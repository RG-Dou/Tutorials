# 代理和RPC

## 代理

![img](https://upload-images.jianshu.io/upload_images/3796264-1b8dbd8ad3b210c7.jpg?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

代理模式（Proxy）是通过代理对象访问目标对象，这样可以在目标对象基础上增强额外的功能，如添加权限，访问控制和审计等功能。







## RPC

客户端想要调用服务器端的函数或者接口。

根据通信协议层面分为：

- 基于HTTP的RPC
- 基于TCP的RPC
- 基于二进制的RPC

根据是否跨平台分为：

- 单语言RPC，如RMI, Remoting;
- 跨平台RPC，如google protobuffer, restful json, http XML

根据调用过程分为：

- 同步RPC：指的是客户端发起调用后，必须等待调用执行完成并返回结果
- 异步RPC：指客户方调用后不关心执行结果返回，如果客户端需要结果，可用通过提供异步 callback 回调获取返回信息。大部分 RPC 框架都同时支持这两种方式的调用



示例：

1、服务端代码
服务端要接收客户端调用，肯定要接收客户端的请求，所以需要一个ServerSocket来接收客户端连接，采用多线程方式，客户端连接比较多时，就会开辟较多线程，可能会占用太多资源，导致服务挂掉，所以我们采用线程池的方式，这样线程可实现复用回收等等。
代码包括ServerService.java和Server.java,如下
Server.java如下,serverSocket用于接收客户端的连接，有客户端连接时，就从线程池threadPool去出一个线程用，用完就是自动回收。线程池里没有可用线程时，会将请求添加到队列，队列有大小，线程池相关知识请自行学习，不做过多介绍。

```java
package Server;

import service.Imp.StudentServiceImp;
import service.StudentService;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class Server {
    private ServerSocket serverSocket;
    private int servPort;

    public Server(int port) throws IOException {
        this.servPort = port;
        serverSocket = new ServerSocket(this.servPort);

    }
    public void start(){
        ThreadPoolExecutor threadPool =new ThreadPoolExecutor(5, 10,
                200, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(10));
        while (true) {
            try {
                Socket sock = serverSocket.accept();
                ServerService service = new ServerService(sock);
                service.registerService(StudentService.class, StudentServiceImp.class);
                threadPool.execute(service);
            } catch (IOException e){
                System.out.println(e.getMessage());
            }
        }
    }
}
```

ServerService.java代码用于接收客户端发过来的请求，socket相关知识不做介绍了。从接收的对象中获取要执行的接口或方法的信息。

注意：私有变量serviceRegistry，该变量用于服务注册，即这个里面存放了我们向外存放的接口，key是接口名或ID，用于查找接口。

```java
package Server;

import Entity.Request;
import Entity.Response;

import java.io.InputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.OutputStream;
import java.lang.reflect.Method;
import java.net.Socket;
import java.util.HashMap;
import java.util.Map;

public class ServerService implements Runnable{
    private Socket sockClient;
    private Map<String, Class> serviceRegistry = new HashMap<String, Class>();
    private Response response = new Response();

    public ServerService (Socket sock) {
        super();
        this.sockClient = sock;
    }

    public void run() {
        try {
            OutputStream out = sockClient.getOutputStream();
            // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
            InputStream in = sockClient.getInputStream();

            ObjectInputStream objIn = new ObjectInputStream(in);
            ObjectOutputStream objOut = new ObjectOutputStream(out);

            // 2. 获取请求数据，强转参数类型
            Object param = objIn.readObject();
            Request request = null;
            if(!(param instanceof Request)){
                response.setMessage("参数错误");
                objOut.writeObject(response);
                objOut.flush();
                return;
            }else{
                request = (Request) param;
            }

            // 3. 查找并执行服务方法
            System.out.println("要執行的类型为：" + request.getClassName());
            Class<?>  service = serviceRegistry.get(request.getClassName());
            if(service != null) {
                Method method = service.getMethod(request.getMethodName(), request.getParamTypes());
                Object result = method.invoke(service.newInstance(), request.getParams());
                // 4. 得到结果并返回
                response.setObj(result);
            }
            objOut.writeObject(response);
            objOut.flush();
            out.close();
            in.close();
            sockClient.close();
        } catch (Exception e){
            System.out.println(e.getMessage());
        }
    }

    public void registerService(Class iface, Class Imp) {
        this.serviceRegistry.put(iface.getName(),Imp);
    }
}

```

2、客户端代码

客户端代码包括socketClient.java和SocketClientProxy两个部分

socketClient.java代码如下，建立socket连接,发送请求并将接收服务器返回信息。

```java
package Client;

import Entity.Request;
import Entity.Response;

import java.io.*;
import java.net.Socket;

public class socketClient {
    private Socket sock;

    public socketClient(){
    }

    public Object invoke(Request request, String Ip, int port) throws IOException {
        sock = new Socket(Ip,port);
        InputStream in = sock.getInputStream();
        OutputStream out = sock.getOutputStream();
        ObjectOutputStream objOut;
        ObjectInputStream objIn;
        Response response = null;
       try {
           objOut = new ObjectOutputStream(out);
           objOut.writeObject(request);
           objOut.flush();
           objIn = new ObjectInputStream(in);
           Object res = objIn.readObject();
           if (res instanceof Response) {
               response = (Response) res;
           } else {
               throw new RuntimeException("返回對象不正确!!!");
           }
       } catch (Exception e) {
           System.out.println("error:   " + e.getMessage());
       } finally {
           out.close();
           in.close();
           sock.close();
           return  response.getObj();
       }
    }
}
```

代理对象SocketClientProxy.java采用动态代理的方式，代理对象调用方法都会转发为InvocationHandler接口的invoke函数调用。

```java
package Client;

import Entity.Request;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class SocketClientProxy {
    private socketClient sock = new socketClient();

    public  <T> T getProxy(Class<T> clazz){
        return (T) Proxy.newProxyInstance(clazz.getClassLoader(),
                new Class<?>[]{clazz}, new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        Request request = new Request();
                        request.setClassName(method.getDeclaringClass().getName());
                        request.setMethodName(method.getName());
                        request.setParamTypes(method.getParameterTypes());
                        request.setParams(args);
                        return  sock.invoke(request,"127.0.0.1",12000);
                    }
                });
    }
}

```

3、测试代码

测试代码如下,首先开启服务端，等待客户端调用。然后生成代理对象StudentService studentService，当调用studentService.getInfo()时，会调用代理对象SocketClientProxy中invoke方法，向服务端发起请求，调用服务端的getInfo方法，并将结果返回，即student的结果是服务端计算后返回的。这样就完成一个简单的RPC实例。

```java
public static void main(String [] arg){
    new Thread( () ->  {
        try {
            Server server = new Server(12000);
            server.start();
        } catch (IOException e){
            System.out.println(e.getMessage());
        }
    } ).start();

    SocketClientProxy proxy = new SocketClientProxy();
    StudentService studentService = proxy.getProxy(StudentService.class);
    Student student = studentService.getInfo();
    System.out.println(student.getName());
}

```



在RPC中，代理就是对目标类做预处理，使得这些类或者方法能够发送到服务器端
# Java Tutorials



## RPC

请看RPC.md







## 序列化和反序列化

**把对象转换为字节序列的过程称为对象的序列化**

**把字节序列恢复为对象的过程称为对象的反序列化**

对象的序列化主要有两种用途：

1） 把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中；

2） 在网络上传送对象的字节序列。



**serialVersionUID: **字面意思上是序列化的版本号。如果不手动设置，编译器会根据类名，接口名，方法和属性等自动生成

用途：java编译器会自动给这个class进行一个摘要算法，类似于指纹算法。如果没有设置**serialVersionUID**，只要这个文件多一个空格，得到的UID就会截然不同的，可以保证在这么多类中，这个编号是唯一的。所以，添加了一个字段后，由于没有显指定serialVersionUID，编译器又为我们生成了一个UID，当然和前面保存在文件中的那个不会一样了，于是就出现了2个序列化版本号不一致的错误。因此，只要我们自己指定了serialVersionUID，就可以在序列化后，去添加一个字段，或者方法，而不会影响到后期的还原，还原后的对象照样可以使用，而且还多了方法或者属性可以用。可以说serialVersionUID是序列化和反序列化之间彼此认识的唯一信物。





## Collection 接口

Collection是set和list的父类

- Set （不包含重复的元素）
  - HashSet
  - SortSet
  - TreeSet
- List （元素可重复，有序）
  - LinkedList
  - ArraryList
  - Vector
    - Stack



## System

System.getenv() : 获取指定的环境变量的值



## Wrapper（包装类）



## 回调（callback）

https://www.cnblogs.com/prayjourney/p/9667835.html
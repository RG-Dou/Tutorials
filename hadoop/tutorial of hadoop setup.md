# Hadoop Setup

## 编译

进入hadoop的目录

`mvn package -Pdist -Pdoc -Psrc -Ptar -DskipTests -e`

-P表示执行pom里面的附属项，比如-Pdist就是执行pom里面的标签为dist的附属项

-D表示给宏一个定义，比如-DskipTests就是在给skipTests定义为true（好像），然后就可以跳过tests。

-e表示显示error。



如果发现编译出的包没有用，试着mvn clean 把旧的包清理一下



## 运行

这样编译完了之后，会将可执行文件放在`hadoop-dist/target`里，需要解压

解压完，yarn的执行文件就在`sbin/start-yarn.sh`

这个时候可能会有找不到JAVA_HOME的错误，需要在`etc/hadoop/hadoop_env.sh`的最后面设置一下JAVA_HOME



## 使用cgoup

首先要获得container-executor.

进入`hadoop-yarn-project/hadoop-yarn/hadoop-yarn-server/hadoop-yarn-server-nodemanager`

执行`mvn package -Pdist,native -DskipTests -Dtar -Dcontainer-executor.conf.dir=/etc/hadoop`

这里，/etc/hadoop是你放`container-executor.cfg`的地方，需要将该文件放到根目录下。

然后得到可执行文件container-executor。在相对目录`target/native/target/usr/local/bin/container-executor`

将该文件放到之前hadoop解压完后的文件夹下的etc/hadoop下面

具体怎么配置yarn-site.xml 和 container-executor.cfg. 参考https://blog.csdn.net/fbj312/article/details/63762891

需要把container-executor.cfg里面的注释删掉

配置文件权重：

`chown root:drg bin/container-executor`

`chmod 6050 bin/container-executor`



在/sys/fs/cgroups/blkio，/sys/fs/cgroup/cpu，/sys/fs/cgroup/memory 下面创建：

`sudo mkdir yarn/`

`sudo chown -R drg:drg /sys/fs/cgroup/blkio/yarn `



## 打包到本地库

mvn install -DskipTests

## Zookeeper zoo.cfg
minSessionTimeout=40000

maxSessionTimeout=60000

## Kafka server.config
log.dirs=/data/drg_data/kafka-logs

log.retention.hours=2

## Task configuration
job.coordinator.zk.session.timeout.ms=400000

task.drop.producer.errors=true

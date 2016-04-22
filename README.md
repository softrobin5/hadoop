# installing java
[Download][df1] java jdk-8u91-linux-x64.tar.gz :  

Uncompress :
```sh
sudo tar -xvf jdk-8u91-linux-x64.tar.gz
```

Now move the JDK 8 directory to /usr/lib

```sh
sudo mkdir -p /usr/lib/jvm
sudo mv ./jdk1.8.0 /usr/lib/jvm/
```
Correct the file ownership
```sh
sudo chown -R root:root /usr/lib/jvm/jdk1.8.0_91
sudo vim /etc/bash.bashrc
#append at the end below 
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_91
export PATH=$JAVA_HOME/bin:$PATH
```
Check if installation was successful
You can check if the installation succeeded with the following command:
```sh
java -version
```

You should see something like

java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)

----------------------------------------------------------------------------------------------

# maven & git

```sh
sudo apt-get install git
```

### Install Maven
```sh
sudo apt-get purge maven maven2 maven3
sudo add-apt-repository ppa:andrei-pozolotin/maven3
sudo apt-get update
sudo apt-get install maven3
```
### check maven version
```sh
mvn -version
```
You should see something like

Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-10T16:41:47+00:00)
Maven home: /usr/share/maven3
Java version: 1.8.0_91, vendor: Oracle Corporation
Java home: /usr/lib/jvm/jdk1.8.0_91/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.13.0-74-generic", arch: "amd64", family: "unix"
-------------------

# mysql installation

login detail:-
hostname:localhost
username:root
password:root

directory structure:-
installed package location: 	/usr/local/mysql
data location: 			/data/mysql/
log location: 			/var/log/nazara/mysql

```sh
run installation script for mysql installtion
cd foldername
sh install_mysql.sh
```
-------------------------

### kafka single broker setups

installed package location: 	/opt/kafka
data location: 			
log location: 			/tmp/kafka-logs
default port zookeeper: 	2181
default port kafka server: 	9092
		
download kafka link-
```sh
cd /opt/
sudo wget http://mirrors.sonic.net/apache/kafka/0.8.2.1/kafka_2.11-0.8.2.1.tgz 
sudo tar xvzf kafka_2.11-0.8.2.1.tgz -C /opt
sudo rm -f kafka_2.11-0.8.2.1.tgz
sudo chown ubuntu.ubuntu -R /opt/kafka_2.11-0.8.2.1
sudo ln -s /opt/kafka_2.11-0.8.2.1 /opt/kafka
```

### copy Kafka start file in supervisor dir
```sh
cd foldername
sudo cp kafka.conf /etc/supervisor/conf.d/
sudo cp zookeeper.conf /etc/supervisor/conf.d/
```

### Start zookeeper & Kafka Service

```sh
sudo supervisorctl reread
sudo supervisorctl update
```
---


# haddop installtion

Hadoop 2.7 Installing on Ubuntu 14.04 (Single-Node Cluster) with YARN

Adding a dedicated Hadoop user
```sh
sudo addgroup hadoop
sudo adduser --ingroup hadoop hduser
# set password is hduser
hduser has root priviledge (add hduser to sudo):
sudo adduser hduser sudo
```
Installing / Updating SSH
```sh
sudo apt-get install ssh
```

Create and Setup SSH Certificates
```sh
su hduser
ssh-keygen -t rsa -P ""
cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
ssh localhost
```
download Hadoop package
```sh
sudo mkdir /usr/local/hadoop
sudo wget http://mirrors.sonic.net/apache/hadoop/common/hadoop-2.7.0/hadoop-2.7.0.tar.gz
sudo tar xvzf hadoop-2.7.0.tar.gz
sudo chown -R hduser:hadoop /usr/local/hadoop
```

Setup Configuration Files
```sh
sudo vim /etc/bash.bashrc
```
### append at the end of above file
```sh
#HADOOP VARIABLES START
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_91
export HADOOP_INSTALL=/usr/local/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib"
#HADOOP VARIABLES END
```
```sh
source /etc/bash.bashrc
```
```sh
vim /opt/hadoop/etc/hadoop/hadoop-env.sh
```
We need to set JAVA_HOME by modifying hadoop-env.sh file
```sh
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_91
```
```sh
vim /opt/hadoop/etc/hadoop/core-site.xml
```

Open the file and enter the following in between the <configuration></configuration> tag:
```sh
<property>
  <name>hadoop.tmp.dir</name>
  <value>/data/hadoop/tmp</value>
  <description>A base for other temporary directories.</description>
 </property>

 <property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:8020</value>
  <description>The name of the default file system.  A URI whose
  scheme and authority determine the FileSystem implementation.  The
  uri's scheme determines the config property (fs.SCHEME.impl) naming
  the FileSystem implementation class.  The uri's authority is used to
  determine the host, port, etc. for a filesystem.</description>
 </property>
```

```sh
cp /opt/hadoop/etc/hadoop/mapred-site.xml.template /opt/hadoop/etc/hadoop/mapred-site.xml
vim /opt/hadoop/etc/hadoop/mapred-site.xml
```

We need to enter the following content in between the
```sh
<configuration></configuration> tag:
 <property>
  <name>mapred.job.tracker</name>
  <value>localhost:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at.  If "local", then jobs are run in-process as a single map
  and reduce task.
  </description>
 </property>

<property>
 <name>mapreduce.framework.name</name>
 <value>yarn</value>
</property>
```

```sh
sudo mkdir -p /data/hadoop/hadoop_store/hdfs/namenode
sudo mkdir -p /data/hadoop/hadoop_store/hdfs/datanode
sudo chown -R hduser:hadoop /data/hadoop/hadoop_store
```
```sh
vim /opt/hadoop/etc/hadoop/hdfs-site.xml
```
We need to enter the following content in between the
<configuration></configuration> tag:

```sh
<property>
  <name>dfs.replication</name>
  <value>1</value>
  <description>Default block replication.
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.
  </description>
 </property>
 <property>
   <name>dfs.namenode.name.dir</name>
   <value>file:/data/hadoop/hadoop_store/hdfs/namenode</value>
 </property>
 <property>
   <name>dfs.datanode.data.dir</name>
   <value>file:/data/hadoop/hadoop_store/hdfs/datanode</value>
 </property>
 <property>
   <name>dfs.data.dir</name>
   <value>file:/data/hadoop/tmp/dfs/datanode</value>
 </property>
```

Format the New Hadoop Filesystem
```sh
cd /data/hadoop/hadoop_store/hdfs/namenode
hadoop namenode -format
```

### How to Starting Hadoop

login hduser
```sh
sudo su hduser
cd /opt/hadoop/sbin
start-all.sh
```
We can check if it's really up and running:
```sh
jps
```
```sh
12800 Jps
10502 ResourceManager
10138 DataNode
9964 NameNode
10349 SecondaryNameNode
10655 NodeManager
```
### How to Stopping Hadoop

login hduser
```sh
sudo su hduser
cd /opt/hadoop/sbin
stop-all.sh
```
# Hadoop Web Interfaces
```sh
Name Node : http://52.90.177.6:50070/
YARN Services : http://52.90.177.6:8088/
Secondary Name Node : http://52.90.177.6:50090/
Data Node 1 : http://52.90.177.6:50075/
logs : http://52.90.177.6:50090/logs/
```
------------------

#gobblin installtion

```sh
cd /opt/gobblin
change folder permission:-
sudo chown ubuntu.ubuntu -R /opt/gobblin
To build Gobblin against a different version of Hadoop 2, e.g., 2.7.0, run the following command:
$ ./gradlew clean build -PuseHadoop2 -PhadoopVersion=2.7.0
```
```sh
sudo vim /etc/bash.bashrc
#append at the end below 
export GOBBLIN_HOME=/opt/gobblin
```

---

# spark

download package in opt folder:-

```sh
cd /opt
sudo wget http://d3kbcqa49mib13.cloudfront.net/spark-1.6.0.tgz
```
Extract spark :-

```sh
sudo tar -xvf spark-1.6.0.tgz
sudo rm -rf spark-1.6.0.tgz
```

### create softlink:-

```sh
sudo ln -s /opt/spark-1.6.0 /opt/spark
```

change ownership:-
```sh
sudo chown ubuntu.ubuntu -R /opt/spark-1.6.0
```

make a compile
```sh
cd /opt/spark
mvn -Pyarn -Phadoop-2.6 -Dscala-2.11 -DskipTests package -X -Phive -Phive-thriftserver
```
---

#install aerospike
```sh
cd /opt
wget -O aerospike.tgz 'http://aerospike.com/download/server/latest/artifact/ubuntu14'
sudo tar zxvf aerospike.tgz
sudo ln -s /opt/aerospike-server-community-3.8.1-ubuntu14.04  /opt/aerospike
sudo rm -rf /opt/aerospike.tgz
```
```sh
sudo ./asinstall
```
```sh
sudo service aerospike start && \
sudo tail -f /var/log/aerospike/aerospike.log | grep cake
```

```sh
sudo vim /etc/aerospike/aerospike.conf
```
append at the end of file below lines
```sh
namespace gameops {
        replication-factor 2
        memory-size 4G
        default-ttl 30d # 30 days, use 0 to never expire/evict.

        # storage-engine memory
        # To use file storage backing, comment out the line above and use the
        # following lines instead.

       storage-engine device {
               file /data/aerospike/gameops.dat
               filesize 4G
               data-in-memory true # Store data in memory in addition to file.
       }
}
```

---
```sh
mkdir /home/ubuntu/nazara
cd /home/ubuntu/nazara/
git clone git@github.com:47Billion/nazaragateway.git

cd /home/ubuntu/nazara/nazaragateway/subscription-pipeline/db2kafka
mvn clean install package

cd /home/ubuntu/nazara/nazaragateway/subscription-pipeline/kafka2s3
mvn clean install

cd /home/ubuntu/nazara/nazaragateway/subscription-pipeline/s3toOpsdb
mvn clean install package
```
---

[df1]: <http://www.oracle.com/technetwork/java/javase/downloads/index.html>

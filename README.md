# Hadoop Cluster
## Description 
The aim of this project is to build a cluster of minimum of 4 machine , first we will building it virtually using virtualbox ,then on physical machines.
In this project we will use hadoop 3.3.3 , ubuntu server 20.04 


## Preparation for hadoop
### Set OS
#### Install ubuntu



1. Run the VM , GRUB will open "set install ubuntu server"
![VirtualBox](/images/10.png "virtualbox")

1. Set the language you want (English in this case)
![VirtualBox](/images/11.png "virtualbox")

1. Set the keyboard layout
![VirtualBox](/images/12.png "virtualbox")

1. Set the network address (Optional)
![VirtualBox](/images/13.png "virtualbox")

1. Configurate proxy (Optional)
![VirtualBox](/images/14.png "virtualbox")

1. Configuration ubuntu archive server
![VirtualBox](/images/15.png "virtualbox")

1. Configuration of the disk
![VirtualBox](/images/16.png "virtualbox")
![VirtualBox](/images/17.png "virtualbox")
![VirtualBox](/images/18.png "virtualbox")

1. Profile setup
![VirtualBox](/images/19.png "virtualbox")

1. set the ssh server 
![VirtualBox](/images/20.png "virtualbox")

1. Installation of the OS
![VirtualBox](/images/21.png "virtualbox")

1. Reboot 
![VirtualBox](/images/22.png "virtualbox")

### Set ssh, install java
If ssh , sshd and java are not installed, this can be done using the following commands under Ubuntu:

```
$ sudo apt-get install ssh
$ sudo apt-get install rsync
$ sudo apt-get install default-jdk 
```

Now that ssh is installed, we create a user named hadoop that will later install and run the HDFS cluster and the MapReduce jobs:

```
$ sudo adduser hadoop
$ sudo usermod -aG sudo hadoop
$ sudo su - hadoop
```

Once the user is created, we open a shell for it, create a SSH keypair for it, copy the content of the public key to the file authorized_keys and check that we can login to localhost using ssh without password:

```
$ ssh-keygen -t rsa -P ""
$ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
$ ssh localhost
```

### Set up networks

Since we are using ubuntu server then we will set static ip address using netplan.

```
$ sudo netplan generate
```

```
$ sudo vi /etc/netplan/00-installer-config.yaml
```

```
network:
        ethernets:
                enp0s3:
                        dhcp4: no
                        addresses: [192.168.1.100/24]
                        gateway4: 192.168.1.1
                        nameservers:
                                addresses: [8.8.8.8,8.8.4.4]

        version: 2

```
then apply the change 
```
$ sudo netplan --debug apply
```

Set the hostname
```
$ sudo vi /etc/hostname
```

```
hadoop-master
```

You have to edit hosts file in /etc/ folder on all nodes, specify the IP address of each system followed by their host names.

```
$ sudo vi /etc/hosts
```


```
192.168.1.100 hadoop-master
192.168.1.101 hadoop-backup
192.168.1.102 hadoop-slave-1
192.168.1.103 hadoop-slave-2
192.168.1.104 hadoop-slave-3
```

### Set up Firewall (Optional)
Set the firewall 

```
$ sudo ufw enable
$ sudo ufw status
$ sudo ufw allow ssh
$ sudo ufw allow proto tcp to 192.168.1.0/24
$ sudo ufw allow proto tcp from 192.168.1.0/24
```


## Configure Hadoop

### Download, install Hadoop
Having setup the basic environment, we can now download the Hadoop distribution and unpack it under `/usr/local/`

```
$ wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.3/hadoop-3.3.3.tar.gz
$ shasum -a 512 hadoop-3.3.3.tar.gz
$ tar -xzvf hadoop-3.3.3.tar.gz
$ mv hadoop-3.3.3 hadoop
$ mv hadoop /usr/local/
```


Starting Hadoop commands just from the command line requires to set its environment variables , and the HDFS binaries are added to the path.
These lines can also be added to the file `.bashrc ` to not type them each time again.


```
$ vi ~/.bashrc

```


```
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

### Stand alone setup


Set up vi `hadoop-env.sh` & vi `core-site.xml` for everything.

```
$ sudo vi $HADOOP_HOME/etc/hadoop/hadoop-env.sh

```

```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_CLASSPATH+=" $HADOOP_HOME/lib/*.jar"
```


```
$ sudo vi $HADOOP_HOME/etc/hadoop/core-site.xml

```

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

`hdfs-site.xml`

```
$ sudo vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml 
```
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```



```
$ sudo vi $HADOOP_HOME/etc/hadoop/mapred-site.xml 
```
```

<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```


```
$ sudo vi $HADOOP_HOME/etc/hadoop/yarn-site.xml 
```
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```




Format the filesystem:


```
$ hdfs namenode -format
```

Start NameNode daemon and DataNode daemon:

```
$ start-dfs.sh
```

Browse the web interface for the NameNode; by default it is available at:

- NameNode - http://localhost:9870/


Start ResourceManager daemon and NodeManager daemon:

```
$ start-yarn.sh
```




## This need more work
### Distributed system



Set up vi `hadoop-env.sh` & vi `core-site.xml` for everything.

```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_CLASSPATH+=" $HADOOP_HOME/lib/*.jar"
```


```
$ sudo vi $HADOOP_HOME/etc/hadoop/core-site.xml

```

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop-master:9000</value>
    </property>
</configuration>

```


### Master ONLY Set-ups
`hdfs-site.xml`

```
$ sudo vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml 
```
```
<configuration>
   <property>
      <name>dfs.replication</name>
      <value>3</value>
   </property>
   <!-- name -->
   <property>
      <name>dfs.namenode.name.dir</name>
      <value>file:///home/hadoop/hdfs/name</value>
   </property>
   <!-- store logs -->
   <property>
      <name>dfs.namenode.edits.dir</name>
      <value>file:///home/hadoop/hdfs/edits</value>
   </property>
   <!-- backup address in http : 50090 -->
   <property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>hadoop-backup:50090</value>
   </property>
</configuration>

```



```
$ sudo vi $HADOOP_HOME/etc/hadoop/mapred-site.xml 
```
```
<configuration>
   <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
   </property>
</configuration>

```


```
$ sudo vi $HADOOP_HOME/etc/hadoop/yarn-site.xml 
```
```
<configuration>
   <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
   </property>

      <property>
      <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
      <value>org.apache.hadoop.mapred.ShuffleHandler</value>
   </property>

</configuration>

```


```
$ sudo vi $HADOOP_HOME/etc/hadoop/workers
```

```
hadoop-slave-1
hadoop-slave-2
hadoop-slave-3
```

### Backup ONLY Set-up



```
$ mkdir -p /home/hadoop/hdfs/namesecondary
```


```
$ sudo vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml 
```
```
<configuration>
   <property>
      <name>dfs.namenode.checkpoint.dir</name>
      <value>file://home/hadoop/hdfs/namesecondary</value>
   </property>
</configuration>

```

### Data node ONLY Set-up
In order to move from “hadoop-backup” to “hadoop-slave-1”, we need to logout from “hadoop-backup” and log into “hadoop-slave-1”.
```
$ exit
$ ssh hadoop-slave-1
```


```
$ mkdir -p /home/hadoop/hdfs/namesecondary
```


```
$ sudo vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml 
```
```
<configuration>
   <property>
      <name>dfs.datanode.data.dir</name>
      <value>file://home/hadoop/hdfs/datanode</value>
   </property>
</configuration>

```


Repeat for other slaves (Workers)


### Format name node & Start using Hadoop!

```
$ exit
```


```
$ hdfs namenode -format
```



















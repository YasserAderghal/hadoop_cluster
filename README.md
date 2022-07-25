# Hadoop Cluster
## Description 
The aim of this project is to build a cluster of minimum of 4 machine , first we will building it virtually using virtualbox ,then on physical machines.
In this project we will use hadoop 3.3.3 , ubuntu server 20.04 


## Installation  & configuration 
### Virtualbox
#### Linux
1. Open a terminal, and update the repositories.
``$ sudo apt-get update  ``
1. Download and install virtualbox
``$ sudo apt-get install virtualbox ``
1.  install the VirtualBox Extension Pack:
`` $ sudo apt-get install virtualbox—ext–pack``
### Ubuntu
1. Download an Ubuntu Image directly from the official website "in our case we will be using ubuntu server".
![The San Juan Mountains are beautiful!](/assets/images/san-juan-mountains.jpg "San Juan Mountains")

1. run VirtualBox

1. Create a new virtual machine
Click new, a then provide the name of the VM , machine folder,

1. Set the memory size( RAM )

1. Set the hard disk file type as VDI 

1. Set storage "dynamically allocated"

1. Set File size ," for my case , it is 80 GB"

1. Open settings of the newly created VM , go to the storage , and set the optical drive to the "ubuntu server" image 

(Optional)
1. Go to system option, and set "Enable EFI"


1. Run the VM , GRUB will open "set install ubuntu server"


1. Set the language you want (English in this case)

1. Set the keyboard layout

1. Set the network address (Optional)

1. Configurate proxy (Optional)

1. Configuration of the disk

1. Profile setup

1. set the ssh server 


1. Reboot 

### hadoop
#### config linux
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
$ cat $HOME/.ssh/id-rsa.pub >> $HOME/.ssh/authorized_keys
$ ssh localhost
```


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
192.168.1.101 hadoop-slave-1
192.168.1.102 hadoop-slave-2
192.168.1.103 hadoop-slave-3
```



#### Config hadoop

Having setup the basic environment, we can now download the Hadoop distribution and unpack it under `/usr/local/`

```
$ wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.3/hadoop-3.3.3.tar.gz
$ shasum -a 512 hadoop-3.3.3.tar.gz
$ tar -xzvf hadoop-3.3.3.tar.gz
$ mv hadoop-3.3.3.tar.gz hadoop
$ mv /usr/local/
```


Starting Hadoop commands just from the command line requires to set its environment variables , and the HDFS binaries are added to the path.
These lines can also be added to the file `.bashrc ` to not type them each time again.

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

- 

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

- The following lines are added to the file $HADOOP_HOME/etc/hadoop/hdfs-site.xml : 

    ```
    
<configuration>
   <property>
      <name>dfs.replication</name>
      <value>1</value>
   </property>

   <property>
      <name>dfs.name.dir</name>
      <value>file:///home/hadoop/hdfs/namenode</value>
   </property>

   <property>
      <name>dfs.data.dir</name>
      <value>file:///home/hadoop/hdfs/datanode</value>
   </property>
</configuration>

    ```
 As user hadoop we create the paths we have configured above as storage:

 ```
 $ mkdir -p /home/hadoop/hdfs/namenode
 $ mkdir -p /home/hadoop/hdfs/datanode
 ```
Before we start the cluster, we have to format the file system:

```
$ $HADOOP_HOME/bin/hdfs namenode -format
```

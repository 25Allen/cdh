#                                  ***\*CDH5.13.3 离线安装文档\****

### \**1. 安装包准备工作:**

服务器: 选用云服务器或者实体服务器

下载相应的安装包: 

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml6276\wps1.jpg) 

附安装包下载地址:(根据自己的安装环境选择不同的安装包)

Parcels：http://archive.cloudera.com/cdh5/parcels/

Cm：http://archive.cloudera.com/cm5/cm/5/

Jdk ：https://www.oracle.com/technetwork/java/javase/downloads/index.html

Mysql：http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-5.7/

Mysql jar：http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.47/

### \**2. 集群搭建前置工作**

(1) Hosts 文件的设置(3台都发)

echo "192.168.3.24 dbsecdata1" >> /etc/hosts

echo "192.168.3.25 dbsecdata2" >> /etc/hosts

echo "192.168.3.26 dbsecdata3" >> /etc/hosts

(2) 关闭防火墙清空规则

service iptables stop

chkconfig iptables off

service iptables status

关闭selinux:

永久有效：vim /etc/sysconfig/selinux

将文本中的SELINUX=enforcing，改为SELINUX=disabled。然后重启reboot

(3) 设置hostname 

vi /etc/sysconfig/network

hostname 改为自己的hosts 里面的主机名

(4) 设置ntp 时间同步(从同步主)

地址: https://www.linuxidc.com/Linux/2015-11/124911.htm

(5) 三台都创建目录

mkdir -p /root/CDH5.13.3/

(6) 把下载完的所有包上传到3.24机器上 /root/CDH5.13.3/ 下，把 cloudera-manager-el6-cm5.13.3_x86_64.tar.gz 放入其余两台服务器的 CDH5.13.3 目录里面。

scp -r /root/CDH5.13.3/cloudera-manager-el6-cm5.13.3_x86_64.tar.gz  [root@dbsecdata1:/root/CDH5.13.3/](mailto:root@dbsecdata3:/root/CDH5.13.3/)

scp -r /root/CDH5.13.3/cloudera-manager-el6-cm5.13.3_x86_64.tar.gz  root@dbsecdata2:/root/CDH5.13.3/

(7) 安装jdk (所有节点)

看服务器有没有/usr/java 目录，没有的话创建 mkdir -p /usr/java

把jdk包发送到其他两台机器的/usr/java 目录下:

scp -r /root/CDH5.13.3/jdk-8u231-linux-x64.tar.gz root@dbsecdata2:/usr/java/

scp -r /root/CDH5.13.3/jdk-8u231-linux-x64.tar.gz root@dbsecdata2:/usr/java/

(8) 三个解压安装jdk，配置环境变量:

cd /usr/java

tar -xvzf jdk-8u231-linux-x64.tar.gz

chown root:root /usr/java/jdk1.8.0_231

echo "export JAVA_HOME=/usr/java/jdk1.8.0_231" >> /etc/profile

echo "export PATH=${JAVA_HOME}/bin:${PATH}" >> /etc/profile

source /etc/profile

java -version 查看是否成功

(9) 安装mysql 到3.24 节点，安装到/usr/local 下，因为cdh有一个文件里面是默认到local 下，用tar 安装，不推荐rpm

mv mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz /usr/local/

cd /usr/local

tar -zxf mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz

后续安装参考我github:  https://github.com/erpengwang/MySQL

(10) Mysql jdbc jar 目录是 /usr/share/java

mv /root/CDH5.13.3/mysql-connector-java-5.1.47.jar mysql-connector-java.jar   #重命名mysql 名字

mv mysql-connector-java.jar /usr/share/java/

(9) 创建CDH以及amon的元数据库和用户

进入 mysql -uroot -p 'password'

create database cmf default character set utf8;

create database amon default character set utf8;

grant all privileges on cmf.* to 'cmf'@'%' identified by 'dbsec123456';

grant all privileges on amon.* to 'amon'@'%' identified by 'dbsec123456';

flush privileges;

### **3.** ***\*CDH 离线部署\****

##### **1.** ***\*离线部署cm server 及agent\****

1.1. 所有节点创建目录及解压

mkdir -p /opt/cloudera-manager

tar -zxf cloudera-manager-el6-cm5.13.3_x86_64.tar.gz -C /opt/cloudera-manager/

1.2. 所有节点修改agent 配置，指向server的节点 dbsecdata1

vim /opt/cloudera-manager/cm-5.13.3/etc/cloudera-scm-agent/config.ini

把里面的hostname 修改成3.24 的主机名

1.3. 主节点修改server的配置

vim etc/cloudera-scm-server/db.properties

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml6276\wps2.jpg) 

1.4. 所有节点创建用户

useradd --system --home=/opt/cloudera-manager/cm-5.13.3/run/cloudera-scm-server/ --no-create-home --shell=/bin/false --comment "clouder cmf user" cloudera-scm

1.5. 目录修改用户及用户组

chown -R cloudera-scm:cloudera-scm /opt/cloudera-manager

##### **2.** ***\*Dbsecdata1 节点离线部署parcel 源\****

2.1. 部署离线parcel 源

mkdir -p /opt/cloudera/parcel-repo

\#把 .sha1 文件名的1 去掉：

mv CDH-5.13.3-1.cdh5.13.3.p0.2-el6.parcel.sha1 CDH-5.13.3-1.cdh5.13.3.p0.2-el6.parcel.sha

cp CDH-5.13.3-1.cdh5.13.3.p0.2-el6.parcel.sha manifest.json /opt/cloudera/parcel-repo

2.2. 目录修改用户及用户组

chown -R cloudera-scm:cloudera-scm /opt/cloudera/

##### \3. 所有节点创建软件安装目录、用户及用户组权限

mkdir -p /opt/cloudera/parcels 

chown -R cloudera-scm:cloudera-scm /opt/cloudera/

##### \4. dbescdata1 启动 server

/opt/cloudera-manager/cm-5.13.3/etc/init.d/cloudera-scm-server start

等待1分钟 打开 http://dbsecdata1:7180 账号密码: admin/admin

假如打不开，找server log 文件，仔细排查错误

##### \5. 所有节点启动agent

/opt/cloudera-manager/cm-5.13.3/etc/init.d/cloudera-scm-agent start

### 4.**Web 界面设置**

界面启动过程中有一个检查，在所有主机运行:

vim /etc/sysctl.conf  加入 

vm.swappiness = 10

 

vim /etc/rc.d/rc.local  加入

echo never > /sys/kernel/mm/transparent_hugepage/defrag

echo never > /sys/kernel/mm/transparent_hugepage/enabled

 

 

 

 

 

 

 

 

 

 

 

 
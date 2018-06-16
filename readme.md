#  集群安装部署手册
-------------------
## [1 关于文档](#about_doc)
## [2 背景知识](#background)
## [3 准备工作](#prepare_work)
### [3.1 安装 expect](#install_expect)
### [3.2 安装 pdsh](#install_pdsh)
### [3.3 超级权限用户免密登录](#root_create_ssh_key)
### [3.4 集群安装 expect](#install_all_expect)
### [3.5 集群安装 pdsh](#install_all_pdsh)
### [3.6 配置Hosts](#hosts_config)
### [3.7 创建用户](#create_user)
### [3.8 创建免密登录](#create_ssh_key)
### [3.9 创建挂载数据盘目录](create_mount_dir#)
### [3.10 挂载数据盘（可选）](#mount_disk)
### [3.11 防火墙设置（可选）](#set_firewalld)
### [3.12 规划集群机器角色](#set_role)
## [4 MySQL 安装(Hive Metastore)](#mysql_install)
## [5 JDK 安装](#jdk_install)
## [6 Zookeeper 安装](#zookeeper_install)


-------------------
## 1. 关于文档 <a name="about_doc"/>
   本文档作为售前运维，技术运维，技术研发安装部署集群参考手册，提供了整个集群安装部署需要注意的事项和部署方法.

## 2. 背景知识 <a name="background"/>
   为了让项目成员从繁琐的集群配置和安装命令中解脱出来，降低集群安装运维成本，整理出这份文档,用以标准化流程.

## 3. 准备工作 <a name="prepare_work"/>
   选择集群服务器中某台服务器作为安装跳板机,以下简称跳板机，所有的安装文件都将会存放在此台服务器中.
### 3.1 安装 expect <a name="install_expect"/>
   * 1) 超级权限用户登录跳板机
   
   ```
   ssh -p 22 user@127.0.0.1
   ```
   
   * 2) install
   
   ```
   sudo yum install -y expect
   ```

### 3.2 安装 pdsh <a name="install_pdsh"/>
   * 1) 超级权限用户登录跳板机
   
   ```
   ssh -p 22 user@127.0.0.1
   ```
   
   * 2) install epel
   
   ```
   sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
   ```
   
   * 3) install
   
   ```
   sudo yum install -y pdsh
   ```   

### 3.3 超级权限用户免密登录 <a name="root_create_ssh_key"/>
   * 1) 超级权限用户登录跳板机,进入安装目录中的install/initial目录
   
   ```
   ssh -p 22 user@127.0.0.1
   cd install/initial
   ```
   
   * 2) 执行命令

   ```
   expect initial.exp distSSHKey 127.0.0.1,127.0.0.2(Service List) user user_password
   ```
   
### 3.4 集群安装 expect <a name="install_all_expect"/>
   * 1) 超级权限用户登录跳板机,进入install目录

   ```
   ssh -p 22 user@127.0.0.1
   cd install
   ```
      
   * 2) 编辑文件
   
   ```
   vi all_hosts
   ```
   
   * 3) 将所有服务器写入文件，一行一个服务器，并保存退出
   
   ```
   172.23.0.21
   172.23.0.22
   172.23.0.23
   ```
      
   * 4) install
   
   ```
   pdsh -w ^all_host sudo yum install -y expect   
   ```
   
### 3.5 集群安装 pdsh <a name="install_all_pdsh"/>
   * 1) 超级权限用户登录跳板机,进入install目录

   ```
   ssh -p 22 user@127.0.0.1
   cd install
   ```
      
   * 2) 编辑文件
   
   ```
   vi all_hosts
   ```
   
   * 3) 将所有服务器写入文件，一行一个服务器，并保存退出
   
   ```
   172.23.0.21
   172.23.0.22
   172.23.0.23
   ```
      
   * 4) install epel
   
   ```
   sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
   ```
   
   * 5) install pdsh
   
   ```
   sudo yum install -y pdsh
   ```

### 3.6 配置Hosts <a name="hosts_config"/>
   * 1) 超级权限用户登录服务器，进入install目录
   
   ```
   ssh -p 22 user@127.0.0.1
   cd install
   ```
      
   * 2) 编辑文件
   
   ```
   vi all_hosts
   ```
   
   * 3) 将所有服务器写入文件，一行一个服务器，并保存退出

   ```
   172.23.0.21
   172.23.0.22
   172.23.0.23
   ```
   
   * 4) 拷贝本地 hosts
   
   ```
   cp /etc/hosts ./
   ```
   
   * 5) 编辑 hosts 文件，将集群服务器对应的 ip地址和hostname 都加进该文件,并保存退出
   
   ```
   172.23.0.21	Hadoop
   172.23.0.22	HBase
   172.23.0.23 Hive   
   ```
   
   * 6) 部署到集群
   
   ```
   pdcp -w ^all_hosts hosts /etc/hosts
   ```
   
### 3.7 创建用户（可选）<a name="create_user"/>
   * 1) 超级权限用户登录跳板机,进入安装目录中的initial目录
   
   ```
   ssh -p 22 user@127.0.0.1
   cd install/initial
   ```
      
   * 2) 执行命令
   
   ```
   expect initial.exp userCreate 127.0.0.1,127.0.0.2(Server List) root(超级权限用户) root_password user user_password user_group
   ```
   
### 3.8 创建免密登录 <a name="create_ssh_key"/>
   * 1) 上传安装文件到3.7步骤创建的用户目录下
   
   ```
   scp -P 2228 -r install-shell suxin@127.0.0.1:/home/suxin/
   ```
   
   * 2) 用3.7步骤创建的用户登录跳板机,进入安装目录中的initial目录
   
   ```
   ssh -p 22 user@127.0.0.1
   cd install/initial
   ```
      
   * 3) 执行命令
   
   ```
   expect initial.exp distSSHKey 127.0.0.1,127.0.0.2(Server List) user user_password
   ```
   
   * 4) 初始化 ssh known keys
   
   ```
   expect initial.exp initKnownKey HadoopRM,HadoopNN(HOSTNAME_LIST) user user_password
   ```
   
   * 5) 重复1 - 4步骤，直至完成所有服务器

### 3.9 创建挂载数据盘目录 <a name="create_mount_dir"/>
   * 1) 用3.7步骤创建的用户登录跳板机,进入install目录
   
   ```
   ssh -p 22 user@127.0.0.1
   cd install
   ```
      
   * 2) 编辑文件
   
   ```
   vi all_hosts
   ```
   
   * 3) 将所有服务器写入文件，一行一个服务器，并保存退出

   ```
   172.23.0.21
   172.23.0.22
   172.23.0.23
   ```
   
   * 4) 创建挂载目录
   
   ```
   pdsh -w ^all_hosts mkdir -p /home/${user}/application
   ```
   
### 3.10 挂载数据盘（可选）<a name="mount_disk"/>
   * 1) 超级权限用户登录服务器
   
   ```
   ssh -p 22 user@127.0.0.1
   ```
      
   * 2) sudo lsblk
   
   ```
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0           2:0    1    4K  0 disk 
sda           8:0    0   32G  0 disk 
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   31G  0 part 
  ├─cl-root 253:0    0 27.8G  0 lvm  /
  └─cl-swap 253:1    0  3.2G  0 lvm  [SWAP]
sdb           8:16   0    2T  0 disk /home/apps/application
sr0          11:0    1 1024M  0 rom
   ```
      
   * 3) 格式化数据盘
   
   ```
   sudo mkfs.ext4 ${步骤2看到的数据盘标识，例：/dev/sdb,/dev/sdc} 
   ```
   
   * 4) 按步骤选择Y即可
   
   ```
mke2fs 1.42.9 (28-Dec-2013)
/dev/sdb is entire device, not just one partition!
Proceed anyway? (y,n)   
   ```
   * 5) 重复1 - 4步骤，直至完成所有服务器
   
   * 6) 超级权限用户登录跳板机,进入install目录
   
   ```
   ssh -p 22 user@127.0.0.1
   cd install
   ```
      
   * 7) 编辑文件
   
   ```
   vi all_hosts
   ```
   
   * 8) 将所有服务器写入文件，一行一个服务器，并保存退出 
   
   ```
   172.23.0.21
   172.23.0.22
   172.23.0.23
   ```
      
   * 9) 执行命令
   
   ```
   pdsh -w ^all_hosts sudo mount ${步骤2看到的数据盘标识，例：/dev/sdb,/dev/sdc}  /home/${USER}/application
   ```
   
   * 10) 改变目录属主
   
   ```
   pdsh -w ^all_hosts sudo chown ${USER}:${USER_GROUP} /home/${USER}/application
   ```
   
   * 11) 查看数据盘 UUID (需要每台服务器独立操作的步骤)
   
   ```
   sudo blkid 
   ```
   
   ```
/dev/sdb: UUID="7348f362-bba7-4b2f-b55f-2fdb4c3a9a41" TYPE="ext4" 
/dev/sda1: UUID="f3a6907b-7ec0-4ad4-be11-d46928592754" TYPE="xfs" 
/dev/sda2: UUID="8GfyzU-aEw6-4WX2-TL0R-gqT0-C1Qg-JvHBST" TYPE="LVM2_member" 
/dev/mapper/cl-root: UUID="1ca5c173-766f-4020-9cd0-c02a228720e8" TYPE="xfs" 
/dev/mapper/cl-swap: UUID="f6361ffa-c19c-4507-a99f-17d32705a793" TYPE="swap"
   ```
   
   * 12) 编辑 /etc/fstat
   
   ```
   sudo vim /etc/fstab
   ```
   
   * 13) 将如下行加到文件
   
   ```
   UUID=${数据盘的UUID}  /home/${USER}/application ext4 defaults 0 0
   ```
   
   ```
#
# /etc/fstab
# Created by anaconda on Wed Jan  3 10:52:36 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=f3a6907b-7ec0-4ad4-be11-d46928592754 /boot                   xfs     defaults        0 0
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
UUID=7348f362-bba7-4b2f-b55f-2fdb4c3a9a41 /home/apps/application ext4 defaults 0 0
   ```
   
   * 14) 重复11 - 13步骤，直至完成所有服务器

### 3.11 防火墙设置（可选）<a name="set_firewalld"/>
   * 1) 超级权限用户登录服务器
   
   ```
   ssh -p 22 user@127.0.0.1
   ```
      
   * 2) 编辑文件
   
   ```
   vi all_hosts
   ```
   
   * 3) 将所有服务器写入文件，一行一个服务器，并保存退出
   
   ```
   172.23.0.21
   172.23.0.22
   172.23.0.23
   ```
      
   * 4) 停止防火墙
   
   ```
   pdsh -w ^all_hosts sudo systemctl stop firewalld.service
   ```
   
   * 5) 关闭自启动
   
   ```
   pdsh -w ^hadoop_cluster sudo systemctl disable firewalld.service
   ```
   
   * 6) 如果不能关闭防火墙，可以使用如下命令添加放行端口号
   
   ```
   pdsh -w ^hadoop_cluster sudo firewall-cmd --zone=public --add-port=${PORT}/tcp --permanent
   ```
   
   * 7) 如果是步骤6 执行如下命令使防火墙端口放行命令生效
   
   ```
   pdsh -w ^hadoop_cluster sudo firewall-cmd --reload
   ```
   
### 3.12 规划集群机器角色 <a name="set_role"/>
   * 1) Zookeeper 集群服务器分配（MYID）
   * 2) kafka 集群服务器分配
   * 3) Flume 集群服务器分配
   * 4) ElasticSearch 集群服务器分配
   * 5) Hadoop 集群服务器分配（RM,NM,NN,SNN,DN,JobHistory）
   * 6) HBase 集群服务器分配（Master,RegionServer）
   * 7) Hive 服务器分配
   * 8) Spark 集群服务器分配 (Master, Slaves)
   
   
## 4. MySQL 安装 <a name="mysql_install"/>
   ###  Step 1. Add MariaDB Yum Repository
   *   1) 编辑安装库
   
   ```
   sudo vi /etc/yum.repos.d/MariaDB.repo
   ```
   
   *   2) 添加如下内容进文件,并保存退出
   
   ```
   [mariadb]
   name = MariaDB
   baseurl = http://yum.mariadb.org/10.1/centos7-amd64
   gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
   gpgcheck=1   
   ```

   ###  Step 2. Install MariaDB in CentOS 7
   * 1) 安装组
   
   ```
   sudo yum groupinstall -y mariadb*
   ```
   
   * 2) 启动 mariadb
   
   ```
   sudo systemctl start mariadb
   ```
   
   * 3) 开启自启动
   
   ```
   sudo systemctl enable mariadb
   ```
   
   * 4) 查看状态
   
   ```
   sudo systemctl status mariadb
   ```
   
   ### Step 3. Secure MariaDB in CentOS 7
   * 1) 设置
   
   ```
   mysql_secure_installation
   ```
   
   ```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!   
   ```
   
## 5 JDK 安装 <a name="jdk_install"/>
  * 1) 用3.7步骤创建的用户登录跳板机,进入install/jdk目录
  
   ```
   ssh -p 22 user@127.0.0.1
   cd install/jdk
   ```
      
   * 2) 编辑文件
   
   ```
   vi jdk_hosts
   ```
   
  * 3) 将所有服务器写入文件，一行一个服务器，并保存退出

   ```
   172.23.0.21
   172.23.0.22
   172.23.0.23
   ```
     
  * 4) 执行命令
  
  ```
  install-jdk [install] [APP_HOME] [JDK_FILE] [JAVA_VERSION] [USER]
  ```
  
  ```
install jdk /home/suxin/test_install jdk-8u171-linux-x64.tar.gz 1.8.0_171 suxin.........

Copying JDK 1.8.0_171 to all hosts...
Installing JDK 1.8.0_171 on all hosts...
Set JAVA_HOME env to all hosts...
Installing JDK 1.8.0_171 on all hosts success  
  ```
  
  * 5) 可选： 用3.7步骤创建的用户登录跳板机, 输入jps命令，如果能运行则安装成功.
  
  ```
  94037 Jps
  ```
  
## 6 Zookeeper 安装 <a name="zookeeper_install"/>
  * 1) 用3.7步骤创建的用户登录跳板机,进入install/zookeeper目录
  
   ```
   ssh -p 22 user@127.0.0.1
   cd install/zookeeper
   ```

   * 2) 编辑文件
   
   ```
   vi zookeeper_hosts
   ```
     
  * 3) 将所有服务器写入文件，一行一个服务器，并保存退出
  
   ```
   172.23.0.21
   172.23.0.22
   172.23.0.23
   ```
   
  * 4) 编辑文件 conf/zoo.cfg
  
  ```
  vi cond/zoo.cfg
  ```
  
  * 5) 修改或添加配置
  
  ```
  # The directory where the snapshot is stored.
  
  dataDir=${ZOOKEEPER_HOME}/data
  dataLogDir=${ZOOKEEPER_HOME}/logs  
  ```
  
  ```
  # To configure a ZooKeeper instance you add the following parameter to the file "zoo.cfg":
  # server.<id> = <zk_host_address>:<zk_port_1>:<zk_port_2>[:<zk_role>];[<client_port_address>:]<client_port>
  
  # example:
  server.1=AZ-TEST-DEV4:2888:3888
  server.2=AZ-TEST-DEV2:2888:3888
  server.3=AZ-TEST-DEV3:2888:3888  
  
  ```
  
  * 6) 执行命令
  
  ```
  install-zookeeper [install] [APP_HOME] [ZookeeperFile] [ZOOKEEPER_VERSION] [MYID_LIST:HOSTNAME:ID,HOSTNAME:ID] [USER]
  ```
  
  ```
  install zookeeper /home/suxin/test_install zookeeper-3.4.10.tar.gz 3.4.10 127.0.0.1:1 suxin .........
  
  Copying Zookeeper 3.4.10 to all hosts...
  Add ZOOKEEPER_HOME to PATH to all hosts...
  Copying ZOOKEEPER config file to all hosts...
  Set myid:127.0.0.1:1 for ZOOKEEPER to all hosts...
  Set myid:1 for ZOOKEEPER to server:127.0.0.1 ...........
  Install ZOOKEEPER service on all hosts success...  
  ```
  
  * 7) 可选：增加服务自启动
  
  ```
  install-zookeeper [checkconfigon] [ZOOKEEPER_HOME]
  ```
  
  ```
  ./install-zookeeper.sh checkconfigon /home/suxin/test_install/zookeeper
  
  check config on /home/suxin/test_install/zookeeper .........
  Check config on for zookeeper ........
  /home/suxin/test_install/zookeeper/bin/zkServer.sh start
  
  ```
  
  * 8) 用3.7步骤创建的用户登录Zookeeper服务器
  
   ```
   ssh -p 22 user@127.0.0.1
   ```
     
  * 9) 执行命令
  
  ```
  ${ZOOKEEPER_HOME}/bin/zkServer.sh start
  ```
  
  * 10) 验证,看到 QuorumPeerMain 进程表示成功
  
  ```
  ${JAVA_HOME}/bin/jps 
  ```
  
   ```
   90448 QuorumPeerMain
   26577 Jps
   ```
   
  * 10) 重复6 - 8步骤，直至完成zookeeper集群所有服务器.
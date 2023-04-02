# Debian11部署Web应用

> 本篇部署名为sandpile的web应用，例子中使用sandpile作为应用名，可将其更换为任意名字

## Linux相关

1. 修改SSH端口

   ```shell
   # 打开配置文件
   nano /etc/ssh/sshd_config
   
   # 找到并修改Port字段
   Port 12200
   
   # 重启服务(注意需先开启防火墙的指定端口)
   systemctl restart ssh
   ```

2. 防火墙配置

   ```shell
   ufw status
   ufw enable
   ufw disable
   
   ufw allow 22/tcp # 开启tcp的22端口
   ufw allow 22/udp # 开启udp的22端口
   ufw allow 22 # 开启指定tcp和udp端口
   ufw deny 22 # 拒绝指定端口(tcp和udp配置同上)
   ```

   

## Nginx部署

1. 安装Nginx

   ```shell
   apt install nginx
   ```

2. 上传前端文件至 `/usr/local/sandpile/frontend` 目录下

3. 修改配置文件：/etc/nginx/nginx.conf

   ```conf
   worker_processes auto;
   pid /run/nginx.pid;
   include /etc/nginx/modules-enabled/*.conf;
   
   events {
   	worker_connections 768;
   	# multi_accept on;
   }
   
   http {
   
   	sendfile on;
   	tcp_nopush on;
   	tcp_nodelay on;
   	types_hash_max_size 2048;
   
   	include /etc/nginx/mime.types;
   	default_type application/octet-stream;
   
   	##
   	# SSL Settings
   	##
   
   	#ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
   	#ssl_prefer_server_ciphers on;
   
   	##
   	# Logging Settings
   	##
   
   	access_log /var/log/nginx/access.log;
   	error_log /var/log/nginx/error.log;
   
   	##
   	# Gzip Settings
   	##
   
   	gzip on;
   
   	gzip_vary on;
   	# gzip_proxied any;
   	gzip_comp_level 3;
   	gzip_min_length 1024;
   	# gzip_buffers 16 8k;
   	# gzip_http_version 1.1;
   	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
   
   	#注释默认配置文件
   	#include /etc/nginx/conf.d/*.conf;
   	#include /etc/nginx/sites-enabled/*;
   
   	server {
           listen 80;
           server_name 域名;
   		default_type text/plain;
   		
   		location / {
               root /usr/local/sandpile/frontend;
               index index.html index.htm;
               try_files $uri $uri/ /index.html; # vue-router history模式适配
   		}
   		
           location /api {
   		    proxy_pass http://127.0.0.1:8090; # 后端地址
   		}
   		
   	}
   }
   
   ```

4. 开启Nginx

   ```shell
   ufw allow 80
   systemctl start nginx
   systemctl enable nginx
   ```



## MySQL部署

1. 下载MySQL APT 存储库

   ```shell
   wget https://repo.mysql.com//mysql-apt-config_0.8.20-1_all.deb
   ```

2. 安装存储库

   ```shell
   apt install ./mysql-apt-config_*_all.deb
   ```

3. 选中 `MySQL Server & Cluster` → `mysql-8.0` → `ok`

4. 以后若想重新配置，输入以下命令：

   ```shell
   sudo dpkg-reconfigure mysql-apt-config
   ```

5. 安装mysql

   ```shell
   sudo apt update
   sudo apt install mysql-server
   ```

   - 若在update时提示没有签名的错误，进入 /etc/apt/apt.conf.d/70debconf 文件，追加：

     ```
     Acquire::AllowInsecureRepositories "true";
     ```
   
6. 确认是否正常运行

   ```shell
   systemctl status mysql
   mysql --version
   ```

7. 可选，对mysql进行安全配置

   ```shell
   mysql_secure_installation
   ```

8. 登陆mysql，设置允许任意ip登陆root用户

   ```shell
   mysql -u root -p
   # 进入mysql> 命令行后
   GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
   ```

   - 若报错“You are not allowed to create a user with GRANT”，执行：

     ```shell
     use mysql;
     update user set host='%' where user='root';
     commit;
     GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
     ```

9. 检查设置是否成功，若root的host参数为%则为成功

   ```shell
   select User, host from mysql.user; # 查看所有用户的情况
   ```

10. 导入备份的数据库文档：

    ```shell
    # 进入mysql命令行模式，新建数据库
    mysql -u root -p
    mysql> create database sandpile;
    mysql> quit;  # 退出数据库命令行模式
     
    # 方法一：比较直接的导入方法: 
    mysql -u username -p password database < database.back.sql
    
    # 方法二：在数据库命令行下的导入方法
    mysql -u root -p
    mysql> use sandpile;
    mysql> set names utf8;  #设定数据库的编码（针对空数据库）
    mysql> source /usr/local/sandpile/backend/db/sand.sql;  #导入的命令，数据库备份文档的绝对路径
    mysql> quit;
    ```
    
11. 修改端口至3307

    ```shell
    # 登陆数据库并查看当前开放端口
    mysql -u root -p
    mysql> show global variables like 'port';
    mysql> exit;
    
    # 修改端口号
    nano /etc/mysql/mysql.conf.d/mysqld.cnf
    # 在[mysqld]节点下增加以下内容
    port=3307
    
    # 重启MySQL
    systemctl restart mysql
    ```

    

## 后端部署

1. [下载JDK17](https://www.oracle.com/java/technologies/downloads/#java17)（免费版本，提供长期支持）

2. 上传jdk至服务器的 `/usr/local/java` ，并解压

   ```shell
   cd /usr/local/java
   tar -zxvf jdk-17_linux-x64_bin.tar.gz
   ```

3. 配置环境变量

   ```shell
   nano /etc/profile
   
   # 文件末尾添加
   export JAVA_HOME=/usr/local/java/jdk-17.0.6
   export JRE_HOME=${JAVA_HOME}/jre
   export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
   export PATH=${JAVA_HOME}/bin:$PATH
   
   # 保存后应用更改
   source /etc/profile
   ```

4. 检查java是否可用

   ```shell
   java -version
   javac
   ```

5. 切换工作目录

   ```shell
   cd /usr/local/sandpile/backend
   ```

6. 上传jar包至该目录，上传配置文件至./config

7. 配置服务：在 `/etc/systemd/system` 目录下，创建文件`sandpile.service`：

   ```shell
   [Unit]
   Description=Sandpile service
   
   [Service]
   User=root
   Type=simple
   DefaultDependencies=no
   ExecStart=/bin/bash -l -c "java -jar /usr/local/sandpile/backend/sandpile-1.0-SNAPSHOT.jar --spring.config.location=/usr/local/sandpile/backend/config/application.yml > /usr/local/sandpile/sandpile.log"
   SuccessExitStatus=143
   
   [Install]
   WantedBy=multi-user.target
   ```

8. 运行服务

   ```shell
   systemctl start sandpile.service
   systemctl enable sandpile.service
   ```

   

## FTP部署

1. 安装ftp

   ```shell
   apt install vsftpd
   ```

2. 编辑配置文件 `/etc/vsftpd.conf`，在末尾追加：

   ```shell
   # 修改端口为12100
   listen_port=12100
   # 禁止匿名登陆和上传文件
   anonymous_enable=NO
   anon_mkdir_write_enable=NO
   # 允许本地用户登陆
   local_enable=YES
   # 允许上传文件
   write_enable=YES
   local_umask=022
   # 设置限制用户访问(只有写到/etc/vsftpd/vsftpd.chroot_list里的用户才可以拥有访问上层目录的权限)
   # chroot_local_user=YES  #一般默认为yes,当为no时,不写到etc/vsftpd/vsftpd.chroot_list里的用户才有上层访问权限
   chroot_list_enable=YES
   chroot_list_file=/etc/vsftpd/vsftpd.chroot_list
   # 设置ftp用户访问进来的根目录
   local_root=/home/ftpuser
   # 在新的ftp版本必须添加这行,不然普通用户无法通过ftp登陆到设置的根目录
   allow_writeable_chroot=YES
   ```

3. 创建目录 `/home/ftpuser`

4. 配置普通用户

   ```shell
   # 先打开/etc/shells文件,查看里面是否有一行/sbin/nologin,如果没有,加入这一行
   
   # 创建普通用户名为ftpuser,登陆执行的终端方式为/sbin/nologin,所属拥有权目录为/home/ftpuser
   sudo useradd -d /home/ftpuser -s /sbin/nologin ftpuser
   # 设置用户密码
   sudo passwd ftpuser
   #设置ftp目录的用户权限
   sudo chown ftpuser:ftpuser /home/ftpuser
   ```

5. 增加管理员用户

   ```shell
   # 首先建立chroot_list文件
   mkdir /etc/vsftpd && touch /etc/vsftpd/vsftpd.chroot_list
   
   # 打开该文件并追加管理员用户
   nano /etc/vsftpd/vsftpd.chroot_list
   # 追加root
   ```

6. 重启ftp服务

   ```shell
   /etc/init.d/vsftpd restart
   ```

   


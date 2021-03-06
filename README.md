#LNMP
centos7 搭建lnmp 环境

一.安装nginx
1.下载对应当前系统版本的nginx包(package)

```wget http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm```

2.建立nginx的yum仓库（默认yum是没有nginx的）

```rpm -ivh nginx-release-centos-7-0.el7.ngx.noarch.rpm```

3.下载并安装nginx
    
```yum install nginx```

4.nginx启动（nginx安装目录下－/usr/sbin/）
	
```systemctl start nginx.service```

5.设置nginx开机启动

	systemctl enable nginx.service

二.防火墙设置(防火墙这一块 如果你不会又害怕出错 你可以先不管,跳过)
查看永久服务  
   	firewall-cmd --permanent --list-services 
添加永久服务	
	firewall-cmd --permanent --add-service=http
	firewall-cmd --permanent --add-service=ssh
	firewall-cmd --permanent --add-service=mysql

查看永久端口
	firewall-cmd --permanent --list-ports
添加永久端口	
	firewall-cmd --permanent --zone=trusted --add-port=9999/tcp (备注:ssh端口,可以任意设置,安全起见不要用默认)	
	firewall-cmd --permanent --zone=trusted --add-port=9000/tcp (备注:php-fpm端口)	
	firewall-cmd --permanent --zone=trusted --add-port=80/tcp   (备注:http端口)	
	firewall-cmd --permanent --zone=trusted --add-port=3306/tcp (备注:mysql端口)
更新防火墙规则
	firewall-cmd --reload

三.安装php
1.查看当前安装的php版本（ yum list installed | grep php）
   如果存在php安装包先删除之前版本  用yum remove 移除 php相关的包
2.rpm 安装 Php7 相应的 yum源
   rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
   rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
3.安装php7.0
   yum install php70w
4.安装php扩展
  php70w-mysql.x86_64       mysql扩展（作为依赖同时安装 php70w-pdo.x8664）
  php70w-gd.x86_64          GD库，是php处理图形的扩展库
  php70w-ldap.x86_64        "轻量级目录访问协议"，是一个用于访问"目录服务器"(Directory Servers)的协议;
  php70w-mbstring.x86_64    mbstring扩展库用于处理多字节字符串
  php70w-mcrypt.x86_64 	    Mcrypt扩展库可以实现加密解密功能，就是既能将明文加密，也可以密文还原。
  php70w-xml.x86_64         php_xml扩展，可以用来生成xml文档（yum install php-xml）
 
 
(注意事项：php 7.1以上版本不在再支持mcrypt,安装php7.1以上版本请注意兼容)

5.安装PHP FPM
   yum install php70w-fpm

第四步：配置nginx
*(修改配置文件之前记得备份,但要注意不要用.conf作为后缀名)

1.nginx配置文件位置：（/etc/nginx/conf.d/default.conf）
​ 配置php解析
​ location ~.php$ {

​ 	fastcgi_pass 127.0.0.1:9000;

​ 	fastcgi_index index.php;

​    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

​ 	include    fastcgi_params;

​ }
(*修改的部分是$document_root这里)

2.php-fpm配置文件位置：（/etc/php-fpm.d/www.conf）
​ 修改
	user =nginx
​ 	group=nginx

3.启动nginx服务：
	systemctl start nginx.service
	如需设置开机自启使用以下命令：
	sudo systemctl enable nginx.service
  查看启动状态：
	systemctl status nginx  

4.启动PHP-FPM：
	systemctl start php-fpm.service
 	如需设置开机自启试用以下命令：
	sudo systemctl enable php-fpm.service
	查看启动状态：
	systemctl status php-fpm.service 

五,安装mysql
	
1.先下载mysql的repo源；相关命令：
	wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
2.安装mysql-community-release-el7-5.noarch.rpm包
	（安装这个包后，会获得两个mysql的yum repo源：/etc/yum.repos.d/mysql-community.repo，/etc/yum.repos.d/mysql-community-source.repo）
	rpm -ivh mysql-community-release-el7-5.noarch.rpm
3.安装MYSQL
	yum install mysql-server
4.重置密码
 	重启服务：
	systemctl restart mysql
	登录，并修改密码：
	mysql -u root
 	mysql> set password for 'root'@'localhost' =password('你的密码');
	mysql> flush privileges;
	为root账户添加远程登陆权限
	mysql> grant all privileges on *.* to root@'%' identified by '远程密码';
	mysql> flush privileges;



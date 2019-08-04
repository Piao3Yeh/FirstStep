# 第一周

任务：Linux+ nginx + php-fpm + MySQL

实现功能：外部主机可以访问服务器的html和php页面， 可以连接到MySQL数据库。



# 1.安装基础环境

## 1.1安装Centos7和Nginx

1.  从Centos官网下载系统镜像, 或者从国内的镜像站下载镜像。**（注意：下载镜像后使用MD5计算工具计算hash值与官网提供的hash对比确定镜像未被修改。）**

2.  在Virtualbox挂上镜像按照安装向导进行安装。**（注意:此时会被要求输入root密码,并且选择是否新建一个具有管理员权限的sudo账户、。要注意弱口令问题，设置的密码应该难于猜解）**

3.  进行网络配置

   将centos的网卡设置为桥接到主机网卡上，则虚拟机与主机在同一网段。由于防火墙原因，主机不能被虚拟机ping通， 此时设置防火墙的入站规则，将ICMPv4 设为启动，并且设置作用域远程ip地址只有虚拟机的ip， 这样主机和虚拟机可以互相ping通

   

4.  更新软件源并且安装vsftpd， 这样就可以在外部通过SecureCRT的ssh2协议连接到虚拟机，避免主机性能不好导致在虚拟机里面很卡顿。并且可以通过sftp协议来上传和下载文件。**（这里感觉会有不少安全问题因为vsftpd安装后是默认配置，但是暂时还不懂有什么毛病）**

   //接下来都是使用远程客户端CRT操作

5. 用sftp上传nginx安装包, 然后解压。依然要注意验证完整性。

6. 进入nginx安装包目录下使用./configure文件 来配置路径等安装配置,之后安装即可。

7. 安装Nginx后发现主机不能访问centos的80端口， 提示为timeout。则应该将虚拟机的80端口打开，centos7的防火墙配置命令已经改为`firewall-cmd`。通过

   命令`firewall-cmd --zone=public --add-port=80/tcp --permanent`来开启80端口， 

   * --zone：作用域

   * -add-port=80/tcp:添加端口, 格式:端口/协议

   * -permanent:永久生效, 无此参数则重启失效

     然后重启防火墙,使其生效。

   此时已经完成Nginx服务器的搭建，完成最基础的部分。

   **此部分最终效果：主机可以使用浏览器访问虚拟机的Nginx欢迎页面。**

   ps:关机前要关闭Nginx 使用`./nginx -s quit`. 更改centos网络配置,路径为`/etc/sysconfig/network-scripts/ifcfg-ensxx `, "ensxx"其中xx根据不同系统而可能有所不同以实际为准.如果改了centos静态ip后使用crt连接经常出现`connect reset` 是因为设置的ip与已有ip冲突,重设即可.

## 1.2  安装PHP

1. 还是下载php安装包, 并且通过tar解压, 然后通过./configure来设置安装信息.配置完成进行make, 但是报错```libtool: link: `ext/opcache/zend_accelerator_hash.lo' is not a valid libtool object```, 网上说法是因为连接库未成功添加,解决方法是`make clean`然后再`make` 

2. 执行完make后提示执行make test ,执行后提示错误内容为`Bug #74090 stream_get_contents maxlength>-1 returns empty string on windows [ext/standard/tests/streams/bug74090.phpt]`, php版本为7.3.7。暂时忽略BUG继续向下进行。

3. `make install` 安装php及其依赖包

4. 编辑php配置文件,位置`/etc/php.ini` 将`cgi.fix_pathinfo=1`从注释中释放出来。

5. 编辑php-fpm配置文件,位置:`/etc/php-fpm.d/www.conf` 。创建nginx用户和组。之后修改的内容：

   ```
   user=nginx
   
   group=nginx
   ```

   

6.在nginx配置文件中,设置关于php的解析路径和解析端口。

```
 location ~ \.php?.*$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
```



**此时的效果为:可以正常访问服务器的php欢迎页面**

php-fpm在默认本地的9000端口监听，nginx将php请求发送到改端口进行处理并返回结果。



## 1.3 安装MySQL

1. 补充mysql的源, 并且更新yum仓库缓存

```
wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum update
```



2. `yum install mysql mysql-community-server`安装mysql包,和服务器包,运行`systemctl start mysqld`, 然后输入mysql即可发现mysql已经安装成功,接下来对其配置且与nginx和php对接。

3. 使用`grep 'temporary password' /var/log/mysqld.log`无返回,则mysql默认密码为空.

4. `mysql -uroot`为不使用密码登录到mysql, 进入之后有`mysql >`标识, 然后输入 `set password for 'root'@'localhost' = password('mypasswd');`(注意命令有分号)返回`Query OK, 0 rows affected (0.00 sec)`则密码设置成功,输入`exit`退出mysql

5. `mysql -uroot -p`输入刚才密码能登录进去.

6. 创建测试文件test.php

   ```
   <?php
   
          //$link_id=mysql_connect('主机名','用户','密码');
   
          $link_id=mysql_connect('localhost','root','mypasswd') or mysql_error();
   
   
   
          if($link_id){
   
                  echo "mysqlsuccessful by linuxidcde lake !";
   
          }else{
   
                   echo mysql_error();
   
          }
   
   ?>
   ```

   **放置到nginx资源文件夹的html目录下, 在主机访问成功!**

## 小结



第一部分主要是基础服务器的搭建, 包括服务器本身, php和数据库MySQL。主要流程为安装nginx， 之后打开防火墙80端口和nginx服务使服务器可以处理外部请求。之后安装php和php-fpm， 安装后启动php-fpm服务来等待nginx发送给它的请求。配置nginx使其将php请求发送给php-fpm。mysql则在php解析中提供数据支撑。



![三者关系](E:\学习\web_security\信安之路\第一周\三者关系.png)

# 2. 可能存在的安全问题

此次环境搭建共涉及到5个部分：Linux系统 、nginx 、php、MySQL和人（也就是管理员我自己）。应确保这5个部分都是可信任的才能在更大的程度上保证安全。既要保证软件的来源和配置可靠， 还要保证管理员有足够的安全意识。总之让信息流出的越少越好 。下面依次是我认为可能存在安全问题的地方：

## 2.1确定安装的不是假冒伪劣软件

在寻找所需软件时 ， 时常会遇到因为网站不在国内而网速过慢的情况， 此时通常做法是寻找国内的镜像网站下载，有时候找不到合适的镜像也可能去博客论坛等地方寻找， 此时应该注意，下载的东西是否是安全的， 至少要验证其md5值是否与官方网站相同，以防被加塞或暗改。官网下的就一定是安全的吗？这就得让管理员要时常关注安全动态。首先保障使用的东西没有问题。

## 2.2 关于linux的问题

安装linux必须要面对的问题就是用户和用户组的关系。root密码不能是弱密码， 文件和文件夹的权限是否合理以及符合最小原则。安装nginx， php和mysql后应对其用户，用户组，其文件夹和相关文件设置合理的权限。对于nginx要确定防火墙开放80端口后是否面临什么问题。

## 2.3 nginx，php和MySQL

这部分刚才接触， 还不太理解。 MySQL安装后要设置密码，同样不能是弱密码。让我困惑的地方是，在安装完MySQL后，未对nginx和php进行任何设置就能通过测试的php文件从主机登录MySQL， 这也太随便了， 这样岂不是有可能访问到其他服务。

# 3. 加固

## 3.1 关于nginx

### 3.1.1 不返回nginx的版本即错误信息

随着版本的更迭，以及辛辛苦苦的漏洞发掘者的发掘， 以往的不同版本都会有或大或小的问题，为防止nginx不是最新或者其他漏洞， 不应该让其他人知道服务器版本， 这样可以增加其找到漏洞的成本。方法：

在nginx.conf中添加`server_tokens off; `

```
http{
    include      mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    server_tokens off; 
    ... ...
```

输入`curl -I http://localhost/wavsep`验证, 发现Server条目只显示nginx,不再显示nginx的版本号。

### 3.1.2 自定义缓存

自定义缓存,对于防止缓冲区溢出攻击能起一点作用是一点吧。在nginx.conf中

```
http{
    ... ...
    server{
        ... ...
        client_body_buffer_size  16K;
       client_header_buffer_size  1k;
        client_max_body_size  1m;
       large_client_header_buffers  4  8k;
        ... ...

```

缓冲区大小依据实际情况而定

### 3.1.3 设置timeout

设置连接的超时时间,不让长时间等待握手连接,能对dos攻击起一点点缓解作用。nginx.conf

```
http {
    ... ...
       client_body_timeout   10;
       client_header_timeout  30;
       keepalive_timeout     30  30;
       send_timeout          10;

```

### 3.1.4 配置日志

记录服务器的访问情况用于分析和查找问题. 此处设置了格式

```
http {
    ......
    log_format  main  '$remote_addr - $remote_user [$time_local]"$request" 				''$status $body_bytes_sent "$http_referer"''"$http_user_agent" 				"$http_x_forwarded_for"';
    access_log logs/ access.log  main;
```

### 3.1.5 限制访问方法

限制请求方法.nginx.conf, 错误回应视实际情况而定

```
server{
       ... ...
       if ($request_method !~ ^(GET|HEAD|POST)$) {        
                     return404;
              }
       ... ...
```

### 3.1.6 限制部分ip的访问

限制某一类ip或者某个网段等的访问

(此条用到了geoip的库, 作用为只允许中国和美国的ip访问)在nginx.conf中

```
if ( $geoip_country_code !~  ^(CN|US)$ ) {
        return 403;
}
```

或者单独进行ip限制, 如在nginx.conf中:

```
location/ {
    deny  192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny  all;
}
```

### 3.1.7 限制不正常的user-agent的访问

user-agent为浏览器标识, 虽然一些扫描工具可以改此项, 但是加上限制还是能避免一些水平不够的恶意骚扰。具体还可以根据日志的分析情况添加

```
    if ($http_user_agent ~* "java|python|perl|ruby|curl|bash|echo|uname|base64|decode|md5sum|select|concat|httprequest|httpclient|nmap|scan" ) {
        return 403;
    }
    if ($http_user_agent ~* "" ) {
        return 403;
    }
```



### 3.1.8 限制特定的文件扩展名

限制对于一些特别的文件类型的访问。

```
    location ~* \.(bak|save|sh|sql|mdb|svn|git|old)$ {
    rewrite ^/(.*)$  $host  permanent;
    }
```

### 3.1.9 url过滤敏感字

过滤url请求的敏感字, 防止被测试出敏感信息.此处列出的仅仅是一些简单的,实际的要复杂的多

```
    if ($query_string ~* "union.*select.*\(") { 
        rewrite ^/(.*)$  $host  permanent;
    } 
     
    if ($query_string ~* "concat.*\(") { 
        rewrite ^/(.*)$  $host  permanent;
    }
```

### 3.1.10 目录权限

没有上传需求的目录应该设为只读。此处的例子为:建立网站数据文件夹data和傀儡只读文件夹/var/www/html, 然后将data文件夹挂载到html文件夹, 这样也访问到data中的数据, 但是却不能对/var/www/html文件夹进行写操作。对网站的数据的更新则是对于data文件夹进行， 而网站的根目录却设置为/var/www/html。

```
mkdir -p /data
mkdir -p /var/www/html
mount --bind /data /var/www/html
mount -o remount,ro --bind /data /var/www/html
```



## 3.2 关于php部分

此部分主要是限制php的函数, 以及目录权限, 修改文件为php.ini

### 3.2.1 禁用危险函数

进制更改用户组, 更改用户等危险操作.

```
disable_functions = passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,fsocket,phpinfo #禁用的函数

```

### 3.2.2 避免暴露php信息

```
expose_php = off
```

### 3.2.3 关闭php的错误信息显示

```
display_errors = off 
```

### 3.2.4 不允许调用dl

enable_dl()函数允许用户在运行时加载PHP扩展的，也就是说，让一个脚本执行。

```
enable_dl = off 
```

### 3.2.5 避免远程调用文件

```
allow_url_include = off
```

### 3.2.6 开启http-only 

使像javascript等浏览器脚本语言不能访问cookie

```
session.cookie_httponly = 1
```

### 3.2.7 定义上传目录

制定用户的上传目录

```
upload_tmp_dir = /tmp
```


### 3.2.8 限制用户访问目录

将用户访问文件的活动范围限制在指定范围, 可以用 "."  来表示当前目录, 注意目录路径要完整

```
open_basedir = /yourpath
```

## 3.3 关于MySQL

### 3.3.1 修改密码

如果还未设置密码的话可以直接输入`mysql`进入, 输入

`update user set password=password('yourpassword')where user='root';`

然后刷新权限`flush privileges;`

### 3.3.2 删除其他默认账号和默认数据库

进入mysql后操作

```
delete from db; //删除存放数据库的表信息，因为还没有数据库信息。
delete from user where not (user=’root’) ; // 删除初始非root的用户
```

### 3.3.3 禁用远程连接数据库

数据库会默认开启3306端口, 不用的话就关闭

文件/etc/my.cnf

在`[mysqld]`条目下添加`skip-networking`, 重启mysql生效

### 3.3.4 限制连接用户数量

位置同上一条

添加`max_user_connections = 2`

### 3.3.5 将mysql用户设置为不可登录系统

在/etc/passwd中, 将mysql用户的/bin/bash更改为`/sbin/nologin`

## 小结

无论加固工作做的如何,一定要经常备份,并且保证备份隔离, 异地隔离更好. 要时常注意系统的变化, 如哪些文件被更改, 更改的是否正确。经常分析日志， 查看是否有异常情况。至此环境搭建已经结束, nginx+php+mysql已经联动成功, 但是加固工作时无休止的, 当理解更进一步时, 加固工作也应到位, 毕竟对人的加固也同样重要。


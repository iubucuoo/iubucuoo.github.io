# 手动安装LNMP与wordpress


<!--more-->

[参考文章](https://www.alibabacloud.com/help/zh/doc-detail/173042.htm)

[参考文章2](https://help.aliyun.com/document_detail/184111.html)

## 步骤1：安装Nginx
因为本人早先已经装过nginx，所以采用的手动安装LNMP。

1.  运行以下命令安装Nginx。
    
    本教程将选用Nginx 1.16.1版本。
    
    **说明**  您可以访问[Nginx官方安装包](http://nginx.org/packages/centos/8/x86_64/RPMS/)获取适用于CentOS 8系统的多版本的Nginx安装包。
    
    ```bash
    dnf -y install http://nginx.org/packages/centos/8/x86_64/RPMS/nginx-1.16.1-1.el8.ngx.x86_64.rpm
    ```
    
2.  运行以下命令查看Nginx版本。
    
    ```undefined
    nginx -v
    ```
    
    查看版本结果如下所示。
    
    ```yaml
    nginx version: nginx/1.16.1
    ```
## 步骤2：安装MySQL

1.  运行以下命令安装MySQL。
    
    ```typescript
    dnf -y install @mysql
    ```
    
2.  运行以下命令查看MySQL版本。
    
    ```undefined
    mysql -V
    ```
    
    查看版本结果如下所示。
    
    ```java
    mysql  Ver 8.0.17 for Linux on x86_64 (Source distribution)
    ```
    

## 步骤3：安装PHP

1.  运行以下命令添加并更新epel源。
    
    ```sql
    dnf -y install epel-release
    dnf update epel-release
    ```
    
2.  运行以下命令删除缓存的无用软件包并更新软件源。
    
    ```python
    dnf clean all
    dnf makecache
    ```
    
3.  启用`php:7.3`模块。
    
    **说明**  本示例使用`php:7.3`版本。如果您需要使用`PHP 7.4`版本，需要先安装remi源。remi源安装命令为dnf -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm。
    
    ```javascript
    dnf module enable php:7.3
    ```
    
4.  运行以下命令安装PHP相应的模块。
    
    ```python
    dnf install php php-curl php-dom php-exif php-fileinfo php-fpm php-gd php-hash php-json php-mbstring php-mysqli php-openssl php-pcre php-xml libsodium
    ```
    
5.  运行以下命令查看PHP版本。
    
    ```undefined
    php -v
    ```
    
    查看版本结果如下所示。
    
    ```yaml
    PHP 7.3.5 (cli) (built: Apr 30 2019 08:37:17) ( NTS )
    Copyright (c) 1997-2018 The PHP Group
    Zend Engine v3.3.5, Copyright (c) 1998-2018 Zend Technologies
    ```
## 步骤4：配置Nginx
本人配置都配在/etc/nginx/nginx.conf内，直接修改当前配置的内容
1.  运行以下命令打开默认配置文件。
    
    ```javascript
    vi /etc/nginx/nginx.conf
    ```
2.  按i进入编辑模式。
3.  在`location`大括号内，修改以下内容。
    
    ```bash
    location / {
        #将该路径替换为您的网站根目录。
        root   /usr/share/nginx/html1/wordpress;
        #添加默认首页信息index.php。
        index  index.html index.htm index.php;
    }
    ```
    
4.  去掉被注释的`location ~ \.php$`大括号内容前的`#`，并修改大括号的内容。
    
    修改完成如下所示。
    
    ```php
    location ~ \.php$ {
        #将该路径替换为您的网站根目录。
        root           /usr/share/nginx/html1/wordpress;
        #Nginx通过unix套接字与PHP-FPM建立联系，该配置与/etc/php-fpm.d/www.conf文件内的listen配置一致。
        fastcgi_pass   unix:/run/php-fpm/www.sock;
        fastcgi_index  index.php;
        #将/scripts$fastcgi_script_name修改为$document_root$fastcgi_script_name。
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        #Nginx调用fastcgi接口处理PHP请求。
        include        fastcgi_params;
    }
    ```
    说明**  Nginx与PHP-FPM进程间通信方式有两种。
    
    -   TCP Socket：该方式能够通过网络，可用于跨服务器通信的场景。
    -   UNIX Domain Socket：该方式不能通过网络，只能用于同一服务器中通信的场景。
    
5.  按下Esc键，并输入`:wq`保存退出文件。
6.  运行以下命令启动Nginx服务。
    
    ```sql
    systemctl start nginx
    ```
    
7.  运行以下命令设置Nginx服务开机自启动。
    
    ```bash
    systemctl enable nginx
    ```
## 步骤5：配置MySQL

1.  运行以下命令启动MySQL，并设置为开机自启动。
    
    ```sql
    systemctl enable --now mysqld
    ```
    
2.  运行以下命令查看MySQL是否已启动。
    
    ```undefined
    systemctl status mysqld
    ```
    
    查看返回结果中`Active: active (running)`表示已启动。
    
3.  运行以下命令执行MySQL安全性操作并设置密码。
    
    ```undefined
    mysql_secure_installation
    ```
    
    命令运行后，根据命令行提示执行如下操作。
    
    1.  输入Y并回车开始相关配置。
    2.  选择密码验证策略强度，输入2并回车。
        
        策略0表示低，1表示中，2表示高。建议您选择高强度的密码验证策略。
        
    3.  设置MySQL的新密码并确认。
        
        本示例设置密码`PASSword123！`。
        
    4.  输入Y并回车继续使用提供的密码。
    5.  输入Y并回车移除匿名用户。
    6.  设置是否允许远程连接MySQL。
        -   不需要远程连接时，输入Y并回车。
        -   需要远程连接时，输入N或其他任意非Y的按键，并回车。
    7.  输入Y并回车删除`test`库以及对`test`库的访问权限。
    8.  输入Y并回车重新加载授权表。
## 步骤6：配置PHP

1.  修改PHP配置文件。
    1.  运行以下命令打开配置文件。
        
        ```bash
        vi /etc/php-fpm.d/www.conf
        ```
        
    2.  按i进入编辑模式。
    3.  找到`user = apache`和`group = apache`，将`apache`修改为`nginx`。
        
        ![php-fpm conf](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/6902649951/p130166.png)
        
    4.  按下Esc键，并输入`:wq`保存退出文件。
1.  运行以下命令启动`PHP-FPM`。
    
    ```sql
    systemctl start php-fpm
    ```
    
2.  运行以下命令设置`PHP-FPM`开机自启动。
    
    ```bash
    systemctl enable php-fpm
    ```
## 步骤7：搭建WordPress网站
1.  远程连接部署好LNMP环境的服务器，配置WordPress数据库。
    1.  远程连接服务器
    2.  进入MySQL数据库。
        
        使用`root`用户登录MySQL，并输入密码。密码为您在搭建环境时为数据库设置的密码。
        
        ```undefined
        mysql -uroot -p
        ```
        
    3.  为WordPress网站创建数据库。
        
        本教程中数据库名为`wordpress`。
        
        ```sql
        create database wordpress;
        ```
        
    4.  创建一个新用户管理WordPress库，提高安全性。
        
        MySQL在5.7版本后默认安装了密码强度验证插件validate_password。您可以登录MySQL后查看密码强度规则。
        
        ```sql
        show variables like "%password%";
        ```
        
        本教程中创建新用户`user`，新用户密码为`PASSword123.`。
        
        ```sql
        create user 'user'@'localhost' identified by 'PASSword123.';
        ```
        
    5.  赋予用户对数据库`wordpress`的全部权限。
        
        ```sql
        grant all privileges on wordpress.* to 'user'@'localhost';
        ```
        
    6.  使配置生效。
        
        ```undefined
        flush privileges;
        ```
        
    7.  退出MySQL。
        
        ```php
        exit;
        ```
        
2.  下载并解压WordPress，然后移动至网站根目录。
    1.  进入Nginx网站根目录，下载WordPress压缩包。
        
        本示例默认安装的是WordPress英文版本。
        
        ```bash
        cd /usr/share/nginx/html
        wget https://wordpress.org/wordpress-5.4.2.zip
        ```
        
        如果您需安装WordPress中文版本，需运行命令wget https://cn.wordpress.org/latest-zh_CN.zip，下载WordPress中文版本压缩包。同时您需要注意，后续操作中压缩包的名称必须替换为latest-zh_CN.zip。
        
    2.  解压WordPress压缩包。
        
        ```python
        unzip wordpress-5.4.2.zip
        ```
        
    3.  将WordPress安装目录下的`wp-config-sample.php`文件复制到`wp-config.php`文件中，并将`wp-config-sample.php`文件作为备份。
        
        ```bash
        cd /usr/share/nginx/html/wordpress
        cp wp-config-sample.php wp-config.php
        ```
        
    4.  编辑`wp-config.php`文件。
        
        ```undefined
        vim wp-config.php
        ```
        
    5.  按i键切换至编辑模式，根据已配置的WordPress数据库信息，修改MySQL相关配置信息，修改代码如下所示。
        
        WordPress网站的数据信息将通过数据库的`user`用户保存在名为`wordpress`的数据库中。
        
        ```sql
        // ** MySQL 设置 - 具体信息来自您正在使用的主机 ** //
        /** WordPress数据库的名称 */
        define('DB_NAME', 'wordpress');
        
        /** MySQL数据库用户名 */
        define('DB_USER', 'user');
        
        /** MySQL数据库密码 */
        define('DB_PASSWORD', 'PASSword123.');
        
        /** MySQL主机 */
        define('DB_HOST', 'localhost');
        ```
        
    6.  修改完成后，按下Esc键后，输入`:wq`并回车，保存退出配置文件。
  
4.  安装并登录WordPress网站。
    1.  在本地物理机上使用浏览器访问`公网IP`，进入WordPress安装页面。
    2.  填写网站基本信息，然后单击安装WordPress。
        
        填写信息参数说明：
        
        -   站点标题(Site Title)：WordPress网站的名称。例如：demowp。
        -   用户名(Username)：登录WordPress时所需的用户名，请注意安全性。例如：testwp。
        -   密码(Password)：登录WordPress时所需的密码，建议您设置安全性高的密码。例如：Wp.123456。
        -   您的电子邮件(Your Email)：用于接收通知的电子邮件。例如：1234567890@qq.com。
        
    3.  单击登录。
    4.  输入在安装WordPress时设置的用户名`testwp`和密码`Wp.123456`，然后单击登录。
        
        成功进入您个人的WordPress网站。
       

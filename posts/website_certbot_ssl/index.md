# cerbot添加ssl证书

<!--more-->
# 使用[certbot](https://certbot.eff.org/)添加ssl证书
参考文献：
1.  [Centos7安装certbot获得并更新证书（使用Snap）](https://blog.csdn.net/AlistairEd/article/details/113804554)
2.  [Nginx on CentOS/RHEL 8](https://certbot.eff.org/lets-encrypt/centosrhel8-nginx.html)
3. [在 CentOS 上安装 snap](https://snapcraft.io/docs/installing-snap-on-centos)
-   开始之前需要先**卸载Certbot和其他Certbot OS包**可参考这篇《[卸载 certbot-auto](https://certbot.eff.org/docs/uninstall.html)》
### 安装Snapd
Snap在CentOS 7.6版本以上均可用，如果不知道你的版本，使用如下命令查看

`cat /etc/centos-release`
-   **添加CentOS repository**
* [什么是EPEL 及 Centos上安装EPEL](https://zhidao.baidu.com/question/1695420895325401068.html)

   CentOS 8

   `1. sudo dnf install epel-release `

   `2. sudo dnf upgrade`

   CentOS 7
   
   `sudo yum install epel-release`

-  **添加EPEL repository后，进行Snapd的安装**

   `sudo yum install snapd`

-  **安装后，需要启用管理snap通信套接字的systemd unit**

   `sudo systemctl enable --now snapd.socket`

-  **为了启用_classic_ snap的支持，需要创建如下软连接**

   `sudo ln -s /var/lib/snapd/snap /snap`

这里可能会出现问题，重启即可，启动之后再执行该命令

###  安装Certbot
-   **升级snap**
执行如下命令以保证Snap为最新的版本
```
	sudo snap install core
	sudo snap refresh core
```
-  **安装Certbot**

   `sudo snap install --classic certbot`

-  **配置Certbot命令行**

   执行如下命令以确保Certbot命令行可用
`sudo ln -s /snap/bin/certbot /usr/bin/certbot`

-  **运行Certbot（二选一）**
	1. 运行此命令获取证书，并让Certbot自动编辑Nginx配置以提供服务，只需一步即可打开HTTPS访问
`sudo certbot --nginx`
    
       **注：** Certbot默认nginx配置文件在 **/etc/nginx/nginx.conf** 或 **/usr/local/etc/nginx/nginx.conf**,若你的nginx配置文件不在此处需在命令后加上 **--nginx-server-root /usr/local/nginx/conf**
	
	2. 仅获得证书。如果你希望手动配置nginx，输入如下命令
`sudo certbot certonly --nginx`

### 自动续期

您系统上的 Certbot 软件包带有一个 cron 作业或 systemd 计时器，它们会在您的证书到期之前自动更新您的证书。除非您更改配置，否则您无需再次运行 Certbot。
您可以通过运行以下命令来测试证书的自动续订：
`sudo certbot renew --dry-run`







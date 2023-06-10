# 配置域名DNS

<!--more-->

### 参考文章原文：[这里](https://www.10besty.com/how-to-start-a-wordpress-website-on-vps/)
### 配置域名DNS

在HostWinds找到域名Nameserver,如

`mdns1.hostwindsdns.com`。

`mdns2.hostwindsdns.com`。

到namesilo自己的域名设置的位置勾选要设置的域名然后changeserver，把这两个hostwinds的nameserver写入。

设置好之后到hostwinds的Domain添加一个domain，输入域名，状态显示为pending，要等待一段时间才会生效，一般很快，最长不超过48小时。

点击右测的Actions子菜单如果为Records表示已经生效，否则为Check。

生效之后点Records输入两个记录

1是`@+ip`

2是`www+ip`

完成后域名与ip就已经建立关系。

#### 配置防火墙

    # 安装防火墙，如果系统中已存在，那就会执行更新
    yum -y install firewalld
    # 开机启动防火墙
    systemctl enable firewalld
    # 立即启动防火墙
    systemctl restart dbus
    systemctl restart firewalld
    # 查看防火墙的状态
    systemctl status firewalld
    # 打开2200（自定义端口，修改SSH默认端口用，见下文）、80（HTTP）、443（HTTPS）端口
    firewall-cmd --zone=public --add-port=2200/tcp --permanent
    firewall-cmd --zone=public --add-port=80/tcp --permanent
    firewall-cmd --zone=public --add-port=443/tcp --permanent
    # 更新防火墙规则
    firewall-cmd --complete-reload
    # 查看所有打开的端口和服务
    firewall-cmd --zone=public --list-ports
    firewall-cmd --list-services
防火墙安装并配置完毕
现在在浏览器中输入域名可以访问wp的设置主页





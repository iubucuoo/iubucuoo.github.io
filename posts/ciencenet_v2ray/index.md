# V2ray安装与使用

<!--more-->
[v2ray官网](https://www.v2ray.com/)
详细教程及原理内容可查看[V2Ray白话文教程](https://toutyrater.github.io/)
## 其他教程
[搭建详细图文](https://github.com/233boy/v2ray/wiki/V2Ray搭建详细图文教程)
[在 Heroku 搭建 V2Ray **(免费)** ](https://ibcl.us/Heroku-V2Ray_20191014/)
## 服务器安装v2ray

1.  安装wget 

    `yum -y install wget`

2. 下载脚本

    `wget https://install.direct/go.sh`

3. 安装unzip

    因为centos不支持apt-get，我们需要安装unzip

    `yum install zip unzip`

4. 执行安装

    `bash go.sh`

5. 安装脚本

    运行以下脚本：

    `curl -Ls https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh | sudo bash`

    里面可能会有报错信息，先不用管

6. 获取用户ID

    可以通过[UUID Generator](https://www.uuidgenerator.net/)这个网站获取，也可以通过Linux命令生成：
    `cat /proc/sys/kernel/random/uuid`

7. 配置

    配置文件位于/etc/v2ray/config.json，可使用vim指令打开文本，也可以使用SSH图形话SFTP客户端如[FileZilla](https://filezilla-project.org/)，进入路径下载文件编辑，

8. 启动V2Ray

    启动：

    `systemctl start v2ray`

    停止

    `systemctl stop v2ray`

    重启

    `systemctl restart v2ray`

9. 打开防火墙

    centos系统防火墙需要开启

    查看已开放端口

    `firewall-cmd --zone=public --list-ports`

    添加开放端口

    `firewall-cmd --zone=public --add-port=12345/tcp --permanent`

    重载防火墙配置，不然查看开放端口都查不到，也不能用，重载配置后即可

    `firewall-cmd --reload`

    如果哪一天发现怎么无法使用了，有可能是IP被屏蔽，也有肯能是端口被封，这个时候就需要换个端口，别忘记防火墙开启新端口，那旧端口就可以删除了。删除端口：

    `firewall-cmd --zone=public --remove-port=123456/tcp --permanent`


## v2ray多用户配置

* 修改时注意json格式

1. 用指令：`cat /etc/v2ray/config.json` 打开配置文件config.json可以看到当前的配置。

2. 运行指令`cat /proc/sys/kernel/random/uuid`或者[UUID Generator](https://www.uuidgenerator.net/)获取uuid

3. 用指令编辑config.json文件，指令： `vim /etc/v2ray/config.json`,输入i进入编辑模式，复制粘贴inbounds[ ]中的一段内容，修改对应参数即可，参数不懂可用搜索引擎查找，接着Esc退出编辑模式，使用指令：`:wq`保存并退出文件。

4. 可再用1步骤查看是否修改成功，然后重启v2ray服务：
    
    `systemctl restart v2ray`
    
    别忘记开启防火墙端口
    
    `firewall-cmd --``zone=public --add-port=12346/tcp --permanent`
    
    重载防火墙配置

    `firewall-cmd --reload`
    
    如果不想改端口，可以复制clients[]中的一段内容并修改参数即可
## 客户端配置

    [下载v2ray-core](https://github.com/v2ray/v2ray-core/releases)

    [下载v2ray-exe](https://github.com/2dust/v2rayN/releases)

1. 两个下载完成解压，将exe文件夹下的V2RayN.exe和一个汉化文件夹复制到core下载解压后的目录。

2. 运行V2RayN.exe并且将配置文件相应的对号入座即可完成配置

3. 开启代理，右击运行的客户端图标，选中HTTP代理->开启PAC 即可

* 手机端软件下载地址：[神一样的工具们](https://www.v2ray.com/awesome/tools.html?tdsourcetag=s_pcqq_aiomsg)



# 安装dnf



<!--more-->

#### 安装 dnf

DNF 并未默认安装在 RHEL 或 CentOS 7系统中

1.  为了安装 dnf ，必须先安装并启用 epel-release 依赖  
    `yum install epel-release`
    
2.  使用 epel-release 依赖中的 YUM 命令来安装 dnf 包  
    `yum install dnf`
    

#### 常见的 dnf 命令介绍

1.  查看 dnf 版本  
    `dnf --version`
2.  查看系统中可用的 dnf 软件库  
    `dnf repolist`
3.  查看系统中可用和不可用的软件库  
    `dnf repolist all`
4.  列出所有RPM包  
    `dnf list`
5.  列出已经安装的RPM包  
    `dnf list installed`
6.  列出可供安装的RPM包  
    `dnf list available`
7.  搜索某包 (以搜索nginx为例)  
    `dnf search nginx`
8.  查看某包的详情  
    `dnf info nginx`
9.  安装包  
    `dnf install nginx`
10.  升级包  
    `dnf update nginx`
11.  检查系统软件包更新  
    `dnf check-update`
12.  升级系统中所有软件包  
    `dnf update OR dnf upgrade`
13.  删除包  
    `dnf remove nginx OR dnf erase nginx`
14.  删除无用孤立的软件包  
    `dnf autoremove`
15.  删除缓存的无用软件包  
    `dnf clean all`
16.  获取有关某条命令的使用帮助  
    `dnf help clean`
17.  重新安装特定软件包  
    `dnf reinstall nginx`
18.  回滚某个特定软件的版本  
    `dnf downgrade nginx`



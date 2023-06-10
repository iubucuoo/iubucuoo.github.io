# php命令行操作



<!--more-->
查看php-fpm状态
``` liunx
    systemctl status php-fpm
```
启动php-fpm
``` liunx
    systemctl start php-fpm
```    
停止php-fpm
``` liunx
    systemctl stop php-fpm
```
重启php-fpm
``` liunx
    systemctl restart php-fpm
```
查看php-fpm位置
```liunx
ps -ef | grep 'php-fpm'
```

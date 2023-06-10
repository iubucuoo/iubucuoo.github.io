# nginx命令行操作


<!--more-->
查看nginx服务器的状态
``` liunx
sudo systemctl status nginx
```
开启nginx
``` liunx
sudo systemctl start nginx
```

停止nginx
``` liunx
sudo systemctl stop nginx
```
重启nginx
``` liunx
sudo systemctl restart nginx
```

上面命令行无法关闭nginx时候使用
关闭：
``` liunx
sudo killall nginx
```
开启：
``` liunx
sudo nginx或者sudo service nginx start
```

设置开机自启动
```liunx
systemctl enable nginx
```


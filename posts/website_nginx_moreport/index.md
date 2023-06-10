# nginx配置多个域名使用同一个端口


<!--more-->
## 方法1
随着服务器性能的提升和业务的需求，一台服务器上往往会同时有多个服务，这些服务都希望监听80端口，比如有a.com和b.com。这时候我们可以使用nginx的代理转发功能帮我们实现共用80端口的需求。

先在两个空闲的端口上分别部署项目（非80，假设是8080和8081）nginx配置如下：
``` conf
# a项目配置nginx
server {
    listen       8080;
    root         /usr/share/nginx/html;    #这里是默认路径，生产中代码存放路径：root /web/vue-base-demo/dist/;
    index        index.html;
    location / {}
}
# b项目配置nginx
server {
    listen       8081;
    root         /usr/share/nginx/html;    #这里是默认路径，生产中代码存放路径：root /web/react-base-demo/build;
    index        index.html;
    location / {}
}
```
紧接着如果已经做好域名解析，希望a.com打开a项目，b.com打开b项目。我们需要再做两个代理，如下:
``` conf
# nginx 80端口配置 （监听a二级域名）
server {
    listen  80;
    server_name     a.com;
    location / {
        proxy_pass      http://localhost:8080; # 转发
    }
}
# nginx 80端口配置 （监听b二级域名）
server {
    listen  80;
    server_name     b.com;
    location / {
        proxy_pass      http://localhost:8081; # 转发
    }
}
```
nginx如果检测到a.com的请求，将原样转发请求到本机的8080端口，如果检测到的是b.com请求，也会将请求转发到8081端口。

测试：浏览器输入http://a.com或http://b.com即可。
## 方法2
```php
server  
{  
listen             80;  
server_name  www.server110.com;                          #绑定域名  
index index.htm index.html index.php;   #默认文件  
root /home/www/server110.com;                     #网站根目录
include location.conf;                         #调用其他规则，也可去除
}
server  
{  
listen             80;  
server_name  msn.server110.com;                          #绑定域名  
index index.htm index.html index.php;   #默认文件  
root /home/www/msn.server110.com;               #网站根目录
include location.conf;                         #调用其他规则，也可去除
}
```
不带www的域名加301跳转
若不带 www 的域名需加 301 跳转，则先绑定不带 www 的域名，且无需写网站目录，直接进行 301 跳转，如：
```ruby
server
{
listen 80;
server_name server110.com;
rewrite ^/(.*) http://www.server110.com/$1 permanent;
}
```
添加404网页  
添加 404 网页，可以直接在配置中添加，如：
```php
server  
{  
listen             80;  
server_name  www.server110.com;                          #绑定域名  
index index.htm index.html index.php;   #默认文件  
root /home/www/server110.com;                     #网站根目录
include location.conf;                         #调用其他规则，也可去除
error_page 404   /404.html;  
}
```


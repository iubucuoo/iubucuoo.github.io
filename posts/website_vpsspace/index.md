# 搭建网页记录

<!--more-->
# 搭建个人页记录

## 常用命令操作
1. **rm -rf 目录名字** 删除文件夹以及文件夹中的所有文件命令
2. **-r  :** 向下递归删除
3.  **-f  :** 直接强行删除，且没有任何提示
4. **rm -f 文件名**  删除文件命令行强行删除文件且无提示
5. **pwd** 查看当前路径
6. **ls -l** 查看当前路径下面的文件
7. **i** 进入编辑状态
8. **:wq** 保存退出

## VPS与域名
根据自己的需求购买VPS 域名并解析IP ,我自己用的VPS是 **[HostWinds](https://www.hostwinds.com/)**  域名是 **[namesilo](https://www.namesilo.com/)** 

## VPS上配置**nginx** 

- **注 :** Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。利用Nginx可以快速在vps上搭建web服务器
1. 远程登陆到vps服务器 先创建三个目录     
    ```bash
      mkdir / blog#目录1，用作git上传的仓库目录，之后会用到
      mkdir / tmp/blog#目录2，用作git clone的目录，之后会用到
      mkdir / usr/share/nginx/html	 #目录3，用作nginx的webserver根目录，如果默认已经创建则忽略
    ```

2. 安装nginx        
    ```bash
      yum update
      yum install nginx
    ```
   此时在本地浏览器访问域名就可以看到nginx的欢迎页面

3. 配置Nginx.conf文件位置通过命令 nginx -t查看 

   vim编辑.conf文件       
   ```bash
    vim /usr/local/nginx/conf/nginx.conf
   ```

   在http{}节点中增加（或修改）
   ```bash
   server {	#服务器
   listen       80;
       server_name  www.test.com; 	# 你的域名
       root /usr/share/nginx/html; # webserver根目录（目录3）
       }
       server {	#跳转，直接输入test.com 或者ip地址，可跳转到www.test.com
       listen	80;
       server_name test.com;
       return 301 http://www.test.com;
       }
   ```
   **注:**  include /etc/nginx/sites-enabled/*; 要注释掉

   否则网站的默认目录会是/var/www/html，而不是/usr/share/nginx/html了   
4. 配置完成
后面只需要将我们博客的内容放到配置文件中指定的webserver目录(/usr/share/nginx/html)，即可通过域名访问我们的博客        	
5. service nginx restart //重启服务
## VPS上建git库
1.  使用git来实现将本地的博客内容推到vps服务器上 
2.  在vps上建一个远程仓库，并且为这个仓库建一个hook，这样每次从本地push时，博客能同步更新  
3.  在vps上，进入之前创建的第一个目录     
    ```bash
      cd /blog     
      git init --bare  
    ```

## 建一个git hook
1. 进入远程仓库的hooks目录进行操作：                 
   ```bash
     cd /blog/hooks
     vim post-receive
   ```

2. 将以下内容写入**post-recevie**文件 
**注:** 这段代码，会将本地每次push到vps上git仓库的内容，放到Nginx webserver目录（目录3）
    ```bash
    GIT_REPO=/blog	#目录1
    TMP_GIT_CLONE=/tmp/blog #目录2 
    NGINX_HTML=/usr/share/nginx/html#目录3
    rm -rf ${TMP_GIT_CLONE} 
    git clone $GIT_REPO $TMP_GIT_CLONE
    rm -rf ${NGINX_HTML}/*
    cp -rf ${TMP_GIT_CLONE}/* ${NGINX_HTML}
    ```
3. 保存并退出，然后修改文件的权限   
    ```bash
    chmod 755 post-receive
    ```
## 编辑网页内容
这里我用到的是[hugo](https://gohugo.io/)

1. 在本地下载好hugo可执行文件

   **Window:**  下载好解压在某个路径  将这个路径添加到环境变量中(不然命令行无法执行到)   

   **Linux:**  `yum install hugo`

2. 然后在hugo主页中进入[教程](https://gohugo.io/getting-started/quick-start/)，按着网页中的方式操作

3. 生成静态文件public后进入下一步部署
## 内容部署到VPS

1. 将前面生成的public目录初始化为git仓库，commit提交，再push到vps上就可以了
    ```bash
    cd public     
    git init     
    git add --all
    git commit -m "Initial blog repo"
    #当前的工作目录下设置远程仓库的地址
    git remote add origin ssh://your_user@VPS_IP:/blog  
    #直接修改工作目录下远程仓库的地址(添加过可以不用再修改)
    git remote set-url origin ssh://your_user@VPS_IP:/blog # 若VPS的ssh默认端口不是22，需另设置ssh端口。  
    git push origin master
    ```
2. 之后会提示输入密码，输入的时候不会显示直接输入就行。
3. push之后就可以通过域名访问到新编辑的网站了

## 上传更新的内容
之后每次编辑完内容，按着之前的方式生成静态网页public，进入public
```bash
  git add --all 添加所有修改
  git commit -m "xxxxxxxxx" 提交修改
  git push origin master
```

## 如何拉取内容到本地


```bash
  cd public # 进入现有的本地目录
  git clone ssh://your_user@VPS_IP:/blog# 第一次使用要把远程仓库的内容克隆到本地
  
  #下拉
  git fetch orgin master # 将远程仓库的 master 分支下载到本地当前 branch 中
  git merge origin/master # 进行合并
  
  # 上传
  git add -all # 把所有变化加入缓存区
  git commit -m "xxxxxxxxx" # 提交到本地仓库
  git push origin master # 推送到远程仓库 origin 的 master 分支
```

## 结束
待后续优化 添加钥避免每次输入密码


# WordPress中安装插件需要ftp怎么办？



<!--more-->

在初次搭建wordpress成功后,老想安装wordpress中有趣的插件时缺发现需要ftp服务，同样的升级插件的话也需要输入ftp的用户名密码。其实不用真的搭建一个ftp服务器,那么,怎么绕过这道程序呢.其实很简单,我们只需在wordpress根目录找到wp-config.php,添加以下代码:
```liunx
define("FS_METHOD","direct");
define("FS_CHMOD_DIR", 0777);
define("FS_CHMOD_FILE", 0777);
```
这时又会提醒无法安装,理由是文件无法创建目录,这个好解决.给wordpress添加权限就好
```liunx
chmod -R 777 wordpress的目录.
```


# python升级pip

<!--more-->
使用pip时报错
#### WARNING: You are using pip version 20.2.4, however version 21.2.2 is available.
#### You should consider upgrading via the 'python -m pip install --upgrade pip' command.
提示正在使用的版本20.2.4，新版本21.2.2，需要升级pip版本

### 升级pip
在命令操作窗口输入
```
python -m pip install --upgrade pip
```

之后等待下载安装，可能会有点久，一直等待就行。

成功之后会提示成功升级到某个新版本：Successfully installed pip-21.2.2

### 查看pip版本
如果不知道版本可通过命令：
```
pip show pip
```

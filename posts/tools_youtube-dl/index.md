# youtube-dl下载利器

<!--more-->

[youtube-dl  **主页**  ](https://ytdl-org.github.io/youtube-dl/index.html)
[youtube-dl github文档](https://github.com/ytdl-org/youtube-dl/blob/master/README.md#readme)

## youtube-dl 视频下载工具
解决平常看到想要下载保存到本地的视频，然而视频网站一般不提供下载接口，这时候需要	借助工具来完成，而youtube-dl刚好能够解决该痛点。


## youtube-dl 安装
###  MAC 端
Mac系统自带python2，所以无需再安装python
调出终端输入：
```
brew install youtube-dl
或者
port install youtube-dl
```
取决与你电脑安装的工具

### Windows 端

先去官网安装Python3  
Python下载官网：[https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/)

下载后，安装的时候记得一定勾选配置环境变量

完成后检测是否安装成功：

按win+R，调出运行，输入cmd回车，再输入

`python`

如果安装成功会显示版本等信息

成功后输入命令：

`exit()`

退出交互端口
装好python后开始安装主角youtube-dl
调用pip安装：
```
pip install youtube-dl
```
回车后会出现下载进度与安装提示

## youtube-dl使用
- 列出支持的网站，终端输入：
    ```
	youtube-dl --list-extractors
    ```
- 下载视频，终端输入：
    ```
	youtube-dl “视频链接”
	如：youtube-dl "www.google.com"
    ```
- 列出所有视频格式，终端输入：
    ```
	youtube-dl -F "视频链接"
	如：youtube-dl -f "www.google.com"
    ```
- 从列表中进行下载
    ```
	youtube-dl -f “视频编码” “视频链接”
	如：youtube-dl -f 1 "www.google.com"
    ```
- 下载到指定目录
    ```
	youtube-dl -o “绝对目录” “视频链接” 
	如：youtube-dl -o '\999\' 'www.google.com'
    ```

## youtube-dl配置

您可以通过将任何受支持的命令行选项放入配置文件来配置 youtube-dl。在 Linux 和 macOS 上，系统范围的配置文件位于 ，`/etc/youtube-dl.conf`用户范围的配置文件位于`~/.config/youtube-dl/config`. 在 Windows 上，用户范围的配置文件位置是`%APPDATA%\youtube-dl\config.txt`或`C:\Users\<user name>\youtube-dl.conf`。请注意，默认配置文件可能不存在，因此您可能需要自己创建它。

例如，使用以下配置文件 youtube-dl 将始终提取音频，而不是复制 mtime，使用代理并将所有视频保存`Movies`在您的主目录中的目录下：
```markdown
# Lines starting with # are comments

# Always extract audio
-x

# Do not copy the mtime
--no-mtime

# Use this proxy
--proxy 127.0.0.1:3128

# Save all videos under Movies directory in your home directory
-o ~/YoutubeDlMovies/%(title)s.%(ext)s

```
下载视频的格式配置示例：

在 Windows 上，您可能需要使用双引号而不是单引号。
```markdown
#下载可用的最佳 mp4 格式或任何其他最佳格式（如果没有可用的 mp4）
#-f ' bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best '

#下载最好的格式，但不超过 480p
#-f ' bestvideo[height<=480]+bestaudio/best[height<=480] '

#下载最佳视频格式但不超过 50 MB
#-f ' best[filesize<50M] '

#通过直接链接通过 HTTP/HTTPS 协议下载可用的最佳格式
#-f ' (bestvideo+bestaudio/best)[protocol^=http] '

#下载最好的视频格式和最好的音频格式而不合并它们
#-f ' bestvideo,bestaudio ' -o ' %(title)sf%(format_id)s.%(ext)s '
```
请注意，配置文件中的选项与常规命令行调用中使用的选项即开关相同，因此 `-` 或  `--` 后不得有空格，例如 `-o` 或 `--proxy` 不是 `- o` 或 `-- proxy`.

`--ignore-config`如果要禁用特定 youtube-dl 运行的配置文件，则可以使用。

`--config-location`如果要为特定的 youtube-dl 运行使用自定义配置文件，也可以使用。


## youtube-dl更新
```
pip install --upgrade youtube-dl
```
如[youtube-dl官网所述](https://ytdl-org.github.io/youtube-dl/download.html)，使用[pip](https://pip.pypa.io/en/stable/)是一种安装youtube-dl的方法，该选项可确保您最终安装了最新的可用版本。

要了解youtube-dl的版本与安装位置，可以使用`pip show youtube-dl`命令

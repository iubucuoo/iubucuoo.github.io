# PotPlayer不支持S/W HEVC(H.265)解码

<!--more-->
**PotPlayer播放MKV格式的视频时跳出视窗显示：不支持S/W HEVC(H.265)解码**
## 解决办法 1
1. [首先下载 FFMPEG64.DLL](https://dll.website/ffmpeg64-dll)
2. 解压下载的压缩包
3. 将文件复制到安装路径D:\MyProgram Files\DAUM\PotPlayer\Module\FFmpeg4下(安装路径视实际情况而定)
4. 打开PotPlayer,点击选项（或者直接按F5），打开设置
5. 选择滤镜-视频解码器-内置解码器/DXVA设置
6. 将左边的「H.265/HEVC」从「内建FFmpeg解码器(建议)」改成「FFmpeg64.dll」
7. 确定之后重新打开就行
## 解决办法 2
1. 直接点弹出提示中的 [**搜索解码器**](https://github.com/Nevcairiel/LAVFilters/releases)
2. 冲github中下载installer.exe文件
3. 自定义安装刚下载好的exe文件
4. 以防万一安装中的选项直接全选
5. 完成后打开potplayer,打开选项-滤镜-全局滤镜优先权
6. 点击 **添加系统滤镜** 打开窗口
7. 如果安装成功会出现“LAV Audio Decoder”、“LAV Splitter”、“LAV Splitter Source”、“LAV Video Decoder”四项,否则安装没成功.
8.  选中 **LAV Splitter Source** 然后确定.
9. 然后在配置窗口中勾选中LAV Splitter Source,优先顺序改成 **强制使用** 并应用
10. 按上面的操作依次添加 **LAV  Video Decoder** 和 **LAV Audio Decoder** 
11. 然后选择左侧菜单栏的 **滤镜** 选项,将激活条件选择为不使用,将使用声音处理过滤(推荐) 前的勾选去掉,然后应用.
12. 到这里重新打开视频就可以正常播放了,此时按 **Ctrl+F1** 快捷键调出播放信息窗口,可以看滤镜列表中启用了刚添加的3项


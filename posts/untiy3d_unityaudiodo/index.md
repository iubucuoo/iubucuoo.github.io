# Unity音频优化Tip

<!--more-->


虽然音频通常不会产生性能瓶颈，但一定的优化仍然可以省下些许内存。

![Inspector dialog box with grey background and white text with an orange voice modulator at the bottom.](https://blog-api.unity.com/sites/default/files/2021-07/Optimizeyourmobilegameperformanceimage8.jpg?imwidth=1260&)

优化AudioClips（音频片段）的导入设置。

### 尽可能地采用未经压缩的原始WAV文件作为资源

Unity会在导入时解压所有的压缩格式（如MP3或Vorbis），在构建时再度进行压缩，这就导致了音频经受两次压缩，最终音质将不可避免地受损。

### 降低音频压缩时的压缩比

当然，我们的确能使用压缩技术来降低音频大小和内存使用：

-   **Vorbis**格式适用于大部分音效（不用循环播放的音效可使用**MP3**）。
-   **ADPCM**格式适用于短促且经常出现的声音（如脚步声、枪声）。相较于未压缩的PCM格式，该格式减小了文件体积，还能在播放时快速解码。

移动设备上的声效最高频率应为22,050赫兹，较低的频率虽然对最终音频的影响不大，但你最好用自己的双耳来做判断。

### 选择恰当的Load Type（加载类型）

Load Type设置应根据音频的体积而异。

-   **短音频（< 200 kb）**可以设为**Decompress on Load（加载时解压），**这时音频会被解压成原始的16位PCM音频数据，产生一定CPU和内存占用，所以音频不宜过大。
-   **中等音频（>=200kb）**可保持**Compressed in Memory（在内存中压缩）。**
-   **长音频（背景音乐）**应设为**Streaming（流播放）**，避免将整个音频资源一次性加载到内存中。

### 释放无声音AudioSources的内存

如果游戏中有静音按钮，静音行为不能只是简单地将音量设置为0。假设玩家无需经常性地开关静音，我们可以调用**Destroy**方法来将**AudioSource**组件从内存中移除。

> Written with [StackEdit](https://stackedit.io/).

# Unity资源优化建议

<!--more-->


资源管线会极大地影响应用的性能。因此，我们最好请一名经验丰富的技术美术来协助制定和实施资源的格式、规格和导入设置，让流程尽可能流畅。

请不要完全依赖默认设置，使用具体平台的重写组件来优化纹理、网格几何形等资源。错误的资源设置可能会导致游戏文件过大、构建时间过长，甚至造成不合理的内存占用。请灵活使用[Presets](https://docs.unity.cn/cn/current/Manual/Presets.html)（预设）功能来制定适合项目的设置基准。

更多详情请参阅[这份美术资源最佳使用指南](https://docs.unity3d.com/cn/current/Manual/ImportingAssets.html)，或在Unity Learn上学习[3D Art Optimization for Mobile Applications](https://learn.u3d.cn/tutorial/arm-unity-optimization-tips)教程。

### **找到正确的纹理导入方式**

一款应用的大部分内存占用一般是用在了纹理上，因此纹理的导入设置非常关键。通常来说，纹理的导入应遵守如下原则：

-   **降低最大分辨率：**在肉眼不易察觉、尽量不破坏原图的前提下使用较低的分辨率，从而达到优化内存占用的目的。
-   **采用二次幂（power of two，POT）压缩格式：**Unity要求移动端的纹理压缩格式（PVRCT或ETC）采用二次幂的纹理尺寸。
-   **制作纹理图集（texture atlas）：**将多张纹理合并到一张纹理图集中可以减少绘制调用次数，加快渲染速度。纹理图集可使用[Unity Sprite Atlas](https://docs.unity3d.com/cn/current/Manual/class-SpriteAtlas.html)或第三方的[TexturePacker](https://www.codeandweb.com/texturepacker)  进行制作。
-   **取消勾选Read/Write Enabled：**该选项在启用时会分别在CPU和GPU可寻址内存中创建一个副本，让纹理的内存占用翻倍。该选项在大多数情况下都可禁用。如果纹理是在运行时生成的，则可通过调用**Texture2D.Apply**，将**makeNoLongerReadable**设为**true**来强制禁用选项。
-   **禁用多余的Mip Map：**Mip Map贴图在2D精灵和UI图形这类大小始终一致的纹理上并无用处，但在随镜头距离变化而变化的3D模型上需要保留。

![Editor screenshot](https://blog-api.unity.com/sites/default/files/2021-08/image6.png?imwidth=3840&)

正确的纹理导入设置可以优化应用包的大小。

### **压缩纹理**

我们来对比一下两个模型和纹理相同的例子：左图所占用的内存几乎是右图的八倍，但在视觉上却没有什么不同。

![Screenshot](https://blog-api.unity.com/sites/default/files/2021-08/image18.jpg?imwidth=5000&)

未经压缩的纹理会占用更多的内存。


因此，请为iOS和Android应用采用自适应可伸缩纹理压缩（ATSC）格式。大多数游戏在开发时都以支持ATSC压缩、配置最低的设备作为目标设备。

当然也有例外，包括：

-   运行于A7及更老的芯片上的iOS游戏（即iPhone 5、5S等）——请使用PVRTC格式
-   运行于2016年以前的安卓设备的安卓游戏——请使用  ETC2  （爱立信的纹理压缩格式）

如果PVRTC和ETC等压缩格式的质量不够理想，或者目标设备不完全支持ASTC格式，我们可以试着将32位纹理替换成16位纹理。

请在完整版手册中详细了解[各平台推荐的纹理压缩格式](https://docs.unity3d.com/cn/current/Manual/class-TextureImporterOverride.html)。

### **调整网格导入设置**

类似于纹理，网格模型如果导入不当，也会占用过多的内存。要想尽量减少网格模型的内存占用，我们可以：

-   **压缩网格**，采用激进的压缩方法来减少磁盘空间占用（运行时的占用不会受影响）。注意，量化模型可能会导致模型失真，请根据实际情况来选择合适的压缩等级。
-   **禁用Read/Write**，该选项会分别在运行内存和GPU内存中创建模型的副本，它在大多数情况下都应禁用（Unity 2019.2及更早的版本会默认启用）。
-   **禁用动画骨架和BlendShapes**，若网格模型不带有骨骼动画或BlendShapes动画，则这两个选项并无多大用处。
-   **禁用法线和切线贴图**，若网格模型的材质并没有法线或切线贴图，则我们能禁用这两个选项来节省性能开支。


![Editor screenshot](https://blog-api.unity.com/sites/default/files/2021-08/image1.jpg?imwidth=1920&)

请仔细检查网格模型的导入设置。

### **检查模型多边形面数**

一个模型如果有更高的分辨率就意味着更多的内存占用和更长的GPU处理时间。通常来说，游戏的背景并不需要数十万的面，并且许多DCC软件导出的模型也可做一定的精简。举例来说，处于摄影机拍摄角之外的多边形可被删去，模型的细节可使用纹理和法线贴图呈现，不必使用过于复杂的网格来保留细节。

### **借助AssetPostprocessor自动设定导入设置**

[AssetPostprocessor](https://docs.unity.cn/ScriptReference/AssetPostprocessor.html)功能支持在导入资源时运行脚本，如此一来我们便能在模型、纹理、音频等资源的导入前/后应用自定义设置。

### **使用Addressable Asset System（可寻址资源系统）**

[Addressable Asset System](https://docs.unity3d.com/Packages/com.unity.addressables@latest)可以一种更为简单的方式来管理内容。系统采用统一的处理方式，通过调用AssetBundle的“地址”或别称，从本地路径或远程的内容分发网络（CDN）异步完成资源的加载。

![Editor screenshot](https://blog-api.unity.com/sites/default/files/2021-08/image11.jpg?imwidth=1920&)

如果把脚本以外的资源（模型、纹理、预制件、音频，甚至整个场景）划分成一个个[AssetBundle](https://docs.unity3d.com/cn/current/Manual/AssetBundlesIntro.html)，就可以将其作为可下载内容（DLC）分开分发。

然后，使用Addressables创建一个最小的应用程序，[Cloud Content Delivery](https://unity.cn/product/cloud-content-delivery)将在游戏进行中完成游戏内容的管理和分发。

![Screenshot](https://blog-api.unity.com/sites/default/files/2021-08/image16.jpg?imwidth=3840&)

使用“地址”来加载资源。


点击[此处](https://unity.cn/projects/addressable-and-cloud-content-delivery)来详细了解Addressable Asset System是怎样让资源管理更为轻松的。

> Written with [StackEdit](https://stackedit.io/).

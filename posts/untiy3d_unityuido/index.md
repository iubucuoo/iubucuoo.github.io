# 如何优化UnityUI

<!--more-->

### 分割Canvas画布

如果某个大型Canvas包含了上千个元素，单个UI元素的更新也会导致整个Canvas更新，造成不必要的CPU占用。

这时我们可利用UGUI的多画布功能，根据更新频率来划分UI元素。将相对静止的UI元素放在一个画布上，将需要时常更新的动态元素放在小型的子画布上，

然后统一画布内所有UI元素的Z轴坐标、材质和纹理。

### 隐藏不可见的UI元素

部分UI元素可能只会间断地出现（譬如角色受伤时出现的血条），就算这个元素变得不可见，它仍可能会发出绘制调用。因此我们需要具体地禁用所有不可见的UI组件，仅在需要时重新激活。

如果你只需隐藏画布，可以禁用Canvas组件，不必禁用整个GameObject，从而避免引擎再度绘制网格和顶点。

### 限制GraphicRaycasters数量、禁用Raycast Target

类似屏幕触摸或点击等输入事件需要使用**GraphicRaycaster**组件处理。组件会循环检测屏幕的每个输入点，检查其与UI的RectTransform是否有重叠。

但我们不必将**GraphicRaycaster**放在默认Canvas层级的顶层，而可以将**其**添加到需要互动的元素（按钮、滚动条等等）上。

![enter image description here](https://blog-api.unity.com/sites/default/files/2021-07/Optimizeyourmobilegameperformanceimage14.jpg?imwidth=1920&)


此外，不一定每个UI文本都要为**Raycast Target**（射线目标）。在不必要的位置禁用这个选项可让UI运算进一步简化，这在复杂的UI上意味着不小的性能收益。

![Image dialog box with a grey background and the text "Raycast Target" outlined in a red square.](https://blog-api.unity.com/sites/default/files/2021-07/Optimizeyourmobilegameperformanceimage2.png?imwidth=1920&)

如无必要，请禁用Raycast Target。


### 避免使用Layout Group（布局组）

由于Layout Group的更新效率较低，所以非动态内容完全可以不使用这项功能，转而借助锚点来实现比例化布局。或者，你也能用自定义代码在布局生成完毕后禁用[Layout Group](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/UIAutoLayout.html?utm_source=demand-gen&utm_medium=pdf&utm_campaign=asset-links-gmg-elevate-your-game&utm_content=optimize-mobile-game-performance-ebook)组件。

如果你确实需要在动态元素上使用（水平、垂直、网格式）Layout Group，请避免嵌套使用。

![enter image description here](https://blog-api.unity.com/sites/default/files/2021-07/Optimizeyourmobilegameperformanceimage5.jpg?imwidth=1920&)

Layout Group，尤其是在嵌套使用时，可能会显著降低性能。


### 避免使用大型的List与Grid视图

大型List和Grid视图会占用大量性能。如果你需要创建一个大型的List或Grid视图（比如一个包含数百个物品的物品栏），可以考虑建立一个UI库，重复使用库中的元素而非为每个物品创建一个元素。请在这个[GitHub项目](https://github.com/boonyifei/ScrollList)样例中查看实例。

### 避免多元素的重叠

将大量的UI元素堆在一起（比如卡牌游戏中的卡堆）会造成过度绘制。我们可使用自定义代码，在运行时将堆叠的多个元素合并成几个或几批较小的元素。

### 使用多种分辨率和长宽比

目前移动设备的分辨率和屏幕尺寸五花八门，而制作[不同分辨率的UI](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/HOWTO-UIMultiResolution.html)、让其完美适配设备也就很有必要了。

[Device Simulator](https://docs.unity3d.com/Manual/com.unity.device-simulator.html)（设备模拟器）支持在引擎中模拟UI在各个设备上的表现。你也可以使用[XCode](https://developer.apple.com/library/archive/documentation/IDEs/Conceptual/iOS_Simulator_Guide/GettingStartedwithiOSSimulator/GettingStartedwithiOSSimulator.html#//apple_ref/doc/uid/TP40012848-CH5-SW10)和[Android Studio](https://developer.android.com/studio/run/managing-avds)来创建虚拟设备。

![Image of the device simulator that has a grey dialog box background and a simulation of a racing boat game on a smartphone.](https://blog-api.unity.com/sites/default/files/2021-07/Optimizeyourmobilegameperformanceimage9.jpg?imwidth=3840&)

在Device Simulator中预览各种样式的屏幕。
### 在UI覆盖全屏时隐藏不可见内容

如果暂停或开始菜单覆盖了整个场景，我们可以禁用渲染场景的摄像机，同样地，隐藏在顶层画布背后的元素也可被禁用。

你甚至可以在UI显示期间使用**Application.targetFrameRate**来降低帧率，因为UI不需要以高帧率刷新。

### 在游戏空间与Camera Space Canvas上摆放摄像机

如果**Event**或**Render Camera**字段为空，则Unity会强行在其中加入**Camera.main**，造成不必要的性能浪费。

我们可以使用在画布的**RenderMode**中使用**Screen Space - Overlay**，来省略掉摄像机的使用。

![Image of the Camera space canvases dialog box with a greyed out background and white text.](https://blog-api.unity.com/sites/default/files/2021-07/Optimizeyourmobilegameperformanceimage11.jpg?imwidth=1920&)


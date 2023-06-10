# Unity项目配置建议

<!--more-->

有几个特定的项目设置会影响移动端游戏的性能。

#### **降低或禁用Accelerometer Frequency（加速度计频率）**

Unity每秒钟会以一定次数统计移动设备的加速度计状态。如果应用并不会用到加速度计，我们完全可以禁用该功能或降低统计频率来获得更好的性能。

![Editor](https://blog-api.unity.com/sites/default/files/2021-08/image3.jpg?imwidth=3840&)

请在游戏不使用加速度计时禁用Accelerometer Frequency。

#### **禁用不必要的Player或Quality设置**

如果目标平台并不支持**Auto Graphics API**，最好禁用**Player**设置中的相应选项，防止驱动程序生成多余的着色器变体。如果应用不支持老式CPU，请禁用**Target Architectures**  。

禁用**Quality**设置里多余的质量等级。

#### **禁用不必要的物理模拟**

如果游戏并不会用到物理模拟，请取消勾选**Auto Simulation**和**Auto Sync Transforms**。这两个选项在物理模拟之外并没有过多的用处，因此只会减缓应用的运行。

#### **选择正确的帧率**

移动端项目必须小心在帧率与电池寿命、过热保护间找到平衡。有时与其用60帧挑战设备的极限，用30帧平稳运行是一种更好的选择，Unity默认会将移动端应用的帧率设为30fps。

你也可以在运行时调用**Application.targetFrameRate**动态地调整帧率，比如，在相对较慢或静止的场景中将帧数降到30帧以下，在游戏进行中保留高帧数。

#### **消除庞杂的对象层级**

把对象层级拆开。如果某个GameObjects不需要嵌到层级里，我们可以简化对象的从属关系。更精简的对象层级可以很好地利用多线程处理来刷新对象的Transform，而复杂的层级结构会导致多余的Transform计算和更高的垃圾数据回收成本。

关于Transform的最佳设置方法，请在[Optimizing the Hierarchy](https://blogs.unity3d.com/2017/06/29/best-practices-from-the-spotlight-team-optimizing-the-hierarchy/?utm_source=demand-gen&utm_medium=pdf&utm_campaign=asset-links-gmg-elevate-your-game&utm_content=optimize-mobile-game-performance-ebook)和这段[Unite演讲](https://youtu.be/W45-fsnPhJY?t=794)中详细了解。

#### **尽量一次完成对象变换**

在移动Transform时，请调用[Transform.SetPositionAndRotation](https://docs.unity.cn/ScriptReference/Transform.SetPositionAndRotation.html)一次性完成对象的移动和旋转，这样做可避免两次修改同一变换，节省运算开销。

如果你需要在运行时[实例化](https://docs.unity.cn/ScriptReference/Object.Instantiate.html)一个GameObject，则可以在实例化之际嵌入并移动对象，来达到优化的目的：

**GameObject.Instantiate(prefab, parent);**

**GameObject.Instantiate(prefab, parent, position, rotation);**

关于Object.Instantiate的更多细节，请参见[Scripting API](https://docs.unity.cn/ScriptReference/Object.Instantiate.html)。

#### **Vsync 在移动平台上默认启用**

大部分移动端设备并不会将图像帧一分为二进行渲染。但即便我们在编辑器选项中禁用Vsync（**Project Settings > Quality**），Vsync功能仍会在硬件层面上启用。因为如果GPU频率不够高，帧将被滞留直到渲染完成，导致帧数降低。


> Written with [StackEdit](https://stackedit.io/).

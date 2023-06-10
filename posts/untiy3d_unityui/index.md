# Unity UI优化原理

<!--more-->

 
尝试优化 Unity UI 系统之前的主要任务是找出观察到的性能问题的确切原因。
Unity UI 用户遇到的常见问题有四类：

 - Gpu片段着色器使用过多（填充率过度使用）
 - 重建Canvas批处理所花费的CPU时间过多
 - Canvas批次的重建数量过多
 - 用于生成顶点的CPU时间过多

**界面源代码**

永远记住，Unity UI 的图形和布局组件是完全开源的。他们的源代码可以在[Unity 的 Bitbucket 存储库的](https://bitbucket.org/Unity-Technologies/)的[UI](https://bitbucket.org/Unity-Technologies/ui/)下找到。
## Unity UI 基础知识
### 渲染细节
在unityui中编写用户界面时，所有由canvas绘制的几何图形都将在透明队列中绘制，也就是说，unityui生成的几何图形始终使用alpha混合从后到前绘制，从性能角度来看，记住重要一点，从多边形栅格化的每个像素都将被采样，即使他完全被不透明的多边形覆盖，在移动设备上，这种高水平的过度绘制会迅速超过GPU的填充率容量。
### 批量构建过程（画布）
批处理构建过程是Canvas组合表示其UI元素的网格并生成适当的渲染命令以发送到unity的图形管道的过程，此过程的结果将被缓存并重用，直到canvas被标记为脏，只要其组成网格之一发生更改，就会发生这种情况。
画布使用的网格取自附加到画布但不包含在任何子画布中的一组画布渲染器组件。
计算批次需要按深度对网格进行排序，并检查他们是否存在重叠、共享材料等。此操作是多线程的，因此其性能在不同的CPU架构之间通常会有很大的差异，尤其是在移动SOC（通常只有很少的cpu内核）和现代桌面cpu（通常4个或更多内核）之间。
### 重建过程（图形）
重建过程是重新计算unityui的c#图形组件的布局和网格的地方。这是在CanvasUpdateRegistry类中执行的。请记住，这是一个c#类，源码可在[Bitbucket](https://bitbucket.org/Unity-Technologies/ui/)上找到
PerformUpdate 运行一个三步过程：
 - [通过ICanvasElement.Rebuild](http://docs.unity3d.com/ScriptReference/UI.ICanvasElement.Rebuild.html)方法请求脏布局组件重建其布局。
 -   任何已注册的 Clipping 组件（例如 Masks）都被要求剔除任何被剪裁的组件。这是通过 ClippingRegistry.Cull 完成的。
 -   脏图形组件被要求重建它们的图形元素。
对于布局和图形重建，该过程分为多个部分。布局重建分三部分（PreLayout、Layout和PostLayout）运行，而图形重建分两个部分（PreRender和LatePreRender）运行。
#### 布局重建
要重新计算包含在一个或多个布局组件中的组件的适当位置（以及可能的大小），有必要以适当的分层顺序应用布局。在 GameObject 层次结构中更接近根的布局可能会改变可能嵌套在其中的任何布局的位置和大小，因此必须首先计算。

为此，Unity UI 按其在层次结构中的深度对脏布局组件列表进行排序。层次结构中较高的项目（即具有较少父变换的项目）被移动到列表的前面。

然后请求布局组件的排序列表来重建它们的布局；这是由 Layout 组件控制的 UI 元素的位置和大小实际改变的地方。有关单个元素的位置如何受布局影响的更多详细信息，请参阅 Unity 手册的[UI 自动布局](http://docs.unity3d.com/Manual/UIAutoLayout.html)部分
####  图形重建

重建 Graphic 组件时，Unity UI 将控制权传递给[ICanvasElement](http://docs.unity3d.com/ScriptReference/UI.ICanvasElement.html?_ga=2.127287373.1786039110.1646701765-101944752.1644809757)接口的[Rebuild](http://docs.unity3d.com/ScriptReference/UI.ICanvasElement.Rebuild.html)方法。Graphic 实现了这一点，并在重建过程的 PreRender 阶段运行两个不同的重建步骤。

-   如果顶点数据已被标记为脏（例如，当组件的 RectTransform 已更改大小时），则重新构建网格。
    

-   如果材质数据已被标记为脏（例如，当组件的材质或纹理已更改时），则附加的 Canvas Renderer 的材质将被更新。
    

图形重建不以任何特定顺序通过图形组件列表进行，并且不需要任何排序操作。

##  填充率、画布和输入
### 修复填充率问题

可以采取两种措施来减少 GPU 片段管道上的压力：

-   降低片段着色器的复杂性。有关详细信息，请参阅“UI 着色器和低规格设备”部分。
    

-   减少必须采样的像素数。
    

由于 UI 着色器通常是标准化的，因此最常见的问题就是过度使用填充率。这通常是由于大量重叠的 UI 元素和/或具有占据屏幕重要部分的多个 UI 元素。这两个问题都可能导致极高水平的透支。

为了减轻填充率过度使用并减少透支，请考虑以下可能的补救措施。

#### 消除不可见的 UI

需要对现有 UI 元素进行最少重新设计的方法是简单地禁用对玩家不可见的元素。这适用的最常见情况是打开具有不透明背景的全屏 UI。在这种情况下，可以禁用放置在全屏 UI 下方的任何 UI 元素。

最简单的方法是禁用包含不可见 UI 元素的根 GameObject 或 GameObjects。有关替代解决方案，请参阅[禁用画布](https://unity3d.com/learn/tutorials/topics/best-practices/other-ui-optimization-techniques-and-tips#disabling-canvases)部分。

最后，通过将 alpha 设置为 0 来确保没有隐藏任何 UI 元素，因为该元素仍将被发送到 GPU 并且可能需要宝贵的渲染时间。如果 UI 元素不需要 Graphic 组件，您只需将其移除，光线投射仍然可以工作。

#### 简化 UI 结构

为了减少重建和渲染 UI 所需的时间，尽可能减少 UI 对象的数量很重要。尽可能多地烤东西。例如，不要使用混合游戏对象来改变元素的色调，而是通过材质属性来做到这一点。另外，不要创建像文件夹一样的游戏对象，并且除了组织场景之外没有其他目的。

#### 禁用隐形相机输出

如果打开具有不透明背景的全屏 UI，世界空间相机仍将渲染 UI 后面的标准 3D 场景。渲染器不知道全屏 Unity UI 会遮挡整个 3D 场景。

因此，如果打开一个完全全屏的 UI，禁用任何和所有被遮挡的世界空间相机将有助于通过简单地消除渲染 3D 世界的无用工作来减轻 GPU 压力。

如果 UI 没有覆盖整个 3D 场景，您可能希望将场景渲染为纹理一次并使用它，而不是连续渲染它。您将无法在 3D 场景中看到动画内容，但这在大多数情况下应该是可以接受的。

注意：如果 Canvas 设置为“Screen Space – Overlay” ，则无论场景中活动的摄像机数量如何，都会绘制它。

#### 多数被遮挡的相机

许多“全屏”用户界面实际上并没有掩盖整个 3D 世界，而是让世界的一小部分可见。在这些情况下，仅捕获渲染纹理中可见的世界部分可能更理想。如果世界的可见部分被“缓存”在渲染纹理中，则可以禁用实际的世界空间相机，并且可以在 UI 屏幕后面绘制缓存的渲染纹理以提供 3D 世界的冒名顶替版本。

#### 基于合成的 UI

设计师通过组合来创建 UI 是很常见的——组合和分层标准背景和元素以创建最终的 UI。虽然这样做相对简单，并且对迭代非常友好，但由于 Unity UI 使用透明渲染队列，它的性能不佳。

考虑一个带有背景、按钮和按钮上的一些文本的简单 UI。因为透明队列中的对象是从后到前排序的，如果一个像素落在文本字形中，GPU 必须采样背景纹理，然后是按钮纹理，最后是文本图集的纹理，总共三个样品。随着 UI 复杂性的增加，以及更多的装饰元素被叠加到背景上，样本的数量会迅速增加。

如果发现大型 UI 受填充率限制，最好的办法是创建专门的 UI 精灵，将 UI 的尽可能多的装饰/不变元素合并到其背景纹理中。这减少了为实现所需设计而必须相互叠加的元素数量，但劳动密集型并增加了项目纹理图集的大小。

这种将创建给定 UI 所需的分层元素的数量压缩到专门的 UI sprite 上的原则也可以用于子元素。考虑一个带有产品滚动窗格的商店 UI。每个产品 UI 元素都有一个边框、一个背景和一些图标来表示价格、名称和其他信息。

商店 UI 将需要背景，但由于其产品在背景中滚动，产品元素无法合并到商店 UI 的背景纹理中。但是，产品的UI元素的边框、价格、名称等元素可以合并到产品的背景上。根据图标的大小和数量，填充率的节省可能相当可观。

组合分层元素有几个缺点。专业元素不能再重复使用，需要额外的艺术家资源来创建。添加大型新纹理可能会显着增加保存 UI 纹理所需的内存量，尤其是在 UI 纹理不是按需加载和卸载的情况下。

#### UI 着色器和低规格设备

Unity UI 使用的内置着色器包含对遮罩、剪辑和许多其他复杂操作的支持。由于这种增加的复杂性，与 iPhone 4 等低端设备上更简单的 Unity 2D 着色器相比，UI 着色器的性能较差。

如果针对低端设备的应用程序不需要遮罩、裁剪和其他“花哨”功能，则可以创建一个省略未使用操作的自定义着色器，例如这个最小的 UI 着色器：

```
 Shader "UI/Fast-Default" { Properties { [PerRendererData] _MainTex ("Sprite Texture", 2D) = "white" {} _Color ("Tint", Color) = (1,1,1,1) }

    SubShader
    {
        Tags
        { "Queue"="Transparent"  "IgnoreProjector"="True"  "RenderType"="Transparent"  "PreviewType"="Plane"  "CanUseSpriteAtlas"="True" }
    
        Cull Off
        Lighting Off
        ZWrite Off
        ZTest [unity_GUIZTestMode]
        Blend SrcAlpha OneMinusSrcAlpha
    
        Pass
        {
        CGPROGRAM #pragma vertex vert  #pragma fragment frag  #include "UnityCG.cginc"  #include "UnityUI.cginc"  struct appdata_t
            {
                float4 vertex   : POSITION;
                float4 color    : COLOR;
                float2 texcoord : TEXCOORD0;
            }; struct v2f
            {
                float4 vertex   : SV_POSITION;
                fixed4 color    : COLOR;
                half2 texcoord  : TEXCOORD0;
                float4 worldPosition : TEXCOORD1;
            };
    
            fixed4 _Color;
            fixed4 _TextureSampleAdd; v2f vert(appdata_t IN) {
                v2f OUT;
                OUT.worldPosition = IN.vertex;
                OUT.vertex = mul(UNITY_MATRIX_MVP, OUT.worldPosition);
    
                OUT.texcoord = IN.texcoord; #ifdef UNITY_HALF_TEXEL_OFFSET OUT.vertex.xy += (_ScreenParams.zw-1.0)*float2(-1,1); #endif OUT.color = IN.color * _Color; return OUT;
            }
    
            sampler2D _MainTex; fixed4 frag(v2f IN) : SV_Target { return (tex2D(_MainTex, IN.texcoord) + _TextureSampleAdd) * IN.color;
            }
        ENDCG
        }
    }
```
### UI 画布重建

要显示任何 UI，UI 系统必须为屏幕上表示的每个 UI 组件构建几何图形。这包括运行动态布局代码、生成多边形来表示 UI 文本字符串中的字符，以及将尽可能多的几何图形合并到单个网格中以最小化绘制调用。这是一个多步骤的过程，在本指南开头的[基础部分中有详细描述。](https://unity3d.com/learn/tutorials/topics/best-practices/fundamentals-unity-ui)

画布重建可能会成为性能问题，主要原因有两个：

-   如果 Canvas 上的可绘制 UI 元素的数量很大，那么计算批次本身就会变得非常昂贵。这是因为排序和分析元素的成本与 Canvas 上可绘制 UI 元素的数量成线性增长。
    

-   如果 Canvas 经常被弄脏，那么可能会花费过多的时间来刷新一个变化相对较少的 Canvas。
    

随着 Canvas 上元素数量的增加，这两个问题都会变得更加严重。

重要提醒：每当给定 Canvas 上的任何可绘制 UI 元素发生变化时，Canvas 必须重新运行批处理构建过程。此过程重新分析 Canvas 上的每个可绘制 UI 元素，无论它是否已更改。请注意，“更改”是影响 UI 对象外观的任何更改，包括分配给精灵渲染器的精灵、变换位置和比例、文本网格中包含的文本等。

### 子订单

Unity UI 是从后到前构建的，对象在层次结构中的顺序决定了它们的排序顺序。层次结构中较早的对象被视为层次结构中较晚的对象。通过从上到下遍历层次结构并收集所有使用相同材质、相同纹理且没有中间层的对象来构建批次。“中间层”是具有不同材质的图形对象，其边界框与两个其他可批处理对象重叠，并放置在两个可批处理对象之间的层次结构中。中间层迫使批次被破坏。

正如 Unity UI 分析工具步骤中所讨论的，UI 分析器和帧调试器可用于检查 UI 的中间层。这是一种情况，其中一个可绘制对象将自己插入到另外两个可批处理的可绘制对象之间。

这个问题最常发生在文本和精灵彼此靠近时：文本的边界框可以不可见地与附近的精灵重叠，因为文本字形的大部分多边形是透明的。这可以通过两种方式解决：

-   重新排序可绘制对象，使可批处理对象不会被不可批处理对象插入；即，将不可批处理对象移动到可批处理对象的上方或下方。
    

-   调整对象的位置以消除不可见的重叠空间。
    

这两个操作都可以在 Unity 编辑器中执行，同时打开并启用 Unity 框架调试器。通过简单地观察 Unity Frame Debugger 中可见的绘制调用数量，可以找到一个顺序和位置，以最大限度地减少由于重叠 UI 元素而浪费的绘制调用数量。

### 分割画布

除了最琐碎的情况外，通过将元素移动到子画布或兄弟画布来拆分画布通常是一个好主意。

兄弟画布最适合用于 UI 的某些部分必须与 UI 的其余部分分开控制其绘制深度的情况，以始终位于其他层之上或之下（例如教程箭头）。

在大多数其他情况下，子画布更方便，因为它们从父画布继承显示设置。

乍一看，将 UI 细分为多个子画布似乎是一种最佳实践，但请记住，画布系统也不会跨单独的画布组合批次。高性能 UI 设计需要在最小化重建成本和最小化绘制调用浪费之间取得平衡。

#### 一般准则

因为 Canvas 会在其任何组成的可绘制组件发生更改时重新批处理，所以通常最好将任何非平凡的 Canvas 拆分为至少两个部分。此外，如果预计元素会同时发生变化，最好尝试将元素放在同一个 Canvas 上。一个例子可能是进度条和倒数计时器。它们都依赖于相同的基础数据，因此需要同时更新，因此它们应该放在同一个 Canvas 上。

在一个 Canvas 上，放置所有静态不变的元素，例如背景和标签。这些将在 Canvas 首次显示时批处理一次，然后不再需要重新批处理。

在第二个画布上，放置所有“动态”元素——那些经常变化的元素。这将确保此 Canvas 主要重新批处理脏元素。如果动态元素的数量变得非常大，则可能需要将动态元素进一步细分为一组不断变化的元素（例如进度条、计时器读数、任何动画）和一组仅偶尔变化的元素。

这在实践中实际上是相当困难的，尤其是在将 UI 控件封装到预制件中时。许多 UI 选择通过将更昂贵的控件拆分到子画布上来细分画布。

#### Unity 5.2 和优化的批处理

在 Unity 5.2 中，批处理代码被大幅重写，与 Unity 4.6、5.0 和 5.1 相比，其性能要高得多。此外，在多于 1 个内核的设备上，Unity UI 系统会将大部分处理移至工作线程。一般来说，Unity 5.2 减少了将 UI 积极拆分为数十个子画布的需要。现在，移动设备上的许多 UI 只需两个或三个 Canvas 即可实现高性能。

有关 Unity 5.2 优化的更多信息，请参阅[此博客文章](http://blogs.unity3d.com/2015/09/07/making-the-ui-backend-faster/)。

### Unity UI 中的输入和光线投射

默认情况下，Unity UI 使用[Graphic Raycaster](http://docs.unity3d.com/ScriptReference/UI.GraphicRaycaster.html)组件来处理输入事件，例如触摸事件和指针悬停事件。这通常由独立输入管理器组件处理。尽管有这个名字，Standalone Input Manager 是一个“通用”输入管理器系统，可以处理指针和触摸。

#### 移动设备上的错误鼠标输入检测 (5.3)

在 Unity 5.4 之前，只要当前没有可用的触摸输入，每个附加了 Graphic Raycaster 的活动 Canvas 将每帧运行一次 raycast 以检查指针的位置。无论平台如何，都会发生这种情况；没有鼠标的 iOS 和 Android 设备仍会查询鼠标的位置，并尝试发现哪些 UI 元素位于该位置下方，以确定是否需要发送任何悬停事件。

这是对 CPU 时间的浪费，并且已经看到消耗了 Unity 应用程序 CPU 帧时间的 5% 或更多。

这个问题在 Unity 5.4 中得到解决。从 5.4 开始，没有鼠标的设备将不会查询鼠标位置，也不会执行不必​​要的光线投射。

如果使用低于 5.4 的 Unity 版本，强烈建议移动开发人员创建自己的 Input Manager 类。这可以像从 Unity UI 源复制 Unity 的标准输入管理器并注释掉 ProcessMouseEvent 方法以及对该方法的所有调用一样简单。

#### 光线投射优化

Graphic Raycaster 是一个相对简单的实现，它遍历所有将“Raycast Target”设置为 true 的 Graphic 组件。对于每个 Raycast Target，Raycaster 执行一组测试。如果一个 Raycast 目标通过了所有的测试，那么它就会被添加到命中列表中。

##### Raycast 实现细节

测试是：

-   如果 Raycast Target 处于活动状态、启用并已绘制（即具有几何图形）
    

-   如果输入点位于 Raycast Target 所附加到的 RectTransform 内
    

-   [如果 Raycast Target 具有或是任何ICanvasRaycastFilter](http://docs.unity3d.com/ScriptReference/ICanvasRaycastFilter.html)组件的子级（在任何深度），并且该 Raycast Filter 组件允许 Raycast。
    

命中的 Raycast Targets 列表然后按深度排序，过滤反向目标，并过滤以确保移除在相机后面渲染的元素（即在屏幕中不可见）。

如果在 Graphic Raycaster 的“Blocking Objects”属性上设置了相应的标志，Graphic Raycaster 也可以将光线投射到 3D 或 2D 物理系统中。（从脚本中，该属性被命名为[blockingObjects](http://docs.unity3d.com/ScriptReference/UI.GraphicRaycaster-blockingObjects.html)。）

如果启用了 2D 或 3D 阻挡对象，则在阻挡光线投射的物理层上的 2D 或 3D 对象下方绘制的任何光线投射目标也将从命中列表中删除。

然后返回最终的命中列表。

##### 光线投射优化技巧

鉴于所有 Raycast Target 都必须由 Graphic Raycaster 测试，最佳实践是仅在必须接收指针事件的 UI 组件上启用“Raycast Target”设置。Raycast Targets 列表越小，必须遍历的层次越浅，每次 Raycast 测试的速度就越快。

对于具有多个必须响应指针事件的可绘制 UI 对象的复合 UI 控件，例如希望其背景和文本都更改颜色的按钮，通常最好将单个 Raycast Target 放置在复合 UI 的根部控制。当单个 Raycast Target 接收到一个指针事件时，它可以将事件转发到复合控件中的每个感兴趣的组件。

##### 层次深度和光线投射过滤器

在搜索光线投射过滤器时，每个图形光线投射都会遍历变换层次结构一直到根。此操作的成本与层次结构的深度成比例线性增长。必须测试附加到层次结构中每个 Transform 的所有组件，以查看它们是否实现[ICanvasRaycastFilter](http://docs.unity3d.com/ScriptReference/ICanvasRaycastFilter.html)，因此这不是一个便宜的操作。

有几个标准的 Unity UI 组件使用[ICanvasRaycastFilter](http://docs.unity3d.com/ScriptReference/ICanvasRaycastFilter.html)，例如[CanvasGroup](http://docs.unity3d.com/ScriptReference/CanvasGroup.html)、[Image](http://docs.unity3d.com/ScriptReference/UI.Image.html)、[Mask](http://docs.unity3d.com/ScriptReference/UI.Mask.html)和[RectMask2D](http://docs.unity3d.com/ScriptReference/UI.RectMask2D.html)，因此不能轻易消除这种遍历。

##### 子画布和 OverrideSorting 属性

Sub-canvas 上的[overrideSorting](http://docs.unity3d.com/ScriptReference/Canvas-overrideSorting.html)属性将导致 Graphic Raycast 测试停止爬升变换层次结构。如果可以启用它而不会导致排序或光线投射检测问题，那么应该使用它来降低光线投射层次遍历的成本。
## 5.优化UI控件
优化 Unity UI 指南的这一部分侧重于特定类型的 UI 控件的问题。虽然大多数 UI 控件在性能方面都比较相似，但有两个是导致游戏接近可交付状态时遇到的许多性能问题的原因。

### 用户界面文本

Unity 的内置文本组件是一种在 UI 中显示光栅化文本字形的便捷方式。然而，有许多行为并不为人所知，但经常作为性能热点出现。在向 UI 添加文本时，请始终记住，文本字形实际上是作为单独的四边形呈现的，每个字符一个。这些四边形往往在字形周围有大量空白空间，具体取决于其形状，并且很容易以无意中破坏其他 UI 元素批处理的方式放置文本。

#### 文本网格重建

一个主要问题是 UI 文本网格的重建。无论何时更改 UI 文本组件，文本组件都必须重新计算用于显示实际文本的多边形。如果文本组件或其任何父游戏对象被简单地禁用和重新启用而不更改文本，也会发生这种重新计算。

对于显示大量文本标签的任何 UI，这种行为都是有问题的，最常见的是排行榜或统计屏幕。由于隐藏和显示 Unity UI 的最常见方法是启用/禁用包含该 UI 的 GameObject，因此具有大量文本组件的 UI 通常会在显示时导致不希望的帧速率中断。

有关此问题的潜在解决方法，请参阅下一步中的禁用画布部分。

#### 动态字体和字体图集

当完整的可显示字符集非常大或在运行前未知时，动态字体是一种显示文本的便捷方式。在 Unity 的实现中，这些字体在运行时根据 UI 文本组件中遇到的字符构建一个字形图集。

加载的每个不同的 Font 对象都将保持自己的纹理图集，即使它与另一种字体在同一个字体系列中。例如，在一个控件上使用带有粗体文本的 Arial 而在另一个控件上使用 Arial Bold 将产生相同的输出，但 Unity 将保持两个不同的纹理图集 - 一个用于 Arial，一个用于 Arial Bold。

从性能的角度来看，最重要的是要理解 Unity UI 的动态字体在字体纹理图集中为每个不同的大小、样式和字符组合保留一个字形。也就是说，如果 UI 包含两个文本组件，都显示字母“A”，则：

-   如果两个 Text 组件共享相同的大小，则字体图集将包含一个字形。
    

-   如果两个 Text 组件的大小不同（例如，一个是 16 磅，另一个是 24 磅），那么字体图集将包含两个不同大小的字母“A”副本。
    

-   如果一个文本组件是粗体而另一个不是，则字体图集将包含一个粗体“A”和一个常规“A”。
    

每当具有动态字体的 UI 文本对象遇到尚未光栅化到字体纹理图集的字形时，必须重新构建字体的纹理图集。如果新字形适合当前图集，则会添加它并将图集重新上传到图形设备。但是，如果当前图集太小，则系统会尝试重建图集。它分两个阶段进行。

首先，以相同大小重建图集，仅使用活动 UI 文本组件当前显示的字形。这包括其父 Canvas 已启用但已禁用 Canvas Renderers 的 UI 文本组件。如果系统成功地将所有当前使用的字形拟合到一个新的图集中，它将对该图集进行光栅化，并且不会继续进行第二步。

其次，如果当前使用的字形集不能放入与当前图集相同大小的图集中，则通过将图集的较短尺寸加倍来创建更大的图集。例如，一个 512x512 的图集扩展为 512x1024 的图集。

由于上述算法，动态字体的图集只有在创建后才会增大。鉴于重建纹理图集的成本，必须在重建期间尽量减少。这可以通过两种方式完成。

尽可能使用非动态字体并为所需的字形集预配置支持。这通常适用于使用严格限制的字符集（例如仅拉丁/ASCII 字符）且大小范围较小的 UI。

如果必须支持非常大范围的字符，例如整个 Unicode 集，则必须将字体设置为 Dynamic。为避免可预测的性能问题，请在启动时通过[Font.RequestCharactersInTexture](http://docs.unity3d.com/ScriptReference/Font.RequestCharactersInTexture.html)使用一组适当的字符来填充字体的字形图集。

请注意，字体图集的重建是针对每个更改的 UI 文本组件单独触发的。当填充大量文本组件时，收集文本组件内容中的所有唯一字符并准备字体图集可能是有利的。这将确保字形图集只需要重建一次，而不是每次遇到新字形时都重建一次。

另请注意，当触发字体图集重建时，当前未包含在活动 UI 文本组件中的任何字符都不会出现在新图集中，即使它们最初是由于调用[Font.RequestCharactersInTexture](http://docs.unity3d.com/ScriptReference/Font.RequestCharactersInTexture.html)。要解决此限制，请订阅Font.textureRebuilt委托并查询[Font.characterInfo](http://docs.unity3d.com/ScriptReference/Font-characterInfo.html)以确保所有需要的字符都保持原样。

Font.textureRebuilt委托当前未记录。它是一个[单参数 Unity Event](http://docs.unity3d.com/ScriptReference/Events.UnityEvent_1.html)。参数是重建纹理的字体。此活动的订阅者应遵循以下签名：

    public  void  TextureRebuiltCallback(Font rebuiltFont)  { /* ... */ }

#### 专门的字形渲染器

对于字形众所周知的情况，每个字形之间的位置相对固定，编写一个自定义组件来显示显示这些字形的精灵会更加有利。这方面的一个例子可能是分数显示。

对于乐谱，可显示的字符取自一个众所周知的字形集（数字 0-9），不会跨地区变化，并且彼此之间的距离固定。将整数分解为其数字并显示适当的数字精灵相对简单。这种专门的数字显示系统可以以一种无需分配且计算、动画和显示速度比 Canvas 驱动的 UI 文本组件快得多的方式构建。

#### 后备字体和内存使用情况

对于必须支持大字符集的应用程序，很容易在字体导入器的“字体名称”字段中列出大量字体。如果字形不能位于主要字体中，则“字体名称”字段中列出的任何字体都将用作备用字体。回退顺序由字体在“字体名称”字段中列出的顺序决定。

但是，为了支持这种行为，Unity 会将“字体名称”字段中列出的所有字体加载到内存中。如果字体的字符集非常大，则备用字体消耗的内存量可能会变得过多。这在包含象形字体时最常见，例如日文汉字或汉字。

### 最佳配合和性能

**一般来说，UI Text 组件的Best Fit设置永远不应该被使用。**

“最佳匹配”将字体大小动态调整为最大整数磅值，可以在文本组件的边界框中显示而不会溢出，并限制在可配置的最小/最大磅值。然而，由于 Unity 为每个不同大小的字符在字体图集中呈现不同的字形，因此使用最佳拟合将迅速淹没具有许多不同字形大小的图集。

从 Unity 2017.3 开始，Best Fit 使用的尺寸检测不是最优的。它为每个测试的大小增量在字体图集中生成字形，这进一步增加了生成字体图集所需的时间。它还容易导致图集溢出，从而导致旧字形被踢出图集。由于最佳拟合计算需要大量测试，这通常会驱逐其他文本组件使用的字形，并在计算出适当的字体大小后强制至少再次重建字体图集。这个特定问题已在 Unity 5.4 中得到纠正，Best Fit 不会不必要地扩展字体的纹理图集，但仍比静态大小的文本慢得多。

频繁的字体图集重建会迅速降低运行时性能并导致内存碎片。设置为 Best Fit 的文本组件的数量越多，这个问题就越严重。

### TextMeshPro 文本

TextMesh Pro (TMP) 是 Unity 现有文本组件（如 Text Mesh 和 UI Text）的替代品。TextMesh Pro 使用有符号距离场 (SDF) 作为其主要的文本渲染管道，从而可以在任何点大小和分辨率下清晰地渲染文本。使用一组旨在利用 SDF 文本渲染功能的自定义着色器，TextMesh Pro 可以通过简单地更改材质属性以添加视觉样式（如膨胀、轮廓、软阴影、斜角）来动态更改文本的视觉外观。纹理、发光等，并通过创建/使用材质预设来保存和调用这些视觉样式。

在 2018.1 发布之前，TextMesh Pro 作为 Asset Store 包包含在一个项目中。自 2018.1 起，TextMesh Pro 将作为[包管理器](https://docs.unity3d.com/Packages/com.unity.package-manager-ui@latest/index.html)包提供。

#### 文本网格重建

很像 Unity 的内置 UIText 组件，对组件显示的文本进行更改将触发对 Canvas.SendWillRendererCanvases 和 Canvas.BuildBatch 的调用，这可能会很昂贵。尽量减少对 TextMeshProUGUI 组件的文本字段的更改，并确保将文本经常更改的父级 TextMeshProUGUI 组件更改为具有自己的 Canvas 组件的父级 GameObject，以确保 Canvas 重建调用保持尽可能高效。

请注意，对于在世界空间中显示的文本，我们建议用户使用普通的 TextMeshPro 组件而不是使用 TextMeshProUGUI，因为在 Worldspace 中使用 Canvases 可能会降低效率。直接使用 TextMeshPro 会更有效，因为它不会产生画布系统开销。

#### 字体和内存使用

鉴于 TMP 中没有动态字体功能，因此必须依赖备用字体。了解如何加载和使用备用字体对于在使用 TMP 时优化内存至关重要。

TMP 中的字形发现是递归完成的——也就是说，当 TMP 字体资源中缺少一个字形时，TMP 会从列表中的第一个回退开始以及它们自己的回退开始迭代当前分配或活动的回退字体资源列表。如果仍未找到字形，TMP 将搜索可能分配给文本对象的任何 Sprite 资源以及分配给此 Sprite 资源的任何回退。如果仍然没有找到所需的字形，TMP 将递归搜索在 TMP 设置文件中分配的一般后备列表，然后是默认 Sprite 资源。如果仍然无法找到此字形，它将搜索在 TMP 设置中分配的默认字体资源。作为最后的手段，TMP 将使用并显示在 TMP 设置文件中定义的缺失字形替换字符。

TextMesh Pro 的字体资源在场景或项目中被引用时被加载。它们主要由 TextMeshPro 文本组件、TMP 设置以及字体资源本身作为后备字体引用。如果在 TMP 设置资源中引用了字体资源，则当第一个带有 TMP 文本组件的场景被激活时，这些字体资源及其所有备用字体资源将被递归加载。如果引用了默认的 sprite sheet 资源，那么它也将被加载。

此外，当给定场景中的 TextMeshPro 组件引用字体资源并且尚未通过 TMP 设置加载时，一旦激活组件，将递归加载所引用的字体资源及其所有后备字体资源。在处理具有多种字体的项目时，请务必牢记此过程，尤其是在可用内存存在问题的情况下。

由于上述原因，在使用 TMP 时本地化项目成为一个问题，因为预先通过 TMP 设置加载所有本地化语言字体资源将对内存压力造成不利影响。如果本地化是必要的要求，我们建议仅在必要时（加载各种场景时）分配这些字体资源或备用或使用资源包以模块化方式加载字体资源的潜在策略。

当应用程序启动时，应该包含一个引导步骤来验证用户的语言环境并为每个字体资产设置字体资产后备：

1.  为基本 TMP 字体资源创建资源包（例如，每种字体的最小拉丁字形）
    

2.  为每种语言所需的备用 TMP 字体资产创建一个资产包（例如，为日语所需的每种字体创建一个 TMP 字体资产的资产包）
    

3.  在引导步骤中加载您的基础资产包
    

4.  根据语言环境，使用备用字体加载所需的 Asset Bundle
    

5.  对于基础资源包中的每种字体，从本地化字体资源包中分配备用字体资源
    

6.  继续引导您的游戏
    

如果不使用图像，也可以从 TMP 设置中删除默认 Sprite 资源引用，以节省额外的适度内存。

#### 最佳配合和性能

再一次，鉴于 TextMesh Pro 没有动态字体功能，不会出现上述 UGUI UIText 部分中关于最佳拟合的问题。在 TextMesh Pro 组件上使用 Best Fit 时唯一需要考虑的是使用二分搜索来找到正确的大小。使用文本自动调整大小时，最好测试最长/最大文本块的最佳点大小。一旦确定了此最佳大小，请禁用给定文本对象的自动调整大小，然后在其他文本对象上手动设置此最佳点大小。这有利于提高性能并避免一组文本对象使用不同的点大小，这被认为是不良的视觉/印刷实践。

### 滚动视图

在填充率问题之后，Unity UI 的滚动视图是运行时性能问题的第二大常见来源。滚动视图通常需要大量的 UI 元素来表示它们的内容。填充滚动视图有两种基本方法：

-   用代表所有滚动视图内容所需的所有元素填充它
    

-   汇集元素，根据需要重新定位它们以表示可见内容。
    

这两种解决方案都有问题。

随着要表示的项目数量的增加，第一个解决方案需要越来越多的时间来实例化所有 UI 元素，并且还增加了重建 Scroll View 所需的时间。如果在 Scroll View 中只需要少量元素，例如在 Scroll View 中只需要显示少量 Text 组件，那么这种方法因其简单性而受到青睐。

第二种解决方案需要大量代码才能在当前 UI 和布局系统下正确实现。下面将更详细地讨论两种可能的方法。对于任何非常复杂的滚动 UI，通常需要某种池化方法来避免性能问题。

**尽管存在这些问题，但所有方法都可以通过将 RectMask2D 组件添加到 Scroll View 来改进**。此组件确保在 Scroll View 视口之外的 Scroll View 元素不包含在可绘制元素列表中，这些元素必须在重建 Canvas 时生成、排序和分析其几何图形。

#### 简单的滚动视图元素池

使用 Scroll View 实现对象池的最简单方法，同时保留使用 Unity 内置 Scroll View 组件的原生便利性，采用混合方法：

要在 UI 中布局元素，这将允许布局系统正确计算 Scroll View 内容的大小并允许滚动条正常工作，请使用带有[Layout Element](http://docs.unity3d.com/Manual/script-LayoutElement.html)组件的 GameObjects 作为可见 UI 元素的“占位符”。

然后，实例化一个足以填充 Scroll View 可见区域的可见部分的可见 UI 元素池，并将这些元素作为定位占位符的父级。当 Scroll View 滚动时，重用 UI 元素来显示滚动到视图中的内容。

这将大大减少必须批处理的 UI 元素的数量，因为批处理的成本只会根据 Canvas 内的 Canvas Renderer 的数量而不是 Rect Transforms 的数量而增加。

##### 简单方法的问题

目前，每当任何 UI 元素重新设置父级或更改其同级顺序时，该元素及其所有子元素都会被标记为“脏”并强制重建其 Canvas。

这样做的原因是 Unity 没有分离用于重新调整变换和更改其兄弟顺序的回调。这两个事件都会触发[OnTransformParentChanged](http://docs.unity3d.com/ScriptReference/MonoBehaviour.OnTransformParentChanged.html)回调。在 Unity UI 的[Graphic](http://docs.unity3d.com/ScriptReference/UI.Graphic.html)类的源代码中（参见源代码中的 Graphic.cs），该回调被实现并调用方法[SetAllDirty](http://docs.unity3d.com/ScriptReference/UI.Graphic.SetAllDirty.html)。通过弄脏 Graphic，系统可以确保 Graphic 在下一帧渲染之前重建其布局和顶点。

可以将画布分配给 Scroll View 中每个元素的根 RectTransform，然后将重建限制为仅重新设置父元素而不是 Scroll View 的全部内容。但是，这往往会增加渲染 Scroll View 所需的绘制调用次数。此外，如果 Scroll View 中的单个元素很复杂并且由十几个 Graphic 组件组成，特别是如果每​​个元素上有大量 Layout 组件，那么重建它们的成本通常高到足以显着降低低端设备的帧速率。

如果 Scroll View UI 元素没有可变大小，则无需重新计算布局和顶点。但是，避免这种行为需要实现基于位置更改而不是父级或同级顺序更改的对象池解决方案。

#### 基于位置的滚动视图池

为了避免上述问题，可以通过简单地移动其包含的 UI 元素的 RectTransforms 来创建一个 Scroll View 池其对象。这避免了在未更改尺寸的情况下重新构建移动的 RectTransform 内容的需要，从而显着提高了 Scroll View 的性能。

要做到这一点，通常最好编写一个自定义的 Scroll View 子类或编写一个自定义的 Layout Group 组件。后者通常是更简单的解决方案，可以通过实现 Unity UI 的[LayoutGroup](http://docs.unity3d.com/ScriptReference/UI.LayoutGroup.html)抽象基类的子类来完成。

自定义 Layout Group 可以分析底层源数据以检查必须显示多少数据元素，并且可以适当地调整 Scroll View 的 Content RectTransform 的大小。然后它可以订阅[Scroll View 更改事件](http://docs.unity3d.com/ScriptReference/UI.ScrollRect-onValueChanged.html)并使用这些事件相应地重新定位其可见元素。

## 其他 UI 优化技巧和技巧
有时没有干净的方法来优化 UI。本节包含一些可能有助于提高 UI 性能的建议，但有些建议在结构上“不干净”，可能难以维护，或者可能有丑陋的副作用。其他可能是 UI 中行为的变通方法，旨在简化初始开发，但也可以相对简单地创建性能问题。

### 基于 RectTransform 的布局

布局组件相对昂贵，因为它们必须在每次标记为脏时重新计算其子元素的大小和位置。（有关详细信息，请参阅基础步骤的图形重建部分。）如果给定 Layout 中的元素数量相对较少且固定，并且 Layout 具有相对简单的结构，则可以将 Layout 替换为 RectTransform - 基于布局。

通过分配 RectTransform 的锚点，可以使 RectTransform 的位置和大小根据其父级进行缩放。例如，一个简单的两列布局可以用两个 RectTransforms 来实现：

-   左列的锚点应该是 X: (0, 0.5) 和 Y: (0, 1)
    

-   右列的锚点应该是 X: (0.5, 1) 和 Y: (0, 1)
    

RectTransform 的大小和位置的计算将由 Transform 系统本身以本机代码驱动。这通常比依赖 Layout 系统更高效。也可以编写设置基于 RectTransform 的布局的 MonoBehaviours。但是，这是一项相对复杂的任务，超出了本指南的范围。

### 禁用画布

当显示或隐藏 UI 的离散部分时，通常在 UI 的根部启用或禁用游戏对象。这可确保禁用 UI 中的任何组件都不会接收输入或 Unity 回调。

但是，这也会导致 Canvas 丢弃其 VBO 数据。重新启用画布将需要画布（和任何子画布）运行重建和重新批处理过程。如果这种情况频繁发生，增加的 CPU 使用率可能会导致应用程序的帧速率卡顿。

一种可能但很麻烦的解决方法是将要显示/隐藏的 UI 放置在其自己的 Canvas 或 Sub-canvas 上，然后仅启用/禁用此对象上的 Canvas 组件。

这将导致 UI 的网格不被绘制，但它们将保留在内存中，并且它们的原始批处理将被保留。此外，不会在 UI 的层次结构中调用[OnEnable](http://docs.unity3d.com/ScriptReference/MonoBehaviour.OnEnable.html)或[OnDisable回调。](http://docs.unity3d.com/ScriptReference/MonoBehaviour.OnDisable.html)

但是请注意，这不会禁用隐藏 UI 中的任何 MonoBehaviours，因此这些 MonoBehaviours 仍将接收 Unity 生命周期回调，例如 Update。

为避免此问题，将以这种方式禁用的 UI 上的 MonoBehaviour 不应直接实现 Unity 的生命周期回调，而应从 UI 根 GameObject 上的“回调管理器”MonoBehaviour 接收其回调。每当显示/隐藏 UI 时都可以通知这个“回调管理器”，并且可以确保根据需要传播或不传播生命周期事件。对此“回调管理器”模式的进一步解释超出了本指南的范围。

### 分配事件摄像机

如果使用 Unity 的内置输入管理器以及设置为在World Space或Screen Space – Camera模式中渲染的画布，请务必分别设置 Event Camera 或 Render Camera 属性。从脚本中，这始终作为[worldCamera](http://docs.unity3d.com/ScriptReference/Canvas-worldCamera.html)属性公开。

如果未设置此属性，则 Unity UI 将通过查找附加到带有 Main Camera 标签的 GameObjects 的 Camera 组件来搜索主摄像头。每个世界空间或相机空间画布至少会发生一次此查找。由于已知[GameObject.FindWithTag](http://docs.unity3d.com/ScriptReference/GameObject.FindWithTag.html)很慢，因此强烈建议所有世界空间和相机空间画布在设计时或初始化时分配其相机属性。

覆盖画布不会出现此问题。

### UI源代码定制

UI 系统旨在支持大量用例。这种灵活性很好，但这也意味着在不破坏其他功能的情况下无法轻松完成某些优化。如果您最终遇到通过更改 C# UI 源代码可以获得一些 CPU 周期的情况，则可以重新编译 UI DLL 并覆盖 Unity 附带的那个。此过程记录在[Bitbucket 存储库](https://bitbucket.org/Unity-Technologies/ui/)的自述文件中。确保获取与您的 Unity 版本对应的源代码。

但这只能作为最后的手段，因为有一些重要的缺点。首先，您必须找到一种方法将这个新的 DLL 分发给您的开发人员和构建机器。然后，每次升级 Unity 时，都必须将更改与新的 UI 源代码合并。在朝着这个方向前进之前，请确保您不能只是扩展现有类或编写自己的组件版本。

# Unity3D中的Quaternion（四元数）

<!--more-->

## **四元数的概念**

四元数，这是一个图形学的概念，一般没怎么见过，图形学中比较常见的角位移的表示方法有“矩阵”、“欧拉角”、“四元数”这三种。可以说各有各的优点和不足，不同的场合用不同的方法。其中四元数的优点有：平滑插值、快速连接、角位移求逆、可以与矩阵形式快速转换、仅用四个数表示。不过，它也有一些缺点：比欧拉角多一个数表示、可能不合法（如：坏的输入数据或者浮点数累计都可能使四元数不合法，不过可以通过四元数标准化来解决这个问题）、晦涩难懂。

那为啥四元数是四个数呢？其实还是有个小故事的。话说当时十九世纪的时候，爱尔兰的数学家Hamilton一直在研究如何将复数从2D扩展至3D，他一直以为扩展至3D应该有两个虚部（可是他错了，哈哈）。有一天他在路上突发奇想，我们搞搞三个虚部的试试！结果他就成功了，于是乎他就把答案刻在了Broome桥上。说到这里，也就明白了，四元数其实就是定义了一个有三个虚部的复数w+xi+yj+zk。记法[w,(x,y,z)]。

好了，上面我们就基本清楚四元数的作用以及好处与坑了，下面开始正式讲讲Unity中我们如何使用一些常见的四元数操作。

## **Unity中的四元数**

基本的旋转，我们可以通过Transform.Rotate来实现，但是当我们希望对旋转角度进行一些计算的时候，就要用到四元数Quaternion了。Quaternion的变量比较少也没什么可说的，大家一看都明白。唯一要说的就是xyzw的取值范围是[-1,1],物体并不是旋转一周就所有数值回归初始值，而是两周。 初始值： (0,0,0,1) 沿着y轴旋转：180°(0,1,0,0) 360°（0,0,0,-1）540°(0,-1,0,0) 720°(0,0,0,1) 沿着x轴旋转：180°(-1,0,0,0) 360°（0,0,0,-1）540°(1,0,0,0) 720°(0,0,0,1) 无旋转的写法是Quaternion.identify。


  **在unity3d中, quaternion 的乘法操作 (operator * ) 有两种操作:**

{{< admonition note "乘法操作"  >}}(1) quaternion * quaternion , 例如 q = t * p; 这是将一个点先进行t 操作旋转,然后进行p操作旋转. 

 (2) Quaternion * Vector3, 例如 p : Vector3, t : Quaternion , q : Quaternion; q = t * p; 这是将点p 进性t 操作旋转; 我进行的是第2种操作,即对一个向量进行旋转; 
首先 ,Quaternion 的基本数学方程为 : 

Q = cos (a/2) + i (x * sin(a/2)) + j (y * sin(a/2)) + k(z * sin(a/2)) (a 为旋转角度) 

Q.w = cos (angle / 2) 

Q.x = axis.x * sin (angle / 2) 

Q.y = axis.y * sin (angle / 2)

Q.z = axis.z * sin (angle / 2) 

我们只要有角度就可以给出四元数的四个部分值,例如我想要让点M=Vector3(o,p,q) 绕x轴顺时针旋转90度;那么对应的quaternion数值就应该为: 

Q : Quaternion; 

Q.x = 1 * sin(90度/2) = sin(45度) = 0.7071 

Q.y = 0; 

Q.z = 0; 

Q.w = cos(90度/2) = cos (45度) = 0.7071 

Q = (0.7071, 0 , 0 , 0.7071); 

m = Q * m; （将点m 绕 x轴(1,0,0) 顺时针旋转了90度) ** {{< /admonition >}}


下面我就按照Unity的API介绍下四元数相关的几个基本函数。

## **一、LookRotation**

声明形式：public static Quaternion LookRotation ( Vector3 forward, Vector3 upwards=Vector3.up )

这个功能很实用，传入的两个参数分别代表前方盯着的方向以及自己的上方向。它可以让一个GameObject转动脑袋盯着另一个物体。如：
  

     public Transform target;
        void Update() {
            Vector3 relativePos = target.position - transform.position;
            Quaternion rotation = Quaternion.LookRotation(relativePos);
            transform.rotation = rotation;
        }

这段代码就可以让当前的object时时盯着target不放，当然，你也可以自定义up朝向，这里默认是Vector3.up。

## **二、Angle**

声明形式：public static float Angle ( Quaternion a, Quaternion b )

这个就比较简单了，它可以计算两个旋转之间的夹角。与Vector3.Angle()作用是一样的。

## **三、Euler**

声明形式：public static Quaternion Euler ( float x, float y, float z )

或者： public static Quaternion Euler ( Vector3 euler )

这个函数可以将一个欧拉形式的旋转转换成四元数形式的旋转。传入的参数分别是欧拉轴上的转动角度。

## **四、Slerp**

声明形式：public static Quaternion Slerp ( Quaternion from, Quaternion to, float t )

基本意思就是线性地从一个角度旋转到另一个角度，其中，旋转匀速增加t。

附加内容：很多时候from 和to都不是固定的，而且上一个脚本也不能保证所有角度下的旋转速度一致。所以我写了这个脚本来保证可以应付大多数情况。

  

    Transform target; 
    float rotateSpeed = 30.0f; 
    Quaternion wantedRotation = Quaternion.FromToRotation(transform.position, target.position); 
    float t = rotateSpeed/Quaternion.Angle(transform.rotation, wantedRotation)*Time.deltaTime; 
    transform.rotation = Quaternion.Slerp(transform.rotation, target.rotation, t);


这个脚本可以保证物体的旋转速度永远是rotateSpeed。如果自身坐标和目标之间的夹角是X度，我们想以s=30度每秒的速度旋转到目标的方向,则每秒旋转的角度的比例为s/X。 再乘以每次旋转的时间Time.deltaTime我们就得到了用来匀速旋转的t。

## **五、FromToRotation**

声明形式：public static Quaternion FromToRotation ( Vector3 from, Vector3 to )

它是得到从一个方向到另一个方向的旋转。就是转一个方向，就这么简单。

## **六、identity**

这个不是一个函数，它是一个只读的变量。它代表世界坐标系或者父物体坐标系中的无旋转方位。


> Written with [StackEdit](https://stackedit.io/).

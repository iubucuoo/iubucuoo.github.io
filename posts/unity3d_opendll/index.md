# 反编译apk内dll的代码

<!--more-->
## DLL反编译工具
1. [dnSpy](https://github.com/dnSpy/dnSpy/releases)
## unity il2cpp apk 反汇编
### unity apk会出现两种情况
 * 解压后asset->bin->Data->Managed中有很多dll,这是用unity mono打包的，直接用dnspy/Reflector等工具打开Assembly-CSharp.dll即可
 * 解压后asset->bin->Data->Managed中有3个文件夹etc、Metadata、Resources,这是使用unity新版的il2cpp模式打包的，使用以下流程破解（破解后的Assembly-CShapr.dll依旧无法看到代码，会被转成16进制文件存储，反编译好像没什么意义，当然小部分项目由于使用的unity版本过旧依旧可以查看代码，可以先查看下apk使用的unity版本再决定是否需要反汇编）
### 针对第二种情况(无法看到代码)
1. 下载[il2cppDumper](https://github.com/Perfare/Il2CppDumper)
2. 将应用apk重命名，后缀名为.rar,并解压
3. 运行il2CppDumper.exe
4. 在apk解压的文件夹中，找到asset->bin->Data,随便用文本框打开一个文件，可以看到一堆乱码，但第一行会有几个数字，比如2019.1.12f1即为版本号
5. 在il2CppDumper.exe中要求找到 ，apk解压的文件夹中lib-->arm64-->libil2cpp. so,（arm64名字可能不一样，找到最后的libil2cpp.so即可，会有多个libil2cpp.so文件，随便选择一个即可）
6. 在il2CppDumper.exe中要求找到 ，apk解压的文件夹中assets->bin->Data->Managed->Metadata->global-metadata.dat文件
7. 出现要求输入版本号，输入步骤4找到的版本号，如：2019.1.12/2019.1.12f1都可，回车，出现 select Mode: i.Manua1 2.Auto 3.Auto(Plus)4.Auto(Symbo1)
8. 选择3，等待一会，就会在il2CppDumper.exe同级目录下生成dump.cs、script. py、DummyDll文件，如果失败可以依次尝试其它模式
9. 最后使用[dnspy](https://github.com/dnSpy/dnSpy)/Reflector等工具打开Assembly-CSharp.dll即可

## 其他
 1. unity版本输入不正确时，会出现：System.IndexOutOfRangeException: 索引超出了数组界限。也可能会出现该模式不可使用，要求尝试其它模式的提醒
 2. 选择1,地址输入不正确时会出现只生成dump.cs和script.py文件而不生成DummyDll文件夹的情况（实际上dump.cs里也只有一些注释，生成错误）


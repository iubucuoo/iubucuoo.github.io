# VSCode的安装与使用在untiy中

<!--more-->
## 下载安装
[下载vscode](https://code.visualstudio.com/download)并安装，安装位置随意。

## 插件安装
**Tip : 插件可以先全部安装好，再对各个插件进行相应设置**
### Chinese(Simplified)(简体中文)
默认VSCode是英文版，先安装中文插件。在左侧导航栏的四个方格的按钮或者快捷键(ctrl+shift+x)，在搜索栏中输入**chinese**,点击中文简体进行安装，看到卸载按钮为安装好了，安装完重启软件。
### lua(作者:sumneko)
安装方式同上
### LuaPanda(调试工具)
1. 安装方式同上
2. 下载github内luapanda_matser整个工程[调式器快速试用页面(github)](https://github.com/Tencent/LuaPanda/blob/master/Docs/Manual/quick-use.md)
3. 把luapanda_matser项目中/Debugger下的LuaPanda.lua文件拷贝到unity的lua工程根目录
4. 在客户端Main.lua中新增 require("LuaPanda").start("127.0.0.1",8818)
5. 增加启动配置launch.json
```
    {
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lua",
            "request": "launch",
            "tag": "normal",
            "name": "LuaPanda",
            "description": "通用模式,通常调试项目请选择此模式 | launchVer:3.2.0",
            "cwd": "${workspaceFolder}",
            "luaFileExtension": ".lua",
            "pathCaseSensitivity": true,
            "connectionPort": 8818,
            "stopOnEntry": false,
            "useCHook": true,
            "autoPathMode": true,
            "logLevel": 1
        },
        {
            "type": "lua",
            "request": "launch",
            "name": "LuaPanda-Attach"
        },
        {
            "type": "lua",
            "request": "launch",
            "tag": "independent_file",
            "name": "LuaPanda-IndependentFile",
            "description": "独立文件调试模式,使用前请参考文档",
            "luaPath": "",
            "packagePath": [],
            "luaFileExtension": ".lua",
            "connectionPort": 8818,
            "stopOnEntry": false,
            "logLevel": 1,
            "useCHook": true
        }
    ]
}
```
6. 启动debug调试，点运行和调试三角箭头。
7. 看调试控制台内容
8. 启动你的unity项目查看控制台是否已经建立
#### 配置说明
launch.json 配置说明 
launch.json 指的是存在于被调试项目的 .vscode/launch.json 文件，它保存调试器的运行配置，这里主要介绍各项配置项的含义，方便大家根据需要求改。 另外需要注意的是，调试器不同版本生成的 launch.json 文件可能会有所不同，如果升级后遇到问题，可以删除launch.json文件并重新生成。
下面是 3.2.0 版本的 launch.json
```lua
{
    "version": "0.2.0",
    "configurations": [
        {
                        "type": "lua",
                        "request": "launch",
                        "tag": "normal",
                        "name": "LuaPanda",
                        "cwd": "${workspaceFolder}",
                        "luaFileExtension": "",
                        "connectionPort": 8818,
                        "stopOnEntry": true,
                        "useCHook": true,
                        "autoPathMode": true,
                    },
                    {
                        "type": "lua",
                        "request": "launch",
                        "tag": "independent_file",
                        "name": "LuaPanda-IndependentFile",
                        "luaPath": "",
                        "packagePath": [],
                        "luaFileExtension": "",
                        "connectionPort": 8820,
                        "stopOnEntry": true,
                        "useCHook": true,
                    }
    ]
}
```
launch.json 包含两个模式
+ LuaPanda 自适应模式
+ LuaPanda-IndependentFile 单文件调试模式

我们定义的自适应模式是一种最频繁使用的模式。有些调试器分为 launch 以及 attach 模式，我们理解 lua 是一种脚本语言，常嵌入 c# ， c++ 中被调用，通常在使用 lua 调试器时，用户会手动启动 unity / unreal，调试器无需再拉起这些被调试程序。如果有启动 vscode 调试器拉起一个二进制程序的需求，可以关注下面的 program 选项。

必要配置

| 项目             | 默认值               | 意义                                                         |
| ---------------- | -------------------- | ------------------------------------------------------------ |
| type             | "lua"                | 插件适用于lua语言，**请勿修改**                              |
| request          | "launch"             | 因为我们使用自适应模式，保持launch不必修改                   |
| tag              | "normal"             | 可以选择normal(通用模式)，attach(附加模式)，区别是启动时是否自动拉起program配置指向的程序。**不建议修改** |
| name             | "LuaPanda"           | 展示在VScode运行按钮旁的目标名，3.2.0 后可根据需要自行修改   |
| cwd              | "${workspaceFolder}" | 被调试的包含lua目录，${workspaceFolder} 指 VScode 打开的目录，通常不用修改。即使要修改，也请用 "\${workspaceFolder}/path" 这样的相对路径 |
| luaFileExtension | ""                   | **重要设置：** 用户设置的lua文件的后缀，如 txt ,  lua.txt 等 |
| connectionPort   | 8818                 | 默认端口号，如果连接无问题，可以不用修改。如果改了这里，请同步修改`require("LuaPanda").start(ip, port)`中的端口号 |
| stopOnEntry      | true                 | 调试器建立连接后立刻停止。接入调试器时建议设置true, 稳定使用后可根据用户需要设置成 false |
| useCHook         | true                 | 运行时尝试加载 c 模块，这个模块作用是加速运行，加载不成功也不会影响调试效果 |
| autoPathMode     | true                 | 是否使用自动路径模式。**强烈建议 true**                      |
| program     | ""                 | 可以填入一个二进制文件路径，启动调试器会自动运行这个文件 |

扩展功能的可选配置
| 项目                    | 默认值      | 意义                                                         |
| ----------------------- | ----------- | ------------------------------------------------------------ |
| isNeedB64EncodeStr      | true        | 对传输的字符串使用 base64 加密，避免一些异常字符干扰协议解析 |
| pathCaseSensitivity     | false       | 路径大小写敏感。默认 false 可兼容 getInfo获取的路径大小写，无需修改 |
| updateTips              | true        | 当检查到项目中的 lua 文件比较旧时，提示用户升级              |
| logLevel                | 1           | 日志等级，开发调试器时可能会使用0级，大量日志会降低运行效率。正常使用请勿修改 |
| distinguishSameNameFile | false       | 调试器默认不做同名文件区分 , 请不要在同名文件中打断点（此时仅依靠文件名进行文件区分）。如需要调试器区分同名文件，可尝试设置为 true，此时会执行较为严格的路径模式。 |
| truncatedOPath          | ""          | 路径裁剪，**通常无需修改**。配合 distinguishSameNameFile: true 模式使用。裁减掉 getinfo 的一部分路径，用剩余的路径进行断点匹配 |
| VSCodeAsClient          | false       | 反转 VScode 和 lua 进程的 C/S                                |
| connectionIP            | "127.0.0.1" | 配合 VSCodeAsClient: true 模式使用，要连接的 lua 进程所在ip  |

+ LuaPanda-IndependentFile 模式的配置
LuaPanda-IndependentFile 我们称之为"独立文件模式" ,  它的作用是打开一个新的终端，并 lua 命令运行当前 VSCode 活动窗口中的 lua 代码，并连接调试器，对其进行调试。

这个模式的目的是方便进行 lua 开发时，测试一些独立的文件 / 函数运行状况。

使用时请确保系统中安装了 lua 命令行二进制文件 以及 luasocket。测试方法是打开一个终端，运行 lua 看是否报错，之后尝试 `require("socket.core")`  不报错即可使用。

单文件模式有一些独立的配置：
| 配置项      | 默认值 | 建议                                                      |
| ----------- | ------ | ------------------------------------------------------------ |
| luaPath     | ""     | Lua 可执行文件的路径，如果已经加入系统path路径，可以不用修改 |
| packagePath | []     | 可以加入用户自己的packagepath路径，比如["./?.lua", "../?.lua" ] |


## 屏蔽某些后缀名文件
1. 从顶部菜单栏打开**文件**-> 首选项-> 设置（或者使用快捷键Ctrl+，）
2. 在设置页面的输入框中输入files:exclude
3. 点击添加模式，输入 **/*.meta 即可
#### 常用快捷键
1. 查找引用：shift+f12
2. 返回上一步：Alt+←
3. 文件搜索：ctrl+p



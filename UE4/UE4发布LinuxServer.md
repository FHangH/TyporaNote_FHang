# UE4发布LinuxServer



:::tip 情景内容

1. 需要开发专用服务器
2. 需要将Server发到云端Linux系统主机
3. 需要关闭SSH，服务一直启用
4. 需要使用UE4

:::



### 1. Visual Studio 2019



#### 1.1 下载 Visual Studio



:::warning

- **UE4**编译源码可以使用**VS2022**，但就此博客发布时间，需要修改配置文件才能正常编译，而2017版本太老了；
- 后续的C++编写在选用**Rider For Unreal**和**Epic**商城的二进制版本UE4；
- 项目写完后，用源码版**UE4**打包发布；
- **UE5**的试过了，很麻烦，问题很多，想想自己是写代码，何必执着于**UE5**，于是就释然了
- 电脑安装源码引擎的硬盘空间至少要有**200GB**
- 内存至少**16GB**，最好去网上查下**扩展虚拟内存**，自动托管到其他磁盘空间较大的磁盘，最好是**固态硬盘**
- 编译源码引擎时，确保关闭无关电脑**后台**，除非报错，否则不操作电脑做其他事，防止**虚拟内存不足**

:::



1. 微软只在**Visual Stuido**主页提供最新版本下载，需要单独去下载 [Visual Studio 2019](https://learn.microsoft.com/zh-cn/visualstudio/releases/2019/release-notes)
2. 下载社区版的就可以了



#### 1.2 配置 Visual Studio



- 安装完成后，会进入**Visual Studio Installer**
- 在**已安装**内可以看见**Visual Studio 2019**的安装程序
- 点击**修改**
- 在**工作负荷**内找到**游戏**，勾选**使用C++的游戏开发**
- 此时在右边的**安装详细信息**内的**使用C++的游戏开发**的可选项内确保勾选：
  - **C++分析工具**
  - **C++ AddressSanitizer**
  - 最新的**Window 10 SDK**
  - **IntelliCode**
  - **Unreal Engine 安装程序**
- **单个组件**内：
  - 默认勾选的**C# 和 Visual Basic Roslyn 编译器**
  - 搜索勾选**MSBuild**
- 最后检查一下是否勾选了**最新的Net Framework SDK**，该博客勾选的是**Net Framework 4.8 SDK**
- 下载安装，安装位置可自定义



:::warning

安装时出现**共享组件、工具和SDK**的路径不能修改，是因为此前可能已经安装过**Visual Studio**，需要修改注册表解决问题

解决步骤：

1. **win + r**
2. **regedit**
3. **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\Setup**删除**SharedInstallationPath**
4. 重启**Visual Studio Installer**

:::



#### 1.3 修改解决方案样式



:::tip 可选步骤

1. **菜单栏**内找到**工具**
2. **工具->自定义->命令->工具栏->标准->解决方案配置->修改所选内容**
3. 更改宽度：例如**200**
4. 若要在生成项目时显示**输出**窗口，请在**选项对话框->项目和解决方案>常规**页上，选择**在生成开始时显示输出窗口**

:::



#### 1.4 管理员模式启动



:::tip 可能重要

1. 在**VS**安装目录内：**\Common7\IDE\devenv.exe**
2. **右键->兼容性疑难解答->疑难解答程序->勾选改程序需要附加权限->测试后下一步->保存设置**

:::



### 2. 交叉编译器



:::danger

下载交叉编译器，千万别去**官方中文**的**虚幻文档**，里面给的**交叉编译工具链接**版本是错的，应该默认进入**官方英文**的文档网站内下载

博客使用的是**UE4.27.2**，下载文档内提供的**链接**，并确保是**-v19**的版本，如果不是就进错网站了，正确的网站：[UE4.27.2 交叉编译](https://docs.unrealengine.com/4.27/en-US/SharingAndReleasing/Linux/GettingStarted/)

:::



#### 2.1 安装交叉编译



直接默认安装即可，也可以选择安装路径

cmd验证：`%LINUX_MULTIARCH_ROOT%x86_64-unknown-linux-gnu\bin\clang++ -v`

无效需手动配置环境变量



#### 2.2 配置环境变量



1. 进入系统环境变量
2. 添加：`LINUX_MULTIARCH_ROOT`
3. 路径：`安装目录\v19_clang-11.0.1-centos7\x86_64-unknown-linux-gnu\bin`



:::warning

交叉编译器安装和配置完成后，需要重启电脑

:::



### 3. 源码UnrealEngine



:::tip 省略步骤

 **github**账号

 加入**Epic**组织

 选择**UE4.27.2 Release**下载

:::



#### 3.1 配置Setup.bat



**Setup.bat**配置表：

```shell
Usage:
   GitDependencies [options]

Options:
   --all                         Sync all folders
   --include=<X>                 Include binaries in folders called <X>
   --exclude=<X>                 Exclude binaries in folders called <X>
   --prompt                      Prompt before overwriting modified files
   --force                       Always overwrite modified files
   --root=<PATH>                 Set the repository directory to be sync
   --threads=<N>                 Use N threads when downloading new files
   --dry-run                     Print a list of outdated files and exit
   --max-retries                 Override maximum number of retries per file
   --proxy=<user:password@url>   Sets the HTTP proxy address and credentials
   --cache=<PATH>                Specifies a custom path for the download cache
   --cache-size-multiplier=<N>   Cache size as multiplier of current download
   --cache-days=<N>              Number of days to keep entries in the cache
   --no-cache                    Disable caching of downloaded files

Detected settings:
   Excluded folders: Mac, Android, Linux
   Proxy server: none
   Download cache: disabled

Default arguments can be set through the UE4_GITDEPS_ARGS environment variable.
Installing prerequisites...
```



该博客的配置：(截取的一段，只有**set PROMPT_ARGUMENT=**需要修改)

```shell
set PROMPT_ARGUMENT=
for %%P in (%*) do if /I "%%P" == "--prompt" goto no_prompt_argument
for %%P in (%*) do if /I "%%P" == "--force" goto no_prompt_argument
set PROMPT_ARGUMENT=--prompt --threads=20 --cache=E:\UrealEngineSource\UE4.27.2\Cache
:no_prompt_argument
```

```shell
--threads=20 # 加快下载速度，不宜过高
--cache # 将下载的源码文件单独保存到指定位置，方便传给其他人和玩坏了，不用全删了重新下载了
--exclude # 排除不想要的模块
```



**--exclude**可选参数：

```shell
平台参数：Win32, Win64, osx32,osx64, Linux, Android, IOS, HTML5
VS版本参数：VS2012,VS2013,VS2015
尽量别排除VS版本的参数，平台参数可以排除不要的，该博客默认不加入 --exclude，全要了
```



#### 3.2 生成UE4.sln文件



上面配置好**Setup.bat**文件后：

- 双击运行**Setup.bat**
- 双击运行**GenerateProjectFiles.bat**



#### 3.3 编译源码编辑器



:::warning

1. 先编译源码 编辑器
2. 网上有的教程说的很乱，这里就编译**Development Editor**即可
3. 注意：
   - 如果是先完成过了源码引擎的编译，后下载**交叉编译**，需要按照博客**2. 交叉编译器**的内容完成检查步骤
   - 再进行博客**3.2 生成UE4.sln文件**的步骤，再接着往下继续看，并完成下面的全部步骤

:::



双击**UE4.sin**文件，根据上面的**VS2019配置**，此时默认**管理员模式**

在解决方案中依次选择：

- **Development Editor**
- **Win64**
- 选中侧边栏的**UE4**，**右键->设为启动项**
- 右键**UE4**选择**生成**，注意：不希望看见**Warning**和**Error**，如果有请深入排查问题，然后右键**UE4**选择**重新生成**

上面的步骤完成后，不关闭**VS2019**，建议进行下一步：

- 依次再**重新生成**：`AutomationTool、UnrealBuildTool、UnrealFrontend`
- 如果全部没**Warning**和**Error**，就算编译完成了



#### 3.4 创建快捷方式



网上有的教程真的傻，非得打开**UE4.sln**在**VS2019**里面启动**源码虚幻引擎**

其实可以进如源码引擎目录：`\Engine\Binaries\Win64`找到**UE4Editor.exe**

直接双击启动，或者创建快捷方式，放到方便使用的地方启动，和正常的Epic商城引擎没什么两样



:::warning

源码引擎编译完成后，先打开引擎看看打包功能是否出现:warning:标志

如果有，大概率是**VS2019**和**交叉编译器**没有配置好，缺失**SDK**，需要**重新生成**

如果没有，再往后继续看

:::



### 4. 打包测试项目



#### 4.1 创建测试项目



1. 打开UE4，是不是源码引擎无所谓，真的
2. 新建一个**C++、第三人称、无初学者内容包、无光追**的项目**Test**



#### 4.2 修改测试项目



1. 进入**ThirdPersonBP**找到**Map**
2. 依次创建两个新**Level**：**Map_Translation**用来服务器过渡用，**Map_Start**这是游戏客户端默认打开的地图，已经存在的模板地图是客户端玩家进入的**服务器地图**
3. 同时创建一个**User Widget**，随便给个按钮，实现按钮**点击事件**，**Open Level**，**Level Name**参数是：`你的云服务器公网IP:7777`，后面会说**云主机的配置**，也可以直接跳到博客**5. 云主机防火墙配置**，先拿到**公网IP**
4. 打开**Map_Start**，打开关卡蓝图
5. 在**Begin Play**实现：`Create Widget`，`Add To Viewport`，`GetPlayerController->SetShowMouse->ture`



#### 4.3 设置测试项目



- 打开项目设置：
  - **全局GameMode**可以设为**none**，默认不设也许，完全不影响测试结果
  - 编辑器初始地图：无所谓
  - 游戏默认地图：**Map_Server**
  - 过渡地图：**Map_Translation**
  - 服务器地图：**第三人称的模板默认地图**
- 在项目设置，**打包**中搜索**list of maps to include in a packaged build**:
  - 添加进上面的三个地图



#### 4.4 修改 build.cs



:::warning

注意：测试项目创建用的是不是源码引擎不重要，后面的**4.5 配置打包程序**要用**源码引擎打开**，具体后面会写

接下来会编译至少两次**项目C++源码**，假设最终打包结果为：一个Linux Server，一个Windows Client

需要先打开项目文件的**.sln**，解决方案依次是分别编译一下两种配置：

Linux Server包：**Development Server**，**Linux**

Windows Client包：**Development**，**Win64**，如果Windows Client想打包成**发行版**，就选择**Shipping**，**Win64**，该博客选择**Development**

如果报错，例如：

1. 交叉编译比需求的版本老了，那就是交叉编译器装错了，顺便查查系统环境变量，再照着博客**3.2 生成UE4.sln文件**重来
2. 缺失对应平台的SDK，去博客的**1.2 配置 Visual Studio**重来
3. 其他的自行上网查询

:::



上面的是警告，需要注意看需求，选择合适解决方案的配置

首先还算要打开项目目录下的**.sln**文件：

1. 找到项目目录中的`项目名Editor.Target.cs`文件

2. 直接复制这个文件，在同级目录下，再重命名为：`项目名Server.Target.cs`，打开

3. 修改内容为：(博客的配置)

   ```c#
   // Fill out your copyright notice in the Description page of Project Settings.
   
   using UnrealBuildTool;
   using System.Collections.Generic;
   
   public class Test01ServerTarget : TargetRules
   {
   	public Test01ServerTarget(TargetInfo Target) : base(Target)
   	{
   		Type = TargetType.Server;
   		DefaultBuildSettings = BuildSettingsVersion.V2;
   
   		ExtraModuleNames.AddRange( new string[] { "Test01" } );
   	}
   }
   ```

4. 分别是`public class Test01ServerTarget` ，`public Test01ServerTarget(TargetInfo Target)`，`Type = TargetType.Server`



修改完成后，再依照上面的**Warning**内容进行编译项目文件



#### 4.5 配置打包程序



首先找到项目目录下的**.uproject**文件，右键**Select Unreal Engine Version**，选择**Soucre build**开头的**UE4**源码引擎

生成一段时间后，双击**.uproject**文件打开源码引擎

配置打包程序：(目的是一次性同时打包出 LinuxServer和windowsClient两个包)

1. 在引擎主界面的**顶部菜单栏**中找到**窗口->项目启动程序**
2. 添加**自定义启动描述文件**
3. 在**烘培**中选择**常规**，勾选**LinuxServer**，**WindowsNoEditor**(也可以是WindowsClient)
4. **烘培语言**：随意，一般默认选**en**
5. **包**：选择**本地打包存储**，修改自己的**本地目录路径**
6. **部署**：选择**不部署** (注意：很重要，不然后面打包会卡在`Launching UAT`)
7. 返回

点击**启动此描述文件**，就可以打包了



### 5. 云主机/ECS



#### 5.1 云服务配置



:::tip 云主机/ECS/轻量应用服务器

随便哪家的云服务都行，系统最好安装**Centos7**，**root密码随你**，重要的是防火墙的配置：

1. 添加**TCP**，端口：**7777**
2. 添加**UDP**，端口：**7777**

:::



#### 5.2 上传和运行Server



:::warning

博客省略了将文件上传到远端服务器的过程，反正把**LinuxServer**包压缩**.zip**格式比较简单操作，容易理解

上传的路径一定别是**root**用户的**home**也就是**~**目录

:::



1. 先创建一个普通用户

   ```bash
   useradd unreal
   passwd unreal // 输入两次相同的秘密
   cd /home/unreal
   ```

2. 把压缩文件上传的当前目录

3. 安装解压程序

   ```bash
   yum install unzip
   ```

4. 解压压缩包

   ```bash
   unzip 压缩包
   ```

5. 通过root设置执行权限

   ```bash
   chmod +x 项目名称Server.sh
   ```

6. 切换到普通用户unreal

   ```bash
   su unreal
   ```

7. 执行

   ```bash
   ./项目名称Server.sh -log
   ```

8. 在本地端，打开WindowsNoEditor包的程序进行测试



注意：在Windows本地段关闭远程连接云服务的**SSH**，正在运行的**Server**服务会停止

解决方案：**nohup**，**screen**

博客采用最简单的**nohup**：

```bash
nohup ./项目名称Server.sh -log &
```

如果想终止这个服务：

```bash
su root
ps aux
// 找到 ./项目名称Server.sh -log 这个命令的进程，记住它的pid
kill p
```








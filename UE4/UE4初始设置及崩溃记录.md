# UE4初始设置及崩溃记录



	1.第一次记录：2020.2.05 版本：4.24.2



[toc]



# UnrealEngine的初始设置	


## 1.在Epic中下载UnrealEngine4之后

###### 1-1. 首先：启动-选项
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205144655903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)
###### 1-2. 接着	：（勾选）输入调试用符号-应用
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205144746667.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)
以后出现新的崩溃问题，方便查看详细原因

## 2.进行UE4相关的文件配置
###### 2-1. 删除C盘中的缓存及修改UE4缓存地址问项目地址
**参照内容来源于CSDN博客文章：**[https://blog.csdn.net/cc13813194235/article/details/53424866](https://blog.csdn.net/cc13813194235/article/details/53424866)
**同时本人以	UE4_4.24.2版本	亲测有效**

1.首先随便创建一个游戏项目，选择自己的项目目录

2.创建完成后关闭UE4

	找到UE4的项目缓存目录
	C:\Users\用户名\AppData\Local\UnrealEngine\Common\DerivedDataCache
	删除DerivedDataCache

3.找到安装UE4的目录
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205151224151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)
4.在 UE_4.24/Engine/Config 中找到 BaseEngine.ini 文件，直接用记事本打开

5.编辑——查找——在查找框中输入 [InstalledDerivedDataBackendGraph]
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205151306657.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)

6.将 [InstalledDerivedDataBackendGraph] 下面的内容进行修改

	1 Local=(Type=FileSystem, ReadOnly=false,Clean=false, Flush=false, PurgeTransient=true, DeleteUnused=true,UnusedFileAge=34, FoldersToClean=-1, Path="%ENGINEVERSIONAGNOSTICUSERDIR%DerivedDataCache")
	
	修改为：
	1 Local=(Type=FileSystem, ReadOnly=false,Clean=false, Flush=false, PurgeTransient=true, DeleteUnused=true,UnusedFileAge=34, FoldersToClean=-1,Path="%GAMEDIR%DerivedDataCache")

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205151509665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)
7.修改后进行验证是否修改成功
	1.在BaseEngine.ini 文件中，在查找框中输入 [DerivedDataBackendGraph]
	![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020515250133.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)
	2.找到图中选中的文本，即修改成功，项目缓存路径以及改到项目文件夹内
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205152632765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)
###### 2-2. 删除联机构建SwarmAgent程序的缓存
缓存目录：C:\Users\用户名\AppData\Local\UnrealEngine\4.24\Saved\Swarm\SwarmCache
删除 SwarmCache

## 3. UE4的相关设置
###### 3-1. 设置UE4启动时不编译蓝图
**参照内容来源于知乎文章：**[https://zhuanlan.zhihu.com/p/104097525](https://zhuanlan.zhihu.com/p/104097525)

1.找到配置文件路径：C:\Users\用户名\AppData\Local\UnrealEngine\4.24\Saved\Config\Windows\Engine.ini

2.打开Engine.ini文件

3.文本内容中添加
	
	[/Script/Engine.Blueprint]
	bRecompileOnLoad=False
	
	[/Script/Engine.LevelScriptBlueprint]
	bRecompileOnLoad=False
	
	[/Script/Engine.AnimBlueprint]
	bRecompileOnLoad=False

4.保存，重启

###### 3-2. 设置仅编译成功时保存
1.在随便打开一个蓝图，在编译的旁边有个倒三角，如图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205171503766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)

###### 3-3. 显示编辑器实时信息
1.编辑-编辑器偏好设置-通用-性能-勾选显示帧率和实时信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205172012598.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)

# 崩溃日志
## UE4崩溃日志（1）
**参照内容来源于虚幻4官方文档：**[https://docs.unrealengine.com/zh-CN/GettingStarted/RecommendedSpecifications/index.html](https://docs.unrealengine.com/zh-CN/GettingStarted/RecommendedSpecifications/index.html)
**里面有虚幻4的硬件配置要求，配套软件版本的需求，以及显卡驱动的需求**

###### 因显卡驱动程序原因导致崩溃
	关键字：D3D11RHI、Rendering、Building
	D3D11 为Direct3D11（渲染管线），RHI 为Render hardware interface 渲染硬件层接口，d3d11.dll 为动态链接库Dynamic Link Library
如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205155157795.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)
解决方法：

1.首先进入 NVIDIA GeForce Experience
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205160827894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)

2.检查更新文件（旁边的三个点）-选择首选项-Studio驱动程序-检查更新文件（如图）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205161143542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)
3.检查到更新文件后，下载最新的驱动，自定义安装，勾选恢复默认设置

4.重启电脑（最好重启一下）

## UE4崩溃日志（2）
**参考内容来源于知乎文章：**[https://zhuanlan.zhihu.com/p/45508890](https://zhuanlan.zhihu.com/p/45508890)
**该知乎作者写了很多UE4的Bug以及部分崩溃解决方案**

**虚幻4官方文档内容参考：**[https://docs.unrealengine.com/zh-CN/Engine/Rendering/Nvidia/NVIDIAAftermath/index.html](https://docs.unrealengine.com/zh-CN/Engine/Rendering/Nvidia/NVIDIAAftermath/index.html)
###### 因NVIDIA Aftermath 崩溃
	关键字：Aftermath、D3D11Query.cpp] [Line: 111] 、GPU has crashed
	Aftermath 是一个轻量化的实用工具，可减轻部分调试工具对性能历史记录的要求。实际上，它相当轻量化，甚至可以包含在发布的游戏中，提供开发者需要的用户电脑数据。程序员可利用 Aftermath 在代码中插入标记，帮助追踪崩溃发生的根源。

解决方法：
	1.找到 ConsoleVariables.ini ，路径：安装的磁盘：UE_4.24\Engine\Config\ConsoleVariables.ini
	2.找到 [Startup]
	3.在该区域文本内容下添加
		r.DX11NVAfterMathEnabled=0
		r.GPUCrashDebugging=0
		如图：
		![在这里插入图片描述](https://img-blog.csdnimg.cn/20200205164450952.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxOTY5Mg==,size_16,color_FFFFFF,t_70)
		4.保存，重启UE4

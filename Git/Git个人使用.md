---
title: Git个人使用
tags: Git
categories: Git
top_img: 'https://cdn.jsdelivr.net/gh/FHangH/FHangBlogCDN/Post_Img/postImg_12.jpg'
cover: 'https://cdn.jsdelivr.net/gh/FHangH/FHangBlogCDN/Post_Img/postImg_12.jpg'
---

# Git个人使用



```
// 2021-02-19：苦于还是不记得git bash的使用，每次使用都要到网上找一会，太麻烦了，决定写成博客，记录一下个人的使用总结
// 开头从最开始的顺序来记录
-----------------------------------------------------------------------------------------------------------------------
// 2021-02-20：又添加了 clone 的方法，修改完善了之前的内容
```



### 1. 准备工作

简单带过：

- 注册Github账号
- 下载Git，并安装



### 2. 本地账号

- 这一步是方便以后使用git时，跳过账号信息验证

```shell
// "" 里填 GitHub 账号的用户名
$ git config --global user.name ""

// "" 里填 GitHub 账号的邮箱
$ git config --global user.email ""

// 查看本地的用户信息配置
$ git config --list
// 结果大概如下
PS C:\Windows\System32> git config --list
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
http.sslbackend=openssl
http.sslcainfo=E:/Git/Git/mingw64/ssl/certs/ca-bundle.crt
core.autocrlf=true
core.fscache=true
core.symlinks=false
pull.rebase=false
credential.helper=manager
user.email=752972182@qq.com
user.name=FHangH
```



### 3 Public SSH Key

- 在 Github 中添加一个 Public SSH key 同时在本地也要有 Public SSH Key 的相关文件
- git 上传和下载过程中需要密钥的验证，以保证安全性

```shell
// 首先验证是否本地存在 SSH Key
$ ssh -T git@github.com

// 如果存在，大概结果如下（可以直接跳过生成本地 SSH Key 的步骤）
PS C:\Windows\System32> ssh -T git@github.com
Hi FHangH! You've successfully authenticated, but GitHub does not provide shell access.
```

```shell
// 查看本地是否已经存在 Public SSH Key
$ cd ~/.ssh
```

```shell
// 查看文件列表
$ ls

// 此时两种情况，一种是什么都没有
// 另一种是，差不多是存在以下文件，至少是有 id_rsa , id_rsa.pub 两个文件才行

PS C:\Users\Admin\.ssh> ls
    Directory: C:\Users\Admin\.ssh
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---           2021/2/17     9:18           2610 id_rsa
-a---           2021/2/17     9:18            575 id_rsa.pub
-a---           2021/2/19    16:05           1385 known_hosts
```

```shell
// 如果有，我们需要 id_rsa.pub 的内的key
$ cat id_rsa.pub

// 出现类似一下内容（全文复制，后面要用）
PS C:\Users\Admin\.ssh> cat .\id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDdrPvLeNqhzEgVU8Ep/9LiDvGpooO6UD8Tq5DM4CJzfiS+I95KjwwhxpQ7Et0pgfMt6ikRBXE1phgxoaK+tArSLcAOr1k8CgHazzB7D2j2X6v4x0Xmescq4dzB+R+6dtYGWhn5qwFjn2KljjYGVwitwdjyuqIqzS3vEpJaIpzI8nOnbGPR42a6t9FmBg3KhYyrcT5Z7DJgQvF1RkXmjeCjtHOOWL7xdDjI8iRwF3Kkiz78ovv2jr8MB2unrgPTNQ56ZPSi34gAGIDgt59VVM14P6GRxvRvtCG644QOEH/4woOmADi28BD3Gkj2+1Z1tXTaj1WPOvyEorHAXVS4L9fDScvaRK3el1LKk1hX1/dq3+ozN+Jpm8KWVtfLGfKxmKgQSJFX6qH49FuCBcD0Rpk3WnJInqz5+HLUlGqQypf0oTjQFpa+vY83/Fa3WKpqAuJM136+3mxeZFv+YCJv7eql2FzNhUAMG6Zur4/Kx5uMP1QFl0y9JYmH7WebS9MMzmE= Admin@DESKTOP-EBFV026
```

```shell
// 如果没有，需要生成一个
// "" 内填写 Github 账号的注册邮箱
$ ssh-keygen -t rsa -C ""

// 接下的步骤大概就是确认，填写密码之类的，回车键跳过（简单点）
```



### 4. Github添加Key

需要在Github账户内，将本地生成的 SSH key 添加进去，Git上传或下载时，才不会出现错误

- 进入Github用户主页
- 用户头像边上的倒三角
- Settings 进入设置页面
- 在侧边栏内找到 SSH and GPG Keys
- 在SSH Keys 内找到 New SSH Key
- 跳转页面后，在Title页面里填写 Public SSH Key 名称（随意）
- 在Key的文本框内，粘贴进之前在 复制的 id_rsa.pub 文件的内容
- Add SSH Key



### 5. Git仓库

- 创建 Git 仓库 （这个也跳过，简单的创建一个公开的仓库）
- 创建后，可以简单记录一下 SSH 的链接，比如：git@github.com:FHangH/FHangBlogCDN_02.git



### 6. 上传本地到Git

- 首先是进入要上传的项目文件夹内

```shell
// 初始化本地仓库
$ git init

// 将文件全部加入到缓存中
$ git add .
// 或者在 add 后面跟上指定的文件或某种类型的文件
$ git add 

// 提交操作记录，"" 内随意填
$ git commit -m ""

// 远程添加到源中，也就是git仓库
$ git remote add origin git@github.com:用户名/仓库名.git
// 有时候出现失败情况，就用下面这条命令，然后再重复上面的命令
$ git remote rm origin

// 最后，将本地缓存中的文件上传的远程的git仓库中（有时候网络会抽风）
$ git push origin master
```



### 7. 从远程Git仓库拉到本地

- 本地创建一个空的项目文件夹，在文件夹内进行

```shell
// 多种 clone 的方法
$ git clone http[s]://example.com/path/to/repo.git
$ git clone http://git.oschina.net/yiibai/sample.git
$ git clone ssh://example.com/path/to/repo.git
$ git clone git://example.com/path/to/repo.git // 这个速度最快
$ git clone /opt/git/project.git 
$ git clone file:///opt/git/project.git
$ git clone ftp[s]://example.com/path/to/repo.git
$ git clone rsync://example.com/path/to/repo.git
```

```shell
// 此处以 ssh 为例
$ git clone git@github.com:FHangH/Cpp-Learn-AddressBook_Clion.git

//运行结果（这样就 clone Git仓库到本地了）
PS C:\Users\Admin\Desktop\Test> git clone git@github.com:FHangH/Cpp-Learn-AddressBook_Clion.git
Cloning into 'Cpp-Learn-AddressBook_Clion'...
remote: Enumerating objects: 65, done.
Receiving objects:  10% (7/65) (65/65), done.
remote: Compressing objects: 100% (55/55), done.
remote: Total 65 (delta 7), reused 65 (delta 7), pack-reused 0
Receiving objects: 100% (65/65), 739.90 KiB | 127.00 KiB/s, done.
Resolving deltas: 100% (7/7), done.
```



### 8. gitignore文件

- 用来上传本地项目到远程仓库时，过滤掉一些不需要上传的文件

```shell
// 生成 .gitignore 文件
$ touch .gitignore

// powershell 里生成 .gitignore 的方法
$ new-item .gitignore
```

- 提供一个现成的 .gitignore 文件的开源库链接

- [gitignore]: https://github.com/FHangH/gitignore

- 使用方法：

  1. 确定自己上传的项目类型
  2. 在上述开源链接中找到对应项目类型的 .gitignore 文件
  3. 复制里面的内容
  4. 粘贴到自己项目中创建的 .gitignore 文件中，保存



##### 8.1 个人常用的 .gitignore 文件



###### 8.1.1 *UnrealEnigne.gitignore*

```
# Visual Studio 2015 user specific files
.vs/

# Compiled Object files
*.slo
*.lo
*.o
*.obj

# Precompiled Headers
*.gch
*.pch

# Compiled Dynamic libraries
*.so
*.dylib
*.dll

# Fortran module files
*.mod

# Compiled Static libraries
*.lai
*.la
*.a
*.lib

# Executables
*.exe
*.out
*.app
*.ipa

# These project files can be generated by the engine
*.xcodeproj
*.xcworkspace
*.sln
*.suo
*.opensdf
*.sdf
*.VC.db
*.VC.opendb

# Precompiled Assets
SourceArt/**/*.png
SourceArt/**/*.tga

# Binary Files
Binaries/*
Plugins/*/Binaries/*

# Builds
Build/*

# Whitelist PakBlacklist-<BuildConfiguration>.txt files
!Build/*/
Build/*/**
!Build/*/PakBlacklist*.txt

# Don't ignore icon files in Build
!Build/**/*.ico

# Built data for maps
*_BuiltData.uasset

# Configuration files generated by the Editor
Saved/*

# Compiled source files for the engine to use
Intermediate/*
Plugins/*/Intermediate/*

# Cache files for the editor to use
DerivedDataCache/*
```



###### 8.1.2 *Unity.gitignore*

```
# This .gitignore file should be placed at the root of your Unity project directory
#
# Get latest from https://github.com/github/gitignore/blob/master/Unity.gitignore
#
/[Ll]ibrary/
/[Tt]emp/
/[Oo]bj/
/[Bb]uild/
/[Bb]uilds/
/[Ll]ogs/
/[Uu]ser[Ss]ettings/

# MemoryCaptures can get excessive in size.
# They also could contain extremely sensitive data
/[Mm]emoryCaptures/

# Asset meta data should only be ignored when the corresponding asset is also ignored
!/[Aa]ssets/**/*.meta

# Uncomment this line if you wish to ignore the asset store tools plugin
# /[Aa]ssets/AssetStoreTools*

# Autogenerated Jetbrains Rider plugin
/[Aa]ssets/Plugins/Editor/JetBrains*

# Visual Studio cache directory
.vs/

# Gradle cache directory
.gradle/

# Autogenerated VS/MD/Consulo solution and project files
ExportedObj/
.consulo/
*.csproj
*.unityproj
*.sln
*.suo
*.tmp
*.user
*.userprefs
*.pidb
*.booproj
*.svd
*.pdb
*.mdb
*.opendb
*.VC.db

# Unity3D generated meta files
*.pidb.meta
*.pdb.meta
*.mdb.meta

# Unity3D generated file on crash reports
sysinfo.txt

# Builds
*.apk
*.aab
*.unitypackage

# Crashlytics generated file
crashlytics-build.properties

# Packed Addressables
/[Aa]ssets/[Aa]ddressable[Aa]ssets[Dd]ata/*/*.bin*

# Temporary auto-generated Android Assets
/[Aa]ssets/[Ss]treamingAssets/aa.meta
/[Aa]ssets/[Ss]treamingAssets/aa/*
```



###### 8.1.3 *VisualStudio.gitignore*

```
## Ignore Visual Studio temporary files, build results, and
## files generated by popular Visual Studio add-ons.
##
## Get latest from https://github.com/github/gitignore/blob/master/VisualStudio.gitignore

# User-specific files
*.rsuser
*.suo
*.user
*.userosscache
*.sln.docstates

# User-specific files (MonoDevelop/Xamarin Studio)
*.userprefs

# Mono auto generated files
mono_crash.*

# Build results
[Dd]ebug/
[Dd]ebugPublic/
[Rr]elease/
[Rr]eleases/
x64/
x86/
[Ww][Ii][Nn]32/
[Aa][Rr][Mm]/
[Aa][Rr][Mm]64/
bld/
[Bb]in/
[Oo]bj/
[Ll]og/
[Ll]ogs/

# Visual Studio 2015/2017 cache/options directory
.vs/
# Uncomment if you have tasks that create the project's static files in wwwroot
#wwwroot/

# Visual Studio 2017 auto generated files
Generated\ Files/

# MSTest test Results
[Tt]est[Rr]esult*/
[Bb]uild[Ll]og.*

# NUnit
*.VisualState.xml
TestResult.xml
nunit-*.xml

# Build Results of an ATL Project
[Dd]ebugPS/
[Rr]eleasePS/
dlldata.c

# Benchmark Results
BenchmarkDotNet.Artifacts/

# .NET Core
project.lock.json
project.fragment.lock.json
artifacts/

# ASP.NET Scaffolding
ScaffoldingReadMe.txt

# StyleCop
StyleCopReport.xml

# Files built by Visual Studio
*_i.c
*_p.c
*_h.h
*.ilk
*.meta
*.obj
*.iobj
*.pch
*.pdb
*.ipdb
*.pgc
*.pgd
*.rsp
*.sbr
*.tlb
*.tli
*.tlh
*.tmp
*.tmp_proj
*_wpftmp.csproj
*.log
*.vspscc
*.vssscc
.builds
*.pidb
*.svclog
*.scc

# Chutzpah Test files
_Chutzpah*

# Visual C++ cache files
ipch/
*.aps
*.ncb
*.opendb
*.opensdf
*.sdf
*.cachefile
*.VC.db
*.VC.VC.opendb

# Visual Studio profiler
*.psess
*.vsp
*.vspx
*.sap

# Visual Studio Trace Files
*.e2e

# TFS 2012 Local Workspace
$tf/

# Guidance Automation Toolkit
*.gpState

# ReSharper is a .NET coding add-in
_ReSharper*/
*.[Rr]e[Ss]harper
*.DotSettings.user

# TeamCity is a build add-in
_TeamCity*

# DotCover is a Code Coverage Tool
*.dotCover

# AxoCover is a Code Coverage Tool
.axoCover/*
!.axoCover/settings.json

# Coverlet is a free, cross platform Code Coverage Tool
coverage*[.json, .xml, .info]

# Visual Studio code coverage results
*.coverage
*.coveragexml

# NCrunch
_NCrunch_*
.*crunch*.local.xml
nCrunchTemp_*

# MightyMoose
*.mm.*
AutoTest.Net/

# Web workbench (sass)
.sass-cache/

# Installshield output folder
[Ee]xpress/

# DocProject is a documentation generator add-in
DocProject/buildhelp/
DocProject/Help/*.HxT
DocProject/Help/*.HxC
DocProject/Help/*.hhc
DocProject/Help/*.hhk
DocProject/Help/*.hhp
DocProject/Help/Html2
DocProject/Help/html

# Click-Once directory
publish/

# Publish Web Output
*.[Pp]ublish.xml
*.azurePubxml
# Note: Comment the next line if you want to checkin your web deploy settings,
# but database connection strings (with potential passwords) will be unencrypted
*.pubxml
*.publishproj

# Microsoft Azure Web App publish settings. Comment the next line if you want to
# checkin your Azure Web App publish settings, but sensitive information contained
# in these scripts will be unencrypted
PublishScripts/

# NuGet Packages
*.nupkg
# NuGet Symbol Packages
*.snupkg
# The packages folder can be ignored because of Package Restore
**/[Pp]ackages/*
# except build/, which is used as an MSBuild target.
!**/[Pp]ackages/build/
# Uncomment if necessary however generally it will be regenerated when needed
#!**/[Pp]ackages/repositories.config
# NuGet v3's project.json files produces more ignorable files
*.nuget.props
*.nuget.targets

# Microsoft Azure Build Output
csx/
*.build.csdef

# Microsoft Azure Emulator
ecf/
rcf/

# Windows Store app package directories and files
AppPackages/
BundleArtifacts/
Package.StoreAssociation.xml
_pkginfo.txt
*.appx
*.appxbundle
*.appxupload

# Visual Studio cache files
# files ending in .cache can be ignored
*.[Cc]ache
# but keep track of directories ending in .cache
!?*.[Cc]ache/

# Others
ClientBin/
~$*
*~
*.dbmdl
*.dbproj.schemaview
*.jfm
*.pfx
*.publishsettings
orleans.codegen.cs

# Including strong name files can present a security risk
# (https://github.com/github/gitignore/pull/2483#issue-259490424)
#*.snk

# Since there are multiple workflows, uncomment next line to ignore bower_components
# (https://github.com/github/gitignore/pull/1529#issuecomment-104372622)
#bower_components/

# RIA/Silverlight projects
Generated_Code/

# Backup & report files from converting an old project file
# to a newer Visual Studio version. Backup files are not needed,
# because we have git ;-)
_UpgradeReport_Files/
Backup*/
UpgradeLog*.XML
UpgradeLog*.htm
ServiceFabricBackup/
*.rptproj.bak

# SQL Server files
*.mdf
*.ldf
*.ndf

# Business Intelligence projects
*.rdl.data
*.bim.layout
*.bim_*.settings
*.rptproj.rsuser
*- [Bb]ackup.rdl
*- [Bb]ackup ([0-9]).rdl
*- [Bb]ackup ([0-9][0-9]).rdl

# Microsoft Fakes
FakesAssemblies/

# GhostDoc plugin setting file
*.GhostDoc.xml

# Node.js Tools for Visual Studio
.ntvs_analysis.dat
node_modules/

# Visual Studio 6 build log
*.plg

# Visual Studio 6 workspace options file
*.opt

# Visual Studio 6 auto-generated workspace file (contains which files were open etc.)
*.vbw

# Visual Studio LightSwitch build output
**/*.HTMLClient/GeneratedArtifacts
**/*.DesktopClient/GeneratedArtifacts
**/*.DesktopClient/ModelManifest.xml
**/*.Server/GeneratedArtifacts
**/*.Server/ModelManifest.xml
_Pvt_Extensions

# Paket dependency manager
.paket/paket.exe
paket-files/

# FAKE - F# Make
.fake/

# CodeRush personal settings
.cr/personal

# Python Tools for Visual Studio (PTVS)
__pycache__/
*.pyc

# Cake - Uncomment if you are using it
# tools/**
# !tools/packages.config

# Tabs Studio
*.tss

# Telerik's JustMock configuration file
*.jmconfig

# BizTalk build output
*.btp.cs
*.btm.cs
*.odx.cs
*.xsd.cs

# OpenCover UI analysis results
OpenCover/

# Azure Stream Analytics local run output
ASALocalRun/

# MSBuild Binary and Structured Log
*.binlog

# NVidia Nsight GPU debugger configuration file
*.nvuser

# MFractors (Xamarin productivity tool) working folder
.mfractor/

# Local History for Visual Studio
.localhistory/

# BeatPulse healthcheck temp database
healthchecksdb

# Backup folder for Package Reference Convert tool in Visual Studio 2017
MigrationBackup/

# Ionide (cross platform F# VS Code tools) working folder
.ionide/

# Fody - auto-generated XML schema
FodyWeavers.xsd
```



###### 8.1.4 *C++.gitignore*

```
# Prerequisites
*.d

# Compiled Object files
*.slo
*.lo
*.o
*.obj

# Precompiled Headers
*.gch
*.pch

# Compiled Dynamic libraries
*.so
*.dylib
*.dll

# Fortran module files
*.mod
*.smod

# Compiled Static libraries
*.lai
*.la
*.a
*.lib

# Executables
*.exe
*.out
*.app
```



###### 8.1.5 *C.gitignore*

```
# Prerequisites
*.d

# Object files
*.o
*.ko
*.obj
*.elf

# Linker output
*.ilk
*.map
*.exp

# Precompiled Headers
*.gch
*.pch

# Libraries
*.lib
*.a
*.la
*.lo

# Shared objects (inc. Windows DLLs)
*.dll
*.so
*.so.*
*.dylib

# Executables
*.exe
*.out
*.app
*.i*86
*.x86_64
*.hex

# Debug files
*.dSYM/
*.su
*.idb
*.pdb

# Kernel Module Compile Results
*.mod*
*.cmd
.tmp_versions/
modules.order
Module.symvers
Mkfile.old
dkms.conf
```



###### 8.1.6 *CMake.gitignore*

```
CMakeLists.txt.user
CMakeCache.txt
CMakeFiles
CMakeScripts
Testing
Makefile
cmake_install.cmake
install_manifest.txt
compile_commands.json
CTestTestfile.cmake
_deps
```

# MultiPlayer-RPG



:::danger 游戏介绍

- 游戏名：**代号-OP**
- 开发人：**Fangh**
- 项目启动时间：**2022-11-12 | 09:30**
- 项目完成时间：****

:::



[toc]



### 0. 项目日志

:::tip 项目日志

**2022-11-12 | 09:30**：项目开始

**2022-11-17 | 09:00**：开始设计部分数据库表

**2022-11-18 | 10:30**：大概完善部分数据库表

**2022-11-22 | 19:35**：项目转移至UE5.1

**2022-11-26 | 10:30**：基本完成登录和注册界面的逻辑

**2022-11-26 | 16:00**：基本搭建好游戏主要逻辑架构

**2022-11-26 | 16:30**：开始设计项目结构和记录项目API

**2022-12-15 | 16:00**：重新修改数据表

**2023-01-28 | 09:30**：解决了玩家进入服务器地图后，UI无法获取数据的bug

**2023-01-31 | 09:30**：完善了项目开放打包迭代测试的流程，从原来接近两小时的测试方式，压缩到一分钟左右

**2023-01-31 | 21:00**：完成了三大核心模块的前两个模块

**2023-02-07 | 18:57**：完成了PropsInfo数据表的设计

**2023-02-08 | 16:00**：完成了注册检测用户是否已经存在的判断

**2023-02-08 | 17:00**：修改了UseInfo数据表，加入用户是否已经登录的字段

**2023-02-08 | 18:50**：完成了用户登录判断是否已经登录的功能提示

:::



### 1. 需求分析



:::tip 游戏大致功能分析

- 数据库
- 用户登录注册
- 弱联机类型
- 开放世界
- 动物
- 联机副本
- 大世界联机
- 加入玩家大世界
- 加入玩家副本
- 聊天功能
- 成就系统
- 多装备(职业切换)
- 玩家等级
- 武器(职业)等级
- 合成系统
- 大世界AI -- 普通AI -- 精英AI -- BossAI
- 副本AI -- 普通AI -- 精英AI -- BossAI
- 部署专用服务器

:::



#### 1.1 数据库



**数据库**：`MultiPlayerRPG`

**数据表**：

- `UserInfo`

- `PlayerInfo`
- `WeaponInfo`
- `PropsInfo`



#### 1.2 用户登录注册



:::warning

- 在云服务器完成
- 在数据库录入和读取
- 通过数据库插件和服务端交互

:::



1. 本地用户注册->RPC服务端->服务端将数据记入服务器数据库
2. 本地用户登录->RPC服务端->验证是否登录->读取服务器数据库->验证信息->RPC本地用户进入世界



#### 1.3 数值规划



- 玩家最大等级：10
- C >> B：PropsB*10
- B >> A：PropsA*20
- A >> S：PropsS*40



### 2. 项目准备



#### 2.1 开发工具



云服务器：

- 部署专用服务器
- MySQL

UE4.27.2 二进制版：

- 开发游戏项目

UE4.27.2 源码版：

- 打包发布游戏项目

RiderForUnreal：

- UE4 C++编写/编译

插件：

- FH_MySQL插件



#### 2.2 项目资源



模型：

- 角色模型
- 角色武器模型
- 动物模型
- 场景模型
- 动物模型
- AI模型

动画：

- 角色基本动画
- 角色武器(职业)动画
- 动物动画
- AI动画

音效：

- 角色音效
- 武器(职业)音效
- 世界音效
- 副本音效
- 动物音效
- AI音效

特效：

- 场景特效
- 武器(职业)特效
- AI特效
- 副本特效



#### 2.3 开发标准



##### 2.3.1 命名标准



- **类名**
  - `GameMode_`
  - `PlayerController_`
- **变量名**：`fh_`
- **对象名**：`Fh_`
  - 虚幻类对象名：`Fh_XXX虚幻类型`，`Fh_XXXComponent`
- **函数名**：`FH_`
- **单播委托**：
  - 委托名：`FHDelegateXXX`
  - 委托变量：`FHDG_XXX`
  - 动态委托名：`FHDynamicDelegateXXX`
  - 动态委托变量：`FHDDG_XXX`
- **多播委托**：
  - 委托名：`FHDelegateMulXXX`
  - 委托变量：`FHDGM_XXX`
  - 动态委托名：`FHDynamicDelegateMulXXX`
  - 动态委托变量：`FHDDGM_XXX`



#### 2.4 打包测试



测试流程：

- `Unreal Rider`中修改完代码，完成编译
- 进入虚幻引擎，使用`项目启动程序`进行打包
- 添加打包方案：`WindowsServer`，`常规烘培`，`勾选所有需要烘焙的地图`，`高级设置：打包到一个Unreal Pak内，使用迭代烘焙打包`，`不部署`
- 完成`WindowsServer`打包后，通过创建快捷键的方式，`-log`形式启动
- 进入虚幻引擎中，关卡播放设置为`Standalone`，通过`open level IP:7777`的方式测试游戏，测试结果和完全打包`WindowsServer`和`Windows`效果一致

优点：打包速度快，测试结果真实可靠，快速迭代项目流程



### 3. 数据表设计



>- `UserInfo`
>- `PlayerInfo`
>- `WeaponInfo`
>- `PlayerChat`
>- `PropsInfo`
>- `DelegateInfo`
>- `BoxInfo`
>- `MaterialInfo`
>



#### 3.1 UserInfo



**UserInfo结构**

| 字段名       | 类型        | 自增 | 不为空 | PK / FK |
| ------------ | ----------- | ---- | ------ | ------- |
| UserID       | int         | Y    | Y      | PK      |
| UserName     | varchar(30) |      | Y      |         |
| UserEmail    | varchar(50) |      | Y      |         |
| UserPassword | varchar(20) |      | Y      |         |
| UserIsLogin  | int         |      | Y      |         |



**UserInfo表**

| UserID | UserName | UserEmail        | UserPassword | UserIsLogin |
| ------ | -------- | ---------------- | ------------ | ----------- |
| 1000   | fang     | 752972182@qq.com | 752972182    | 0           |
| 1001   | qq       | 123456789@qq.com | 123456789    | 1           |



**UserInfo.sql**

```sql
create table UserInfo(
	UserID int not null auto_increment primary key,
    UserName varchar(30) not null,
    UserEmail varchar(50) not null,
    UserPassWord varchar(20) not null,
    UserIsLogin int not null
) ENGINE=INNODB DEFAULT CHARSET=utf8 auto_increment=1000;
```





#### 3.2 PlayerInfo



**PlayerInfo**

| 字段名               | 类型        | 自增 | 不为空 | PK / FK |
| -------------------- | ----------- | ---- | ------ | ------- |
| UserID               | int         |      | Y      | PK      |
| PlayerGrade          | int         |      | Y      |         |
| PlayerMaxHealth      | float       |      | Y      |         |
| PlayerCurHealth      | float       |      | Y      |         |
| PlayerDamage         | float       |      | Y      |         |
| PlayerCritRate       | float       |      | Y      |         |
| PlayerCritMultiplier | float       |      | Y      |         |
| PlayerDefense        | float       |      | Y      |         |
| PlayerLocation       | varchar(30) |      | Y      |         |
| PlayerColor          | enum        |      | Y      |         |
| PlayerCurExp         | float       |      | Y      |         |
| PlayerNeedExp        | float       |      | Y      |         |
| PlayerCurWeaponSlot  | int         |      | Y      |         |
| PlayerWeaponIDSet    | varchar(30) |      | Y      |         |



**PlayerInfo表**

| UserID | PlayerGrade | PlayerMaxHealth | PlayerCurHealth | PlayerDamage | PlayerCritRate        | PlayerCritMultiplier   | PlayerDefense   | PlayerLocation  | PlayerColor | PlayerCurExp | PlayerNeedExp   | PlayerCurWeaponSlot | PlayerWeaponIDSet |
| ------ | ----------- | --------------- | --------------- | ------------ | --------------------- | ---------------------- | --------------- | --------------- | ----------- | ------------ | --------------- | ------------------- | ----------------- |
| 1000   | 1           | PlayerGrade*100 | 100.0           | PlayerGrade  | 0.09+PlayerGrade*0.01 | 1.0+(PlayerGrade/10.0) | 9.0+PlayerGrade | (0.0, 0.0, 0.0) | WHITE       | 0.0          | PlayerGrade*3.0 | 0                   | (0,0)             |
| 1001   | 2           | PlayerGrade*100 | 200.0           | PlayerGrade  | 0.09+PlayerGrade*0.01 | 1.0+(PlayerGrade/10.0) | 9.0+PlayerGrade | (0.0, 0.0, 0.0) | BLUE        | 0.0          | PlayerGrade*3.0 | 1                   | (1,2)             |



**PlayerInfo.sql**

```sql
create table PlayerInfo(
    UserID int not null primary key,
    PlayerGrade int not null,
	PlayerMaxHealth float not null,
    PlayerCurHealth float not null,
    PlayerDamage float not null,
    PlayerCritRate float not null,
    PlayerCritMultiplier float not null,
    PlayerDefense float not null,
    PlayerLocation varchar(30) not null,
    PlayerColor enum('WHITE', 'BLUE', 'RED', 'YELLOW', 'GREEN', 'BLACK', 'PINK', 'GOLD', 'SILVER') not null,
    PlayerCurExp float not null, 
    PlayerNeedExp float not null,
    PlayerCurWeaponSlot int not null,
    PlayerWeaponIDSet varchar(30) not null
) ENGINE=INNODB DEFAULT CHARSET=utf8;
```



#### 3.3 WeaponInfo



**WeaponInfo**

| 字段名               | 类型  | 自增 | 不为空 | PK / FK |
| -------------------- | ----- | ---- | ------ | ------- |
| WeaponID             | int   | Y    | Y      | PK      |
| UserID               | int   |      | Y      |         |
| WeaponQuality        | enum  |      | Y      |         |
| WeaponDamage         | float |      | Y      |         |
| WeaponCritRate       | float |      | Y      |         |
| WeaponCritMultiplier | float |      | Y      |         |
| WeaponType           | enum  |      | Y      |         |



**WeaponInfo表**

| WeaponID | UserID | WeaponQuality | WeaponDamage | WeaponCritRate | WeaponCritMultiplier | WeaponType |
| -------- | ------ | ------------- | ------------ | -------------- | -------------------- | ---------- |
| 1        | 1000   | C             | 3.0          | 10.0           | 0.2                  | SWORD      |
| 2        | 1000   | C             | 1.0          | 3.5            | 0.1                  | GUN        |
| 3        | 1001   | B             | 6.0          | 15.0           | 0.4                  | SWORD      |
| 4        | 1001   | B             | 2.0          | 7.0            | 0.2                  | GUN        |
| 5        | 1002   | A             | 12.0         | 20.0           | 0.8                  | SWORD      |
| 6        | 1002   | A             | 4.0          | 10.0           | 0.4                  | GUN        |
| 7        | 1003   | S             | 24.0         | 25.0           | 1.6                  | SWORD      |
| 8        | 1003   | S             | 8.0          | 13.0           | 0.8                  | GUN        |



**WeaponInfo.sql**

```sql
create table WeaponInfo(
    WeaponID int not null auto_increment primary key,
    UserID int not null,
    WeaponQuality enum('S','A','B','C') not null,
    WeaponDamage float not null,
    WeaponCritRate float not null,
    WeaponCritMultiplier float not null,
    WeaponType enum('SWORD','GUN') not null
) ENGINE=INNODB DEFAULT CHARSET=utf8 auto_increment=1;
```



#### 3.4 PropsInfo



**PropsInfo**

| 字段名 | 类型 | 自增 | 不为空 | PK / FK |
| ------ | ---- | ---- | ------ | ------- |
| UserID | int  |      | Y      | PK      |
| PropsB | int  |      | Y      |         |
| PropsA | int  |      | Y      |         |
| PropsS | int  |      | Y      |         |



**PropsInfo表**

| UserID | PropsB | PropsA | PropsS |
| ------ | ------ | ------ | ------ |
| 1000   | 0      | 0      | 0      |
| 1001   | 1      | 2      | 3      |



**PropsInfo.sql**

```sql
create table PropsInfo(
    UserID int not null primary key,
    PropsB int not null,
    PropsA int not null,
    PropsS int not null
) ENGINE=INNODB DEFAULT CHARSET=utf8;
```





### 4. 功能分析



#### 4.1 用户登录注册



##### 4.1.2 用户登录



:::tip 功能设计

- 在关卡**Map_Start**中设置**GameModeBase_Start**
- 项目全局设置为**GameInstanceRPG**
- **GameInstanceRPG**中创建并保存**Fh_UI_Start**，**Fh_MySQlConnector**
- **PlayerController_Start**调用**GameInstanceRPG**中的**Fh_UI_Start**和**Fh_MySQlConnector**
- 通过**PlayerController_Start**本地输入账号和密码
- 调用**GameInstanceRPG**中的**Fh_MySQlConnector**获取登录信息
- 调用**PlayerController_FH**调用**FH_CheckPutLoginInfo**，将登录信息提交到数据库，得到用户密码，判定是否可以登录

:::



##### 4.1.3 用户注册



:::tip 功能设计

- 大部分功能同**用户登录**一致
- 调用**PlayerController_FH**调用**FH_CheckPutRegInfo**，将注册信息提交到数据库，判断是否成功
- 成功则主界面切换到**用户登录**界面

:::





### 5. 项目结构-API



#### 5.1 项目结构



>MultiPlayerRPG
>
>>Plugins
>>
>>Source



##### 5.1.1 插件源码



**Plugins**

- **Source**
  - **FH_MySQL**
    - **Private**
      - BPFuncLib_FHSQL.cpp
      - FH_ConnectionObject.cpp
      - FH_MySQL.cpp
    - **Public**
      - BPFuncLib_FHSQL.h
      - FH_ConnectionObject.h
      - FH_MySQL.h



##### 5.1.2 项目源码



**Source**

- **MultiPlayerRPG**
  - **Data**
    - FFhMySQLConnectorData.cpp
    - FFhMySQLConnectorData.h
  - **GamePlay**
    - **Character**
      - Character_FH.cpp
      - Character_FH.h
    - **GameInstance**
      - GameInstanceRPG.cpp
      - GameInstanceRPG.h
    - **GameMode**
      - GameMode_MainLevel.cpp
      - GameMode_MainLevel.h
      - GameModeBase_Start.cpp
      - GameModeBase_Start.h
    - **GameState**
      - GameState_FH.cpp
      - GameState_FH.h
    - **PlayerController**
      - PlayerController_FH.cpp
      - PlayerController_FH.h
      - PlayerController_Start.cpp
      - PlayerController_Start.h
    - **PlayerState**
      - PlayerState_FH.cpp
      - PlayerState_FH.h
    - **UI**
      - UI_MainLevel.cpp
      - UI_MainLevel.h
      - UI_Start.cpp
      - UI_Start.h
  - **Utils**
    - MySqlUtil.cpp
    - MySqlUtil.h



#### 5.2 项目API





# 异步执行

虚幻C++中，如何异步执行一个方法

在虚幻C++中，可以使用`AsyncTask`类来异步执行一个方法。`AsyncTask`类可以在后台线程中执行任务，而不会影响主线程的性能。使用`AsyncTask`类可以让你的应用程序更加响应式，并且可以更有效地利用多核处理器。使用`AsyncTask`类可以简化异步任务的实现，并且可以更好地控制任务的执行。

下面是一个使用`AsyncTask`类来异步执行一个方法的示例：

```cpp
class MyTask : public AsyncTask
{
public:
    MyTask() {}
    ~MyTask() {}
    void DoWork()
    {
        // Do something here
    }
};

MyTask* task = new MyTask();
task->StartBackgroundTask();
```

在上面的示例中，我们创建了一个`MyTask`类，并实现了`DoWork`方法。然后，我们创建了一个`MyTask`对象，并调用`StartBackgroundTask`方法来异步执行`DoWork`方法。

这个DoWork是虚函数吗？task->StartBackgroundTask()最终回回调DoWork()？

是的，`DoWork`是一个虚函数，`StartBackgroundTask`方法会回调`DoWork`方法，从而实现异步执行。

我如何在别的类中使用这个异步类

你可以在别的类中使用`AsyncTask`类，只需要在该类中声明一个`MyTask`类的对象，然后调用`StartBackgroundTask`方法来异步执行`DoWork`方法即可。

我想将别的类的某个方法，传递到异步类中执行

可以通过使用lambda表达式来实现，例如：

```cpp
class MyTask : public AsyncTask
{
public:
    MyTask(std::function<void()> func) : func(func) {}
    ~MyTask() {}
    void DoWork()
    {
        func();
    }
private:
    std::function<void()> func;
};

MyClass myClass;
MyTask* task = new MyTask([&myClass](){ myClass.DoSomething(); });
task->StartBackgroundTask();
```

在上面的示例中，我们创建了一个`MyTask`类，并将`MyClass`的`DoSomething`方法传递给`MyTask`类的构造函数。然后，我们调用`StartBackgroundTask`方法来异步执行`DoSomething`方法。



在虚幻引擎中，可以使用 `AsyncTask` 宏来创建异步任务。该宏可以将一个 Lambda 表达式放入指定的线程中执行。以下是一个示例代码，用于在游戏线程以外的线程中异步执行 `MyFunction` 方法：

```c++
void MyFunction();

void MyClass::MyFunctionAsync()
{
    AsyncTask(ENamedThreads::AnyBackgroundThreadNormalTask, [this]()
    {
        MyFunction();
    });
}
```

在上面的代码中，`MyFunctionAsync` 方法是一个公共方法，它将在游戏线程以外的线程中异步执行 `MyFunction` 方法。在 `AsyncTask` 宏中，我们指定了一个 `ENamedThreads` 枚举值，该值指定了我们要将 Lambda 表达式放入哪个线程中执行。在这个例子中，我们使用了 `ENamedThreads::AnyBackgroundThreadNormalTask`，它表示我们将在任何后台线程中执行该任务。

在 Lambda 表达式中，我们调用了 `MyFunction` 方法，这个方法将在异步任务中执行。请注意，在异步任务中，我们不能直接访问 `this` 指针，因为它可能不在异步任务的上下文中。如果需要访问 `this` 指针，可以将其传递给 Lambda 表达式，如上面的代码所示。

# MultiPlayer-RPG-OpenLevel



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

**2022-11-17 | 9:00**：开始设计部分数据库表

**2022-11-18 | 10:30**：大概完善部分数据库表

**2022-11-26 | 10:30**：基本完成登录和注册界面的逻辑

**2022-11-26 | 16:00**：借本搭建好游戏主要逻辑架构

**2022-11-26 | 16:30**：开始设计项目结构和记录项目API

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

- `PlayerInfo`
- `PlayerChat`
- `PropsInfo`
- `DelegateInfo`
- `BoxInfo`
- `MaterialInfo`



#### 1.2 用户登录注册



:::warning

- 在云服务器完成
- 在数据库录入和读取
- 通过数据库插件和服务端交互

:::



1. 本地用户注册->RPC服务端->服务端将数据记入服务器数据库
2. 本地用户登录->RPC服务端->验证是否登录->读取服务器数据库->验证信息->RPC本地用户进入世界



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



**UserInfo表**

| UserID | UserName | UserEmail        | UserPassword |
| ------ | -------- | ---------------- | ------------ |
| 1000   | fang     | 752972182@qq.com | 752972182    |
| 1001   | qq       | 123456789@qq.com | 123456789    |



**UserInfo.sql**

```sql
create table UserInfo(
	UserID int not null auto_increment primary key,
    UserName varchar(30) not null,
    UserEmail varchar(50) not null,
    UserPassWord varchar(20) not null
);
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
| PlayerWeaponSlotID1  | int         |      | Y      |         |
| PlayerWeaponSlotID2  | int         |      | Y      |         |
| PlayerWeaponSlotID3  | int         |      | Y      |         |



**PlayerInfo表**

| UserID | PlayerGrade | PlayerMaxHealth | PlayerCurHealth | PlayerDamage | PlayerCritRate        | PlayerCritMultiplier   | PlayerDefense   | PlayerLocation  | PlayerColor | PlayerCurExp | PlayerNeedExp   | PlayerCurWeaponSlot | PlayerWeaponSlotID1 | PlayerWeaponSlotID2 | PlayerWeaponSlotID3 |
| ------ | ----------- | --------------- | --------------- | ------------ | --------------------- | ---------------------- | --------------- | --------------- | ----------- | ------------ | --------------- | ------------------- | ------------------- | ------------------- | ------------------- |
| 1000   | 1           | PlayerGrade*100 | 100.0           | PlayerGrade  | 0.09+PlayerGrade*0.01 | 1.0+(PlayerGrade/10.0) | 9.0+PlayerGrade | (0.0, 0.0, 0.0) | WHITE       | 0.0          | PlayerGrade*3.0 | 1                   | 0                   | 1                   | 2                   |
| 1001   | 2           | PlayerGrade*100 | 200.0           | PlayerGrade  | 0.09+PlayerGrade*0.01 | 1.0+(PlayerGrade/10.0) | 9.0+PlayerGrade | (0.0, 0.0, 0.0) | BLUE        | 0.0          | PlayerGrade*3.0 | 2                   | 3                   | 4                   | 5                   |



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
    PlayerWeaponSlotID1 int not null,
    PlayerWeaponSlotID2 int not null,
    PlayerWeaponSlotID3 int not null
);
```



#### 3.3 WeaponInfo



**WeaponInfo**

| 字段名               | 类型        | 自增 | 不为空 | PK / FK |
| -------------------- | ----------- | ---- | ------ | ------- |
| WeaponID             | int         | Y    | Y      | PK      |
| UserID               | int         |      | Y      |         |
| WeaponName           | varchar(10) |      | Y      |         |
| WeaponQuality        | enum        |      | Y      |         |
| WeaponDamage         | float       |      | Y      |         |
| WeaponCritRate       | float       |      | Y      |         |
| WeaponCritMultiplier | float       |      | Y      |         |
| WeaponType           | enum        |      | Y      |         |



**WeaponInfo表**

| WeaponID | UserID | WeaponName | WeaponQuality | WeaponDamage | WeaponCritRate | WeaponCritMultiplier | WeaponType |
| -------- | ------ | ---------- | ------------- | ------------ | -------------- | -------------------- | ---------- |
| 0        | 1000   | sword      | A             | 30.0         | 10.0           | 0.5                  | SWORD      |
| 1        | 1001   | gun        | B             | 10.0         | 3.5            | 0.2                  | GUN        |



**WeaponInfo.sql**

```sql
create table WeaponInfo(
    WeaponID int not null primary key,
    UserID int not null,
	WeaponName varchar(10) not null,
    WeaponQuality enum('S','A','B','C') not null,
    WeaponDamage float not null,
    WeaponCritRate float not null,
    WeaponCritMultiplier float not null,
    WeaponType enum('SWORD','SPEAR','GUN','BOW','EPEE','GLOVES','SICKLE') not null
);
```





### 4. 功能分析



#### 4.1 用户登录注册



##### 4.1.2 用户登录



:::tip 功能设计

- 在关卡**Map_Start**中设置**GameModeBase_Start**
- 项目全局设置为**GameInstanceRPG**
- **GameInstanceRPG**中创建并保存**Fh_UI_Start**，**Fh_MySQlConnector**
- **PlayerController_FH**调用**GameInstanceRPG**中的**Fh_UI_Start**和**Fh_MySQlConnector**
- 通过**PlayerController_FH**本地输入账号和密码
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

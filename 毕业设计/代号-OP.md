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
| PlayerMaxHealth      | varchar(20) |      | Y      |         |
| PlayerCurHealth      | varchar(20) |      | Y      |         |
| PlayerGrade          | varchar(10) |      | Y      |         |
| PlayerDamage         | varchar(20) |      | Y      |         |
| PlayerCritRate       | varchar(10) |      | Y      |         |
| PlayerCritMultiplier | varchar(10) |      | Y      |         |
| PlayerDefense        | varchar(20) |      | Y      |         |
| UserID               | int         |      |        | FK      |



**PlayerInfo表**

| PlayerMaxHealth | PlayerCurHealth | PlayerGrade | PlayerDamage | PlayerCritRate | PlayerCritMultiplier | PlayerDefense | UserID |
| --------------- | --------------- | ----------- | ------------ | -------------- | -------------------- | ------------- | ------ |
| PlayerGrade*100 | 100             | 1           | PlayerGrade  | 9+PlayerGrade  | 1.0+(PlayerGrade/10) | 9+PlayerGrade | 1000   |
| PlayerGrade*100 | 200             | 2           | PlayerGrade  | 9+PlayerGrade  | 1.0+(PlayerGrade/10) | 9+PlayerGrade | 1001   |



**PlayerInfo.sql**

```sql
create table PlayerInfo(
	PlayerMaxHealth varchar(20) not null,
    PlayerCurHealth varchar(20) not null,
    PlayerGrade varchar(10) not null,
    PlayerDamage varchar(20) not null,
    PlayerCritRate varchar(10) not null,
    PlayerCritMultiplier varchar(10) not null,
    PlayerDefense varchar(20) not null,
    UserID int,
    foreign key(UserID) references UserInfo(UserID)
);
```



#### 3.3 WeaponInfo



**WeaponInfo**

| 字段名               | 类型         | 自增 | 不为空 | PK / FK |
| -------------------- | ------------ | ---- | ------ | ------- |
| WeaponName           | varchar(10)  |      | Y      |         |
| WeaponQuality        | enum         |      | Y      |         |
| WeaponDescribe       | varchar(100) |      | Y      |         |
| WeaponDamage         | varchar(20)  |      | Y      |         |
| WeaponCritRate       | varchar(10)  |      | Y      |         |
| WeaponCritMultiplier | varchar(10)  |      | Y      |         |
| WeaponType           | enum         |      | Y      |         |
| UserID               | int          |      |        | FK      |



**WeaponInfo表**

| WeaponName | WeaponQuality | WeaponDescribe | WeaponDamage | WeaponCritRate | WeaponCritMultiplier | WeaponType | UserID |
| ---------- | ------------- | -------------- | ------------ | -------------- | -------------------- | ---------- | ------ |
| sword      | A             | a sword        | 30           | 10             | 0.5                  | Sword      | 1000   |
| gun        | B             | a gun          | 10           | 3.5            | 0.2                  | Gun        | 1001   |



**WeaponInfo.sql**

```sql
create table WeaponInfo(
	WeaponName varchar(10) not null,
    WeaponQuality enum('S','A','B','C') not null,
    WeaponDescribe varchar(100) not null,
    WeaponDamage varchar(20) not null,
    WeaponCritRate varchar(10) not null,
    WeaponCritMultiplier varchar(10) not null,
    WeaponType enum('Sword','Spear','Gun','Bow','Epee','Gloves','Sickle') not null,
    UserID int,
    foreign key(UserID) references UserInfo(UserID)
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

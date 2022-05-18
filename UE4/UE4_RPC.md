# UE4 RPC



### 1. RPC 架构



#### 1.1 架构简介



- 一个服务器， 一个或多个客户端
- 不能信任客户端，所有重要信息都需要通过服务器验证
- `Listen Server`监听服务器和`Dedicated Server`专有服务器





#### 1.2 基本GamePlay结构



- **Server**
  - `GameMode` 仅存在于Server
  - `GameState` 同时存在与Server和Client
  - `Pawn_Server`, `Pawn_A`, `Pawn_B` 同时存在与Server和Client
  - `PlayerState_Server`, `PlayerState_A`, `PlayerState_B` 同时存在与Server和Client
  - `PlayerController_Server` 仅存在于Server
  - `PlayerController_A`, `PlayerController_B` 同时存在与Server和各自所拥有的Client
  - `GameInstance`, `UI` 同时存在与Server和各自所拥有的Client
- **Client**
  - 每个客户端都有独立的`PlayerController`和`UI`
  - 其余与`Server`同时拥有





### 2. Listen Server



#### 2.1 Replication



- **说明：**
  - 信息从服务端同步到客户端`(单向)`
  - `Actor`及其派生类才有`Replication`的能力





#### 2.2 Replication类型



- **类型：**
  1. `Actor Replication`
  2. `Property Replication`
  3. `Component Replication`



- 在服务端进行操作：

  ```c++
  if (HasAuthority())
  {
  	TestNum = 999.f;
  }
  ```

  



##### 2.2.1 Actor Replication



- **两层意义：**
  1. 服务端生成，客户端也跟着生成`(在服务端生成一个replication对象)`
  2. 当前`Actor`的所有属性复制，组件复制，RPC的总开关



- **开启Replication**

  - 蓝图：勾选`Replicates`

  - C++：在构造函数中实现

    ```c++
    bReplicates = true;
    SetReplicates(true);
    
    其中Set要慢一点
    ```





##### 2.2.2 Property Replication



- **开启方式：**

  - 前提：`Actor Replication`是`true`

  - 蓝图：`Replication`设置为`Replicated`

  - C++：

    ```c++
    UPROPERTY(Replicated)
    float TestNum;
    ```

    ```c++
    #include "Net/UnrealNetwork.h"
    
    void AMyActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
    {
    	Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    	
    	DOREPLIFETIME(AMyActor, TestNum);
    }
    ```

  - `AMyActor`是需要复制属性的类

  - `TestNum`是待复制的属性





##### 2.2.3 Rep Notify



- **说明：**变量设为`Rep_Notify`，当变量发生复制时，`服务端`和`收到的客户端`都可以`调用`一个`自定义的函数`
- **注意：**`C++中`，自定义函数`仅在客户端`中调用



- **设置方式：**

  - 蓝图：`Replication`设置为`RepNotify`，会自动生成`OnRep_属性()`函数

  - C++：

    ```c++
    UPROPERTY(ReplicatedUsing=OnRep_TestNum, BlueprintReadWrite)
    int TestNum;
    
    UFUNCTION()
    void OnRep_TestNum();
    ```

    ```c++
    #include "MyActor.h"
    #include "Kismet/KismetSystemLibrary.h"
    #include "Net/UnrealNetwork.h"
    
    AMyActor::AMyActor()
    {
    	PrimaryActorTick.bCanEverTick = true;
    
    	bReplicates = true;
    }
    
    void AMyActor::BeginPlay()
    {
    	Super::BeginPlay();
    }
    
    void AMyActor::Tick(float DeltaTime)
    {
    	Super::Tick(DeltaTime);
    
    	if (HasAuthority())
    	{
    		++TestNum;
    	}
    }
    
    void AMyActor::OnRep_TestNum()
    {
    	if (HasAuthority())
    	{
    		UKismetSystemLibrary::PrintString(GetWorld(), FString::Printf(TEXT("%d"), TestNum));
    	}
    	else
    	{
    		UKismetSystemLibrary::PrintString(GetWorld(), FString::Printf(TEXT("%d"), TestNum));
    	}
    }
    
    void AMyActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
    {
    	Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    
    	DOREPLIFETIME(AMyActor, TestNum);
    }
    ```






#### 2.3 OwnerShip



- **作用：**
- RPC需要确定哪个客户端将执行于客户端的RPC
  
- Actor复制与连接相关性
  
- 在涉及所有者时的Actor属性复制条件



- **说明：**
  - 当 Pawn Actor 被 PlayerController 拥有时，它们的所有者将是它们所拥有的 PlayerController。
  - 在此期间，它们归 PlayerController 的连接所有。
  - Pawn 仅在同时由 PlayerController 拥有/拥有时由该连接拥有。
  - 因此，一旦 PlayerController 不再拥有 Pawn，Pawn 就不再由连接拥有。



- **注意：**
  - 连接所有权对于 RPC 之类的东西很重要，因为当在 Actor 上调用 RPC 函数时，除非 RPC 被标记为多播，否则它需要知道在哪个客户端上执行该 RPC。
  - 它通过查找拥有的连接来确定将 RPC 发送到的连接。



- **使用：**

  - 蓝图

    - 设置
      - SpawnActor中又Owner可以引用
      - SetOwner
    - 改变
      - Possess(OnPossess > PossessedBy > SetOwner), UnPossess
    - 获得
      - GetOwner

  - C++

    - 设置

      ```c++
      //SpawnParameters 内可以设置 Onwer
      GetWorld()->SpawnActor(Class, const* UserTransformPtr, const SpawnParameters);
      	
      // SetOwner
      SetOwner(NewOwner);
      ```

    - 获取

      ```c++
      GetOwner();
      ```

    

    

    

#### 2.4 Actor Role



- **分类：**
  1. Authority 权威
  2. Simulated Proxy 模拟代理
  3. Autonomous Proxy 自主代理



- **基本结构：**A， B， C `A为房主`
  - ServerA：A, B, C `Authority`
  - ClientB：A, C `Simulated`, B `Autonomous`
  - ClentC: A, B `Simulated`, C `Autonomous`



- **说明：**当涉及到复制时，actor 有两个很重要的属性。`Role`和`RemoteRole`。
- **作用：**
  - 谁对演员有权力
  - 演员是否被复制
  - 复制模式



- 服务器复制到客户端的条件：`Role == ROLE_Authority`and `RemoteRole == ROLE_SimulatedProxy`or `ROLE_AutonomousProxy`







### 3. RPC介绍



- 类似于函数调用，不过不一定是在本地执行



- 可以实现：
  - 客户端调用，服务端执行
  - 服务端调用，客户端执行
- 不可以有返回值
- 默认不可靠（可以设置成Reliable）





#### 3.1 RPC设置



**蓝图：**

- CustomEvent 的 Replicates 选项设置为其中一个
  - Run On Server
  - Run On Owning Client
  - Net MultiCast
- 要勾选`Reliable`



**C++：**

- 将一个自定义的函数声明为RPC，需要添加反射`UFUNCTION()`
  - Server
  - Client
  - NetMultiCast
- 额外添加`Reliable`





#### 3.2 RPC要求和注意



要使 RPC 完全正常运行，需要满足一些要求：

- 它们必须从 Actors 中调用
- 必须复制 Actor
- 如果从服务器调用 RPC 以在客户端上执行，则只有实际拥有该 Actor 的客户端将执行该函数
- 如果从客户端调用 RPC 以在服务器上执行，则客户端必须拥有正在调用 RPC 的 Actor
- 多播 RPC 是一个例外：
  - 如果从服务器调用它们，服务器将在本地执行它们以及在所有当前连接的客户端上执行它们
  - 如果从客户端调用，它们只会在本地执行，不会在服务器上执行
  - 目前，我们有一个简单的多播事件限制机制：多播函数在给定 Actor 的网络更新周期内不会复制超过两次。从长远来看，我们希望对此进行改进，并为跨渠道流量管理和节流提供更好的支持

​	

**从Server调用RPC**

| 演员所有权           | 未复制         | `NetMulticast`             | `Server`       | `Client`                 |
| -------------------- | -------------- | -------------------------- | -------------- | ------------------------ |
| **客户拥有的演员**   | 在服务器上运行 | 在服务器和所有客户端上运行 | 在服务器上运行 | 在演员拥有的客户端上运行 |
| **服务器拥有的演员** | 在服务器上运行 | 在服务器和所有客户端上运行 | 在服务器上运行 | 在服务器上运行           |
| **无名演员**         | 在服务器上运行 | 在服务器和所有客户端上运行 | 在服务器上运行 | 在服务器上运行           |



**从Client调用RPC**

| 演员所有权           | 未复制             | `NetMulticast`     | `Server`       | `Client`           |
| -------------------- | ------------------ | ------------------ | -------------- | ------------------ |
| **由调用客户端拥有** | 在调用客户端时运行 | 在调用客户端时运行 | 在服务器上运行 | 在调用客户端时运行 |
| **由不同的客户拥有** | 在调用客户端时运行 | 在调用客户端时运行 | 掉落           | 在调用客户端时运行 |
| **服务器拥有的演员** | 在调用客户端时运行 | 在调用客户端时运行 | 掉落           | 在调用客户端时运行 |
| **无名演员**         | 在调用客户端时运行 | 在调用客户端时运行 | 掉落           | 在调用客户端时运行 |


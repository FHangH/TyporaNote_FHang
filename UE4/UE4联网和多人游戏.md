# UE4 联网和多人游戏



**复制Replication**：在客户端服务器间同步数据和调用程序的过程



### 1. 网络概述



#### 1.1 尽早规划多人游戏



- 若项目可能需要多人游戏功能，则从项目开始阶段起，构建所有gameplay时都应将多人游戏功能考虑在内，便于进行调试和维护，且支持单人
- 若是单人游戏改为多人，重构无网络情况下编译的基本代码需要梳理整个项目，几乎所有gameplay都需要重新编写





#### 1.2 客户端-服务器模型



- **单人游戏或本地多人游戏**：

  - 游戏在 `独立`游戏上本地运行。玩家将输入连接到一台计算机，直接控制其上所有内容，而包括Actor、场景和各玩家的用户界面在内的所有游戏项目均存在于这台本地机器上

    

- **网络多人游戏**：

  - 虚幻引擎使用 `客户端-服务器`模型

  - 网络中的一台计算机作为 `服务器` 主持多人游戏会话，而所有其他玩家的计算机作为 `客户端` 连接到该服务器。然后，服务器与连接的客户端分享游戏状态信息，并提供一种客户端之间通信的方法

    

    **客户端和服务端**：

    - 在网络多人游戏中，游戏将在服务器（1）与多个与之连接的客户端（2）之间进行。服务器处理gameplay，客户端向用户显示游戏
    - 服务器是多人游戏实际发生的地方
    - 客户端会远程控制其在服务器上各自拥有的 `Pawn`， 发送过程调用以使其执行游戏操作
    - 服务器不会将视觉效果直接流送至客户端显示器。服务器会将游戏状态信息 **复制** 到各客户端，告知应存在的Actor、此类Actor的行为，以及不同变量应拥有的值
    - 客户端使用此信息，对服务器上正在发生的情况进行高度模拟





##### 1.2.1 客户端-服务器游戏范例



- 分别以`本地游戏`和`多人游戏`为范例，说明`GamePlay`的处理逻辑
- 本地游戏：玩家1
- 多人游戏：玩家2



| 本地游戏                                                     | 网络游戏                                                     |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| ![本地运行范例](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Overview/LocalMultiplayerExample.jpg) | ![网络运行范例2](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Overview/ClientServerExample.jpg) |
| `玩家1按下输入以发射武器`<br />玩家1的Pawn将发射其当前武器以响应此操作。<br />玩家1的武器生成发射物，并播放附带音效和视觉效果。 | `玩家1在本地机器上按下输入以发射武器`<br />玩家1的本地Pawn将武器发射命令传送给服务器上对应的Pawn。<br />玩家1在服务器上的武器生成发射物。<br />服务器告知所有连接的客户端各自生成玩家1发射物的副本。<br />玩家1在服务器上的武器告知所有客户端播放武器发射音效和视觉效果。 |
| `玩家1的发射物从武器中射出并前移`                            | `玩家1的发射物从在服务器上的武器中射出并前移`<br />此时，服务器告知所有客户端复制玩家1发射物发生的移动，因此各客户端上的玩家1发射物便相应移动。 |
| `玩家1的发射物撞击玩家2的Pawn`<br />碰撞将触发摧毁玩家1发射物的函数，对玩家2的Pawn造成伤害，并播放附带音效和视觉效果。<br />玩家2播放画面效果，作为对伤害的响应。 | `玩家1在服务器上的发射物撞击玩家2的Pawn`<br />碰撞触发摧毁服务器上玩家1发射物的函数。<br />服务器自动告知所有客户端各自摧毁玩家1发射物副本。<br />碰撞触发告知所有客户端播放附带碰撞音效和视觉效果的函数。<br />玩家2在服务器上的Pawn承受发射物碰撞造成的伤害。<br />玩家2在服务器上的Pawn告知玩家2客户端播放画面效果，作为对伤害的响应。 |



- **网络游戏中**：
  - 此类交互发生在多个不同场景，这一过程将在基础游戏交互（碰撞、移动、伤害）、美化效果（视觉效果和音效）和私人玩家信息（HUD更新）间进行划分。这三者各自与网络中的特定机器或机组关联
  - 此信息的复制过程并非完全自动，游戏编程时须指定要复制的信息和接收副本的机器
  - 主要的难点在于选择应复制的信息及方式，以向所有玩家提供一致的游戏体验，同时需最小化信息复制量，尽可能减少网络带宽占用率







#### 1.3 基本网络概念



##### 1.3.1 网络模式和服务器类型



- **网络模式**：
  - 描述了计算机与网络多人游戏会话的关系
  - 游戏实例可采用以下任意网络模式



| 网络模式     | 说明                                                         |
| :----------- | ------------------------------------------------------------ |
| `独立`       | 游戏作为服务器运行，不接受远程客户端连接<br />参与游戏的玩家必须为本地玩家<br />此模式用于单人游戏和本地多人游戏<br />其将运行本地玩家适用的服务器逻辑和客户端逻辑 |
| `客户端`     | 游戏作为网络多人游戏会话中与服务器连接的客户端运行<br />其不会运行服务器逻辑 |
| `聆听服务器` | 游戏作为主持网络多人游戏会话的服务器运行<br />其接受远程客户端中的连接，且直接在服务器上拥有本地玩家<br />此模式通常用于临时合作和竞技多人游戏 |
| `专属服务器` | 游戏作为主持网络多人游戏会话的服务器运行<br />其接受远程客户端中的连接，但无本地玩家，因此为了高效运行，其将废弃图形、音效、输入和其他面向玩家的功能<br />此模式常用于需要更固定、安全和大型多人功能的游戏 |



- 独立游戏服务器可同时作为服务器和客户端，为多人游戏创建的逻辑可在无需额外工作的情况下，在单人游戏中运行





##### 1.3.2 Actor复制



- **描述**：
  - 复制是指在网络会话中的不同机器间复制游戏状态信息
  - 若正确设置复制，将可同步不同机器的游戏实例
  - 在C++ Actor类中设置 `bReplicates` 变量，或将Actor蓝图的 `复制（Replicates）`设置设为 `true`，可启用给定类的Actor复制



###### 1.3.2.1 常见复制功能



| 复制功能                | 说明                                                         |
| :---------------------- | :----------------------------------------------------------- |
| **创建和销毁**          | 服务器上生成复制Actor的授权版本时，其会在所有连接客户端上自动生成远程代理。<br />其之后会将信息复制到这些远程代理。<br />若销毁授权Actor，则将自动销毁所有连接客户端上的远程代理。 |
| **移动复制**            | 若授权Actor启用了 **复制移动**，或将C++中的 `bReplicateMovement` 设为 `true`，其将自动复制位置、旋转和速度。 |
| **变量复制**            | 在指定为复制变量的值变更时，其将自动从授权Actor复制到其远程代理。 |
| **组件复制**            | Actor组件复制为其所属Actor的一部分。<br />组件内指定为复制变量将复制，而组件内调用的RPC将与Actor类中调用的RPC保持一致。 |
| **远程过程调用（RPC）** | RPC是传输到网络游戏中特定机器的特殊函数。<br />无论初始调用RPC的是哪台机器，其的实现仅在目标机器上运行。<br />此类RPC可指定为服务器（仅在服务器上运行）、客户端（仅在Actor的拥有客户端上运行）或NetMulticast（在连接会话的所有机器上运行，包括服务器）。 |



- 虽然创建、销毁和移动等常见使用可自动处理，但即使启用复制，其他所有gameplay功能也不会默认自动复制
- 必须根据游戏的需求明确指定要复制的变量和函数



`Actor`、`Pawn`和`角色`的部分常用功能不会复制：

- **骨架网格体** 和 **静态网格体** 组件
- **材质**
- **动画蓝图**
- **粒子系统**
- **音效发射器**
- **物理对象**





###### 1.3.2.2 网络角色和授权



- **描述**：
  - Actor的 **网络角色** 将决定网络游戏期间控制Actor的机器
  - **授权** Actor被认为可控制Actor的状态，并可将信息复制到网络多人游戏会话中的其他机器上
  - **远程代理** 是该Actor在远程机器上的副本，其将接收授权Actor中的复制信息，由 **Local Role** 和 **Remote Role** 变量进行追踪



| 网络角色     | 说明                                                         |
| :----------- | :----------------------------------------------------------- |
| **无**       | Actor在网络游戏中无角色，不会复制。                          |
| **授权**     | Actor为授权状态，会将其信息复制到其他机器上的远程代理。      |
| **模拟代理** | Actor为远程代理，由另一台机器上的授权Actor完全控制。<br />网络游戏中如拾取物、发射物或交互对象等多数Actor将在远程客户端上显示为模拟代理。 |
| **自主代理** | Actor为远程代理，能够本地执行部分功能，但会接收授权Actor中的矫正。<br />自主代理通常为玩家直接控制的actor所保留，如pawn。 |



- 虚幻引擎使用的默认模型是 **服务器授权**，意味着服务器对游戏状态固定具有权限，而信息固定从服务器复制到客户端
- 服务器上的Actor应具有授权的本地角色，而其在远程客户端上的对应Actor应具有模拟或自主代理的本地角色





###### 1.3.2.3 客户端拥有权



- 特定客户端机器上的 **PlayerController** 拥有网络游戏中的pawn
- Pawn调用纯客户端函数时，其将无视调用函数的机器，而仅指向拥有玩家的机器
- 将Actor的 **Owner** 变量设为特定Pawn，则通关关联，该Actor属于该Pawn的拥有客户端，并将纯客户端函数指向其拥有者的机器
- C++中的 `IsLocallyControlled` 函数，或蓝图中的 **Is Locally Controlled** 节点，以决定Pawn是否在其拥有客户端上
- 由于构造期间Pawn可能未指定控制器，因此避免在自定义Pawn类的构造函数中使用 `IsLocallyControlled`





###### 1.3.2.4 相关性和优先级



- **相关性**：用于决定是否需要在多人游戏期间复制Actor

  - 复制期间将剔除被认为不相关的actor，此操作可节约带宽，以便相关Actor可更加高效地复制
  - 若Actor未被玩家拥有，且不在玩家附近，将其被视为不相关，而不会进行复制
  - 不相关Actor会存在于服务器上，且会影响授权游戏状态，但在玩家靠近前不会向客户端发送信息
  - 覆盖 `IsNetRelevantFor` 函数以手动控制相关性，并可使用 `NetCullDistanceSquared` 属性决定成为相关Actor所需距离

  

- **优先级**：有时在游戏单帧内，没有足够带宽供复制所有相关Actor，因此，Actor拥有 **优先级(Priority)** 值，用于决定优先复制的Actor

  - Pawn和PlayerController的 `NetPriority` 默认为 **3.0**，从而使其成为游戏中最高优先级的Actor，基础Actor的 `NetPriority` 为 **1.0**
  - Actor在被复制前经历的时间越久，每次成功通过时所处的优先级便越高





##### 1.3.3 变量复制



- **描述**：授权Actor上复制变量的值变更时，其信息将自动从授权Actor发送到连接会话的远程代理

  - C++中使用对应 `UPROPERTY` 宏内的 `Replicated` 或 `ReplicateUsing` 说明符

  - 蓝图的细节面板中将它们指定为已复制，可将复制添加到变量和对象引用





###### 1.3.3.1 RepNotify



- **描述**：可指定在Actor成功接收特定变量的复制信息时要调用的 **RepNotify** 函数

  - RepNotify仅在变量更新时本地触发
  - 触发gameplay逻辑响应授权Actor上的变量更改时，使用RepNotify可减少开销
  - 在C++中使用变量的 `UPROPERTY` 宏的 `ReplicatedUsing` 说明符可访问此功能
  - 蓝图中变量的复制设置以使用RepNotify

  

- **补充**：由于RepNotify可添加到需复制的变量中，而无需考虑其他gameplay功能，创建额外网络调用时刻节约大量带宽，因此RepNotify比RPC或复制函数更加好用







##### 1.3.4 远程过程调用(RPC)



- **描述**：远程过程调用也称为复制函数
  - 可在任何机器上进行调用，但会指示其的实现在与网络会话连接的特定机器上发生
  - 有三种类型的RPC



| RPC类型          | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| **Server**       | 仅在主持游戏的服务器上调用。                                 |
| **Client**       | 仅在拥有该函数所属Actor的客户端上调用。若Actor无拥有连接，将不会执行此逻辑。 |
| **NetMulticast** | 在与服务器连接的所有客户端及服务器本身上调用。               |



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
- 其代码将在代码实现中使用后缀 `_Implementation`



- **ExampleClass.h**

  ```c++
  //服务器RPC MyFunction的声明。
  UFUNCTION(Server, Reliable, WithValidation)
  void MyFunction(int myInt);
  ```

- **ExampleClass.cpp**

  ```c++
  //服务器RPC MyFunction的实现。
  void AExampleClass::MyFunction_Implementation(int myInt)
  {
      //游戏代码在此。
  }
  ```

  



###### 1.3.4.1 可靠性



- **描述**：必须将RPC指定为 **可靠** 或 **不可靠**

  - **不可靠**：

    - 不可靠RPC无法保证必会到达预定目的地，但其发送速度和频率高于可靠的RPC
    - 最适用于对gameplay而言不重要或经常调用的函数
    - 例如，由于Actor移动每帧都可能变换，因此使用不可靠RPC复制该Actor移动

  - **可靠**：

    - 可靠的RPC保证到达预定目的地，并在成功接收之前一直保留在队列中
    - 最适合用于对gameplay很关键或者不经常调用的函数
    - 相关例子包括碰撞事件、武器发射的开始或结束，或生成Actor

    

- **注意**：

  - 滥用可靠函数可能导致其队列溢出，此操作将强制断开连接
  - 若逐帧调用复制函数，应将其设为不可靠
  - 若拥有与玩家输入绑定的可靠函数，应限制玩家调用该函数的频率





###### 1.3.4.2 验证



- **描述**：`WithValidation` 说明符表明除函数的实现外，还有可验证传入函数调用的数据的函数
  - 此验证函数与其负责的函数使用同一签名，但其将返回布尔而非原本返回值
  - 若返回 `true`，则其允许执行RPC的 `Implementation`；若返回 `false`，则防止执行



- **ExampleClass.cpp**

  ```c++
  //服务器RPC MyFunction的验证
  bool AExampleClass::MyFunction_Validation(int myInt)
  {
      /* 
          若myInt的值为负，建议不允许运行MyFunction_Implementation。 
          因此仅在myInt大于零时返回true。
      */
      return myInt >= 0;
  }
  ```







### 2. 提示和深入阅读



- **描述**：戏中实现高效、稳定多人游戏系统的基本指南





#### 2.1 基本复制Actor清单



- 将Actor的复制设置设为True
- 若复制Actor需要移动，将复制移动（Replicates Movement）设为True
- 生成或销毁复制Actor时，确保在服务器上执行该操作
- 设置必须在机器间共享的变量，以便进行复制。这通常适用于以gameplay为基础的变量
- 尽量使用虚幻引擎的预制移动组件，其已针对复制进行构建
- 若使用服务器授权模型，需确保玩家可执行的新操作均由服务器函数触发





#### 2.2 网络提示



- 尽可能少用RPC或复制蓝图函数。在合适情况下改用RepNotify
- 组播函数会导致会话中各连接客户端的额外网络流量，需尤其少用
- 若能保证非复制函数仅在服务器上执行，则服务器RPC中无需包含纯服务器逻辑
- 将可靠RPC绑定到玩家输入时需谨慎。玩家可能会快速反复点击按钮，导致可靠RPC队列溢出。应采取措施限制玩家激活此项的频率
- 若游戏频繁调用RPC或复制函数，如tick时，则应将其设为不可靠
- 部分函数可重复使用。调用其响应游戏逻辑，然后调用其响应RepNotify，确保客户端和服务器拥有并列执行即可
- 检查Actor的网络角色可查看其是否为 `ROLE_Authority`。此方法适用于过滤函数中的执行，该函数同时在服务器和客户端上激活
- 使用C++中的 `IsLocallyControlled` 函数或蓝图中的Is Locally Controlled函数，可检查Pawn是否受本地控制。基于执行是否与拥有客户端相关来过滤函数时，此方法十分有用
- 构造期间Pawn可能未被指定控制器，因此避免在构造函数脚本中使用 `IsLocallyControlled`







### 3. Actor 复制



Actor 主要通过两种方式进行更新：

- 属性更新
- RPC （远程过程调用）
- 属性更新和 RPC 的主要区别在于，属性可以在发生变化时随时自动更新，而 RPC 只能在被执行时获得调用更新



复制例子：Actor 的健康值

- 当健康值发生变化时，您通常都希望告知客户端。如果健康值没有变化，则不会发送任何数据
- 即使这个属性没有变化（因此不消耗任何带宽），它仍然会消耗 CPU 资源来判断这个值是否发生变化
- 适合那些经常变化的属性



RPC例子：同一场爆炸

- 可以以位置和半径为参数的 RPC 函数，同时在每次发生爆炸时调用它
- 也可以将此存储为一组属性，通过同步的方式将其传达给客户端
- 这种做法会损失一些效率，因为爆炸出现的频繁度也许不会高得有必要将它们作为属性





#### 3.1 组件复制



##### 3.1.1 组件复制介绍



介绍：

- 虚幻引擎 4 支持组件复制
- 大多数组件都不会复制
- 多数游戏逻辑都是在 Actor 类和组件中完成，而它们 通常只代表了构成 Actor 的零散部分
- 实际复制的是 Actor 中的游戏逻辑，而这样做的结果，有时会调用/更改组件
- 有些情况下，组件本身的属性或事件必须要直接复制
- 一旦复制了 Actor，它就可以复制自身组件
- 这些组件 可以按 Actor 的方式复制属性和 RPC
- 组件必须以 Actor 的方式实施 `::GetLifetimeReplicatedProps (...)` 函数



组件复制涉及两大类组件：

- 静态组件：一种是随 Actor 一起创建的组件

  - 在客户端或服务器上生成 所属 Actor 时，这些组件也会同时生成，与组件是否被复制无关
  - 服务器不会告知客户端显式生成这些组件
  - 静态组件无需通过复制存在于客户端
  - 只有在属性或事件需要在服务器和客户端之间自动同步时，才需要进行复制

  

- 动态组件：运行时在服务器上生成的组件种，其创建和删除操作也将被复制到客户端

  - 运行方式与 Actor 极为一致
  - 动态组件需通过复制的方式存在于所有客户端
  - 客户端可以生成自己的本地非复制组件，当那些在服务器上触发的 属性或事件需要自动同步到客户端时，才会出现复制行为





##### 3.1.2 使用方式



- 在组件上设置属性和 RPC 的过程与 Actor 并无区别
- 将一个类设置为具有复本后，这些组件的实际实例也必须经过设置后才能复制



- C++

- 调用 `AActorComponent::SetIsReplicated(true)` 即可

- 如果组件是一个默认子对象，就应当在生成组件之后通过类构造函数来完成此调用

- 示例：

  ```c++
  ACharacter::ACharacter()
  {
      // Etc...
  
      CharacterMovement = CreateDefaultSubobject<UMovementComp_Character>(TEXT("CharMoveComp"));
      if (CharacterMovement)
      {
          CharacterMovement->UpdatedComponent = CapsuleComponent;
  
          CharacterMovement->GetNavAgentProperties()->bCanJump = true;
          CharacterMovement->GetNavAgentProperties()->bCanWalk = true;
          CharacterMovement->SetJumpAllowed(true);
          CharacterMovement->SetNetAddressable(); // Make DSO components net addressable
          CharacterMovement->SetIsReplicated(true); // Enable replication by default
  
      }
  }
  ```



- 蓝图

- 要进行静态蓝图组件复制，只需在组件默认设置中切换 **Replicates** 布尔变量

- 静态组件需要在客户端和服务器上隐式创建

- 并非所有组件都会如此显示，必须要支持某种复制形式才会显示

  ![components_checkbox.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/Components/components_checkbox.jpg)

- 通过动态生成的组件来实现这一点，可以调用 **SetIsReplicated** 函数

  ![components_function.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/Components/components_function.jpg)





##### 3.1.3 时间轴



- 时间轴必须通过其属性中的 Replicated 选项来启用复制
- 会将服务器控制的运行位置、速率和方向复制到客户端
- 大多数时间轴都无需复制
- 时间轴复本只应当在服务器上直接 操作 (start/stop etc)
- 客户端只应当查看运行位置的复本，而不应尝试改变时间轴本身
- 在进行复制更新的间歇，客户端将推测 运行位置





##### 3.1.4 带宽开销



- 复制组件时的资源开销是比较低的
- 复制的 Actor 中的每个组件都需要添加一个额外的 NetGUID（4 字节）"标头"和一个大约 1 字节的"标脚"（footer） 及其属性
- 在 CPU 层面上，基于 Actor 的属性复制与基于组件的复制之间应当有一个最小差异





##### 3.1.5 一般性子对象复制



- 所有 Actor 子对象都可以复制，而不只限于组件



- 对于希望复制非 ActorComponent 子对象的类，应当实施三种方法

  ```c++
  /** FActory 方法，用于对模板化 TobjectReplicator 类进行实例化，以便实现子对象复制 */
  virtual class FObjectReplicatorBase * InstantiateReplicatorForSubObject(UClass *SubobjClass);
  
  /** 能让 Actor 在其 Actor 通道上复制子对象的方法 */
  virtual bool ReplicateSubobjects(class UActorChannel *Channel, class FOutBunch *Bunch, FReplicationFlags *RepFlags);
  
  /** 通过复制来动态创建新的子对象时，在 Actor 上进行调用 */
  virtual void OnSubobjectCreatedFromReplication(UObject *NewSubobject);
  ```





###### 3.1.5.1 使用情形



- 能在 Actor 通道的层面上使用 UObject 和多态（polymorphism）
- 之前用于复杂数据结构的复制方法只适合那些 在 Actor 类中对类型进行静态定义的结构
- 利用子对象复制，建立一个道具栏系统，使其中的每个物品作为一个从基本道具栏类扩展而来的类， 也可以进行完整复制，同时无需让这些项成为 Actor（资源负担太大）





###### 3.1.5.2 优化



- 有很多子对象需要复制，Actor 只需了解哪些子对象（如存在）最近发生过变化且需要复制，从而节省了大量时间

- 通过访问器（accessor）函数来持续跟踪子对象的更改情况

- 所用的接口位于 UActorChannel 中

  ```c++
  bool KeyNeedsToReplicate(int32 ObjID, int32 RepKey);
  ```

- 该函数应当由 Actor 在其 `::ReplicateSubobjects` 实施中调用

- Actor 类可以设置一个任意的对象 ID 和复制键，供复制系统跟踪每个客户端

- 对象 ID 和复制键完全是任意指定的

- 对象 ID 仅用于引用"事情"

  - 可以是整个子对象列表、部分列表或单个对象

- 复制键同样可以任意指定

  - 可以是一个在对象 ID 跟踪变化时递增的计数器







#### 3.2 Actor及其所属连接



##### 3.2.1 连接



- 每个连接都有一个专门为其创建的 PlayerController
- 确定一个 actor 是否归某一连接所有，您可以查询这个 actor 最外围的所有者
- 所有者是一个 PlayerController，则这个 actor 同样归属于拥有 PlayerController 的那个连接





##### 3.2.2 确定连接



- 在确定所属连接方面，组件有一些特殊之处
  - 首先确定组件所有者，方法是遍历组件的"外链"，直到找出所属的 actor
  - 确定这个 actor 的所属连接，像上面那样继续下去
  - 连接所有权是以下情形中的重要因素：
    - RPC 需要确定哪个客户端将执行运行于客户端的 RPC
    - Actor 复制与连接相关性
    - 在涉及所有者时的 Actor 属性复制条件



##### 3.2.3 连接的作用



- 连接所有权对于 RPC 这样的机制至关重要，因为当您在 actor 上调用 RPC 函数时
- 除非 RPC 被标记为多播，否则就需要知道要在哪个客户端上执行该 RPC
- 它可以查找所属连接来确定将 RPC 发送到哪条连接



- 连接所有权会在 actor 复制期间使用，用于确定各个 actor 上有哪些连接获得了更新
- 对于那些将 bOnlyRelevantToOwner 设置为 true 的 actor，只有拥有此 actor 的连接才会接收这个 actor 的属性更新
- 默认情况下，所有 PlayerController 都设置了此标志，正因如此，客户端才只会收到它们拥有的 PlayerController 的更新
- 最主要的是防止玩家作弊和提高效率



- 对于那些要用到所有者的 [需要复制属性的情形](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/Actors/Properties/Conditions) 来说，连接所有权具有重要意义：
  - 当使用 `COND_OnlyOwner` 时，只有此 actor 的所有者才会收到这些属性更新
- 所属连接对那些作为自治代理的 actor（角色为 `ROLE_AutonomousProxy`）来说也很重要
  - 这些 actor 的角色会降级为 `ROLE_SimulatedProxy`，其属性则被复制到不拥有这些 actor 的连接中







#### 3.3 Actor相关性与优先级



##### 3.3.1 相关性



前提：

- 场景的规模可能非常大，在特定时刻某个玩家只能看到关卡中的一小部分 Actor
- 场景中的其他大多数 Actor 都不会被看到和听到， 对玩家也不会产生显著的影响
- 被服务器认为可见或能够影响客户端的 Actor 组会被视为该客户端的相关 Actor 组



虚幻引擎的网络代码中包含一处重要的带宽优化：

- 服务器只会让客户端知道其相关组内的 Actor



参照以下规则确定玩家的相关 Actor 组：在虚拟函数 `AActor::IsNetRelevantFor()` 中实施

- 如果 Actor 是 bAlwaysRelevant、归属于 Pawn 或 PlayerController、本身为 Pawn 或者 Pawn 是某些行为（如噪音或伤害）的发起者，则其具有相关性
- 如果 Actor 是 bNetUseOwnerRelevancy 且拥有一个所有者，则使用所有者的相关性
- 如果 Actor 是 bOnlyRelevantToOwner 且没有通过第一轮检查，则不具有相关性
- 如果 Actor 被附加到另一个 Actor 的骨架模型，它的相关性将取决于其所在基础的相关性
- 如果 Actor 是不可见的 (bHidden == *true*) 并且它的 Root Component 并没有碰撞，那么则不具有相关性
  - 如果没有 Root Component 的话，`AActor::IsNetRelevantFor()` 会记录一条警告，提示是否要将它设置为 bAlwaysRelevant=*true*
- 如果 AGameNetworkManager 被设置为使用基于距离的相关性，则只要 Actor 低于净剔除距离，即被视为具有相关性



注意：

- Pawn 和 PlayerController 将覆盖 `AActor::IsNetRelevantFor()` 并最终具有不同的相关性条件



缺点：

- 距离检查在遇到大型 Actor 时可能会出现漏报（尽管我们用了一些启发式方法来应对），也不能处理环境声音的吸收
- 相对于互联网的延迟和数据包丢失这些网络环境所固有的问题来说，这种近似法产生的错误就不那么明显了





##### 3.3.2 优先级设定



介绍：虚幻引擎采用了负载平衡技术来安排所有 Actor 的优先级，并根据它们对游戏的重要性为其分别提供一个公平的带宽份额



原理：

- 每个 Actor 都有一个名为 NetPriority 的浮点变量
- 变量的数值越大，Actor 相对于其他"同伴"的带宽就越多
- 和优先级为 1.0 的 Actor 相比，优先级是 2.0 的 Actor 可以得到两倍的更新频度
- 唯一影响优先顺序的就是它们的比值
- 所以无法通过提高所有优先级的数值来增加虚幻引擎的网络性能



例子：

- Actor = 1.0
- Matinee = 2.7
- Pawn = 3.0
- PlayerController = 3.0



使用：

- 计算 Actor 的当前优先级时使用了虚拟函数 `AActor::GetNetPriority()`
- 为避免出现`饥荒（starvation）`，`AActor::GetNetPriority()` 使用 Actor 上次复制后经过的时间 去乘以 `NetPriority`
- 同时，`GetNetPriority` 函数还考虑了 Actor 与观察者的相对位置以及两者之间的距离







#### 3.4 Actor复制流程详述



介绍：

- 大多数 actor 复制操作都发生在 `UNetDriver::ServerReplicateActors` 内
- 服务器将收集所有被认定与各个客户端相关的 actor，并发送那些自上次（已连接的）客户端更新后出现变化的所有属性





##### 3.4.1 复制流程



**复制连接流程**：指定了 actor 的更新方式、要调用的特定框架回调，以及在此过程中使用的特定属性

- `AActor::NetUpdateFrequency` - 用于确定 actor 的复制频度
- `AActor::PreReplication` - 在复制发生前调用
- `AActor::bOnlyRelevantToOwner` - 如果此 actor 仅复制到所有者，则值为 true
- `AActor::IsRelevancyOwnerFor` - 用于确定 bOnlyRelevantToOwner 为 true 时的相关性
- `AActor::IsNetRelevantFor` - 用于确定 bOnlyRelevantToOwner 为 false 时的相关性



**高级流程**：

- 循环每一个主动复制的 actor（`AActor::SetReplicates( true )`）

  - 确定这个 actor 是否在一开始出现休眠（`DORM_Initial`），如果是这样，则立即跳过
  - 通过检查 NetUpdateFrequency 的值来确定 actor 是否需要更新，如果不需要就跳过
  - 如果 `AActor::bOnlyRelevantToOwner` 为 true，则检查此 actor 的所属连接以寻找相关性（对所属连接的观察者调用 `AActor::IsRelevancyOwnerFor`），如果相关，则添加到此连接的已有相关列表
    - 此时，这个 actor 只会发送到单个连接
  - 对于任何通过这些初始检查的 actor，都将调用 `AActor::PreReplication`
    - PreReplication 可以让您决定是否针对连接来复制属性，这时要使用 `DOREPLIFETIME_ACTIVE_OVERRIDE`
  - 如果同过了以上步骤，则添加到所考虑的列表

  

- 对于每个连接：

  - 对于每个所考虑的上述 actor
    - 确定是否休眠
    - 是否还没有通道
      - 确定客户端是否加载了 actor 所处的场景
        - 如未加载则跳过
      - 针对连接调用 `AActor::IsNetRelevantFor`，以确定 actor 是否相关
        - 如不相关则跳过
  - 在归连接所有的相关列表上添加上述任意 actor
  - 这时，我们拥有了一个针对此连接的相关 actor 列表
  - 按照优先级对 actor 排序
  - 对于每个排序的 actor：
    - 如果连接没有加载此 actor 所在的关卡，则关闭通道（如存在）并继续
    - 每 1 秒钟调用一次 AActor::IsNetRelevantFor，确定 actor 是否与连接相关
    - 如果不相关的时间达到 5 秒钟，则关闭通道
    - 如果相关且没有通道打开，则立即打开一个通道
    - 如果此连接出现饱和
      - 对于剩下的 actor
        - 如果保持相关的时间不到 1 秒，则强制在下一时钟单位进行更新
        - 如果保持相关的时间超过 1 秒，则调用 `AActor::IsNetRelevantFor` 以确定是否应当在下一时钟单位更新
    - 对于通过了以上这几点的 actor，将调用 `UChannel::ReplicateActor` 将其复制到连接







##### 3.4.2 Actor复制到连接



流程：`UChannel::ReplicateActor` 将负责把 actor 及其所有组件复制到连接中

- 确定这是不是此 actor 通道打开后的第一次更新
  - 如果是，则将所需的特定信息（初始方位、旋转等）序列化
- 确定该连接是否拥有这个 actor
  - 如果没有，而且这个 actor 的角色是 `ROLE_AutonomousProxy`，则降级为 `ROLE_SimulatedProxy`
- 复制这个 actor 中已更改的属性
- 复制每个组件中已更改的属性
- 对于已经删除的组件，发送专门的删除命令







#### 3.5 Role和RemoteRole



介绍： Actor 的复制过程中，有两个属性扮演了重要角色，分别是 `Role` 和 `RemoteRole`



作用：有了这两个属性，可以知道

- 谁拥有 actor 的主控权
- actor 是否被复制
- 复制模式



前提：

- 首先一件要确定的事，就是谁拥有特定 actor 的主控权
- 确定当前运行的引擎实例是否有主控者，需要查看 Role 属性是否为 `ROLE_Authority`
  - 如果是，就表明这个运行中的引擎实例负责掌管此 actor（决定其是否被复制）
  - 如果 Role 是 `ROLE_Authority`，RemoteRole 是 `ROLE_SimulatedProxy` 或 `ROLE_AutonomousProxy`
  - 就说明这个引擎实例负责将此 actor 复制到远程连接



注意：

- 只有服务器能够向已连接的客户端同步 Actor （客户端永远都不能向服务器同步）
- *只有* 服务器才能看到 `Role == ROLE_Authority` 和 `RemoteRole == ROLE_SimulatedProxy` 或者 `ROLE_AutonomousProxy`





##### 3.5.1 Role/RemoteRole对调



对于不同的数值观察者，它们的 Role 和 RemoteRole 值可能发生对调



例子：

- 服务器上有这样的配置：

  - `Role == ROLE_Authority`
  - `RemoteRole == ROLE_SimulatedProxy`

- 客户端会将其识别为以下形式：

  - `Role == ROLE_SimulatedProxy`
  - `RemoteRole == ROLE_Authority`

  

- 这种情况是正常的，因为服务器要负责掌管 actor 并将其复制到客户端

- 而客户端只是接收更新，并在更新的间歇模拟 actor





##### 3.5.2 复制模式



前提：

- 服务器不会在每次更新时复制 actor
- 会消耗太多的带宽和 CPU 资源
- 服务器会按照 `AActor::NetUpdateFrequency` 属性指定的频度来复制 actor
- 因此在 actor 更新的间歇，会有一些时间数据被传递到客户端
- 会导致 actor 呈现出断续、不连贯的移动
- 弥补这个缺陷，客户端将在更新的间歇中模拟 actor



两种模拟：

- `ROLE_SimulatedProxy`

  - 标准的模拟途径，通常是根据上次获得的速率对移动进行推算
  - 服务器为特定的 actor 发送更新时，客户端将向着新的方位调整其位置，然后利用更新的间歇，根据由服务器发送的最近的速率值来继续移动 actor

  

- `ROLE_AutonomousProxy`
  - 这种模拟通常只用于 PlayerController 所拥有的 actor
  - 说明此 actor 会接收来自真人控制者的输入，所以在我们进行推算时，我们会有更多一些的信息，而且能使用真人输入内容来补足缺失的信息（而不是根据上次获得的速率来进行推算）







#### 3.6 RPC



简介：

- RPC （远程过程调用）是在本地调用但在其他机器（不同于执行调用的机器）上远程执行的函数
- 允许客户端或服务器通过网络连接相互发送消息



作用：

- 执行那些不可靠的暂时性/修饰性游戏事件
- 包括播放声音、生成粒子或产生其他临时效果 之类的事件，它们对于 Actor 的正常运作并不重要
- 在此之前，这些类型的事件往往要通过 Actor 属性进行复制





##### 3.6.1 使用RPC



- 声明：将一个函数声明为 RPC，您只需将 `Server`、`Client` 或 `NetMulticast` 关键字添加到 `UFUNCTION` 声明



- 例子1：要将某个函数声明为一个要在服务器上调用、但需要在客户端上执行的 RPC

  ```c++
  UFUNCTION( Client )
  void ClientRPCFunction();
  ```



- 例子2：将某个函数声明为一个要在客户端上调用、但需要在服务器上执行的 RPC，您可以采取类似的方法，但需要使用 `Server` 关键字

  ```c++
  UFUNCTION( Server )
  void ServerRPCFunction();
  ```



- 例子3：多播 RPC 可以从服务器调用，然后在服务器和当前连接的所有客户端上执行，需使用 `NetMulticast` 关键字

  ```c++
  UFUNCTION( NetMulticast )
  void MulticastRPCFunction();
  ```

- 注：多播 RPC 还可以从客户端调用，但这时就只能在本地执行





##### 3.6.2 快速提示



- 在函数的开头预置 `Client`、`Server` 或 `Multicast` 关键字
- 这是我们在内部所做的一个约定，用来告诉程序员所用的函数将分别在客户端、服务器或所有客户端上调用
- 事先确定该函数将在多人游戏会话期间被哪些机器调用





##### 3.6.3 要求和注意事项



前提：

- 它们必须从 Actor 上调用
- Actor 必须被复制
- 如果 RPC 是从服务器调用并在客户端上执行，则只有实际拥有这个 Actor 的客户端才会执行函数
- 如果 RPC 是从客户端调用并在服务器上执行，客户端就必须拥有调用 RPC 的 Actor
- 多播 RPC 则是个例外：
  - 如果它们是从服务器调用，服务器将在本地和所有已连接的客户端上执行它们
  - 如果它们是从客户端调用，则只在本地而非服务器上执行
  - 有一个简单的多播事件限制机制：`在特定 Actor 的网络更新期内，多播函数将不会复制两次以上`





###### 3.6.3.1 从服务器调用的RPC



| Actor 所有权           | 未复制         | `NetMulticast`             | `Server`       | `Client`                    |
| :--------------------- | :------------- | :------------------------- | :------------- | :-------------------------- |
| **Client-owned actor** | 在服务器上运行 | 在服务器和所有客户端上运行 | 在服务器上运行 | 在 actor 的所属客户端上运行 |
| **Server-owned actor** | 在服务器上运行 | 在服务器和所有客户端上运行 | 在服务器上运行 | 在服务器上运行              |
| **Unowned actor**      | 在服务器上运行 | 在服务器和所有客户端上运行 | 在服务器上运行 | 在服务器上运行              |





###### 3.6.3.2 从客户端调用的RPC



| Actor 所有权                    | 未复制                   | `NetMulticast`           | `Server`       | `Client`                 |
| :------------------------------ | :----------------------- | :----------------------- | :------------- | :----------------------- |
| **Owned by invoking client**    | 在执行调用的客户端上运行 | 在执行调用的客户端上运行 | 在服务器上运行 | 在执行调用的客户端上运行 |
| **Owned by a different client** | 在执行调用的客户端上运行 | 在执行调用的客户端上运行 | 丢弃           | 在执行调用的客户端上运行 |
| **Server-owned actor**          | 在执行调用的客户端上运行 | 在执行调用的客户端上运行 | 丢弃           | 在执行调用的客户端上运行 |
| **Unowned actor**               | 在执行调用的客户端上运行 | 在执行调用的客户端上运行 | 丢弃           | 在执行调用的客户端上运行 |





##### 3.6.4 可靠性



- 默认情况下，RPC 并不可靠

- 要确保在远程机器上执行 RPC 调用，可以指定 `Reliable` 关键字

  ```c++
  UFUNCTION( Client, Reliable )
  void ClientRPCFunction();
  ```





##### 3.6.5 蓝图



前提：

- 如果被标记为 RPC 的函数是从蓝图中调用，它们也会被复制
- 它们将遵循相同的规则，就像是从 C++ 调用一样
- 在此情况下，无法将函数动态标记为蓝图的 RPC
- 自定义事件可以从蓝图编辑器内部被标记为复制



使用：

- 使用此功能，您需要在您的事件图表中新建一个自定义事件

- 单击自定义事件并在详细信息视图中编辑复制设置

  ![RPC_BP.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/RPCs/RPC_BP.jpg)







##### 3.6.6 验证



使用验证的原因：

- 作为检测错误数据/输入的一个手段
- 如果 RPC 的验证函数检测到任何 参数存在问题，就会通知系统将发起 RPC 调用的客户端/服务器断开
- 会通知系统将发起 RPC 调用的客户端/服务器断开



例：

- 要为 RPC 声明一个验证函数，只需将 `WithValidation` 关键字添加到 `UFUNCTION` 声明语句

  ```c++
  UFUNCTION( Server, WithValidation )
  void SomeRPCFunction( int32 AddHealth );
  ```

- 然后在实施函数旁边加入验证函数

  ```c++
  bool SomeRPCFunction_Validate( int32 AddHealth )
  {
      if ( AddHealth > MAX_ADD_HEALTH )
      {
          return false;                       // This will disconnect the caller
      }
  return true;                              // This will allow the RPC to be called
  }
  
  void SomeRPCFunction_Implementation( int32 AddHealth )
  {
      Health += AddHealth;
  }
  ```



- 注：
  - 被添加到 UHT，以便要求客户端 -> 服务器 RPC 具有一个 _Validate 函数
  - 鼓励使用安全的服务器 RPC 函数，同时尽可能方便其他人 添加代码以检查所有参数，确保其符合所有已知的输入限制







#### 3.7 属性复制



说明：

- 每个Actor维护一个全属性列表，其中包含[`Replicated` 说明符](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/GameplayArchitecture/Properties/Specifiers)
- 每当复制的属性值发生变化时，服务器会向所有客户端发送更新
- 客户端会将其应用到Actor的本地版本上
- 这些更新只会来自服务器，客户端永远不会向服务器或其他客户端发送属性更新



注意：

- 不推荐在客户端上更改复制的变量值
- 该值将始终与服务器端的值不一致，直到服务器下一次侦测到变更并发送更新为止
- 如果服务器版本的属性不是经常更新，那客户端就需要等待很长时间才能被纠正



Tips：

- Actor属性复制可靠
- 意味着，Actor的客户端版本的属性最终将反映服务器上的值，但客户端不必接受服务器上某个属性的每一个单独变更



- 例：
  - 如果一个整数属性的值快速从100变成200，然后又变成了300
  - 客户端将最终接受一个值为300的变更，但客户端不一定会知道这个值曾经变成过200





##### 3.7.1 设置要复制的属性



- 复制属性：在定义属性的Actor类标头处，您需要确保`replicated`关键字作为`UPROPERTY`声明的参数之一

  ```c++
  class ENGINE_API AActor : public UObject
  {
      UPROPERTY( replicated )
      AActor * Owner;
  };
  ```



- 在Actor类的实现过程中，需要实现`GetLifetimeReplicatedProps`函数

  ```c++
  void AActor::GetLifetimeReplicatedProps( TArray< FLifetimeProperty > & OutLifetimeProps ) const
  {
      DOREPLIFETIME( AActor, Owner );
  }
  ```



- 在Actor的构造函数中，确保将`bReplicates`标记设置为`true`

  ```c++
  AActor::AActor( const class FPostConstructInitializeProperties & PCIP ) : Super( PCIP )
  { 
      bReplicates = true;
  }
  ```

- 注：对于当前实例化的Actor类型的每个副本，成员变量"Owner"现在将同步到所有连接的客户端（在本例中为基础Actor类）





##### 3.7.2 网络更新优化



###### 3.7.2.1 数据驱动型网络更新频率



- Actor将观察在其`NetUpdateFrequency`变量中设置的最大更新频率
- 通过在不太重要或不太频繁变化的Actor上降低该变量，网络更新可以变得更高效，同时在有限带宽的场景中可能会带来更流畅的游戏体验
- 常见的更新频率值为：
  - 重要且不可预知的Actor：射击游戏中由玩家控制的角色，为10（每0.1秒更新一次）
  - 对于行动缓慢的角色：
    - 合作类游戏中由AI控制的怪物，为5（每0.2秒更新一次）
    - 对于游戏进程不是很重要但仍通过网络同步的以及/或者由服务器端逻辑控制因而需要复制的后台Actor，为2（每0.5秒更新一次）





###### 3.7.2.2 自适应型网络更新频率



Tips：

- 在默认情况下，该功能是关闭的
- 将控制台变量 `net.UseAdaptiveNetUpdateFrequency` 设置到 `1` 可以将其激活



作用：

- 节省CPU周期，这些CPU周期通常会在没有任何实际更改的情况下多次尝试复制Actor而浪费掉
- 启用了该功能时，系统将根据各个Actor的更新是否有意义，动态调整其更新频率



- 有意义：
  - 初始化了Actor、添加或删除了子对象（即拥有的组件）
  - 更改了Actor上或其任何子对象上复制字段值的任何更新



- 每个Actor可能的更新速率范围由Actor本身的两个变量决定：`NetUpdateFrequency`和`MinNetUpdateFrequency`
- `NetUpdateFrequency`表示Actor每秒尝试更新自己的最大次数，而`MinNetUpdateFrequency`表示每秒尝试更新的最小次数
- 使用该功能可以大大提高复制性能





###### 3.7.2.3 更新频率降低算法



- 在更新尝试期间，Actor将确定最近一次有意义的更新发送到现在有多长时间，如果它们发送了有意义的更新，将记录新的时间
- 例1：
  - 进行更新的Actor超过2秒没有发送有意义的更新，那么它将开始降低更新频率
  - 在没有发送有意义的更新的情况下，更新频率将在7秒后达到最小
- 例2：
  - 更新延迟在0.1秒到0.6秒之间的Actor在3秒内没有任何有意义的更新
  - 那么它将在0.2秒内尝试下一次更新





###### 3.7.2.4 更新频率增加算法



- 在发送一个有意义的更新之后，Actor将安排下一个更新发生的时间，使其比前两次有意义的更新之间的时间短30%，并且处于最小更新频率与最大更新频率之间



- 如果Actor在两次有意义的更新之间恰好间隔了一秒，那么它会将下一次更新尝试安排在未来0.7秒
- 或者接近指定的最小与最大更新频率的时间
- 接下来每次有意义的更新，都将重复该计算，如果Actor开始频繁地进行数据或子对象更改，将快速缩短更新之间的时间







#### 3.8 条件属性复制



前提：

- 当属性被注册进行复制后，您将无法再取消注册（涉及到生存期）
- 因为要预制尽可能多的信息，以便针对同一组属性将某一工作分担给多个连接
- 可以节省大量的计算时间



说明：

- 默认情况下，每个复制属性都有一个内置条件：如果不发生变化就不会进行复制

- 为了加强对属性复制的控制，使用一个专门的宏来添加附加条件

  - 这个宏被称为 `DOREPLIFETIME_CONDITION`

  ```c++
  void AActor::GetLifetimeReplicatedProps( TArray< FLifetimeProperty > & OutLifetimeProps ) const
  {
      DOREPLIFETIME_CONDITION( AActor, ReplicatedMovement, COND_SimulatedOnly );
  }
  ```

- 传递给条件宏的 `COND_SimulatedOnly` 标志甚至可以在考虑复制属性前执行一次额外检查

- 这时，它只会复制到拥有此 actor 模拟复本的客户端



作用：

- 最明显的好处在于节省带宽
- 因为我们确信拥有此 actor 的自治代理版本的客户端无需了解这个属性
- 例：
  - 该客户端为了进行预测而直接设置了这一属性
  - 对于不接收该属性的客户端而言，服务器无需干涉这个客户端的本地复本



使用：

- 目前支持的条件，在 `Engine\Source\Runtime\CoreUObject\Public\UObject\CoreNet.h` 的 `ELifetimeCondition` 枚举中指定

| 条件                      | 说明                                                         |
| :------------------------ | :----------------------------------------------------------- |
| `COND_InitialOnly`        | 该属性仅在初始数据组尝试发送                                 |
| `COND_OwnerOnly`          | 该属性仅发送至 actor 的所有者                                |
| `COND_SkipOwner`          | 该属性将发送至除所有者之外的每个连接                         |
| `COND_SimulatedOnly`      | 该属性仅发送至模拟 actor                                     |
| `COND_AutonomousOnly`     | 该属性仅发送给自治 actor                                     |
| `COND_SimulatedOrPhysics` | 该属性将发送至模拟或 bRepPhysics actor                       |
| `COND_InitialOrOwner`     | 该属性将发送初始数据包，或者发送至 actor 所有者              |
| `COND_Custom`             | 该属性没有特定条件，但需要通过 SetCustomIsActiveOverride 得到开启/关闭能力 |



补充：

- 有一个名叫 `DOREPLIFETIME_ACTIVE_OVERRIDE` 的宏可以进行全面控制

- 利用任何定制条件来决定何时复制/不复制某个属性

- 需要注意的是：

  - 这种控制需针对每个 actor（而不是每条连接）逐一进行
  - 在定制条件中使用一个可根据连接而发生变化的状态，会存在一定的安全风险

  ```c++
  void AActor::PreReplication( IRepChangedPropertyTracker & ChangedPropertyTracker )
  {
      DOREPLIFETIME_ACTIVE_OVERRIDE( AActor, ReplicatedMovement, bReplicateMovement );
  }
  ```

- 现在 ReplicatedMovement 属性只会在 bReplicateMovement 为 true 时复制

- 不使用这个宏的原因：

  - 如果定制条件的值变化太大，这种做法会降低执行速度
  - 您不能使用根据连接而变化的条件（此时不检查 RemoteRole）



条件属性复制的好处：

- 属性复制条件可以很好的实现控制力与性能之间的平衡
- 可以使引擎以更快的速度针对多条连接检查并发送属性
- 同时让程序员对复制属性的方式和时机进行精细控制







#### 3.9 复制对象引用



前提：

- 一般而言，对象引用会在 UE4 多人游戏架构中自动处理
- 一个已经复制的 UObject 属性，该对象的引用将作为服务器分配的专门 ID 通过网络进行发送
- 这个专门 id 是一个 FNetworkGUID
- 服务器将负责分配此 id，然后向所有已连接的客户端告知这一分配



使用：

- 需将一个 UObject 属性标记为已复制

  ```c++
  class ENGINE_API AActor : public UObject
  {
      UPROPERTY( replicated )
      AActor * Owner;
  };
  ```

- `Owner` 属性将作为其引用的 actor 的一个复制引用

- 注意：

  - 通过网络合法引用的对象，必须对其提供支持以保证网络连接
  - 要进行检查，可以调用 `UObject::IsSupportedForNetworking()`
  - 这是一个底层函数，所以一般不需要在游戏代码中对其进行检查



其他方式：

- 任何复制的 actor 都可以复制为一个引用
- 任何未复制的 actor 都必须有可靠命名（直接从数据包加载）
- 任何复制的组件都可以复制为一个引用
- 任何未复制的组件都必须有可靠命名。
- 其他所有 UObject（非 actor 或组件）必须由加载的数据包直接提供







##### 3.9.1 拥有可靠命名的对象



说明：

- 拥有可靠命名的对象指的是存在于服务器和客户端上的同名对象
- 如果 Actor 是从数据包直接加载（并非在游戏期间生成），它们就被认为是拥有可靠命名



方式：

- 从数据包直接加载
- 通过简单构建脚本添加
- 采用手动标记（通过 `UActorComponent::SetNetAddressable` 进行）
  - 只有当您知道要手动命名组件以便其在服务器和客户端上具有相同名称时，才应当使用这种方法
  - （最好的例子就是 `AActor` C++ 构造函数中添加的组件）







#### 3.10 蓝图使用RPC



远程调用函数主要包括 3 种类型：

1. **Multicast 广播**：广播函数在服务器上调用和执行，然后自动转发给客户端
   - Server的函数，Server调用，Server执行，复制给所有在场的Actor
2. **Run on Server 在服务端执行**：在服务端执行的函数由客户端调用，然后仅在服务器上执行
   - Server的函数，Client调用，Server执行
3. **Run on owning Client 在客户端执行**：在客户端执行的函数由服务器调用，然后仅在自有客户端上执行
   - Client的函数，Server调用，Client执行





##### 3.10.1 Multicast广播



1. 自定义事件

2. 然后在 **Details** 面板中将 **Replicates** 下拉菜单设置为 **Multicast**

   [![HowTo5.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo5.jpg)](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo5.png)

3. 从 **Space Bar** 按键事件连出来，搜索并添加调用函数**MulticastSpawn**

   ![HowTo6.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo6.jpg)

4. **编译** 并 **保存**，关闭蓝图，然后单击 **运行** 按钮在编辑器中开始游戏

5. 在游戏中，定位服务器所在的窗口，然后按 **Space Bar** 跳转

   ![HowTo7.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo7.jpg)

6. 服务器上的玩家按跳转时，服务器和所有客户端上都将生成喷火效果

7. 其他玩家跳转仍只能在本地喷火，因为我们并未告知服务器客户端已生成该效果







##### 3.10.2 RunOnServer



1. 选择 **MulticastSpawn** 自定义事件，然后将 **Replicates** 下拉选项更改为 **Run on Server**

   [![HowTo8.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo8.jpg)](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo8.png)

2. **编译** 并 **保存**，关闭蓝图，然后单击 **运行** 按钮在编辑器中开始游戏

3. 在游戏中，定位非服务器的任何游戏窗口，然后按 **Space Bar** 跳转

   ![HowTo9.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo9.jpg)

4. 注意，喷火效果只在服务器上生成，无论哪个玩家跳转，其他玩家都看不到，只有服务器端才能看到

5. 我们需确保该效果已设置为复制，以便其传递给所有客户端和服务器

6. 在 **Content/StarterContent/Blueprints** 文件夹中，打开 **Blueprint_Effect_Fire** 蓝图

7. 从主工具栏中选择 **Class Defaults** 后，在 **Details** 面板中，选中 **Replicates** 复选框

   ![HowTo10.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo10.jpg)

8. **编译** 并 **保存**，关闭蓝图，然后单击 **运行** 按钮在编辑器中开始游戏

9. 在游戏中，定位非服务器的任何游戏窗口，然后按 **Space Bar** 跳转

   ![HowTo11.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo11.jpg)

10. 向服务器发送消息，以在服务器运行部分脚本来生成 Actor，由于此 Actor 已设置为复制，因此我们可以在所有客户端上看到它







##### 3.10.3 RunOnOwningClient



说明：在此示例中，我们要做到是创建一个发生服务器事件时仅在特定客户端上更新的变量



步骤：设置 **Run on owning Client** 复制函数

1. 在 **Content/ThirdPersonBP/Blueprints** 文件夹中，打开 **ThirdPersonCharacter** 蓝图

2. 在 **MyBlueprint** 窗口中，创建新变量并将其命名为 **Inventory**，然后单击 **Compile**

   ![Inventory.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/Inventory.jpg)

3. 在此变量的 **Details** 面板中，将其设置为 **String**、**Editable** 和 **Replicated**，然后为 **Default Value** 输入 **Empty**

   ![HowTo12.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo12.jpg)

   - 将此变量设置为 **Replicated** 可确保其通过网络复制到所连接的机器上

   - 我们将使用此变量模拟人物在多人游戏中进入触发卷时收集道具，退出触发卷时删除道具

4. 添加一个与 **Print String** 连接的 **P** 按键事件，然后按住 **Control** 并拖入 **Inventory** 变量，并按所示方式连接

   ![HowTo13.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo13.jpg)

5. **Compile** 并 **Save**，然后关闭 **ThirdPersonCharacter** 蓝图

6. 在 **放置Actor（Place Actors）** 的 **基础（Basic）** 选项卡中，将 **盒体触发器（Box Trigger）** 拖入你的关卡

   ![HowTo14.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo14.jpg)

   - 当玩家人物进入触发器时，我们将更新创建的变量，但仅在进入触发器盒的客户端上更新。

7. 在 **Rendering** 下 **Box Trigger** 的 **Details** 面板中，取消选中 **Actor Hidden In Game**。

   ![UnHideBox.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/UnHideBox.jpg)

   - 在编辑器中玩游戏时，这可让我们在关卡中看到此盒，使得测试更加轻松

8. 单击 **Box Trigger** 将其选中，然后从主工具栏打开**Level Blueprint**。

   ![HowTo15.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo15.jpg)

9. 在图表中 **右键** ，然后搜索 **Begin Overlap** 并选择 **Add On Actor Begin Overlap** 事件

   ![HowTo16.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo16.jpg)

10. 重复上一步，但搜索并添加 **Add On Actor End Overlap** 事件

11. 将每个节点连接到 **Switch Has Authority** 节点

    ![HowTo17.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo17.jpg)

    - Switch Has Authority 节点用于检查当前正在运行的脚本正在从何处执行，然后基于脚本是在网络授权者（通常为服务器）还是远程机器（客户端）上运行将其分成两个不同的方向

    - 通常，你会对只希望在服务器上发生的事情使用授权者（这些通常为游戏关键性事件，例如调整玩家的生命值或赠送奖励或掠夺物品，因为你不想让客户端确定这些更改何时发生，以防作弊）

    - 在此示例中，我们将更新文本变量，此变量也完全可以是包含玩家生命值的变量，或所收集道具的变量

12. 在图表中 **右键** 并添加一个称作 **Add Item** 的 **Custom Event** 节点

13. 将 Replicates 选项设置为 **Run on owning Client**，并添加称作 **Character** 的输入，将其设置为 **Actor**

    ![HowTo18.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo18.jpg)

14. 创建另一个称作 **Remove Item** 的 **Custom Event**，其设置与 **Add Item** 事件的设置相同

15. 如下所示，从两个重叠事件连出来，接 **Add Item** 和 **Remove Item** 节点

    ![HowTo19.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo19.jpg)

    - 这里我们指的是，当重叠触发器时，如果重叠发生在服务器上，在服务器上运行 **Add Item** 事件，并将它复制到自有客户端（即重叠触发器的人物所在的客户端）
    - "它"是指 **Add Item** 启动并仅在服务器上执行但复制到客户端时所调用的脚本
    - 当人物退出触发器盒时 — 这也由服务器决定，在服务器上运行 **Remove Item** 事件，并将其复制到自有客户端

16. 从 **Add Item** 事件连出来，添加 **Print String** （文本设置为 **Item Added** ），然后拖开 **Character** 和 **Cast To ThirdPersonCharacter**

    ![HowTo20.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo20.jpg)

17. 从 **As Third Person Character** 针连出来，搜索并添加 **Set Inventory** 节点，将文本设置为 **Has the Item**

    [![HowTo21.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo21.jpg)](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo21.png)

    - 此处，我们选取的是在服务器上执行并复制到客户端的事件，此事件将在屏幕上显示`item added`
    - 然后将自有客户端的 **Inventory** 文本变量设置为`Has the item`

18. 在 **Add Item** 事件之后复制三个节点，并将其连接到 **Remove Item** 事件

19. 将 **Print String** 更改为 **Item Removed**，将 **Inventory** 文本变量更改为 **Empty**

    [![HowTo22.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo22.jpg)](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Actors/ReplicatingFunctions/HowTo22.png)

    - 现在，当人物退出触发器时，文本变量将在服务器上更新并复制到自有客户端

20. **编译** 并 **保存**，然后关闭 **Level Blueprint** 并在编辑器中开始游戏

    <div style="position: relative; width: 100%; height: 0; padding-bottom: 75%;">
    	<iframe src="https://www.youtube.com/embed/KiBrcLbXbYQ?rel=0&amp;modestbranding=1&amp;showinfo=0" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="position: absolute; width: 100%; height: 100%; left: 0; top: 0;">
        </iframe>
    </div>

    - 在以上视频中，当游戏启动时，按 P 可将文本变量打印到屏幕上，可对每个角色显示 "empty"
    - 但是，当其中一个角色退出触发器盒时，将显示文本 "item added"
    - 当人物按 P 时，文本将更改为 "has the item"，但其他人物仍显示 "empty"
    - 当人物退出触发器盒时，将显示文本 "item removed"，再次按 P 时，文本将更改回 "empty"







### 4. 客户端-服务器模式



说明：

- UE4 多人游戏基于客户端-服务器模式

- 一个服务器担当游戏状态的主控者，而连接的客户端将保持近似复本



作用：

- 做出所有重要决定，包含所有的主控状态，处理客户端连接，转移到新的地图以及处理比赛开始/结束时的总体游戏流程等





#### 4.1 启动服务器



基本命令：

| 类型       | 命令                                                   |
| :--------- | :----------------------------------------------------- |
| 监听服务器 | `UE4Editor.exe ProjectName MapName?Listen -game`       |
| 专用服务器 | `UE4Editor.exe ProjectName MapName -server -game -log` |
| 客户端     | `UE4Editor.exe ProjectName ServerIP -game`             |



注：

- 专用服务器在默认情况下并不会显示窗口
- 如果不使用 -log，您将不会看到任何呈现专用服务器的窗口







#### 4.2 服务器游戏流程



说明：

- 服务器的职责是在游戏开始/结束以及 actor 复制更新等情况下通知客户端转移到新地图
- 游戏状态和流程一般是通过 GameMode 这一 actor 来驱动
- 只有服务器才包含此 actor 的有效复本（客户端不包含复本）
- 要向客户端传达该状态，可以使用 GameState actor 显示 GameMode actor 的重要状态
- 这个 GameState actor 被标记为复制到每个客户端
- 客户端将包含此 GameState actor 的一个近似复本，而且能使用这个 actor 作为引用，用于了解游戏的一般状态





#### 4.3 连接过程



前提：一个服务器需要从网络连接的角度实现某种目的，它就必须要有客户端连接



说明：新的客户端初次连接时

- 首先，客户端要向即将连接的服务器发送一个请求
- 服务器将处理这条请求
- 如果它不拒绝连接，服务器会向客户端发回一个包含了继续运行所需信息的响应



主要步骤如下：

1. 客户端发送连接请求
2. 如果服务器接受连接，则发送当前地图
3. 服务器等待客户端加载此地图
4. 加载之后，服务器将在本地调用 AGameModeBase::PreLogin
   - 这样可以使 GameMode 有机会拒绝连接
5. 如果接受连接，服务器将调用 AGameModeBase::Login
   - 该函数的作用是创建一个 PlayerController，可用于在今后复制到新连接的客户端
   - 成功接收后，这个 PlayerController 将替代客户端的临时 PlayerController （之前被用作连接过程中的占位符）
   - 此时将调用 APlayerController::BeginPlay
   - 应当注意的是，在此 actor 上调用 RPC 函数尚存在安全风险。您应当等待 AGameModeBase::PostLogin 被调用完成
6. 如果一切顺利，AGameModeBase::PostLogin 将被调用
   - 这时，可以放心的让服务器在此 PlayerController 上开始调用 RPC 函数







### 5. 角色移动组件



说明：

- **角色移动组件** 是一种 **Actor 组件**，提供封装的移动系统和类人 **角色** 的常见移动模式，包括行走、跌倒、游泳和飞行
- 角色移动组件还具备强大的网络gameplay整合
- 默认移动模式全都默认用于复制，并提供框架来帮助开发者创建自定义的联网移动





#### 5.1 角色移动基础



- [`UCharacterMovementComponent`](https://docs.unrealengine.com/en-US/API/Runtime/Engine/GameFramework/UCharacterMovementComponent)前附于 [`ACharacter`](https://docs.unrealengine.com/en-US/API/Runtime/Engine/GameFramework/ACharacter)Actor类及其派生的所有 **蓝图**

  ![角色移动组件](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/CharacterMovementComponent/CharacterMovement_ComponentAttached.jpg)

- 在其 `TickComponent` 函数期间，`UCharacterMovementComponent` 将调用 `PerformMovement`

- 基于当前所用 **移动模式** 以及玩家的输入变量来计算世界场景中所需的加速度

- 通常在[`APlayerController`](https://docs.unrealengine.com/en-US/API/Runtime/Engine/GameFramework/APlayerController)中以 *control input* 变量表示

- 一旦完成移动计算，`UCharacterMovementComponent` 将把最终移动应用于拥有的角色



注：

- 虽然 `ACharacter` 派生自[`APawn`](https://docs.unrealengine.com/en-US/API/Runtime/Engine/GameFramework/APawn)，但角色并不只是增加了角色移动组件的Pawn
- `UCharacterMovementComponent` 和 `ACharacter` 需要一同使用，因为 `ACharacter` 覆盖数个复制的变量和函数，专为在 `UCharacterMovementComponent` 中进行复制







#### 5.2 PerformMovement



说明：

- `PerformMovement` 函数负责游戏世界场景中的角色物理移动
- 在非联网游戏中，`UCharacterMovementComponent` 每次tick将直接调用一次 `PerformMovement`
- 在联网游戏中，由专用函数为服务器和客户端调用 `PerformMovement`，在玩家的本地机器上执行初始移动，或在远程机器上再现移动



作用：

- 应用外部物理效果，例如脉冲、力和重力
- 根据动画根运动和 **根运动源** 计算移动
- 调用 `StartNewPhysics`，它基于角色使用的移动模式选择 `Phys*` 函数







#### 5.3 移动物理效果



每个移动模式都有各自的 `Phys*` 函数，负责计算速度和加速度：

- `PhysWalking` 决定角色在地面上移动时的移动物理效果
-  `PhysFalling` 决定在空中移动时的移动物理效果



若移动模式在一个tick内发生变化（例如角色开始跌倒或撞到某个对象）：

- `Phys*` 函数会再次调用 `StartNewPhysics`，在新移动模式中继续角色的运动
- `StartNewPhysics` 和 `Phys*` 函数各自通过已发生的 `StartNewPhysics` 迭代的次数
- 参数 `MaxSimulationIterations` 是此递归所允许的最大次数





#### 5.4 移动复制摘要



`UCharacterMovementComponent` 使用其所有者的 **网络角色** 来确定移动的复制方式：

| 网络角色                         | 说明                                                         |
| :------------------------------- | :----------------------------------------------------------- |
| **自主代理（Autonomous Proxy）** | 角色在其 *所属客户端* 机器上，由玩家本地控制                 |
| **权威（Authority）**            | 角色存在于建立游戏的服务器上                                 |
| **模拟代理（Simulated Proxy）**  | 角色存在于可查看远程控制角色的其他客户端上，无论角色受服务器AI控制，还是由不同客户端上的自主代理控制 |



- 复制进程遵循 `TickComponent` 函数中的循环，每tick重复一次
- 角色执行移动时，为同步移动信息，网络游戏中所有不同机器上的副本会相互进行 **远程进程调用 (RPC)**，不同网络角色使用相应的不同执行路径





下表逐步概述此进程中 `UCharacterMovementComponent` 在各个机器上所执行的操作：

| 步骤                         | 说明                                                         |
| :--------------------------- | :----------------------------------------------------------- |
| 自主代理（所属玩家的客户端） |                                                              |
| **1**                        | 所属客户端本地控制自主代理。`PerformMovement` 运行移动组件的物理移动逻辑 |
| **2**                        | 代理构建 `FSavedMove_Character`，其中包含其所做移动的相关数据，然后在 `SavedMoves` 中将其排入队列 |
| **3**                        | 类似的 `FSavedMove` 条目组合在一起。自主代理通过 **ServerMove** RPC将压缩版数据发送到服务器 |
| 权威Actor（服务器）          |                                                              |
| **4**                        | 服务器接收ServerMove并使用 `PerformMovement` 复制客户端移动  |
| **5**                        | 服务器检查其在ServerMove后的位置是否与客户端报告的最终位置匹配 |
| **6**                        | 若服务器和客户端的最终位置匹配，则向客户端发回信号，表示移动有效。否则将使用 **ClientAdjustPosition** RPC 发送矫正 |
| **7**                        | 服务器复制 `ReplicatedMovement` 结构，将其位置、旋转和当前状态发送到其他已连接客户端上的模拟代理 |
| 自主代理（所属玩家的客户端） |                                                              |
| **8**                        | 若客户端收到ClientAdjustPosition，则复制服务器的移动，并使用它的 `SavedMoves` 队列重新追踪其步骤，以获得新的最终位置。成功解析移动后，从队列中移除已保存的移动 |
| 模拟代理（所有其他客户端）   |                                                              |
| **9**                        | 模拟代理直接应用复制的移动信息。**网络平滑** 提供最终运动的可视化清理 |







#### 5.5 复制角色移动详解



##### 5.5.1 所属客户端上的本地移动



说明：自主代理在 `TickComponent` 中本地处理移动，予以记录，然后发送到服务器以授权方式再现和应用





###### 5.5.1.1 编译客户端预测数据



说明：

- 自主代理编译名为 `ClientPredictionData` 的 `FNetworkPredictionData_Client_Character` 对象
- 其部分流程负责记录移动情况和处理来自服务器的矫正



参数：

- 客户端与服务器通信时的时间戳
- 已保存或待定移动情况的列表
- 来自服务器矫正的已保存信息
- 指示如何应用矫正的标记
- 决定平滑行为的参数



- `ClientPredictionData` 还包括与这些参数交互的效用函数

- 可在[`FNetworkPredictionData_Client_Character` 的API参考](https://docs.unrealengine.com/en-US/API/Runtime/Engine/GameFramework/FNetworkPredictionData_Client_Ch-)中找到此对象信息和函数的完整列表

- 客户端执行本地移动、准备要发送到服务器的移动并处理矫正时，它的参数将被频繁引用和更改







###### 5.5.1.2 复制服务器矫正



- 处理玩家的输入或世界场景中的力之前，自主代理将调用 `ClientUpdatePositionAfterServerUpdate`
- 这将检查服务器是否已向所属玩家发送矫正
- 若是，`ClientPredictionData` 中的变量 `bUpdatePosition` 将为true，且该角色将再现服务器通过客户端矫正进程发送的移动





###### 5.5.1.3 执行和记录移动



- 自主代理角色在 `TickComponent` 期间调用 `ReplicateMoveToServer`，而非直接调用 `PerformMovement`

- 此函数围绕 `PerformMovement` 提供必要的逻辑，以便在角色执行它时记录移动，然后将移动提交到服务器

- `FSavedMove_Character` 结构记录自主代理在每个tick中如何开始和结束移动，之后通过ServerMove RPC将其数据的最小子集发送到服务器

- 其参数包括：

  - 角色最终位置和旋转的相关信息

  - 已采集哪些移动输入

  - 角色拥有的速度和加速度

  - 从 **AnimMontages** 采集的根运动信息

- 可在[`FSavedMove_Character` 的API参考](https://docs.unrealengine.com/en-US/API/Runtime/Engine/GameFramework/FSavedMove_Character)中查看此结构的完整参数列表

- 此信息使服务器能够再现玩家执行的移动，然后检查客户端的最终位置



- 处理 `PerformMovement` 之后，`ReplicateMoveToServer` 函数利用名为 `NewMove` 的 `FSavedMove_Character` 结构在客户端预测数据中记录角色移动的结果，然后将其添加到名为 `SavedMoves` 的缓冲
- 此缓冲将保存的移动按最旧到最新的顺序排列，并用作队列，直至保存的移动可提交到服务器为止。提交之前，为缓解带宽压力，缓冲中的所有类似移动将合并成单个 `FSavedMove_Character`
- 移动正等着与即将到来的移动相结合，那么参数 `PendingMove` 将用于存储移动



- 当这些移动被确认或 *ACK* 后，会从缓冲中移除
- 服务器可通过确认客户端位置有效来直接确认移动，客户端也可以在处理来自服务器的矫正时确认移动
- 最新确认的移动保存在 `LastAckedMove` 中，以供处理未来的矫正时使用





###### 5.5.1.4 向服务器提交移动



说明：

- `ReplicateMoveToServer` 通过运行函数 `CallServerMove` 来完成，该函数接受队列中未获得服务器确认的最新和最旧移动
- 这将执行向服务器提交移动的最后准备工作，先尝试提交旧移动（若适用），然后调用正确的ServerMove函数来提交新移动的最终移动
- 最终ServerMove直接提交给 `UCharacterMovementComponent` 的所属角色，作为 *不可靠的* 服务器RPC



ServerMove函数不可靠的原因有二：

1. 正常游戏期间经常会调用ServerMove函数，如果它们被视为可靠，大量调用可靠函数可能会导致缓冲溢出，迫使所属玩家断开连接
2. 用于缓冲已保存移动的系统可确保传输进程中丢失的移动信息会被重新提交和计算，这为可靠函数提供了类似的安全保障，但却没有可靠RPC缓冲溢出的风险，并增加了一些规定来确保丢弃太旧的移动数据







##### 5.5.2 计算服务器上的移动



说明：

- 服务器不会定期tick移动来与游戏的tick循环同步，而是等待从自主代理处接收ServerMove调用
- `ServerMove_Implementation` 处理服务器端的移动，重新构造客户端的移动并检查细微差异





###### 5.5.2.1 编译服务器预测数据



- 角色移动组件的权威版本创建名为 `ServerPredictionData` 的 `FNetworkPredictionData_Server_Character` 对象，该对象在角色生命周期内存在
- 在 `ServerMove_Implementation` 期间，此对象将存储信息，供以后的进程再现所属客户端的移动
- 服务器接收数据时，此对象在后台被不断修改，参数包括：
  - 用于计算服务器差量时间的时间戳
  - 待定的客户端调整
  - 与解决时间差异相关的标记
  - 指示服务器确认还是矫正移动的标记
- 可在[`FNetworkPredictionData_Server_Character` 的API参考指南](https://docs.unrealengine.com/en-US/API/Runtime/Engine/GameFramework/FNetworkPredictionData_Server_Ch-)中查看它的参数和函数的完整列表







###### 5.5.2.2 验证客户端时间戳



- 与ServerMove RPC一同发送的信息包括移动发生时的时间戳
- 若服务器时间戳和客户端时间戳差异太大，客户端时间戳将视为已过期，移动将被丢弃
- 否则该差异将标记为已解决，且 `UCharacterMovementComponent` 使用 `ProcessClientTimeStampForTimeDiscrepancy` 在下一步中创建差量时间的覆盖







###### 5.5.2.3 计算差量时间



说明：

- 差量时间通常是通过追踪当前tick与上一个tick之间经过的时间来获取
- 但服务器上的角色并不使用 `TickComponent` 来计算移动



原理：

- `ServerMove_Implementation` 调用 `GetServerMoveDeltaTime` 并在收到ServerMove后计算移动
- 若服务器预测数据被标记为尝试解决时间戳差异，它将使用 `TimeDiscrepancyResolutionMoveDeltaOverride`
- 若无时间差异，则根据当前ServerMove RPC时间戳和最后一个ServerMove RPC时间戳之间的差异，使用服务器预测数据来创建差量时间
- 为多一层安全，执行这些计算时多半使用服务器时间戳而非客户端时间戳，以防止客户端加快本地游戏时间进行速度作弊







###### 5.5.2.4 计算移动



说明：

- 服务器使用来自ServerMove RPC的数据重新构造所属玩家控制器的控制旋转
- 然后调用函数 `MoveAutonomous` 来处理角色的加速、旋转和跳跃输入



原理：

- `MoveAutonomous` 使用 `PerformMovement` 函数，根据之前重新构造的数据和上一步中提供的差量时间来模拟角色的移动物理效果
- 服务器从获取ServerMove调用时自身角色副本所在的位置模拟运动，而非从客户端开始位置模拟



注意：

- 若角色正在计算来自动画的根运动，MoveAutonomous还会使用提供的差量时间tick角色的动画姿势
- 所有动画事件都将相应触发。否则动画将正常tick







##### 5.5.3 处理客户端错误和矫正



说明：

- 服务器移动的工作原理是假设服务器和所属客户端在相同位置开始移动，且若服务器执行的移动与客户端报告的相同，则移动结束的位置也将相同
- 若客户端的移动由于连接问题而丢失，或者客户端提交了错误数据，则两者的移动将在不同的位置结束，因此需要矫正
- 函数 `ServerMoveHandleClientError` 负责这些操作





###### 5.5.3.1 确定是否需要调整



说明：

- 频繁发送矫正会导致带宽紧张，并导致客户端过于频繁地重新模拟大量已保存的移动
- 因此要首先检查从 `WithinUpdateDelayBounds` 返回的值，查看移动间歇是否已超过最小时间量
- 若返回 `false`，则不发出矫正，若返回true，则可运行其余进程



原理：

- 使用 `ServerCheckClientError` 查看服务器与客户端之间的误差是否大到需要矫正
- 若返回true 或 `bForceClientUpdate` 因故被设为true而强制矫正，则 `ServerMoveHandleClientError` 将继续执行余下进程



注：

- 可在 `BaseGame.ini` 中找到用于调整这两个操作的参数，并可在 `DefaultGame.ini` 中提供项目特定的覆盖
- `ClientErrorUpdateRateLimit` 值决定着服务器向客户端发送错误矫正的最小延迟（单位为秒）`MAXPOSITIONERRORSQUARED` 值是网络游戏中可接受未矫正最大位置误差的平方
- 两者都可在配置文件的 `[/Script/Engine.GameNetworkManager]` 部分中找到

- 若需要调整，服务器预测数据用从角色的服务器副本中采集的当前移动变量样本填充名为 `PendingAdjustment` 的 **[`FClientAdjustment`](https://docs.unrealengine.com/en-US/API/Runtime/Engine/GameFramework/FClientAdjustment)**，包括位置、旋转、速度和可能充当角色移动基础的对象
- 否则将 `PendingAdjustment` 的 `bAckGoodMove` 值设为 `true`，将客户端的移动标记为有效







###### 5.5.3.2 发送客户端调整或确认移动



说明：

- 确认到客户端的移动的最后一步由 `SendClientAdjustment` 完成
- 此函数不作为 `ServerMove_Implementation` 的一部分，而是作为 `UNetDriver::ServerReplicateActors` 的一部分，在服务器端的tick结束时调用，并同样负责调用其他客户端调整RPC
- 调用 `SendClientAdjustment` 时，操作方式将取决于之前步骤中编译的预测数据的标记方式



作用：

- 若服务器预测数据的 `PendingAdjustment` 的 `bAckGoodMove` 标记为 `true`，则将调用 `ClientAckGoodMove` RPC来确认移动，告知所属客户端机器上的自主代理该移动为有效
- 这将从所属客户端的 `SavedMoves` 缓冲中移除原始移动，并将其记录为 `LastAckedMove`，用于编译未来的预测数据



注：

- 若 `PendingAdjustment` 的 `bAckGoodMove` 标记为false，则将调用客户端调整函数，将最终矫正发送到客户端





###### 5.5.3.3 自主代理接收客户端调整



说明：

- 客户端调整RPC包括 `ClientAdjustPosition`、`ClientAdjustRotation`、这两者在速度为零时发生的缩略版、以及这两者专用于根运动移动的版本
- 作为 `SendClientAdjustment` 的一部分，服务器可调用多个此类函数，具体取决于待矫正内容的性质和严重程度
- 每个此类函数都可告知 `ClientPredictionData` 在应用必要矫正后确认移动，且都会将 `bUpdatePosition` 标记为true



注：

- 然后在客户端的下一个 `TickComponent` 开始时使用 `ClientUpdatePosition` 应用最终矫正







##### 5.5.4 将移动复制到模拟代理



说明：

- 客户端机器上的角色（其所有者除外）为模拟代理，而非自主代理、
- 由于模拟代理的唯一工作就是响应服务器，因此将移动从服务器复制到模拟代理的进程为高度简化
- 从服务器接收移动更新时不会模拟运动物理效果，而是将其位置、旋转和速度设为服务器想要的值，并使用一些额外的进程让移动更流畅、更可信





###### 5.5.4.1 存储复制的移动信息



说明：

- Actor复制移动时，其不会直接复制自身的变换。所有Actor都维持名为 `ReplicatedMovement` 的复制变量，该变量使用结构[`FRepMovement`](https://docs.unrealengine.com/en-US/API/Runtime/Engine/Engine/FRepMovement)



作用：

- 布尔 `bReplicateMovement` 由蓝图中的 **Replicate Movement** 变量表示，标记Actor将移动信息存储在此结构中，并将其复制到客户端
- 客户端收到 `ReplicatedMovement` 的更新时，**RepNotify** 函数 `OnRep_ReplicatedMovement` 将解压缩存储的移动数据，并相应地更新Actor的位置和速度

- 在蓝图中无法访问 `ReplicatedMovement` 或其 OnRep，但可在C++中覆盖 `OnRep_ReplicatedMovement`，还可在`GetLifetimeReplicatedProps` 中覆盖 `ReplicatedMovement` 的复制条件
- 用户因此能够自定义移动复制在基于C++的Actor类中的行为
- 在 `ACharacter` 中，仅为模拟代理复制 `ReplicatedMovement` 结构
- 自主代理上会忽略此结构，而使用服务器移动和客户端调整RPC来处理移动



注：

- 若角色用另一个Actor作为基础，则将使用 `ReplicatedBasedMovement`，从而应用额外的逻辑来确保客户端是依据服务器而正确建立
- 若角色使用根运动系统，则忽略所有这些进程，而倾向于使用 `RepRootMotion`







###### 5.5.4.2 在模拟代理上tick移动



- 当 `UCharacterMovementComponent` 在模拟代理上运行 `TickComponent` 时，其将调用 `SimulatedTick` 来处理模拟移动的逻辑
- 这不会执行上述复制移动。相反，`SimulatedTick` 会根据最近提供的复制移动数据继续移动
- 执行标准移动物理效果时，调用 `SimulateMovement` 函数，然后使用 `SmoothClientPosition` 执行最终的验证和网络平滑





###### 5.5.4.3 执行模拟移动



说明：

- `SimulateMovement` 函数负责移动模拟代理角色
- 除了会被 `SimulatedTick` 调用，其还会被 `OnRep_ReplicateMovement` 调用
- 此函数执行以下进程：
  1. 调用所属角色的 `GetReplicatedMovement` 函数，以获取对 `ReplicatedMovement` 的引用
  2. 执行安全检查，确保复制的移动数据有效，且客户端的基础被解析
  3. 检查是否已收到网络更新
  4. 从服务器应用通过 `GetReplicatedMovementMode` 获取的角色移动模式
  5. 重置有关网络更新的所有标记
  6. 基于当前 `MovementMode` 和角色的当前状态信息执行模拟移动的逻辑



注：

- 与标准运动物理效果相比，模拟运动的逻辑高度简化，因此主要包含在 `SimulateMovement` 函数本身中，不会分解成更小的函数
- 但是，此函数仍负责更新角色的本地移动状态，包括应转换为哪个移动模式，角色是否已经降落至地面，以及应拥有的速度
- 这些信息可确保角色正确更新动画，让动作看起来十分准确







###### 5.5.4.4 网络平滑



说明：

- 若单纯通过复制角色的位置和旋转来复制移动，则角色看起来每隔数分钟就会瞬移一次
- 这是因为本地机器的渲染速率比网络发送数据速率更快
- 举例而言，客户端可能以240 Hz刷新率渲染显示器，而复制的移动可能仅以30 Hz发送



作用：

- 网络平滑是使此运动平滑的进程，将角色从源位置逐渐向目标位置插入，而非立即将其对齐到目标位置
- 源位置由角色当前位置提供，而目标位置由客户端预测数据提供
- 内插本身在 `SmoothClientPosition` 中处理，使用[`NetworkSmoothingMode`](https://docs.unrealengine.com/en-US/API/Runtime/Engine/Engine/ENetworkSmoothingMode)确定应使用的内插类型









#### 5.6 特殊移动情况



- 包含常见特殊移动情况的信息，包括传送、自定义移动，**以及可能在特殊技能中看到的代码驱动移动**





##### 5.6.1 多人游戏中传送角色



说明：可调用 **SetLocation** 函数或 **Teleport** 蓝图节点在网络游戏中传送角色



1. 必须在服务器上调用
2. 若使用 `SetLocation` 函数，将 `bTeleport` 变量设为true，使其将该移动视为传送

![传送角色](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/CharacterMovementComponent/CharacterMovement_Teleport.jpg)

- 若满足这些条件，移动会作为传送记录到服务器的预测数据和复制移动中，且所有客户端会将角色对齐到所需位置（而非应用平滑）来做出相应的响应







##### 5.6.2 自定义移动模式



说明：移动模式 `MOVE_Custom` 会中止所有其他移动物理效果，可实现自定义移动逻辑，不受 `UCharacterMovementComponent` 正常进程的干扰



作用：

- `UCharacterMovementComponent` 通常不可蓝图化，因此蓝图中的自定义移动通常是使用 **UpdateCustomMovement** 事件直接在角色内实现

- 可使用 **Custom Movement Mode** 字节变量通过整数开关或自定义列举转换提供子模式

  ![蓝图中的自定义移动模式](UE4联网和多人游戏.assets/CharacterMovement_CustomMovementModeBP.jpg)

- `UpdateCustomMovement` 由 `UCharacterMovementComponent` 中的 `PhysCustom` 函数调用

- 函数 `StartNewPhysics`、`PhysCustom` 和所有其他移动物理效果函数都是虚拟函数，因此若要在C++中创建自定义 `UCharacterMovementComponent`，可直接将其覆盖







##### 5.6.3 通根运动复制



说明：

- 有时需要在短时间内直接控制角色的移动，例如在使用 **Gameplay Ability System** 创建的技能发动期间、或在动画驱动操作发生期间
- 虽然这在纯本地游戏中很容易做到，但复制的特殊移动情况需要使用根运动（通常指应用动画中的移动）
- 根运动系统也已调整为允许代码驱动的特殊移动情况



作用：

- 无论 `UCharacterMovementComponent` 使用哪种运动模式，根运动始终优先于标准移动物理状态。完成根运动后将继续正常移动







###### 5.6.3.1 来自动画蒙太奇



说明：

- 根运动的大部分应用来自动画蒙太奇，而动画蒙太奇用于代码触发的一次性动画
- 根运动的这种用法会中止角色正在执行的其他移动，直至动画结束
- 角色从其骨架的根骨骼使用移动，并将其转换为场景空间的移动，使动画能够控制角色的移动方式
- 完成这一步后，角色恢复使用普通物理效果



注：

- 若角色处于下落移动模式，即使角色正在执行根运动，重力仍将作用于角色的Z轴移动

- 在上述复制进程中，根运动信息由 `FSavedMove_Character` 结构采集，包括作为其来源的动画蒙太奇、角色在蒙太奇中的轨迹位置，以及角色移动本身的参数



服务器和所属客户端上的自主代理不会检查播放的动画是否相同，因为这通常被视为是一种装饰功能

- 因此必须对游戏逻辑进行编程，确保所有动画蒙太奇都能在所有连接游戏的机器上正确触发
- 但是模拟代理拥有上述进程的并行进程，用于同步基于根运动的移动



Gameplay Ability System插件将复制触发动画蒙太奇和根运动的技能，以此同步这些动画蒙太奇和根运动。





###### 5.6.3.2 来自根运动源



说明：

- 有时需要手动控制角色的位置来处理特殊情况
- 举例而言，可能需要创造一种特殊技能，使角色跳到空中特定的高度，然后落到移动目标上



原理：

- 在standalone游戏中，可以用 `SetLocation` 和 `SetRotation` 手动控制角色，但在网络游戏中，上述复制进程无法捕捉此运动，因此服务器将把客户端的最终位置视为错误并发出矫正
- 同时，来自动画蒙太奇的根运动仅遵循动画中预计算的运动
- 这意味着根运动通常无法从游戏世界获取实时信息，比如其他角色的位置，而且也不能使用游戏变量进行轻松微调



注：

- **根运动源** 为程序员提供了一种手动控制角色根运动的方法
- 因此可以通过编程来控制角色的移动，同时还可以利用上述系统在联网期间处理根运动

- 根运动源应应用于所属客户端上的自主代理



使用：

- 必须创建新的FRootMotionSource结构

- 不同类型的移动对应不同的FRootMotionSource变体

- 举例而言，FRootMotionSource_MoveToForce用于从开始位置到目标位置的直线移动，而FRootMotionSource_JumpForce遵循的是弹跳式弧线移动

- 创建适当的根运动源后，可使用所需源位置、目标位置和移动行为相关参数将其属性初始化

  

- 函数 `UCharacterMovementComponent::ApplyRootMotionSource` 将把根运动源应用于角色，并返回一个可供之后引用的柄
- 根运动源本身不处理移动，而是由角色移动组件执行与所提供 `FRootMotionSource` 中参数一致的移动，以代替动画
- 最终这将添加到 `FSavedMove_Character` 结构中的 `SavedRootMotion`，并在复制周期中采集，前提是 `FRootMotionSource` 应用于自主代理



- 完成移动后，必须调用 `UCharacterMovementComponent::RemoveRootMotionSource`，使用从 `ApplyRootMotionSource` 返回的柄将其移除

Gameplay Ability System插件包含几个利用根运动源的技能任务，使技能可以执行复杂的程序化移动序列

- 基础范例请参见 `AbilityTask_ApplyRootMotionMoveToForce`







### 6. OnlineBeacon



介绍：

- **Online Beacons** 是一个特殊类型的 **Actor**，可提供一种简易的方式与服务器联络并进行交互（通过 RPC），而无需让普通游戏连接来执行此任务
- 可以使用内置类，因为在一些情况下它们将被延展为自定义类，执行项目特定的交互、逻辑和信息请求





#### 6.1 OnlineBeacon 基类



说明：

- Beacon 类执行的常规操作是请求服务质量信息
- 在客户端需要加入的游戏中预留空位
- 接收游戏中玩家名列表、
- 获取正在进行的游戏中的得分和运行时间，等等







##### 6.1.1 AOnlineBeacon



说明：

- 这是 `AOnlineBeaconClient` 和 `AOnlineBeaconHost` 的基类
- 它直接派生自 `AActor`





##### 6.1.2 AOnlineBeaconHost



说明：

- 此类使用其自身的 `UNetDriver` 获得来自远程客户端电脑的传入 Online Beacon 连接
- 接收到连接时，它将在注册 `AOnlineBeaconHostObject` 实例列表中进行查找，找到与传入客户端匹配的实例并转交连接
- 此类通常不需要被派生，因其只管理客户端和注册 `AOnlineBeaconHostObject` 之间的初始连接





##### 6.1.3 AOnlineBeaconClient



说明：

- 此类的子项连接到主机并执行实际的 RPC
- 它们其中一个将在客户端电脑上生成，一个由正确的 `AOnlineBeaconHostObject`（注册到服务器的 `AOnlineBeaconHost`）在服务器上生成
- `GetBeaconType` 函数的输出（即为类名称）将用于对比此类的实例和正确主机对象类的注册实例



注意：

- 这和普通的 Actor 生成方式（服务器生成 Actor 然后复制到客户端）不同
- 然而，客户端和服务器对象副本之间的连接建立后，对象复制将正常进行，任意一方均可向对方执行 RPC，而对象的服务器版本可对属性复制发送命令
- 该基类实现 `OnConnected` 和 `OnFailure` 函数
- 这两个函数可由子类覆盖，在连接时执行 RPC，或处理失败连接
- 此类是 Online Beacon 系统的主力，将执行 Beacon 所需的客户端端的工作
- 在成功连接事件中，服务器上将生成和源实例同步的另一个实例，此例也可执行服务器端的工作，通过客户端和服务器 RPC（或服务器到客户端的复制属性）进行协调和交流





##### 6.1.4 AOnlineBeaconHostObject



说明：

- 此类也应被覆盖，使其和覆盖的 `AOnlineBeaconClient` 类配对
- 将客户端 `GetBeaconType` 的返回值和保存在 `BeaconTypeName` 成员变量中的值进行匹配即可完成配对
- 服务器的 `AOnlineBeaconHost` 检测到传入 `AOnlineBeaconClient` 的配对 `AOnlineBeaconHostObject` 时，它将指示 `AOnlineBeaconHostObject` 通过虚拟 `SpawnBeaconActor` 函数生成 `AOnlineBeaconClient` 的本地副本
- 此函数默认使用 `ClientBeaconActorClass` 成员变量确定要生成的 actor 类，此类应被设为配对的 `AOnlineBeaconClient` 类
- 它还将在生成对象的服务器副本上调用 `SetBeaconOwner`，以便客户端对象的服务器端实例与主机对象进行交流
- 此设置多数建立在基类中，无需被覆盖







#### 6.2 OnlineBeacon 范例



说明：

- 引擎中已自带大量预建 Beacon 类配对
- 也可自行构建，Test Beacon 就是一个简单的好例子
- `ATestBeaconClient` 派生自 `AOnlineBeaconClient`，`ATestBeaconHost` 派生自 `AOnlineBeaconHostObject`



使用：Test Beacon 就是简单地在客户端和服务器之间来回执行重复的函数调用

1. 在 `ATestBeaconHost` 的构建函数中，`ClientBeaconActorClass` 设为 `ATestBeaconClient` 的类、`BeaconTypeName` 设为 `ClientBeaconActorClass` 的命名（调用 `GetName` 函数获得），这样可确保 `AOnlineBeaconHost` 将两个 Test Beacon 类正确配对
2. 客户端的 `ATestBeaconClient` 成功连上服务器时，服务器的 `AOnlineBeaconHost` 实例将指引注册 `ATestBeaconHost` 生成其自身的 `ATestBeaconClient`，并在服务器上将其和 `ATestBeaconClient` 关联， 这很重要，因为它使客户端和服务器 RPC 在两个 `ATestBeaconClient` 实例之间传递
3. 服务器的 `ATestBeaconHost` 将在它的 `ATestBeaconClient` 实例上调用 `OnClientConnected`（客户端 RPC）， 因为此函数为客户端 RPC，调用函数的命令将被传至客户端电脑，而不会在服务器上调用函数
4. RPC 将在客户端的 `ATestBeaconClient` 实例上运行虚拟 `OnConnected` 函数，此函数已被覆盖，以便调用 `ClientPing`
5. `ClientPing` 将输出一些文本到调试日志，然后调用服务器 RPC `ServerPong`，它将在网络连接中传递，并在服务器的 `ATestBeaconClient` 实例上执行
6. `ServerPong` 将输出调试文本并调用 `ClientPing`，再触发对客户端联网 RPC 调用
7. 此过程将无限继续，这通常并非理想状态，尤其是这些 RPC 并未执行具体任务，它可用于记录相同 RPC 在两次分步运行之间的时间，以追踪服务器的 ping 时，因为可能存在延迟，所以这个 ping 检查不会按电脑和网络的最佳频率进行，或者客户端/服务器可能直接在数次 ping 后直接停止返回 ping，其他的使用范例可能涉及到唯一的 RPC 回叫到客户端，而不是继续循环，如连续询问一连串的问题 `该服务器的游戏使用的是哪张地图？`或 ` 这是一个夺旗服务器吗？`，这些问题的内容视特定游戏或项目的相关信息而定







### 7. 测试多人游戏



介绍：

- 更改编辑器的某些设置
- 测试不同的多人场景





#### 7.1 设置玩家数量



**增加玩家数量**

- 单击 **Play** 按钮旁的下箭头，然后输入 **Number of Players** 的值

  ![NumberOfPlayers.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/TestMultiplayer/NumberOfPlayers.jpg)

- 默认情况下，服务器将使用 **Selected Viewport** 作为游戏窗口，并为添加的每位玩家创建新窗口

![PIE.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/TestMultiplayer/PIE.jpg)









#### 7.2 调整游戏窗口



**为服务器调整窗口**

- 单击 **Play** 按钮旁的下箭头，然后选择 **New Editor Window**

  ![NewEditorWindow.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/TestMultiplayer/NewEditorWindow.jpg)

- 虽然模拟客户端拥有自己的窗口，但此设置还会模拟服务器创建独立窗口







#### 7.3 高级设置



**调整窗口大小**

1. 单击 **Play** 按钮旁的下箭头，然后选择 **Advanced Settings**

   ![AdvancedSettings.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/TestMultiplayer/AdvancedSettings.jpg)

2. 在 **Play in New Window** 部分下设置所需的窗口大小

   ![WindowSize.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/TestMultiplayer/WindowSize.jpg)

   - **Play in New Window** 大小选项允许您设置所创建的窗口的大小和默认位置
   - 可以从若干预设窗口大小中选择或手动输入窗口大小（此例中，我们指定为 640x480)
   - 可以将所创建的窗口设置为 **Left** 和 **Top** 位置或单击 **Always center window to screen** 的复选框

3. 输入窗口大小后，当您在编辑器中玩游戏时，每个新窗口的大小相同。

![MultipleWindows.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/TestMultiplayer/MultipleWindows.jpg)

- 当在编辑器中使用每个游戏会话的新窗口玩游戏时，将注意到每个窗口顶部将显示玩家是服务器玩家还是客户端玩家
- 同时，在 **Play** 模式下，移动窗口时，系统将为下次在编辑器会话中 **Play** 记住其位置（因此，无需一直移动窗口，这样使得更简单）





#### 7.4 多人选项



**Advanced Settings** 中还包括用于设置附加多人选项的部分：

![MultiplayerOptions.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/TestMultiplayer/MultiplayerOptions.jpg)

| 选项                                     | 说明                                                         |
| :--------------------------------------- | :----------------------------------------------------------- |
| **Number of Players**                    | 此选项定义游戏启动时其中将生成的玩家数量<br />编辑器和监听服务器均视为玩家，而专用服务器则不会<br />客户端组成剩余玩家 |
| **Server Game Options**                  | 您可在此指定将作为 URL 参数传输到服务器的附加选项            |
| **Run Dedicated Server**                 | 如果选中，则将启动独立的专用服务器<br />否则，第一个玩家将充当所有其他玩家均可连接的监听服务器 |
| **Route 1st Gamepad to 2nd Client**      | 在单个进程中运行多个玩家窗口时，此选项将决定游戏手柄输入的路由方式<br />如果未选中（默认），第一个游戏手柄将连接到第一个窗口，第二个连接到第二个窗口，以此类推<br />如果选中，则第一个游戏手柄将连接第二个窗口<br />然后即可通过键盘/鼠标控制第一个窗口，如果两人在同一台计算机上进行测试则十分便利 |
| **Use Single Process**                   | 此选项将在虚幻引擎 4 的单个实例中生成多个玩家窗口<br />其加载速度更快，但也有存在更多问题的可能性。未选中此选项时，附加选项将可用 |
| **Create Audio Device for Every Player** | 启用此选项将允许从每位玩家的角度渲染精确音频，但将占用更多 CPU 资源 |
| **Play In Editor Description**           | 此选项根据当前应用的多人设置说明玩游戏时将发生的事情         |

- 当选中 **Use Single Process** 时，虚幻引擎 4 的单个实例中将生成多个窗口
- 当未选中此选项时，将为分配的每位玩家启动多个 UE4 的实例，而且附加选项将可用：

![NoSingleProcess.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/TestMultiplayer/NoSingleProcess.jpg)

| 选项                                    | 说明                                                         |
| :-------------------------------------- | :----------------------------------------------------------- |
| **Editor Multiplayer Mode**             | 此选项是用于 Play In Editor（**Play Offline**、**Play As Listen Server** 或 **Play As Client**）的 NetMode |
| **Command Line Arguments**              | 您可在此分配将传输到独立游戏实例的附加命令行选项             |
| **Multiplayer Window Size (in pixels)** | 定义生成附加独立游戏实例时所采用的宽度/高度                  |





#### 7.5 监听服务器与专用服务器



介绍：启动多人游戏时，游戏托管方式有两种

1. 通过使用 **Listen Server**（默认设置），这表示拥有权限的机器也可以运行客户端，并可在为其他玩家托管时正常玩游戏
2. 通过使用 **Dedicated Server**，顾名思义，专用服务器仅专用于托管游戏，而没有本地玩家在此机器上玩游戏，因为连接的每位玩家均是客户端玩家

- 通常情况下，相比作为监听服务器运行，作为专用服务器运行性能因为没有图像或输入而更优化
- 默认情况下，在编辑器或独立游戏中玩游戏时，服务器类型设置为监听服务器



使用：**运行专用服务器**

- 单击 **Play** 按钮旁的下箭头，然后选中 **Run Dedicated Server** 复选框

  ![DedicatedServer.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/TestMultiplayer/DedicatedServer.jpg)







### 8. 蓝图中的多人功能



#### 8.1 Gameplay框架评估



说明：要向游戏添加多人功能，必须理解引擎提供的主要Gameplay类的角色，以及它们彼此的协作关系，尤其是它们在多人情境下如何工作：

- GameInstance
- GameMode
- GameState
- Pawn（以及从Pawn继承而来的Character）
- PlayerController
- PlayerState



使用：参阅[Gameplay框架](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Framework)文档以了解更多信息，但在设计多人游戏模式时，至少需要记住以下提示：

- `GameInstance`
  - 在引擎会话的持续时间内一直存在，意味着在引擎启动时创建，并在引擎关闭后才会销毁或更换
  - 服务器和每个客户端上都存在一个独立的GameInstance，这些实例彼此不通信
  - 由于GameInstance存在于游戏会话之外，并且是在关卡加载期间唯一存在的游戏结构体，因此非常适合于保存特定类型的持久数据，如终生玩家统计信息（如获胜总次数）、帐户信息（如特殊物品的锁定/解锁状态），甚至在像《虚幻竞技场》等竞技游戏中，用来保存用来切换的地图列表
- `GameMode`
  - 对象仅存在于服务器上
  - 它通常存储客户端不需要明确知道的游戏信息
  - 例如，如果游戏有"仅火箭发射装置"等特殊规则，客户端或许不需要知道这条规则，但在地图上随机产生武器时，服务器需要知道仅从"火箭发射装置"类别中选取
- `GameState`
  - 存在于服务器和客户端上，因此服务器可以在GameState上使用复制变量让所有客户端保持最新的游戏数据
  - 与所有玩家和旁观者有关、而不是与任何一个特定玩家有关的信息最适合于GameState复制
  - 例如，棒球游戏可以通过GameState复制每个团队的分数和当前局次
- `PlayerController`
  - 每一台客户端上的每一个玩家存在一个`PlayerController`
  - 它们在服务器和关联的客户机之间进行复制，但不会复制到其他客户端，因此在服务器上每个玩家都有PlayerController，但本地客户端只有本地玩家的PlayerController
  - 客户端保持连接时存在PlayerController，PlayerController与Pawn关联，但不会像Pawn一样被销毁和重新产生
  - 它们非常适用于在客户端和服务器之间传达信息，而不必将该信息复制到其他客户端
  - 例如，服务器告知客户端对其迷你地图进行ping，以响应只有该玩家能检测到的游戏事件
- `PlayerState`
  - 服务器和客户端上存在与游戏相连的每个玩家的`PlayerState`
  - 这个类可以用于所有客户端感兴趣的复制属性，而不仅仅是所属客户端，如单个玩家在自由竞赛游戏中的当前分数
  - 与PlayerController类似，它们与单个Pawn关联，但不会像Pawn那样被销毁和重新产生
- `Pawn（包括Character）`
  - 也存在于服务器和所有客户端上，可以包含复制变量和事件
  - 决定对特定变量或事件使用PlayerController、PlayerState还是Pawn取决于具体情况，但务必要记住的是，只要所属玩家保持与游戏相连，且游戏没有加载新关卡，则PlayerController和PlayerState就保持不变，而`Pawn`则不然
  - 例如，如果Pawn在游戏期间死亡，它通常会被销毁并替换为一个新Pawn，而PlayerController和PlayerState将持续存在，并在新Pawn产生后与新Pawn关联
  - 因此，Pawn的生命值将存储在Pawn自身上，因为该信息特定于该Pawn的实际实例，应在该Pawn替换为新Pawn时重置





#### 8.2 Actor复制



![replicates.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Blueprints/replicates.jpg)

- UE4中的联网技术的核心是Actor复制
- "复制"标志设置为true的Actor将自动从服务器同步到与该服务器相连的客户端
- 必须理解的一点是，Actor仅从服务器复制到客户端，而不能从客户端复制到服务器
- 当然，客户端仍需要能够向服务器发送数据，它们通过复制的"在服务上运行"事件来实现

请参阅概[在蓝图中同步 Actor](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/Actors/ReplicatingActors)指南以了解具体示例的逐步演示，以及[Actor 的复制](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/Actors)文档





##### 8.2.1 权限



- 对于场景中的每个Actor，会将其中一个连接的玩家视为拥有对该Actor的权限
- 对于服务器上存在的每个Actor，服务器拥有对该Actor的权限，包括所有复制的Actor
- 因此，**拥有权限** 函数在客户端上运行，而目标是复制到客户端的Actor时，将返回false
- 你还可以使用 **切换拥有权限** 便捷宏，作为针对复制Actor中不同服务器和客户端行为进行分支的快捷方法

![switch_has_authority.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Blueprints/switch_has_authority.jpg)





##### 8.2.2 变量



- 在Actor上的变量的细节面板中，有一个 **复制（Replication）** 下拉列表，让你可以控制如何复制变量

![variable_replication.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Blueprints/variable_replication.jpg)

| 选项                      | 说明                                                         |
| :------------------------ | :----------------------------------------------------------- |
| **无（None）**            | 这是新变量的默认值，表示不通过网络向客户端发送值             |
| **复制（Replicated）**    | 服务器复制该Actor时，它会将该变量发送到客户端。接收客户端上的变量值将自动更新，因此下一次访问该客户端时，它会反映服务器上的值<br />当然，在真实网络上游戏时，更新会有所延迟，具体取决于网络的延迟时间<br />请记住，复制的变量仅沿一个方向传递，从服务器到客户端！要从客户端向服务器发送数据，请参见"事件"部分 |
| **复制通知（RepNotify）** | 变量将按照"复制"（Replicated）选项进行复制，此外，还将在蓝图中创建`OnRep_`函数<br />当该变量的值发生变化时，引擎会自动在客户端和服务器上调用该函数<br />你可以根据游戏需要随意实现该函数<br /> ![onrep.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Blueprints/onrep.jpg) |

- 引擎内置类中的许多变量已经启用了复制，因此许多功能在多人情境下会自动工作

请参阅概[在蓝图中同步变量](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/Actors/ReplicatingVariables)指南以了解具体变量复制示例的逐步演示，以及[属性复制](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/Actors/Properties)文档





##### 8.2.3 产生和销毁



介绍：

- 在服务器上产生复制的Actor时，这个信息会传达给客户端，他们也会自动产生该Actor的副本
- 但总体而言，由于不会从客户端复制到服务器，因此如果在客户端上产生了复制的Actor，则该Actor只会存在于产生这个Actor的客户端上



影响：

- 服务器和任何其他客户端都不会收到该Actor的副本
- 但产生客户端拥有对该Actor的权限
- 这对于某些Actor还是十分有用的，比如对Gameplay没有实际影响的装饰Actor，但对于会影响Gameplay并应当复制的Actor，最好确保在服务器上产生这类Actor



销毁复制的Actor也属于类似情况：

- 如果服务器销毁一个复制的Actor，则所有客户端也会销毁相应的副本
- 客户端可以随意销毁他们拥有权限的Actor，即，客户端自己产生的Actor，因为这些Actor不会复制到其他玩家，也不会影响其他玩家
- 如果客户端尝试销毁没有权限的Actor，这个销毁请求会被忽略
- 关键在于，产生Actor亦是如此：如果你需要销毁复制的Actor，则需要在服务器上销毁





#### 8.3 事件复制



介绍：

- 在蓝图中，除了复制Actor及其变量，还可以跨客户端和服务器运行事件

请参阅[在蓝图中使用远程调用函数](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/Actors/ReplicatingFunctions)指南以了解具体示例的逐步演示，以及[RPC](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/Actors/RPCs)文档



注：

- 可能还会看到RPC（远程过程调用）一词
- 如果看到，只需注意蓝图中的复制事件基本上编译到引擎中的RPC，这是C++中的常用说法





##### 8.3.1 所有权



介绍：

- 在使用多人时需要理解的一个重要概念是（尤其是在使用复制事件时），哪个连接[被视为特定Actor或组件的所有者](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/Actors/OwningConnections)

- 就我们的目的而言，需要知道"在服务器上运行"事件只会从客户端拥有的Actor（或其组件）调用

- 通常，这表示只能从以下Actor或其中某个Actor的组件发送"在服务器上运行"事件：

  - 客户端本身的PlayerController

  - 客户端PlayerController支配的Pawn

  - 客户端的PlayerState



注：

- 同样，对于发送"在所属客户端上运行"事件的服务器，这些事件也应当在上述某个Actor上调用
- 否则，服务器不知道要将事件发送到哪个客户端，这样就只会在服务器上运行







##### 8.3.2 事件



- 在自定义事件的细节面板中，你可以设置如何复制事件

![event_replication.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Blueprints/event_replication.jpg)

| 选项                                           | 说明                                                         |
| :--------------------------------------------- | :----------------------------------------------------------- |
| **不复制（Not Replicated）**                   | 这是默认值，表示该事件不进行复制<br />如果在客户端上调用，它仅在该客户端上运行，如果在服务器上调用，则仅在服务器上运行 |
| **组播（Multicast）**                          | 如果在服务器上调用组播事件，则该事件将复制到所有连接的客户端，无论拥有目标对象的是哪个连接<br />如果客户端调用组播事件，该事件会被视为未复制，并仅在调用该事件的客户端上运行 |
| **在服务器上运行（Run on Server）**            | 如果从服务器调用该事件，则仅在该服务器上运行<br />如果从客户端调用，并且目标归该客户端所有，则事件将复制到服务器并在服务器上运行<br />"在服务器上运行"事件是客户端向服务器发送数据的主要方法 |
| **在所属客户端上运行（Run on Owning Client）** | 如果从服务器调用，则该事件将在拥有目标Actor的客户端上运行<br />由于服务器可以拥有Actor本身，因此"在所属客户端上运行"事件实际上可以在服务器上运行，而无论名称为何<br />如果从客户端调用，该事件会被视为未复制，并仅在调用该事件的客户端上运行 |





以下各表描述了不同复制模式根据调用方式对事件运行位置的影响

- 如果事件从服务器调用，假设左列是目标，则运行位置将是...

|                    | 未复制 | 组播               | 在服务器上运行 | 在所属客户端上运行 |
| :----------------- | :----- | :----------------- | :------------- | :----------------- |
| **客户端拥有目标** | 服务器 | 服务器和所有客户端 | 服务器         | 目标的所属客户端   |
| **服务器拥有目标** | 服务器 | 服务器和所有客户端 | 服务器         | 服务器             |
| **无主目标**       | 服务器 | 服务器和所有客户端 | 服务器         | 服务器             |

- 如果事件从客户端调用，假设左列是目标，则运行位置将是...

|                          | 未复制     | 组播       | 在服务器上运行 | 在所属客户端上运行 |
| :----------------------- | :--------- | :--------- | :------------- | :----------------- |
| **目标归调用客户端所有** | 调用客户端 | 调用客户端 | 服务器         | 调用客户端         |
| **目标归其他客户端所有** | 调用客户端 | 调用客户端 | 丢弃           | 调用客户端         |
| **服务器拥有目标**       | 调用客户端 | 调用客户端 | 丢弃           | 调用客户端         |
| **无主目标**             | 调用客户端 | 调用客户端 | 丢弃           | 调用客户端         |

- 从上表可见，任何从一个客户端调用并且没有设为"在服务器上运行"的事件都被视为没有复制



注：

- 从客户端向服务器发送复制的事件是唯一一种从客户端向服务器传达信息的方法，因为一般的Actor复制设计为仅从服务器到客户端

- 此外，还需注意组播事件只能从服务器发送
- 虚幻采用客户端-服务器模型，因此客户端不会直接连接任何其他客户端，它们只是连接到服务器
- 因此，客户端无法直接向其他客户端发送组播事件，只能与服务器通信
- 但你可以使用两个复制事件来模拟这种行为：一个"在服务器上运行"事件和一个"组播"事件
- "在服务器上运行"事件实现可以在需要时执行验证，然后调用组播事件
- "组播"事件实现将执行你想要对所有连接玩家运行的逻辑
- 下图代表了不执行任何验证的示例：

![forward_multicast.png](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/Blueprints/forward_multicast.jpg)







###### 8.3.2.1 "进行中加入"注意事项



说明：

- 在使用复制事件以传达游戏状态更改时需要记住的一点是它们如何与支持"进行中加入"的游戏交互
- 如果玩家在游戏进行期间加入游戏，则加入前发生的任何复制事件都不会对新玩家执行
- 重点是，如果你希望游戏正确地支持"进行中加入"，通常最好通过复制的变量同步重要的Gameplay数据
- 经常用的一种模式是客户端在场景中执行一些操作，通过"在服务器上运行"事件将操作通知给服务器，然后在该事件的实现中，服务器根据该操作更新一些复制的变量
- 然后没有执行该操作的其他客户端仍会通过复制的变量看到操作结果
- 此外，任何在操作发生后在进行期间加入的客户端仍会看到场景的正确状态，因为它们从服务器获得了最新的复制变量值
- 如果服务器仅发送事件，则进行中加入的玩家不会知道之前执行的操作





###### 8.3.2.2 可靠性



介绍：

- 对于任何复制的事件，你可以选择是 **可靠** 还是 **不可靠**



可靠事件：

- 可靠事件保证可到达目标（假设遵守上述所有权规则），但为了兑现这个承诺，它们会使用更多带宽，还可能会存在延迟
- 尽可能频繁地尝试避免发送可靠事件，例如每个tick事件，因为引擎的内部可靠事件缓冲区可能会溢出，如果溢出，则关联的玩家会断开连接



不可靠事件：

- 不可靠事件顾名思义，它们不能到达目标，比如网络上丢包，或者引擎确定有许多更高优先级的流量需要发送
- 因此，不可靠事件使用的带宽低于可靠事件，你可以更频繁地安全调用此类事件









### 9. Replication Graph



介绍：

- 概述Replication Graph功能
- Replication Graph节点



说明：

- **Replication Graph** 插件是一个用于多人游戏的网络复制系统，它的设计可以很好地适应大量玩家和复制Actor
- 例如，Epic自己的[Fortnite Battle Royale](https://www.epicgames.com/fortnite)从一开始就支持每场比赛100名玩家，包含大约50,000个复制的Actor
- 标准的网络复制策略（要求每个复制的Actor决定是否应该向每个连接的客户端发送更新）在这种情况下表现很差，并且会限制服务器的CPU
- 像将Actor划分为交错的组，或者只是降低更新频率，这样的解决方案可能会缓解这个问题，但降低更新频率也会降低客户端体验
- Replication Graph消除了Actor单独评估每个连接的客户端的需要，同时在不牺牲客户端体验的情况下，解决了CPU性能问题





#### 9.1 结构



介绍：

- Replication Graph包含一系列 **Replication Graph节点**，它们负责按需创建要复制到每个客户端的Actor列表
- 由于此系统是由持久对象构建的，而不是仅仅由复制的Actor自己处理函数调用，因此数据可以跨多个帧存储并在客户端连接之间共享
- 这种持久、共享的数据减少了Replication Graph系统为每个客户端生成复制列表所需的时间



说明：

- Replication Graph节点（我们简称为"节点"）执行以下实际工作：确定哪些Actor可能需要更新、将它们分组、存储要发送到客户端的预计算列表等
- 它们的最终任务是，尽可能快地为每个客户端连接按需提供Actor"复制列表"，以便服务器在每个Actor、每个客户端上花费尽可能少的CPU周期
- 每个节点都可以以一种独特的方式运行，鼓励开发者根据需要为自己的游戏编写自定义节点
- 节点可能与游戏无关，也可能利用特定于游戏的信息
- 根据Actor在游戏中的角色将它们放到不同的节点中，可以让你更好地控制它们复制的方式和时间
- 构建新节点，并根据Actor在游戏中的行为方式使用Replication Graph将Actor分配到最佳节点，这将极大地减少服务器CPU为准备网络复制列表所花费的时间





#### 9.2 启用此系统



你可以通过以下两种方式之一，配置你的项目使用自定义 **Replication Driver**（Replication Graph的父类）：

- 在"DefaultEngine.ini"文件中指定Replication Driver类
- 将返回Replication Driver类实例的函数绑定到Replication Driver的默认创建委托



注：

- ShooterGame项目是如何设置和实现Replication Graph的一个很好的示例
- 但是，请注意Replication Graph在控制台版本中是禁用的，因为它目前不适用于分屏游戏





##### 9.2.1 配置(.ini)文件



使用：

- 要配置引擎的默认Replication Driver

  1. 打开项目的"DefaultEngine.ini"文件

  2. 找到（或添加）`[/Script/OnlineSubsystemUtils.IpNetDriver]`部分，并设置（或添加）"ReplicationDriverClassName"条目，以便它指示希望使用的Replication Driver（或Replication Graph）类的名称

  3. 这应该大致如下所示，用实际项目的名称替换"ProjectName"，用自定义类名称替换"ClassName"：

     ```c++
     [/Script/OnlineSubsystemUtils.IpNetDriver]
     ReplicationDriverClassName="/Script/ProjectName.ClassName"
     ```

     



##### 9.2.2 在代码中绑定



使用：

- 如果你的项目有多个游戏模式或地图，而这些模式或地图的网络要求有很大的不同，那么绑定到委托将使你能够在代码中为当前游戏模式或地图创建适当的Replication Driver

- 要使用此方法，将一个函数绑定到名为`CreateReplicationDriverDelegate`的`UReplicationDriver`函数

- 绑定函数必须返回所需Replication Driver类的一个有效实例，就像下面的lambda函数示例所做的：

  ```c++
  UReplicationDriver::CreateReplicationDriverDelegate().BindLambda([](UNetDriver* ForNetDriver, const FURL& URL, UWorld* World) -> UReplicationDriver*
  {
  	return NewObject<UMyReplicationDriverClass>(GetTransientPackage());
  });
  ```

  



#### 9.3 高级示例



说明：

- 对于具有大量连接客户端甚至更多同步Actor的游戏而言，根据不同节点的类型和状态分配Actor的Replication Graph可以节省大量CPU时间
- 这使得构建传统复制方法无法实现的游戏成为可能



- 在概念层面上，这种规模的游戏可以构建具有以下功能的Replication Graph和Replication Graph节点，以处理数量庞大的复制Actor和连接客户端：

  - **根据位置将Actor分组**

    - 场景可以划分为游戏的网格空间（在大逃杀、MOBA或MMORPG流派游戏中），预定义房间或区域（对于地牢爬行游戏或走廊风格的第一人称或第三人称射击游戏），或任何适合你游戏空间的方法
    - 将Actor添加到每个网格单元或房间中，以便可以从其中潜在地看到或听到Actor，这将加快客户端的更新速度，因为节点可以简单地为客户端摄像机所在的任何网格单元或房间内，向客户端提供持久性Actor列表

    

  - **确定"休眠"放置的Actor，并将它们放在单独的列表中**

    - 虽然一些Actor，比如那些表示玩家或由AI控制角色的Actor，可能需要频繁的更新，但也有许多Actor可能是预先放置在关卡中的，在玩家与它们交互之前，它们是不会自己移动或改变状态的
    - 这些Actor可能会维持很长时间（可能是整个游戏会话），而无需发送网络更新
    - 例如，在Fortnite Battle Royale中，玩家和发射物可能会不断地更新，直到从游戏中移除为止
    - 另一方面，树可能会休眠很长时间，无需对任何客户端进行更新
    - 当树被损坏时，任何能够看到树的客户端都会需要接收关于此树的更新
    - 最后，当树被销毁时，任何接收描述树被销毁更新的客户端都不需要接收关于此树的任何进一步更新

    

  - **如果你游戏中的角色** **能够拾取并携带物品，随物品的携带者一起更新这些物品**

    - 当一个玩家取出一个物品或者武器并携带它，或者穿一件衣服或盔甲时，将表示该物品的Actor（假设这是一个单独的Actor，而不仅仅是一个组件）添加到一个特殊组，该组始终在拥有玩家更新时进行更新，否则绝不进行更新

    

  - **制定所有客户端始终已知的特殊Actor的列表**

    - 特殊Actor始终通过与每个玩家相关，并且可以放在一个简单的节点中来跟踪这些Actor，这样就可以将它们排除在其他列表之外，它们可能会占用CPU周期来进行不必要的计算

    

  - **制定始终（或从不）与特定客户端相关的特殊Actor的列表**

    - 可以为单个玩家或玩家团队创建一个类似的始终相关的列表节点
    - 这对于类似以下情况特别有用：确保玩家的队友始终更新，或者通过特殊的游戏内检测能力"暴露"对手，使对手对玩家的整个团队都是可见。如果"暴露"到期，可以将这些Actor添加回它们的默认节点





注：

- 构建一个Replication Graph，并根据对Actor在游戏中角色的了解，明智地将Actor分配到不同的节点，这可以充分利用服务器的CPU时间
- 最终的结果是为游戏提供稳定的服务器性能，否则游戏将无法在当前硬件上运行
- Replication Graph插件包含几个Replication Graph节点类，你可以在大型在线游戏中使用它们
- 此外，鼓励开发者基于对特定游戏内部工作原理的了解来构建自定义节点类









### 10. 多人游戏中的关卡切换



#### 10.1 无缝与非无缝切换



介绍：

- UE4 中主要有两种转移方式：
  - 无缝和非无缝方式
  - 两者的主要区别在于，无缝转移是一种非阻塞（non-blocking）操作，而非无缝转移则是一种阻塞（blocking）操作
- 当客户端执行非无缝转移时，客户端将与服务器断开连接，然后重新连接到同一服务器，而服务器将准备新的地图以供加载



建议：

- UE4 多人模式游戏尽量采用无缝转移
- 这样做通常可以提供更流畅的体验，同时避免重新连接过程中可能出现的问题



有三种情形中必然产生非无缝转移：

- 初次加载地图时
- 初次作为客户端连接服务器时
- 想要终止一个多人模式游戏并启动新游戏时

有三个用来驱动转移的主要函数：`UEngine::Browse`、`UWorld::ServerTravel` 和 `APlayerController::ClientTravel`

在确定使用哪个函数时，您可能会感到有些困惑，所以请遵循下面的准则：



详细：

`UEngine::Browse`

- 就像是加载新地图时的硬重置
- 将始终导致非无缝切换
- 将导致服务器在切换到目标地图前与当前客户端断开连接
- 客户端将与当前服务器断开连接
- 专用服务器无法切换至其他服务器，因此地图必须存储在本地（不能是 URL）



`UWorld::ServerTravel`

- 仅适用于服务器
- 会将服务器跳转到新的世界/场景
- 所有连接的客户端都会跟随
- 这就是多人游戏在地图之间转移时所用的方法，而服务器将负责调用此函数
- 服务器将为所有已连接的客户端玩家调用 `APlayerController::ClientTravel`



`APlayerController::ClientTravel`

- 如果从客户端调用，则转移到新的服务器
- 如果从服务器调用，则要求特定客户端转移到新地图（但仍然连接到当前服务器）





#### 10.2 启用无缝切换



前提：启用无缝切换，您需要设置一个过渡地图

- 需要通过 `UGameMapsSettings::TransitionMap` 属性进行配置
- 该属性默认为空，如果您的游戏保持这一默认状态，就会为过渡地图创建一个空地图



原理：

- 因为必须始终有一个被加载的世界（用于存放地图），所以在加载新地图之前，我们不能释放原有的地图
- 由于地图可能会非常大，因此让新旧地图同时存放在存储器内绝对是个坏主意，这时就需要过渡地图来帮忙了



好处：由于过渡地图非常小，因此在"中转"当前地图和最终地图时不会造成太大的资源消耗



注意：设置好过渡地图后，您需要将 `AGameModeBase::bUseSeamlessTravel` 设置为 true，这样就可以实现无缝切换了







#### 10.3 无缝切换流程



下面是执行无缝切换时的一般流程：

1. 标记出要在过渡关卡中存留的 actor（更多信息请见下面）
2. 转移到过渡关卡
3. 标记出要在最终关卡中存留的 actor（更多信息请见下面）
4. 转移到最终关卡







#### 10.4 无缝切换中的存留 Actor



介绍：

- 在使用无缝切换时，可以将（存留） actor 从当前关卡带到新的关卡

- 这适用于一些特定的 actor，如道具栏物品和玩家等



默认情况下，这些 actor 将自动存留：

- `GameMode` actor（仅限服务器）
  - 通过 `AGameModeBase::GetSeamlessTravelActorList` 额外添加的任何 actor
- 拥有一个有效的 `PlayerState` （仅限服务器）的所有控制器
- 所有 `PlayerControllers` （仅限服务器）
- 所有本地 `PlayerControllers` （服务器和客户端）
  - 通过 `APlayerController::GetSeamlessTravelActorList` （在本地`PlayerControllers`上调用）额外添加的任何 actor









### 11. 网络性能分析工具



介绍：

- Network Profiler 是一款独立工具，可用于显示网络流量和性能信息，上述信息可在游戏运行时由引擎捕捉
- 它是一种极其有效的方式，可识别多人游戏中占用极高带宽的区域，因为您可以查看各个 actor、PRC 和属性在总带宽中的占比





#### 11.1 录制分析会话



前提：

- 在使用 network profiler 之前，您必须录制相同的数据才能进行分析

- 为此，您需要使用启用了进程状态跟踪功能的引擎，通常这说明是个调试 build、用于非调试配置或编辑器 build（更具体地来说，对引擎进行编译时，STATS 宏已定义为非零值）

- 还可以在引擎启动的时候增加 "networkprofiler=true" 命令行参数，以便于引擎已启动就能够开始记录，也可以用以下这些命令行指令在运行时控制是否要记录网络数据：

  - netprofile：切换录制的开启和关闭

  - netprofile enable：如果尚未录制，则开始录制

  - netprofile disable：如果当前在录制，则停止录制



使用：

- 数据将被保存至 `<project directory>/Saved/Profiling/<project name>-<timestamp>.nprof`
- 如果您在相同文件夹中看到一个名为 NetworkProfiling.tmp 的文件，这只是引擎在收集数据时使用的临时文件
- 当前分析会话停止时，该文件将根据以上方案进行重命名，然后即可在独立工具中打开它









#### 11.2 查看分析会话



**`Chart, Filters, Details选项卡`**

- 默认条件下，选定的是 "Chart, Filters, Details（图表、筛选器和详细信息）" 选项卡
- 下文将对此选项卡的不同区域进行介绍

![image alt text](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/NetworkProfiler/image_1.jpg)

1. [图表视图](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/NetworkProfiler/#图表视图)
2. [统计数据列表](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/NetworkProfiler/#统计数据列表)
3. [汇总视图](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/NetworkProfiler/#汇总视图)
4. [帧详细信息](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/NetworkProfiler/#帧详细信息)
5. [筛选器](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/NetworkProfiler/#筛选器)
6. [性能视图](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/NetworkProfiler/#性能视图)



**`图表视图`**

- 这是主图表视图。它类似于其他分析应用程序中的图表，可显示随时间推移已启用统计数据的图表
- 单击特定帧可使用该帧相关数据填充其他视图，也可在图表上点击并拖动以放大相应区域，并查看该时间范围内的汇总情况。



**`统计数据列表`**

- 该列表中包含了 network profiler 所跟踪的所有统计数据
- 切换复选框将显示或隐藏图表中的特定统计数据。对于大多数统计数据，您可选择原始计数、每秒计数、字节数或每秒字节数



**`汇总视图`**

- 该视图中显示图表中所选时间范围内的数据汇总
- 其中显示了原始总数以及每秒信息
- 注意，"Outgoing bandwidth（输出带宽）" 将大于 "Game socket send size（游戏套接字发送大小）"，因为 "Outgoing bandwidth（输出带宽）" 也包括了 IP 和 UDP 标头大小的近似值



**`帧详细信息`**

- 该视图对当前选定帧所发送的数据进行了分类统计
- 可以查看发送了哪些 actor、属性和远程函数，以及各项分别占到多少字节
- 注意，数据组中包括了 actor 和 PRC，因此 "SEND BUNCH" 部分中的 "NumBytes" 字段中包含了 "SEND RPC" 和 "REPLICATE ACTOR" 部分的字节
- 最后，"SOCKET SEND TO" 条目将精确显示引擎发送到输出套接字的字节数
- 注意，此处不包含标准 UDP 数据包资源消耗



**`筛选器`**

- 可为 actor、属性和 RPC 输入筛选器，在单击 "Apply Filters（应用筛选器）" 按钮后，图表将更新为只显示包含筛选器字段中输入文本的相关 actor、属性和 RPC
- 在应用了筛选器之后，选择图表中的单个帧或一系列帧也将相应地筛选汇总和帧详细信息视图中的数据



**`性能视图`**

- 选择了图表中的单个帧时，该视图将显示该帧内复制的 actor 类型列表，并按照 CPU 复制它们所需的时间排序
- 如果某个 actor 具有相关的复制属性，则展开该 actor 的树形视图将显示各属性
- 在 actor 选项卡中，第一栏数值表示 CPU 复制该 actor 所花费的时间（以毫秒计）
- 第二栏是用于表示数据包中 actor 的字节数，第三栏是当前帧中复制的指定类型的 actor 数
- 在属性行中，第一列是用于属性的字节数，第二列表示为 actor 复制了多少具有指定名称的属性



**`"Actors" 选项卡、"Properties（属性）" 选项卡和 "RPCs" 选项卡`**

![image alt text](https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Networking/NetworkProfiler/image_2.jpg)

- 每个选项卡都显示了整个分析会话中各复制数据的汇总情况
- 注意，这些选项卡中显示的数据不受 "Chart, Filters, Details" 选项卡的图表中选定的当前帧或时间范围的影响
- 选项卡上分别显示了 actor、属性或 PRC 的列表
- "Total Size（总体大小）" 和 "Average Size（平均大小）" 列显示了特定项需要多少带宽，"Count（计数）" 列表示在分析会话期间复制了多少次特定项。单击各列的表头，可在升序或降序之间切换







#### 11.3 服务器和客户端



- 从客户端和服务器都可录制分析数据
- 但是，请注意，由于只有服务器复制 actor 及其属性，因此在查看客户端上录制的分析数据时，只能看到从该客户端发送的 RPC 的详细数据









### 12. 性能与带宽提示



介绍：

- 复制 actor 是一件耗费时间的工作

- 引擎会尽量让这个过程变得更有效率，但您也可以做一些额外的工作来简化这个过程





使用：在收集 actor 用于复制时，服务器将检查一些事项，如相关性、更新频度、休眠情况等。您可以调整这些检查项以改善性能

要最大限度提升这一过程的效率，最好是遵循以下优先顺序：

- 关闭复制（`AActor::SetReplicates( false )`）
  - 当 actor 未进行复制时，它最初不会出现在列表中，我们可以充分利用这一点，确保那些无需复制的 actor 会有相应标记
- 减少 NetUpdateFrequency 值
  - actor 的更新次数越少，更新所用的时间就越短
  - 最好是尽量压低这个数值。该数值代表了这个 actor 每秒复制到客户端的频度
- 休眠情况
- 相关性
- NetClientTicksPerSecond


注：

- 如果属性并非是绝对必需，则不要将其标记为复制

- 如果可以，最好能尝试从现有的已复制属性中派生状态

- 尝试利用已有的量化函数，如 `FVector_NetQuantize`

- 这样能大大减少向客户端复制此状态时所需的大小，如果使用得当，就不会导致任何明显的偏差

- `FName` 一般不会被压缩，所以在使用它们作为 RPC 的参数时，请记住它们通常会向字符串发送所有调用

- 这会产生很大的资源消耗

# UE4 NetWorking



[toc]



### 1. Introduction



- **网络拓扑和NAT**
- **C++ Replication**
- **Low Level NetWorking**
- **诊断工具**



### 2. Network Address Translation



- **Network Address Translation (NAT)**



三种类型：

- **Open**：接受未经请求的链接
- **Moderate (中等)**：有时可以接受未经请求的链接
- **Strict (严格)**：只能接受知道地址的链



### 3. NAT Implications



- **Clinet / Server**
  - 服务器应该开放
  - 客户端可以是任何类型的NAT
- **Peer / Peer**
  - 所有客户端都必须开放NAT
  - 约有20%的玩家无法加入游戏



### 4. Server Authoritative



- **客户端只能与服务器对话**
- **服务器复制到客户端**
- **数据流是服务器->客户端**
- **RPC是双向的**



### 5. C++ Replication



#### 5.1 Basic Gameplay Classes



| Server                                                       | Client                                                       |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `GameMode`<br />`GameState`<br />`PlayerController`:**每个玩家一个**<br />`PlayerState`:**每个玩家一个** | `GameState`<br />`PlayerController`:**每个玩家一个**<br />`PlayerState`:**每个玩家一个** |



#### 5.2 Gameplay Classes



- **GameMode**：制定游戏规则
- **GameState**：关于**GameMode**的复制信息
- **PlayerController**：和**GameMode**的主要交互
- **PlayerState**：玩家的复制信息



#### 5.3 Data Replication



- **bReplicates**：设置为Actor构造函数的一部分
- **UPROPERTY Macro**：
  - `Replicated`
  - `ReplicatedUsing`
- **GetLifetimeReplicatedProps Function**：确定复制的属性集



#### 5.4 Conditional Replication



- 仅在条件满足时复制属性
- 可以减少带宽消耗
- 通过类似的宏完成



**Common Replication Conditions**：

- **COND_InitialOnly**：此属性将仅尝试在初始复制时发送
- **COND_OwnerOnly**：这个属性只会发送给所有者
- **COND_SkipOwner**：此属性发送到除所有者之外的每个连接
- **COND_SimulatedOnly**：此属性只会发送给模拟演员
- **COND_AutonomousOnly**：该属性只会发送给自治参与者



#### 5.5 Function Replication



**UFUNCTION Macro**

- `Reliable`
- `Unreliable`
- `Client`
- `Server`
- `NetMulticast`
- `WithValidation`
- `BlueprintAuthorityOnly`:(仅蓝图授权)
- `BlueprintCosmetic`



##### 5.5.1 Reliable



- 函数被保证调用
- 网络出现错误时重新发送
- 带宽饱和时会延迟



```c++
/** notify player about started match */
UFUNCTION(Reliable, Client)
void ClientGameStarted();

void AShooterPlayerController::ClientGameStarted_Implementation()
{
	bAllowGameActions = true;
	// Enable controls and disable cinematic mode now the game has started
	SetCinematicMode(false, false, true, true, true);
	AShooterHUD* ShooterHUD = GetShooterHUD();
	if (ShooterHUD)
	{
		ShooterHUD->SetMatchState(EShooterMatchState::Playing);
		ShooterHUD->ShowScoreboard(false);
	}
	…
}
```



##### 5.5.2 Unreliable



• 试图发送函数
• 出现错误时不再发送
• 带宽饱和时跳过



```c++
/** Replicated function sent by client to server - contains
client movement and view info. */
UFUNCTION(Unreliable, Server, WithValidation)
virtual void ServerMove(float TimeStamp, …);

void UCharacterMovementComponent::ServerMove_Implementation(float TimeStamp, …)
{
	…
}
```



##### 5.5.3 NetMulticast



- 发送给所有的客户端：`Reliable`和`Unreliable`也同样适用



```c++
/** broadcast death to local clients */
UFUNCTION(Reliable, NetMulticast)
void BroadcastDeath(…);

void AShooterPlayerState::BroadcastDeath_Implementation(…)
{
	for (auto It = GetWorld()->GetPlayerControllerIterator(); It; ++It)
	{
		// all local players get death messages so they can update their huds.
		AShooterPlayerController* TestPC = Cast<AShooterPlayerController>(*It);
		if (TestPC && TestPC->IsLocalController())
		{
			TestPC->OnDeathMessage(KillerPlayerState, this, KillerDamageType);
		}
	}
}
```



##### 5.5.4 WithValidation



- 在目标函数之前调用
- 用于验证函数的参数：旨在检测作弊/黑客行为
- 返回值影响函数是否被调用：`false`跳过呼叫并终止连接



```c++
bool UCharacterMovementComponent::ServerMove_Validate(float TimeStamp, …)
{
	bool bIsValidMove = false;
	// Perform move validation here
	return bIsValidMove;
}
```



### 6. Actor Relevancy



- 以 CPU 时间换取网络带宽
- 基于距离
  - 这些演员是否足够接近
  - 现在的默认实现
- 视线
  - 这些演员可以互相看到吗
  - UE3 默认实现
- 始终相关是一个选项
  - 以带宽换取 CPU 时间



### 7. Low Level Details



- **UNetDriver**
- **UNetConnection**
- **UChannel**
  - **UControlChannel**
  - **UVoiceChannel**
  - **UActorChannel**



#### 7.1 UNetDriver



• 包含与 Tick 的连接列表
• 在客户端，一个连接
• 在服务器上，N 个连接



#### 7.2 UNetConnection



- 包含需要复制的通道列表
- **UChildConnection**：用作分屏游戏的优化



#### 7.3 UChannel Objects



- 逻辑结构：将数据路由到正确的对象
- 由 ChannelID 访问：某些频道具有预定义的 ID



#### 7.4 UControlChannel



• 处理连接握手
• 处理对象加载请求
• 处理杂项：非游戏性交流



#### 7.5 UVoiceChannel



• 发送和接收语音数据：语音通道将数据路由到平台处理程序
• 语音数据取决于平台
• 语音数据作为扬声器 ID 和有效负载发送



#### 7.6 UActorChannel



• 处理参与者复制：包括任何复制的组件
• 每个复制的参与者一个参与者通道
• 参与者按频道 ID 复制：根据阵列位置动态分配



### 8. Voice Considerations



• 游戏可以选择支持或不支持
• 平台可以让其他玩家静音：玩家在游戏外静音另一个
• 玩家可以静音其他玩家
• 游戏玩法可以让玩家静音：基于团队，基于距离，一键通



### 9. Diagnostic Tools



#### 9.1 Network Logging



- **LogNet**：有关通道和连接状态的详细信息
- **LogNetPlayerMovement**：有关来自客户端的移动和来自服务器的更正的详细信息
- **LogNetTraffic**：有关在连接上发送的数据的详细信息



#### 9.2 Network Statistics



- **Stat Net**：列出 ping、通道计数、输入/输出字节等
- **Stat Game**：网络处理信息列表



#### 9.3 Network Simulation Options



- **PktLag**：将数据包的发送延迟 N ms
- **PktLagVariance**：为 PktLag 选项提供一些随机性
- **PktLoss**：不发送数据包的百分比
- **PktDup**：发送重复数据包的百分比
- **PktOrder**：启用时乱序发送数据包



#### 9.4 Setting the Simulation Options



• **Console**
```ini
Net PktLag=100
```

• **INI File**

```ini
[PacketSimulationSettings]
PktLag=50
PktLagVariance=10
PktLoss=3
PktOrder=0
PktDup=10
```


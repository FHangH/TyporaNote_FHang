# UE4 功能整理





### 1. SpawnActor



情景：

- 我有一个`Cpp类`
- 这个`Cpp类`要生成一个其他`Cpp`或`蓝图`类
- 可以使用`TSubclassOf<>`



示例：

- 定义

  ```c++
  private:
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	TSubclassOf<AActor> SpawnActorTest;
  ```

- 实现

  ```c++
  void AActor_SpawnTest::BeginPlay()
  {
  	Super::BeginPlay();
  
  	FActorSpawnParameters SpawnParameters;
  	SpawnParameters.Name = FName("SpawnActorTest");
  	SpawnParameters.Owner = this;
  	SpawnParameters.Instigator = nullptr;
      
  	GetWorld()->SpawnActor<AActor>(SpawnActorTest, GetActorTransform(), SpawnParameters);
  }
  ```



说明：

- 定义中的`SpawnActorTest`是私有变量，若要保持私有，其蓝图可以`Get`, `Set`和参数列表中显示继承中：
  - `UPOPERTY()`中加入`BlueprintReadWrite`，`meta=(AllowPrivateAccess=true)`

- `FActorSpawnParameters`是可以自定义生成参数，默认可以不写
- `GetActorTransform()`可以是`GetActorLocation`和`GetActorRotation`







### 2. LineTrace



- Trace模式
  - TraceSingle 单个结果
  - TraceMulti 多个结果
- Trace 的检测依据
  - ByChanne
  - ByObjectType
  - ByProfile





#### 2.1 UWorld



**`LineTraceSingleByChannel`**

情景：

- 场景中的某个`Actor`需要发射检测射线
- 可以直接在`Actor`上写，也可以通过组件`SceneComponent`，`ActorComponent`
- 示例采用`SceneComponent`



Syntax：

```c++
bool LineTraceSingleByChannel(
	struct FHitResult& OutHit,
	const FVector& Start,
	const FVector& End,
	ECollisionChannel TraceChannel,
	const FCollisionQueryParams& Params = FCollisionQueryParams::DefaultQueryParam,
	const FCollisionResponseParams& ResponseParam = FCollisionResponseParams::DefaultResponseParam
) const;
```



示例：

- 定义

  ```c++
  private:
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category="Trace", meta=(AllowPrivateAccess=true))
  	float TraceDistance;
  ```

- 实现

  ```c++
  const FVector Start = GetOwner()->GetActorLocation();
  const FVector End = Start + (GetForwardVector() * TraceDistance);
  
  FHitResult HitResult;
  const bool IsHit = GetWorld()->LineTraceSingleByChannel(HitResult, Start, End, ECC_Visibility);
  
  DrawDebugLine(GetWorld(), Start, End, FColor::Red, false, 0.5f);
  
  if (!IsHit){return;}
  GEngine->AddOnScreenDebugMessage(-1, 1.f, FColor::Green,
  		FString::Printf(TEXT("Trace Hit: %s"), *HitResult.GetActor()->GetName()));
  ```

  

说明：

- `HitResult`会包含射线检测到的`Actor`的信息







#### 2.2 Kismet



##### 2.2.1 LineTraceSingle



情景：

- 根据 `Channel` 检测单个物体



Syntax：

```c++
static bool LineTraceSingle(
	const UObject* WorldContextObject,
	const FVector Start,
	const FVector End, 
	ETraceTypeQuery TraceChannel, 
	bool bTraceComplex, 
	const TArray<AActor*>& ActorsToIgnore, 
	EDrawDebugTrace::Type DrawDebugType,
	FHitResult& OutHit, 
	bool bIgnoreSelf, 
	FLinearColor TraceColor = FLinearColor::Red, 
	FLinearColor TraceHitColor = FLinearColor::Green, 
	float DrawTime = 5.0f
);
```



示例：

- 定义：

  ```c++
  private:
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category="Trace", meta=(AllowPrivateAccess=true))
  	float TraceDistance;
  ```



- 实现：

  ```c++
  const FVector Start = GetOwner()->GetActorLocation();
  const FVector End = Start + (GetForwardVector() * TraceDistance);
  
  FHitResult HitResult;
  const TArray<AActor*> ActorsToIgnore;
  const bool IsHit = UKismetSystemLibrary::LineTraceSingle(
  		this, Start, End, TraceTypeQuery1,
  		false, ActorsToIgnore, EDrawDebugTrace::ForDuration, HitResult, true,
  		FLinearColor::Red, FLinearColor::Green, 1.f);
  
  if (!IsHit){return;}
  GEngine->AddOnScreenDebugMessage(-1, 1.f, FColor::Green,
  		FString::Printf(TEXT("Trace Hit: %s"), *HitResult.GetActor()->GetName()));
  ```



说明：

- `ETraceTypeQuery` 说明
  - 默认 `TraceTypeQuery1` —— `Visibility`
  - 默认 `TraceTypeQuery2` —— `Camera`
  - 可在 `ProjectSettings`->`Engine`->`Collision`->`Trace Channels` 添加自定义





##### 2.2.2 LineTraceSingleForObjects



情景：

- 根据 `Object Type` 检测单个物体



Syntax

```cpp
static bool LineTraceSingleForObjects(
	const UObject* WorldContextObject,
	const FVector Start,
	const FVector End,
	const TArray<TEnumAsByte<EObjectTypeQuery> > & ObjectTypes,
	bool bTraceComplex,
	const TArray<AActor*>& ActorsToIgnore,
	EDrawDebugTrace::Type DrawDebugType,
	FHitResult& OutHit,
	bool bIgnoreSelf,
	FLinearColor TraceColor = FLinearColor::Red,
	FLinearColor TraceHitColor = FLinearColor::Green,
	float DrawTime = 5.0f 
);
```



示例：

- 实现：

  ```c++
  // 设置要检测的 Object Type
  TArray<TEnumAsByte<EObjectTypeQuery> > ObjectTypes;
  ObjectTypes.Add(EObjectTypeQuery::ObjectTypeQuery1);
  
  //开始检测
  bool bIsHit = UKismetSystemLibrary::LineTraceSingleForObjects(GetWorld(), BeginLoc, EndLoc, ObjectTypes, false, IgnoreActors, EDrawDebugTrace::ForDuration, HitResult, true);
  if (bIsHit)
  {
  	UKismetSystemLibrary::PrintString(GetWorld(), HitResult.GetActor()->GetName());
  }
  ```



说明：

- `EObjectTypeQuery` 对应 `ObjectType`
  - 默认 `ObjectTypeQuery1` —— `WorldStatic`
  - 默认 `ObjectTypeQuery2` —— `WorldDynamic`
  - 默认 `ObjectTypeQuery3` —— `Oawn`
  - 默认 `ObjectTypeQuery4` —— `PhysicasBody`
  - 默认 `ObjectTypeQuery5` —— `Vehicle`
  - 默认 `ObjectTypeQuery6` —— `Destructible`
  - 可以再 `ProjectSettings`->`Engine`->`Collision`->`Object Channels` 添加自定义





##### 2.2.3 LineTraceSingleByProfile



情景：

- 根据 `Collision Preset` 检测单个物体



Syntax

```cpp
static bool LineTraceSingleByProfile(
	const UObject* WorldContextObject,
	const FVector Start, 
	const FVector End, 
	FName ProfileName,
	bool bTraceComplex,
	const TArray<AActor*>& ActorsToIgnore,
	EDrawDebugTrace::Type DrawDebugType,
	FHitResult& OutHit,
	bool bIgnoreSelf,
	FLinearColor TraceColor = FLinearColor::Red,
	FLinearColor TraceHitColor = FLinearColor::Green,
	float DrawTime = 5.0f
);
```



示例：

- 实现：

  ```c++
  bool bIsHit = UKismetSystemLibrary::LineTraceSingleByProfile(
      GetWorld(), BeginLoc, EndLoc,TEXT("BlockAll"), 
      false, IgnoreActors, EDrawDebugTrace::ForDuration, 
      HitResult, true);
  
  if (bIsHit)
  {
  	UKismetSystemLibrary::PrintString(GetWorld(), HitResult.GetActor()->GetName());
  }
  ```



说明：

- `ProfileName` 对应 `Collision Preset` 的名称






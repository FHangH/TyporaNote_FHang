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









### 3. SweepTrace



情景：

- 使用`UWorld`生成

- 生成一个范围检测周边的`Actor`



示例：

- 定义：

  ```c++
  private:
  	UPROPERTY(EditDefaultsOnly, Category="Trace", BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float TraceRadius;
  ```



- 实现：

  ```c++
  TArray<FHitResult> HitResults;
  const FVector Start, End = GetOwner()->GetActorLocation();
  const FCollisionShape CollisionShape = FCollisionShape::MakeSphere(TraceRadius);
  
  const bool IsHit = GetWorld()->SweepMultiByChannel(
  			HitResults, Start, End, FQuat::Identity,
  			ECC_Visibility, CollisionShape);
  
  DrawDebugSphere(GetWorld(), Start, TraceRadius, 50, FColor::Green, true);
  
  if (!IsHit){return;}
  for (const auto &result : HitResults)
  {
  	GEngine->AddOnScreenDebugMessage(-1, 10.f, FColor::Green,
  		FString::Printf(TEXT("Hit: %s"), *result.GetActor()->GetName()));
  }
  ```



说明：

- `FQuat`四元数
  - `Indentity`：无旋转







### 4. SphereTrace



情景：

- 使用`Kismet`生成
- 生成一个范围检测周边的`Actor`



示例：

- 定义：

  ```c++
  private:
  	UPROPERTY(EditDefaultsOnly, Category="Trace", BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float TraceRadius;
  ```



- 实现：

  ```c++
  const FVector Start, End = GetOwner()->GetActorLocation();
  TArray<AActor*> ActorsToIgnore;
  ActorsToIgnore.Add(GetOwner());
  TArray<FHitResult> HitResults;
  	
  const bool IsHit = UKismetSystemLibrary::SphereTraceMulti(
  		this,
  		Start,
  		End,
  		TraceRadius,
  		TraceTypeQuery1, 
  		false,
  		ActorsToIgnore,
  		EDrawDebugTrace::ForDuration,
  		HitResults,
  		true,
  		FLinearColor::Green,
  		FLinearColor::Red,
  		60.f);
  	
  if (!IsHit){return;}
  for (const auto &result : HitResults)
  {
  	GEngine->AddOnScreenDebugMessage(
          -1, 60.f, FColor::Green,
  		FString::Printf(TEXT("Hit: %s"), 
          *result.GetActor()->GetName()));
  }
  ```









### 5. Character



情景：

- 第三人称
- 自由视角
- 角色朝向鼠标输入和控制器指向



示例：

- 准备：
  1. 编辑 >> 项目设置 >> 引擎 >> 输入
  2. 添加操作映射
     - Jump == 空格键
  3. 添加轴映射
     - MoveForward
       - W == 1.0
       - S == -1.0
     - MoveRight
       - D == 1.0
       - A == -1.0
     - PitchCamera
       - 鼠标Y == -1.0
     - YawCamera
       - 鼠标X == 1.0



- 定义：

  ```c++
  class USpringArmComponent;
  class UCameraComponent;
  
  private:
  	UPROPERTY(VisibleDefaultsOnly, BlueprintReadWrite, Category="Player", meta=(AllowPrivateAccess=true))
  	USpringArmComponent *PlayerSpringArmComponent;
  
  	UPROPERTY(VisibleDefaultsOnly, BlueprintReadWrite, Category="Player", meta=(AllowPrivateAccess=true))
  	UCameraComponent *PlayerCameraComponent;
  	
  public:
  	UFUNCTION()
  	void MoveForward(float Value);
  
  	UFUNCTION()
  	void MoveRight(float Value);
  ```



- 实现：

  ```c++
  ACharacterBase::ACharacterBase()
  {
  	PrimaryActorTick.bCanEverTick = true;
  
  	GetCapsuleComponent()->InitCapsuleSize(36.f, 92.f);
  	
  	PlayerSpringArmComponent = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
  	PlayerSpringArmComponent->SetupAttachment(RootComponent);
      PlayerSpringArmComponent->bUsePawnControlRotation = true;
      // 以下是详细配置
  	PlayerSpringArmComponent->SetRelativeLocation(FVector(0.f, 0.f, 90.f));
  	PlayerSpringArmComponent->TargetArmLength = 300.f;
  	PlayerSpringArmComponent->bEnableCameraLag = true;
  	PlayerSpringArmComponent->bEnableCameraRotationLag = true;
  	PlayerSpringArmComponent->CameraLagSpeed = 10.f;
  	PlayerSpringArmComponent->CameraRotationLagSpeed = 10.f;
  	PlayerSpringArmComponent->CameraLagMaxDistance = 100.f;
  
  	PlayerCameraComponent = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
  	PlayerCameraComponent->SetupAttachment(PlayerSpringArmComponent, USpringArmComponent::SocketName);
  	PlayerCameraComponent->bUsePawnControlRotation = false;
  
  	GetCharacterMovement()->bOrientRotationToMovement = true;
      // 以下是详细配置
  	GetCharacterMovement()->RotationRate = FRotator(0.f, 90.f, 0.f);
  	GetCharacterMovement()->GravityScale = 1.5f;
  	GetCharacterMovement()->MaxAcceleration = 980.f;
  	GetCharacterMovement()->JumpZVelocity = 600.f;
  	GetCharacterMovement()->AirControl = 0.2f;
  
  	bUseControllerRotationPitch = false;
  	bUseControllerRotationRoll = false;
  	bUseControllerRotationYaw = false;
  }
  
  void ACharacterBase::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
  {
  	Super::SetupPlayerInputComponent(PlayerInputComponent);
  
  	// Bind Axis => MoveForward, MoveRight
  	PlayerInputComponent->BindAxis(TEXT("MoveForward"), this, &ACharacterBase::MoveForward);
  	PlayerInputComponent->BindAxis(TEXT("MoveRight"), this, &ACharacterBase::MoveRight);
      // Pawn, Character 已经写好了内部的鼠标输入
  	PlayerInputComponent->BindAxis(TEXT("PitchCamera"), this, &ACharacter::AddControllerPitchInput);
  	PlayerInputComponent->BindAxis(TEXT("YawCamera"), this, &ACharacter::AddControllerYawInput);
  
  	// Bind Action => Jump
  	PlayerInputComponent->BindAction(TEXT("Jump"), IE_Pressed, this, &ACharacter::Jump);
  	PlayerInputComponent->BindAction(TEXT("Jump"), IE_Released, this, &ACharacter::StopJumping);
  }
  
  void ACharacterBase::MoveForward(float Value)
  {
      // 这种方法有问题：人物应用摄像机旋转，摄像机应用控制器旋转时，鼠标完全朝上或朝下，将无法正常行走
  	const FVector Direction = FRotationMatrix(GetController()->GetControlRotation()).GetScaledAxis(EAxis::X);
      
      // 这个方法简单，通用
      const FVector Direction = GetActorForwardVector();
  	AddMovementInput(Direction, Value);
  }
  
  void ACharacterBase::MoveRight(float Value)
  {
  	const FVector Direction = FRotationMatrix(GetController()->GetControlRotation()).GetScaledAxis(EAxis::Y);
  	AddMovementInput(Direction, Value);
  }
  ```









### 6. Pawn



情景：

- 使用`Pawn`去写非人型



示例：

- 定义：

  ```c++
  private:
  	UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, Category="Pawn", meta=(AllowPrivateAccess=true))
  	FVector MovementDirection;
  
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category="Pawn", meta=(AllowPrivateAccess=true))
  	float MovementSpeed;
  	
  public:
  	UFUNCTION()
  	void MoveForward(float Value);
  
  	UFUNCTION()
  	void MoveRight(float Value);
  ```



- 实现：

  ```c++
  void APawnBase::Tick(float DeltaTime)
  {
  	Super::Tick(DeltaTime);
  
  	if (!MovementDirection.IsZero())
  	{
  		const FVector NewLocation = GetActorLocation() + (MovementDirection * DeltaTime * MovementSpeed);
  		SetActorLocation(NewLocation);
  	}
  }
  
  void APawnBase::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
  {
  	Super::SetupPlayerInputComponent(PlayerInputComponent);
  
  	// Bind Axis => MoveForward, MoveRight
  	PlayerInputComponent->BindAxis(TEXT("MoveForward"), this, &APawnBase::MoveForward);
  	PlayerInputComponent->BindAxis(TEXT("MoveRight"), this, &APawnBase::MoveRight);
  }
  
  void APawnBase::MoveForward(float Value)
  {
  	MovementDirection.X = FMath::Clamp(Value, -1.f, 1.f);
  }
  
  void APawnBase::MoveRight(float Value)
  {
  	MovementDirection.Y = FMath::Clamp(Value, -1.f, 1.f);
  }
  ```







### 7. Impulse Force



情景：

- 反射一条射线，朝向`可移动`，开启`模拟物理`的`Actor`
- 使得`Actor`往射线方向添加`脉冲力`



示例：

- 定义：

  ```c++
  private:
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float TraceDistance;
  
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float ImpulseForce;
  ```



- 实现：

  ```c++
  FHitResult HitResult;
  const FVector Start = GetComponentLocation();
  const FVector End = Start + (GetComponentRotation().Vector() * TraceDistance);
  
  const bool IsHit = GetWorld()->LineTraceSingleByChannel(
  	HitResult,
  	Start,
  	End,
  	ECC_Visibility);
  
  if (!IsHit){return;}
  DrawDebugLine(GetWorld(), Start, End, FColor::Green, false, 0.1f);
  UStaticMeshComponent *StaticMeshComponent = Cast<UStaticMeshComponent>(HitResult.GetActor()->GetRootComponent());
  
  if (!StaticMeshComponent || !HitResult.GetActor()->IsRootComponentMovable()){return;}
  StaticMeshComponent->AddImpulse(GetForwardVector() * ImpulseForce * StaticMeshComponent->GetMass());
  ```



说明：

- `HitResult.GetActor()->GetRootComponent()`是针对根组件是`UStaticMeshComponent`
- 最好改为`HitResult.GetActor()->GetStaticMeshComponent()`
- 目的确保任意`可移动`，`模拟物理`的`Actor`都能收到影响
- `GetStaticMeshComponent()`可在任意`Actor`内实现此函数，返回`UStaticMeshComponent*`





### 8. Add Force



情景：

- 不发射射线
- 给可以模拟物理的`Actor`添加力



示例：

- 定义：

  ```c++
  class UStaticMeshComponent;
  
  private:
  	UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, meta=(AllowPrivateAccess=true))
  	UStaticMeshComponent *StaticMeshComponent;
  
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float Force;
  ```



- 实现：

  ```c++
  void UAddForce_SComp::BeginPlay()
  {
  	Super::BeginPlay();
  
  	StaticMeshComponent = Cast<UStaticMeshComponent>(GetOwner()->GetRootComponent());
  }
  
  void UAddForce_SComp::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
  {
  	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
  
  	const FVector UpForce = GetUpVector();
  	StaticMeshComponent->AddForce(UpForce * Force * StaticMeshComponent->GetMass());
  }
  ```







### 9. Radia Impulse Force



情景：

- 实现`爆炸力`



示例：

- 定义：

  ```c++
  private:
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float ImpulseRadius;
  	
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float ImpulseForceStrength;
  ```



- 实现：

  ```c++
  void URadiaImpulse_SComp::BeginPlay()
  {
  	Super::BeginPlay();
  
  	TArray<FHitResult> HitResults;
  	const FVector Start, End = GetComponentLocation();
  
  	const bool IsHit = GetWorld()->SweepMultiByChannel(
  		HitResults,
  		Start,
  		End,
  		FQuat::Identity,
  		ECC_WorldStatic,
  		FCollisionShape::MakeSphere(ImpulseRadius));
  
  	DrawDebugSphere(
  		GetWorld(),
  		Start,
  		ImpulseRadius,
  		50,
  		FColor::Green,
  		true);
  	
  	if (!IsHit){return;}
  	for (const auto &result : HitResults)
  	{
  		UStaticMeshComponent *MeshComponent = Cast<UStaticMeshComponent>(result.GetActor()->GetRootComponent());
  		if (!MeshComponent){continue;}
  		MeshComponent->AddRadialImpulse(
  			Start,
  			ImpulseRadius,
  			ImpulseForceStrength,
  			RIF_Linear,
  			true);
  	}
  }
  ```








### 10. TimerHandle



情景：

- 需要使用定时器



示例：

- 定义：

  ```c++
  #define PrintScreen(String) GEngine->AddOnScreenDebugMessage(-1, 10.f, FColor::Green, String)
  
  private:
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	int32 CallTracker;
  
  	UPROPERTY(VisibleDefaultsOnly, BlueprintReadOnly, meta=(AllowPrivateAccess=true))
  	FTimerHandle TimerHandle;
  	
  public:
  	UFUNCTION()
  	void TimerFunction();
  ```



- 实现：

  ```c++
  void ATimerHandle_Actor::BeginPlay()
  {
  	Super::BeginPlay();
  
  	GetWorldTimerManager().SetTimer(
  		TimerHandle,
  		this,
  		&ATimerHandle_Actor::TimerFunction,
  		1.f,
  		true,
  		1.f);
  }
  
  void ATimerHandle_Actor::TimerFunction()
  {
  	CallTracker == 0 ?
  		PrintScreen("Timer End"), GetWorldTimerManager().ClearTimer(TimerHandle) :
  		PrintScreen(FString::Printf(TEXT("Timer: %d"), CallTracker));
  
  	--CallTracker;
  }
  ```



- 注意：
  - 但类作为父类时，要使子类也可使用`TimerHandle`，需要用`protected`修饰






### 11. Disable Actor



情景：

- 编辑场景中的`Actor`
- 不希望直接从场景中删除
- 可以在生成的实例编辑中关闭
- 不消耗性能



示例：

- 定义：

  ```c++
  #define PrintScreen(String) GEngine->AddOnScreenDebugMessage(-1, 10.f, FColor::Green, String)
  
  private:
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	bool isOverrideTick = false;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	bool isAutoDisable = false;
  
  public:
  	UFUNCTION()
  	void SetActive(bool isActive);
  ```



- 实现：

  ```c++
  ADisableActor::ADisableActor()
  {
  	PrimaryActorTick.bCanEverTick = true;
  
  }
  
  void ADisableActor::BeginPlay()
  {
  	Super::BeginPlay();
  
  	isOverrideTick = !PrimaryActorTick.bCanEverTick;
  	if (isAutoDisable){SetActive(false);}
  }
  
  void ADisableActor::Tick(float DeltaSeconds)
  {
  	Super::Tick(DeltaSeconds);
  
  	PrintScreen("Tick");
  }
  
  void ADisableActor::SetActive(bool isActive)
  {
  	if (isOverrideTick)
  	{
  		SetActorTickEnabled(false);
  	}
  	else
  	{
  		SetActorTickEnabled(isActive);
  	}
  
  	SetActorHiddenInGame(!isActive);
  	SetActorEnableCollision(isActive);
  }
  ```



说明：

- 在场景中，只需要实例中的`isAutoDisable`进行勾选
- 重新运行，`Actor`将会被禁用，不会作用于场景中







### 12. Hit Event



情景：

- 通过一个`Actor`和场景中的其他`Actor`碰撞
- 产生碰撞事件



示例：

- 定义：

  ```c++
  #define PrintScreen(String) GEngine->AddOnScreenDebugMessage(-1, 10.f, FColor::Green, String)
  
  class UBoxComponent;
  
  private:
  	UPROPERTY(VisibleDefaultsOnly, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UBoxComponent *HitBox;
  
  public:
  	UFUNCTION()
  	void OnHitComp(UPrimitiveComponent* HitComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp,
  		FVector NormalImpulse, const FHitResult& Hit);
  ```



- 实现：

  ```c++
  AHitEventActor::AHitEventActor()
  {
  	PrimaryActorTick.bCanEverTick = false;
  
  	HitBox = CreateDefaultSubobject<UBoxComponent>(TEXT("Hit Box"));
  }
  
  void AHitEventActor::BeginPlay()
  {
  	Super::BeginPlay();
  
  	HitBox->OnComponentHit.AddDynamic(this, &AHitEventActor::OnHitComp);
  }
  
  void AHitEventActor::OnHitComp(UPrimitiveComponent* HitComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp,
  	FVector NormalImpulse, const FHitResult& Hit)
  {
  	PrintScreen(FString::Printf(TEXT("Hit: %s"), *OtherActor->GetName()));
  }
  ```



说明：

- `HitBox`需要些设置，碰撞事件才能生效
- 配置：
  - 方便查看：
    - 设置`形状`
    - 设置`渲染`
  - 产生事件：设置`碰撞`
    - 打开`模拟生成命中事件`
    - 设置`碰撞预设`







### 13. Set Material



情景：

- 材质的创建
- 材质的使用



示例：

- 定义：

  ```c++
  class UStaticMeshComponent;
  class UMaterialInterface;
  class UMaterial;
  class UMaterialInstance;
  
  private:
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UStaticMeshComponent *Mesh;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UMaterialInterface *MaterialOne;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UMaterialInterface *MaterialTwo;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UMaterial *Material;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UMaterialInstance *MaterialInstance;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	bool IsChooseOne = true;
  ```



- 实现：

  ```c++
  ASetMaterial::ASetMaterial()
  {
  	PrimaryActorTick.bCanEverTick = false;
  
  	Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh_Comp"));
  	RootComponent = Mesh;
  
  	MaterialOne = CreateDefaultSubobject<UMaterialInterface>(TEXT("Material_One"));
  	MaterialTwo = CreateDefaultSubobject<UMaterialInterface>(TEXT("Material_Two"));
  }
  
  void ASetMaterial::BeginPlay()
  {
  	Super::BeginPlay();
  
  	Mesh->SetMaterial(0, IsChooseOne ? MaterialOne : MaterialTwo);
  }
  ```







### 14. Dynamic Material



情景：

- 有一个基本的材质
- 通过创建材质实例
- 对其中的`ScaleParam`，`VectorParam`进行动态修改



示例：

- 定义：

  ```c++
  class UStaticMeshComponent;
  class UMaterialInterface;
  class UMaterialInstanceDynamic;
  
  private:
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UStaticMeshComponent *Mesh;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UMaterialInterface *MaterialInterface;
  
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UMaterialInstanceDynamic *DynamicInstance;
  ```



- 实现：

  ```c++
  ADynamicMaterial::ADynamicMaterial()
  {
  	PrimaryActorTick.bCanEverTick = false;
  
  	Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComp"));
  	RootComponent = Mesh;
  }
  
  void ADynamicMaterial::BeginPlay()
  {
  	Super::BeginPlay();
  
  	MaterialInterface = Mesh->GetMaterial(0);
  	DynamicInstance = UMaterialInstanceDynamic::Create(MaterialInterface, this);
  	
  	Mesh->SetMaterial(0, DynamicInstance);
  
  	DynamicInstance->SetScalarParameterValue(TEXT("EmissiveStrength"), 50.f);
  	DynamicInstance->SetVectorParameterValue(TEXT("Color"), FLinearColor::Yellow);
  }
  ```







### 15. Interp Target



情景：

- 指定`Actor`插值移动到`Target`



示例：

- 定义：

  ```c++
  private:
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	AActor *Origin = nullptr;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	AActor *Target;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float InterpSpeed = 3.f;
  	
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float WaitTime = 1.f;
  ```



- 实现：

  ```c++
  void UInterpTarget_SComp::BeginPlay()
  {
  	Super::BeginPlay();
  	
  	Origin = GetOwner();
  }
  
  void UInterpTarget_SComp::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
  {
  	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
  
  	if (WaitTime > 0)
  	{
  		WaitTime -= DeltaTime;
  		return;
  	}
  
  	if (!Target || !Origin){return;}
  	
  	Origin->SetActorLocation(
  		FMath::VInterpTo(
  			Origin->GetActorLocation(),
  			Target->GetActorLocation(),
  			DeltaTime,
  			InterpSpeed));
  }
  ```



说明：

- `Origin = GetOwner();`写在构造函数里无效







### 16. Lerp



情景：

- 需要使用`Lerp`
- 修改`Actor`的`位置`和`材质`



示例：

- 定义：

  ```c++
  class UMaterialInterface;
  class UMaterialInstanceDynamic;
  class UStaticMeshComponent;
  
  private:
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UMaterialInterface *MaterialInter;
  
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UMaterialInstanceDynamic *InstanceDynamic;
  
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	AActor *OriginActor = nullptr;
  
  	UPROPERTY(VisibleDefaultsOnly, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	UStaticMeshComponent *Mesh;
  	
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	FVector StartLocation;
  
  	UPROPERTY(EditAnywhere, meta=(MakeEditWidget=true))
  	FVector TargetLocation;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float TimeElapsed = 0;
  	
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float LerpDuration = 3.f;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	float WaitTime = 1.f;
  ```



- 实现：

  ```c++
  #include "Lerp_SComp.h"
  
  ULerp_SComp::ULerp_SComp()
  {
  	PrimaryComponentTick.bCanEverTick = true;
  
  	Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComp"));
  }
  
  void ULerp_SComp::BeginPlay()
  {
  	Super::BeginPlay();
  
  	OriginActor = GetOwner();
  	StartLocation = GetComponentLocation();
  	MaterialInter = Mesh->GetMaterial(0);
  	InstanceDynamic = UMaterialInstanceDynamic::Create(MaterialInter, this);
  	Mesh->SetMaterial(0, InstanceDynamic);
  	InstanceDynamic->SetVectorParameterValue(TEXT("Color"), FLinearColor::Blue);
  }
  
  void ULerp_SComp::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
  {
  	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
  
  	if (!OriginActor){return;}
  
  	if (WaitTime > 0){WaitTime -= DeltaTime; return;}
  
  	if (TimeElapsed < LerpDuration)
  	{
  		OriginActor->SetActorLocation(
  			FMath::Lerp(
  				StartLocation,
  				TargetLocation,
  				TimeElapsed / LerpDuration));
  
  		InstanceDynamic->SetVectorParameterValue(
  			TEXT("Color"),
  			FMath::Lerp(
  				FLinearColor::Blue,
  				FLinearColor::Red,
  				TimeElapsed / LerpDuration));
  
  		TimeElapsed += DeltaTime;
  	}
  }
  ```

  





### 18. 黑洞



情景：

- 一个球，可以在一定范围内吸引`开启模拟物理的Actor`
- 吸到黑洞的`Actor`会被销毁



示例：

- 简单的材质
- 黑洞要黑，不能反光
- 创建材质`Mat_BlackHole`
- 创建节点`VectorParameter`，设置`RGBA(0, 0, 0, 1)`，连接`基础颜色`
- 创建常量节点，设置为`0`， 连接其余的选项



定义：

`BlackHole_Actor.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "BlackHole_Actor.generated.h"

class UStaticMeshComponent;
class USphereComponent;
struct FTimerHandle;

UCLASS()
class FPSGAME_API ABlackHole_Actor : public AActor
{
	GENERATED_BODY()
	
public:
	ABlackHole_Actor();
	virtual void BeginPlay() override;

/* My Code */
	// Property
private:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="BlackHole", meta=(AllowPrivateAccess=true))
	float BlackHoleActionRate;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="BlackHole", meta=(AllowPrivateAccess=true))
	float BlackHoleStrength;
	
public:
	FTimerHandle BlackHoleActionHandle;
	
	// Component
private:
	UPROPERTY(VisibleAnywhere, Category="A_Hole")
	UStaticMeshComponent *BlackHoleStaticMeshComp;

	UPROPERTY(VisibleAnywhere, Category="A_Hole")
	USphereComponent *InnerSphereComp;

	UPROPERTY(VisibleAnywhere, Category="A_Hole")
	USphereComponent *OuterSphereComp;

public:
	UFUNCTION()
	void OverlapInnerSphere(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor,
		UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult &SweepResult);

	UFUNCTION()
	void OnBlackHoleAction();
};
```



实现

`BlackHole_Actor.cpp`

```c++
#include "BlackHole_Actor.h"
#include "Components/SphereComponent.h"

ABlackHole_Actor::ABlackHole_Actor()
{
	PrimaryActorTick.bCanEverTick = false;

	BlackHoleActionRate = 0.05f;
	BlackHoleStrength = 10000.f;

	BlackHoleStaticMeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BlackStaticMesh"));
	BlackHoleStaticMeshComp->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	BlackHoleStaticMeshComp->CastShadow = false;
	RootComponent = BlackHoleStaticMeshComp;

	InnerSphereComp = CreateDefaultSubobject<USphereComponent>(TEXT("InnerSphereComp"));
	InnerSphereComp->SetSphereRadius(100.f);
	InnerSphereComp->SetupAttachment(BlackHoleStaticMeshComp);
	
	OuterSphereComp = CreateDefaultSubobject<USphereComponent>(TEXT("OuterSphereComp"));
	OuterSphereComp->SetSphereRadius(3000.f);
	OuterSphereComp->SetCollisionEnabled(ECollisionEnabled::QueryOnly);
	OuterSphereComp->SetCollisionResponseToAllChannels(ECR_Overlap);
	OuterSphereComp->SetupAttachment(BlackHoleStaticMeshComp);
}

void ABlackHole_Actor::BeginPlay()
{
	Super::BeginPlay();

	check(InnerSphereComp);
	check(OuterSphereComp);
	
	if (InnerSphereComp)
	{
		InnerSphereComp->OnComponentBeginOverlap.AddDynamic(this, &ABlackHole_Actor::OverlapInnerSphere);
	}

	if (GetWorld())
	{
		GetWorldTimerManager().SetTimer(
			BlackHoleActionHandle,
			this,
			&ABlackHole_Actor::OnBlackHoleAction,
			BlackHoleActionRate,
			true
			);
	}
}

void ABlackHole_Actor::OverlapInnerSphere(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor,
	UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	if (!OtherActor){return;}
	OtherActor->Destroy();
}

void ABlackHole_Actor::OnBlackHoleAction()
{
	TArray<UPrimitiveComponent*> OverlappingComp;

	if (!OuterSphereComp){return;}
	OuterSphereComp->GetOverlappingComponents(OverlappingComp);

	for (int i = 0; i < OverlappingComp.Num(); ++i)
	{
		if (OverlappingComp[i] && OverlappingComp[i]->IsSimulatingPhysics())
		{
			const float SphereRadius = OuterSphereComp->GetScaledSphereRadius();
			OverlappingComp[i]->AddRadialForce(
				GetActorLocation(),
				SphereRadius,
				-BlackHoleStrength,
				RIF_Constant,
				true);
		}
	}
}
```







### 19. 玩家死亡后进入观察



情景：

- 但玩家死亡后
- 播放死亡动画
- 禁用玩家输入
- 控制器切换控制到`Spectator`
- 玩家不与其他玩家产生碰撞
- 玩家外的胶囊体组件无碰撞
- 死亡动画播放带有布娃娃效果
- 一定时间后销毁



示例：

- 前提：
  - 将死亡动画创建为`蒙太奇动画`
  - 设置蒙太奇的`启用自动混出`为`false`



- 代码：

  - 定义

    ```c++
    class UAnimMontage;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Montage", meta=(AllowPrivateAccess=true))
    UAnimMontage *AnimMontage_Death;
    
    UFUNCTION()
    void OnDead();
    ```

  

  - 实现

    ```c++
    void ASTUCharacterBase::OnDead()
    {
    	if (!AnimMontage_Death){return;}
    	PlayAnimMontage(AnimMontage_Death);
    	GetCharacterMovement()->DisableMovement();
    
    	if (GetController())
    	{
    		GetController()->ChangeState(NAME_Spectating);
    	}
    	GetCapsuleComponent()->SetCollisionResponseToAllChannels(ECR_Ignore);
    	SetLifeSpan(5.0f);
    
    	GetMesh()->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
    	GetMesh()->SetSimulatePhysics(true);
    }
    ```

    



### 20. 高处坠落伤害



情景：

- 高处下落的速度达到指定值
- 玩家受到相应的伤害
- 使用`ACharacter`自带的功能实现
  - `virtual void Landed(const FHitResult& Hit);`
  - `FLandedSignature LandedDelegate;`



示例：

- 介绍：

  `ACharacter.h`

  ```c++
  	/**
  	 * Called upon landing when falling, to perform actions based on the Hit result. Triggers the OnLanded event.
  	 * Note that movement mode is still "Falling" during this event. Current Velocity value is the velocity at the time of landing.
  	 * Consider OnMovementModeChanged() as well, as that can be used once the movement mode changes to the new mode (most likely Walking).
  	 *
  	 * @param Hit Result describing the landing that resulted in a valid landing spot.
  	 * @see OnMovementModeChanged()
  	 */
  	virtual void Landed(const FHitResult& Hit);
  
  /**
  * 落地时调用，根据命中结果执行动作。 触发 OnLanded 事件。
  * 请注意，此活动期间移动模式仍为“下降”。 当前速度值是着陆时的速度。
  * 还要考虑 OnMovementModeChanged()，因为一旦移动模式更改为新模式（很可能是步行），就可以使用它。
  *
  * @param Hit Result 描述了导致有效着陆点的着陆。
  * @see OnMovementModeChanged()
  */
  	FLandedSignature LandedDelegate;	
  ```



- 定义：

  ```c++
  UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="FallingDamage", meta=(AllowPrivateAccess=true))
  FVector2D LandedDamageVelocity;
  
  UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="FallingDamage", meta=(AllowPrivateAccess=true))
  FVector2D LandedDamage;
  
  UFUNCTION()
  void OnGroundLanded(const FHitResult& HitResult);
  ```



- 实现：

  ```c++
  void ASTUCharacterBase::BeginPlay()
  {
  	Super::BeginPlay();
  
  	LandedDelegate.AddDynamic(this, &ASTUCharacterBase::OnGroundLanded);
  }
  
  void ASTUCharacterBase::OnGroundLanded(const FHitResult& HitResult)
  {
  	const float FallVelocityZ = -GetVelocity().Z;
  
  	if (FallVelocityZ < LandedDamageVelocity.X){return;}
  	const float FinalDamage = FMath::GetMappedRangeValueClamped(LandedDamageVelocity,
  		LandedDamage, FallVelocityZ);
  	TakeDamage(FinalDamage, FDamageEvent{}, nullptr, nullptr);
  }
  
  
  # FMath::GetMappedRangeValueClamped() --> UnrealMathUtility.h
  // For the given Value clamped to the [Input:Range] inclusive, returns the corresponding percentage in [Output:Range] Inclusive
  // 对于钳制到 [Input:Range] 的给定值，返回 [Output:Range] 包含的相应百分比
  ```

  





### 21. FindAnimNotifyByClass



情景：

- 按类型`UAnimSequenceBase`中查找`FAnimNotifyEvent`
- 再绑定对应的通知事件



示例：

- 前提：

  创建`AnimNotifyEventUntility.h`

  ```c++
  #pragma once
  
  class AnimNotifyEventUntility
  {
  public:
  	template<typename T>
  	static T* FindNotifyByClass(UAnimSequenceBase* AnimSequenceBase)
  	{
  		if (!AnimSequenceBase){return nullptr;}
  
  		const auto NotifyEvents = AnimSequenceBase->Notifies;
  		for (const auto& NotifyEvent : NotifyEvents)
  		{
  			if (const auto AnimNotify = Cast<T>(NotifyEvent.Notify))
  			{
  				return AnimNotify;
  			}
  		}
  		return nullptr;
  	}
  };
  ```

  

  创建`AnimNotifyBase.h`

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "Animation/AnimNotifies/AnimNotify.h"
  #include "STUAnimNotifyBase.generated.h"
  
  class USkeletalMeshComponent;
  
  DECLARE_MULTICAST_DELEGATE_OneParam(FOnNotifySignature, USkeletalMeshComponent*)
  
  UCLASS()
  class B_01_TPS_API UAnimNotifyBase : public UAnimNotify
  {
  	GENERATED_BODY()
  
  public:
  	virtual void Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation) override;
  
  /* My Code */
  	// Property
  	FOnNotifySignature OnNotify;	
  };
  
  # Notify
  // 是 AnimNotify 自带的函数，同时不建议 UE5 使用
  ```

  

  实现`AnimNotifyBase.cpp`

  ```c++
  #include "AnimNotifyBase.h"
  
  void USTUAnimNotifyBase::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
  {
  	Super::Notify(MeshComp, Animation);
  
  	OnNotify.Broadcast(MeshComp);
  }
  ```

  



- 定义：

  ```c++
  UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Weapon", meta=(AllowPrivateAccess=true))
  UAnimMontage *EquipAnimMontage;
  
  UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category="Weapon", meta=(AllowPrivateAccess=true))
  bool IsEquipAnimInProgress;
  
  UFUNCTION()
  void InitAnimations();
  
  UFUNCTION()
  void OnEquipFinished(USkeletalMeshComponent* SkeletalMeshComponent);
  ```

  

- 实现：

  ```c++
  #include "AnimNotify/AnimNotifyEventUntility.h"
  
  void USTUWeaponActorComponent::BeginPlay()
  {
  	Super::BeginPlay();
  
  	InitAnimations();
  }
  
  void USTUWeaponActorComponent::InitAnimations()
  {
  	if (const auto EquipFinishedNotify = 
          AnimUtils::FindNotifyByClass<USTUAnimNotifyEquipFinished>(EquipAnimMontage))
  	{
  		EquipFinishedNotify->OnNotify.AddUObject(this, &USTUWeaponActorComponent::OnEquipFinished);
  	}
  	else
  	{
  		checkNoEntry();
  	}
  }
  
  void USTUWeaponActorComponent::OnEquipFinished(USkeletalMeshComponent* SkeletalMeshComponent)
  {
  	const ASTUCharacterBase *Character = Cast<ASTUCharacterBase>(GetOwner());
  
  	if (!Character || !(Character->GetMesh() == SkeletalMeshComponent)){return;}
  	IsEquipAnimInProgress = false;
  }
  
  # USTUAnimNotifyEquipFinished
  // 是AnimNotifyBase的子类
  // 里面不用写东西
  // 主要是用来：修改备注名，修改颜色，绑定事件
  ```

  





### 22. 类生成附加到插槽



情景：

- 用指定的组件开始生成

- `BeginPlay()`时开始

- 指定要生成的`class`
- 以有需要附加的插槽``
- 将生成的对象附加到插槽上，并应用插槽的`transform`
- `EndPlay()`结束后，生成的`Actor`也要销毁



示例：

- 定义：`UWeaponActorComponent.h`

  ```c++
  UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
  class B_01_TPS_API USTUWeaponActorComponent : public UActorComponent
  {
  	GENERATED_BODY()
  
  public:
  	USTUWeaponActorComponent();
  
  protected:
  	virtual void BeginPlay() override;
  	virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override;
  	
  /* My Code */
  	// Property
  private:
      UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Weapon", meta=(AllowPrivateAccess=true))
  	TSubclassOf<ASTUWeaponBase> WeaponClass;
      
      UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Weapon", meta=(AllowPrivateAccess=true))
  	FName WeaponEquipSocketName;
      
      UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Weapon", meta=(AllowPrivateAccess=true))
  	ASTUWeaponBase *CurrentWeapon;
      
      // Function
  public:
  	UFUNCTION()
  	void SpawnWeapons();
      
      UFUNCTION()
  	void AttachWeaponToSocket(ASTUWeaponBase* Weapon, USceneComponent* SceneComponent, const FName& SocketName);
  }
  ```



- 实现：

  ```c++
  USTUWeaponActorComponent::USTUWeaponActorComponent()
  {
  	PrimaryComponentTick.bCanEverTick = false;
  
  	WeaponEquipSocketName = TEXT("WeaponSocket");
      WeaponClass = nullptr
  	CurrentWeapon = nullptr;
  }
  
  void USTUWeaponActorComponent::BeginPlay()
  {
  	Super::BeginPlay();
  
  	SpawnWeapons();
  }
  
  void USTUWeaponActorComponent::EndPlay(const EEndPlayReason::Type EndPlayReason)
  {
  	CurrentWeapon->DetachFromActor(FDetachmentTransformRules::KeepWorldTransform);
  	CurrentWeapon->Destroy();
      CurrentWeapon = nullptr;
  
  	Super::EndPlay(EndPlayReason);
  }
  
  void USTUWeaponActorComponent::SpawnWeapons()
  {
  	if (!GetWorld()){return;}
  	ASTUCharacterBase *Character = Cast<ASTUCharacterBase>(GetOwner());
  
  	if (!Character){return;}
      const auto Weapon = GetWorld()->SpawnActor<ASTUWeaponBase>(WeaponClass);
      
  	if (!Weapon){continue;}
      CurrentWeapon = Weapon;
  	CurrentWeapon->SetOwner(Character);
      
  	AttachWeaponToSocket(CurrentWeapon, Character->GetMesh(), WeaponEquipSocketName);
  }
  
  void USTUWeaponActorComponent::AttachWeaponToSocket(ASTUWeaponBase* Weapon, USceneComponent* SceneComponent,
                                                      const FName& SocketName)
  {
  	if (!Weapon || !SceneComponent){return;}
  	const FAttachmentTransformRules AttachmentTransformRules(EAttachmentRule::SnapToTarget, false);
  	Weapon->AttachToComponent(SceneComponent, AttachmentTransformRules, SocketName);
  }
  ```

  



### 23. CameraShake



情景：

- 在`UHealthActorComponent`中实现`CameraShake`



示例：

- 定义：

  ```c++
  public:
  	USTUHealthActorComponent();
  
  private:
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="HPComp|Camera", meta=(AllowPrivateAccess=true))
  	TSubclassOf<UCameraShakeBase> CameraShake;
  
  public:
  	UFUNCTION()
  	void PlayCameraShake();
  ```



- 实现：

  ```c++
  #include "STUHealthActorComponent.h"
  
  USTUHealthActorComponent::USTUHealthActorComponent()
  {
  	PrimaryComponentTick.bCanEverTick = false;
  
  	CameraShake = nullptr;
  }
  
  void USTUHealthActorComponent::PlayCameraShake()
  {
  	const auto Player = Cast<APawn>(GetOwner());
  
  	if (!Player){return;}
  	const auto Controller = Player->GetController<APlayerController>();
  
  	if (!Controller || !Controller->PlayerCameraManager){return;}
  	Controller->PlayerCameraManager->StartCameraShake(CameraShake);
  }
  ```

  

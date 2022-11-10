# UE4 功能整理



[toc]



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
  	//const FVector Direction = FRotationMatrix(GetController()->GetControlRotation()).GetScaledAxis(EAxis::X);
      
      // 这个方法简单，通用
      const FVector Direction = GetActorForwardVector();
  	AddMovementInput(Direction, Value);
      
      ###### 官方写法 ######
  	if ((Controller != nullptr) && (Value != 0.0f))
  	{
  		// find out which way is forward
  		const FRotator Rotation = Controller->GetControlRotation();
  		const FRotator YawRotation(0, Rotation.Yaw, 0);
  
  		// get forward vector
  		const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
  		AddMovementInput(Direction, Value);
  	}
  }
  
  void ACharacterBase::MoveRight(float Value)
  {
  	// const FVector Direction = FRotationMatrix(GetController()->GetControlRotation()).GetScaledAxis(EAxis::Y);
      
      const FVector Direction = GetActorRightVector();
  	AddMovementInput(Direction, Value);
      
      ###### 官方写法 ######
      if ( (Controller != nullptr) && (Value != 0.0f) )
  	{
  		// find out which way is right
  		const FRotator Rotation = Controller->GetControlRotation();
  		const FRotator YawRotation(0, Rotation.Yaw, 0);
  	
  		// get right vector 
  		const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);
  		// add movement in that direction
  		AddMovementInput(Direction, Value);
  	}
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
      
      // 此处留一个伏笔，当前武器指定了 Onwer 是当前的玩家
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








### 24. HealthComponent



情景：

- 设计一个角色的生命组件
- 有生命值
- 有伤害处理
- 呼吸回血效果
- 有处理玩家死亡效果



前提：

- 定义玩家类：`ACharacterBase`
- 定义生命组件类：`UHealthComponent`



结构分析：

- 玩家类中的`OnDead`效果对应于生命组件类中的`OnDead`
- 生命组件类：
  - 可以通过蓝图获得`CurrentHP`，`MaxHP`，`IsDead`
  - 通过`TimerHandle`实现呼吸回血效果



示例：

- 生命组件类

- 定义：

  ```c++
  DECLARE_MULTICAST_DELEGATE(FOnDead);
  DECLARE_MULTICAST_DELEGATE_OneParam(FOnHealthChanged, float);
  
  struct FTimerHandle;
  
  UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
  class B_01_TPS_API USTUHealthActorComponent : public UActorComponent
  {
  	GENERATED_BODY()
  
  public:
  	USTUHealthActorComponent();
  
  protected:
  	virtual void BeginPlay() override;
  
  /* My Code */
  	// Property
  private:
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="HPComp", meta=(AllowPrivateAccess=true))
  	float Health;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="HPComp", meta=(AllowPrivateAccess=true))
  	float MaxHealth;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="HPComp|Heal", meta=(AllowPrivateAccess=true))
  	bool IsAutoHeal;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="HPComp|Heal",
  		meta=(AllowPrivateAccess=true, EditCondition="IsAutoHeal"))
  	float HealUpdateTime;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="HPComp|Heal",
  	meta=(AllowPrivateAccess=true, EditCondition="IsAutoHeal"))
  	float HealDelayTime;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="HPComp|Heal",
  	meta=(AllowPrivateAccess=true, EditCondition="IsAutoHeal"))
  	float HealModifierValue;
  	
  public:
  	FTimerHandle HealTimerHandle;
  	FOnDead OnDead;
  	FOnHealthChanged OnHealthChanged;
  	
  	// Function	
  public:
  	UFUNCTION(BlueprintCallable)
  	FORCEINLINE float GetHP() const {return Health;}
  	
  	UFUNCTION()
  	void OnTakeAnyDamage(AActor* DamagedActor, float Damage, const class UDamageType* DamageType,
  		class AController* InstigatedBy, AActor* DamageCauser);
  	
  	UFUNCTION(BlueprintCallable)
  	bool IsDead() const {return FMath::IsNearlyZero(Health);}
  
  	UFUNCTION()
  	void HealUpdate();
      
  	UFUNCTION()
  	void SetHealth(float NewHealth);
  };
  ```



- 实现：

  ```c++
  #include "STUHealthActorComponent.h"
  
  USTUHealthActorComponent::USTUHealthActorComponent()
  {
  	PrimaryComponentTick.bCanEverTick = false;
  
  	Health = 0.f;
  	MaxHealth = 100.f;
  
  	IsAutoHeal = true;
  	HealUpdateTime = 1.f;
  	HealDelayTime = 4.f;
  	HealModifierValue = 10.f;
  }
  
  void USTUHealthActorComponent::BeginPlay()
  {
  	Super::BeginPlay();
  
      SetHealth(MaxHealth);
  
  	if (GetOwner())
  	{
  		GetOwner()->OnTakeAnyDamage.AddDynamic(this, &USTUHealthActorComponent::OnTakeAnyDamage);
  	}
  }
  
  /* My Code */
  void USTUHealthActorComponent::OnTakeAnyDamage(AActor* DamagedActor, float Damage, const UDamageType* DamageType,
  	AController* InstigatedBy, AActor* DamageCauser)
  {
  	if (Health <= 0.f || IsDead() || !GetWorld()){return;}
  	SetHealth(Health -  Damage);
  
  	if (IsDead())
  	{
  		OnDead.Broadcast();
  	}
  	else if (IsAutoHeal)
  	{
  		GetWorld()->GetTimerManager().SetTimer(HealTimerHandle, this, &USTUHealthActorComponent::HealUpdate,
  			HealUpdateTime, IsAutoHeal, HealDelayTime);
  	}
  }
  
  void USTUHealthActorComponent::HealUpdate()
  {
  	SetHealth(Health += HealModifierValue);
  
  	if (!FMath::IsNearlyEqual(Health, MaxHealth) || !GetWorld()){return;}
  	GetWorld()->GetTimerManager().ClearTimer(HealTimerHandle);
  }
  
  void USTUHealthActorComponent::SetHealth(float NewHealth)
  {
  	Health = FMath::Clamp(NewHealth, 0.f, MaxHealth);
  }
  ```



- 玩家类

- 定义：

  ```c++
  class USTUHealthActorComponent;
  
  public:
  	ASTUCharacterBase();
  
  protected:
  	virtual void BeginPlay() override;
  
  /* My Code */
  	// Component
  private:
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category="Component", meta=(AllowPrivateAccess=true))
  	USTUHealthActorComponent *HealthActorComponent;
  
  	// Function
  public:
  	UFUNCTION()
  	void OnDead();
  ```



- 实现：

  ```c++
  #include "STUHealthActorComponent.h"
  
  ASTUCharacterBase::ASTUCharacterBase()
  {
      PrimaryActorTick.bCanEverTick = false;
      
      HealthActorComponent = CreateDefaultSubobject<USTUHealthActorComponent>(TEXT("HPComp"));
  }
  
  void ASTUCharacterBase::BeginPlay()
  {
  	Super::BeginPlay();
  
  	check(HealthActorComponent);
  	HealthActorComponent->OnDead.AddUObject(this, &ASTUCharacterBase::OnDead);
  }
  
  void ASTUCharacterBase::OnDead()
  {
      // OnDead Event
  }
  ```

  



### 25. GetActorComponent



情景：

- 需要通过已有的`Actor`获得其已添加的`Component`
- 将这个方法提取成一个`模板工具类`



示例：

- 定义：

  ```c++
  #pragma once
  
  class FH_Utility
  {
  public:
  	template<typename T>
  	FORCEINLINE static T* GetActorComponent(const AActor* Actor)
  	{
  		if (!Actor){return nullptr;}
  		const auto Component = Actor->GetComponentByClass(T::StaticClass());
  		T* ResultComponent = Cast<T>(Component);
  
  		if (!ResultComponent){return nullptr;}
  		return ResultComponent;
  	}
  };
  ```








### 26. 设计结构体和委托



情景：

- 将一类`结构体`或`Delegate`写进单独的类中
- 便于管理



示例：

- 定义：`FH_CoreType.h`

  ```c++
  #pragma once
  
  #include "FH_CoreType.generated.h" // 因为使用了Delegate 和 USTRUCT，这里要手动加入
  
  class ASTUWeaponBase;
  class UAnimMontage;
  
  // Delegate 的定义可以写在这里
  DECLARE_MULTICAST_DELEGATE(FOnDead);
  DECLARE_MULTICAST_DELEGATE_OneParam(FOnHealthChanged, float);
  
  // 可以定义结构体，USTRCUT() 内加入 BlueprintType, 蓝图就可以或这个结构体
  USTRUCT(BlueprintType)
  struct FAmmoData
  {
  	GENERATED_USTRUCT_BODY() // 原：GENERATED_BODY() 需改为 GENERATED_USTRUCT_BODY()
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Ammo",  meta=(AllowPrivateAccess=true))
  	int32 Bullets = 0;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Ammo",
  			  meta=(AllowPrivateAccess=true, EditCondition="!IsInfinity"))
  	int32 Clips = 0;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Ammo",  meta=(AllowPrivateAccess=true))
  	bool IsInfinity = false;
  };
  
  USTRUCT(BlueprintType)
  struct FWeaponData
  {
  	GENERATED_USTRUCT_BODY()
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="ReloadWeapon", meta=(AllowPrivateAccess=true))
  	TSubclassOf<ASTUWeaponBase> WeaponClass = nullptr;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="ReloadWeapon", meta=(AllowPrivateAccess=true))
  	UAnimMontage* ReloadAnimMontage = nullptr;
  };
  ```







### 27. AIPerception获得最近Actor



情景：

- `AI`绑定了`AIPerceptionComponent`
- 设置`AISense`：例如--目光`UAISense_Sight`
- 利用`GetCurrentlyPerceivedActors`获得目光内的有效`Actor`
- 通过`Actor`获得其前面`HealthComponent`，判断`Actor`是否符合需求
- 最后通过算法，求得并返回离`AI`最近的`Actor`



示例：

- 定义：

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "Perception/AIPerceptionComponent.h"
  #include "STUAIPerceptionComponent.generated.h"
  
  UCLASS()
  class B_01_TPS_API USTUAIPerceptionComponent : public UAIPerceptionComponent
  {
  	GENERATED_BODY()
  
  /* My Code */
  	// Function
  public:
  	UFUNCTION()
  	AActor* GetClosetActor() const;
  };
  ```



- 实现：

  ```c++
  #include "STUAIPerceptionComponent.h"
  #include "AIController.h"
  #include "STUHealthActorComponent.h"
  #include "B_01_TPS/Dev/STUUtils.h"
  #include "Perception/AISense_Sight.h"
  
  AActor* USTUAIPerceptionComponent::GetClosetActor() const
  {
  	TArray<AActor*> PerceptionActors;
      
      // AIPerceptionComponent自带的函数，获得所有已经被感知的Actor
  	GetCurrentlyPerceivedActors(UAISense_Sight::StaticClass(), PerceptionActors);
  
      // 判断组件的拥有AI是否有AIController
  	if (PerceptionActors.Num() <= 0){return nullptr;}
  	const auto Controller = Cast<AAIController>(GetOwner());
  
      // 判断当前AI是否有效
  	if (!Controller){return nullptr;}
  	const auto Pawn = Controller->GetPawn();
  
      // 开始计算最近的Actor
  	if (!Pawn){return nullptr;}
      
      // MAX_FLT 是 UnrealEngine 自带的 UrealMathUtility 中的 宏
      // #define MAX_FLT 3.402823466e+38F
      // 表示可虚幻引擎可表达的 最大浮点数
  	float NealDistance = MAX_FLT;
  	AActor* NealPawn = nullptr;
  	for (const auto& PerceptionActor : PerceptionActors)
  	{
          // 此处使用了前面创建的 HealthComponent 和 FH_Utility
          // 判断 感知到的Actor是否有生命组件，是否还活着
  		const auto HealthComp = STUUtils::GetSTUPlayerComponent<USTUHealthActorComponent>(PerceptionActor);
  		
          if (!HealthComp || HealthComp->IsDead()){return nullptr;}
          const auto CurrentDistance = (PerceptionActor->GetActorLocation() - Pawn->GetActorLocation()).Size();
  		
          // 找出最近距离的Actor
          if (CurrentDistance < NealDistance)
  		{
  			NealDistance = CurrentDistance;
  			NealPawn = PerceptionActor;
  		}
  	}
  	return NealPawn;
  }
  ```







### 28. GameMode



情景：

- 通过`GameModeBase`进行初始化



示例：

- 实现：

  ```c++
  ASTUGameModeBase::ASTUGameModeBase()
  {
  	DefaultPawnClass = ASTUCharacterBase::StaticClass();
  	PlayerControllerClass = ASTUPlayerController::StaticClass();
  	HUDClass = ASTUGameHUD::StaticClass();
  }
  ```







### 29. 拾取物-PickUp



情景：

- 创建`PickUpBase`
- 可被放置在场景中
- 可以`Actor`重叠检查
- 不与`Actor`产生`BlockHit`
- 拾取后不销毁，隐藏起来，一段时间重生，`TimerHandle`
- 物品默认旋转，拾取后关闭旋转，重生后开启旋转，`TimerHandle`
- 被拾取后，让拾取的`Actor`执行指定的`GivePickUpTo`，`Virtual Function`
- 具体执行的功能，通过子类去实现



示例：

- 定义：

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "GameFramework/Actor.h"
  #include "STUBasePickUp.generated.h"
  
  class USphereComponent;
  class UStaticMeshComponent;
  struct FTimerHandle;
  
  UCLASS()
  class B_01_TPS_API ASTUBasePickUp : public AActor
  {
  	GENERATED_BODY()
  	
  public:
  	ASTUBasePickUp();
  
  protected:
  	virtual void BeginPlay() override;
  	virtual void NotifyActorBeginOverlap(AActor* OtherActor) override;
  
  /* My Code */
  	// Property
  protected:
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="PickUp", meta=(AllowPrivateAccess=true))
  	float RespawnTime;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="PickUp", meta=(ClampMin=0.0333, ClampMax=0.0083))
  	float RotationYawRate;
      
      FTimerHandle RespawnHandle;
  
  	FTimerHandle RotationYawHandle;
  	
  	// Component
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category="PickUp", meta=(AllowPrivateAccess=true))
  	USphereComponent* CollisionComponent;
  
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category="PickUp", meta=(AllowPrivateAccess=true))
  	UStaticMeshComponent* StaticMeshComponent;
  
  	// Function
  public:
  	UFUNCTION()
  	void PickUpWasTaken();
  
  	UFUNCTION()
  	void Respawn();
  
  	UFUNCTION()
  	void LoopRotationYawHandle();
  
  	UFUNCTION()
  	void BeginRotationYaw();
  
  	UFUNCTION()
  	virtual bool GivePickUpTo(APawn* PlayerPawn) { return false; }
  };
  
  # NotifyActorBeginOverlap(AActor* OtherActor)
  	/** 
  	 *	Event when this actor overlaps another actor, for example a player walking into a trigger.
  	 *	For events when objects have a blocking collision, for example a player hitting a wall, see 'Hit' events.
  	 *	@note Components on both this and the other Actor must have bGenerateOverlapEvents set to true to generate overlap events.
  	 */
  ```



- 实现：

  ```c++
  #include "STUBasePickUp.h"
  #include "Components/SphereComponent.h"
  
  ASTUBasePickUp::ASTUBasePickUp()
  {
  	PrimaryActorTick.bCanEverTick = false;
  
      RespawnTime = 5.0f;
  	RotationYawRate = 0.0333f;
      
      CollisionComponent = CreateDefaultSubobject<USphereComponent>(TEXT("CollisionComp"));
  	RootComponent = CollisionComponent;
  	CollisionComponent->SetCollisionEnabled(ECollisionEnabled::QueryOnly);
  	CollisionComponent->SetCollisionResponseToAllChannels(ECR_Overlap);
  	CollisionComponent->InitSphereRadius(50.f);
  
  	StaticMeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComp"));
  	StaticMeshComponent->SetupAttachment(RootComponent);
  	StaticMeshComponent->SetCollisionEnabled(ECollisionEnabled::NoCollision);
  	StaticMeshComponent->SetCollisionResponseToAllChannels(ECR_Ignore);
  }
  
  void ASTUBasePickUp::BeginPlay()
  {
  	Super::BeginPlay();
  
  	check(CollisionComponent);
  	check(StaticMeshComponent);
  
  	LoopRotationYawHandle();
  }
  
  void ASTUBasePickUp::NotifyActorBeginOverlap(AActor* OtherActor)
  {
  	Super::NotifyActorBeginOverlap(OtherActor);
  
  	const auto Player = Cast<APawn>(OtherActor);
  	
  	if (!GivePickUpTo(Player)){return;}
  	PickUpWasTaken();
  }
  
  /* My Code */
  void ASTUBasePickUp::PickUpWasTaken()
  {
  	if (!GetRootComponent()){return;}
  	CollisionComponent->SetCollisionResponseToAllChannels(ECR_Ignore);
  	GetRootComponent()->SetVisibility(false, true);
      
  	GetWorldTimerManager().SetTimer(RespawnHandle, this, &ASTUBasePickUp::Respawn, RespawnTime, false);
  	GetWorldTimerManager().ClearTimer(RotationYawHandle);
  }
  
  void ASTUBasePickUp::Respawn()
  {
  	if (!GetRootComponent()){return;}
  	CollisionComponent->SetCollisionResponseToAllChannels(ECR_Overlap);
  	GetRootComponent()->SetVisibility(true, true);
      
  	GetWorldTimerManager().ClearTimer(RespawnHandle);
  	LoopRotationYawHandle();
  }
  
  void ASTUBasePickUp::LoopRotationYawHandle()
  {
  	if (!StaticMeshComponent && !RespawnHandle.IsValid()){return;}
  	GetWorldTimerManager().SetTimer(
          RotationYawHandle, 
          this, 
          &ASTUBasePickUp::BeginRotationYaw, 
          RotationYawRate, 
          true);
  }
  
  void ASTUBasePickUp::BeginRotationYaw()
  {
  	StaticMeshComponent->AddRelativeRotation(FRotator(0, 1.f, 0.f));
  }
  ```



- 注意：

  ```c++
  # 示例中是通过 AActor 自带的 NotifyActorBeginOverlap(AActor* OtherActor) 实现交互
  # 也可以通过 USphereComponent->UShapeComponent->UPrimitiveComponent 内的 FComponentHitSignature OnComponentHit; 实现交互
  
  ################## 委托名称 ###############
  /** 
  	 *	Event called when a component hits (or is hit by) something solid. This could happen due to things like Character movement, using Set Location with 'sweep' enabled, or physics simulation.
  	 *	For events when objects overlap (e.g. walking into a trigger) see the 'Overlap' event.
  	 *
  	 *	@note For collisions during physics simulation to generate hit events, 'Simulation Generates Hit Events' must be enabled for this component.
  	 *	@note When receiving a hit from another object's movement, the directions of 'Hit.Normal' and 'Hit.ImpactNormal'
  	 *	will be adjusted to indicate force from the other object against this object.
  	 *	@note NormalImpulse will be filled in for physics-simulating bodies, but will be zero for swept-component blocking collisions.
  	 */
  	UPROPERTY(BlueprintAssignable, Category="Collision")
  	FComponentHitSignature OnComponentHit;
  
  ################### 委托绑定的函数参数列表 ##################
  
  /**
   * Delegate for notification of blocking collision against a specific component.  
   * NormalImpulse will be filled in for physics-simulating bodies, but will be zero for swept-component blocking collisions. 
   */
  DECLARE_DYNAMIC_MULTICAST_SPARSE_DELEGATE_FiveParams( FComponentHitSignature, UPrimitiveComponent, OnComponentHit, UPrimitiveComponent*, HitComponent, AActor*, OtherActor, UPrimitiveComponent*, OtherComp, FVector, NormalImpulse, const FHitResult&, Hit );
  ```

  





### 30. AIController



情景：

- 与前面`AIPerception获得最近Actor`想关联
- 利用`AIPerceptionComponent`获取`最近的Actor`
- 通过`TimerHandle`控制检测频率
- 通过获得黑板组件，获得设置的`最近的Actor`
- 让`AIControlller`控制`AI`面向`最近的Actor`
- `AIController`获得自己绑定的`AI`，执行有效的`BehaviorTree`
- 通过`FName`手动指定获得黑板组件中指定的`Key`



示例：

- 定义：

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "AIController.h"
  #include "STUAIController.generated.h"
  
  class USTUAIPerceptionComponent;
  struct FTimerHandle;
  
  UCLASS()
  class B_01_TPS_API ASTUAIController : public AAIController
  {
  	GENERATED_BODY()
  
  public:
  	ASTUAIController();
  	
  protected:
  	virtual void OnPossess(APawn* InPawn) override;
  	virtual void BeginPlay() override;
  
  /* My Code */
  	// Property
  private:
  	UPROPERTY()
  	bool IsRunBehavior;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="AI", meta=(AllowPrivateAccess=true))
  	float CheckRate;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="AI", meta=(AllowPrivateAccess=true))
  	FName FocusOnKeyName;
  	
  public:
  	FTimerHandle CheckClosetEnemyTimerHandle;
  	
  	// Component
  private:
  	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category="AI", meta=(AllowPrivateAccess=true))
  	USTUAIPerceptionComponent* AIPerceptionComponent;
  
  	// Function
  public:
  	UFUNCTION()
  	void OnCheckClosetEnemy();
  
  	UFUNCTION()
  	AActor* GetFocusOnActor() const;
  };
  ```



- 实现：

  ```c++
  #include "STUAIController.h"
  #include "STUAI.h"
  #include "BehaviorTree/BlackboardComponent.h"
  #include "B_01_TPS/Component/STUAIPerceptionComponent.h"
  
  ASTUAIController::ASTUAIController()
  {
  	IsRunBehavior = false;
  	CheckRate = 0.1f;
      
      // 此处指定要访问 黑板组件 的 keyName
  	FocusOnKeyName = "EnemyActor";
  	
      // 此处指定初始化 AIPerceptionComponent
  	AIPerceptionComponent = CreateDefaultSubobject<USTUAIPerceptionComponent>(TEXT("AIPerceptionComp"));
  	if (AIPerceptionComponent){SetPerceptionComponent(*AIPerceptionComponent);}
  }
  
  void ASTUAIController::OnPossess(APawn* InPawn)
  {
  	Super::OnPossess(InPawn);
  
  	const auto STUCharacter = Cast<ASTUAI>(InPawn);
  
      // 此处执行 AI绑定的 BehaviorTree
  	if (!STUCharacter || !STUCharacter->GetBehaviorTreeAsset()){return;}
  	IsRunBehavior = RunBehaviorTree(STUCharacter->GetBehaviorTreeAsset());
  }
  
  void ASTUAIController::BeginPlay()
  {
  	Super::BeginPlay();
  
      // 开始按频率 检测最近的Actor
  	if (GetWorld() && IsRunBehavior)
  	{
  		GetWorldTimerManager().SetTimer(
  			CheckClosetEnemyTimerHandle,
  			this,
  			&ASTUAIController::OnCheckClosetEnemy,
  			CheckRate,
  			true);
  	}
  }
  
  // 设置 AI面向 最近的 Actor
  void ASTUAIController::OnCheckClosetEnemy()
  {
  	const auto AimActor = GetFocusOnActor();
  	SetFocus(AimActor);
  }
  
  // 在黑板组件的指定key中 拿到以设置的 最近的Actor
  AActor* ASTUAIController::GetFocusOnActor() const
  {
  	if (!GetBlackboardComponent()){return nullptr;}
  	return Cast<AActor>(GetBlackboardComponent()->GetValueAsObject(FocusOnKeyName));
  }
  ```







### 31. AI初始化和过渡旋转



情景：

- 初始化一个基本的`AI`
- 通过前面的`AIController`已经可以面向最近的`Actor`，但旋转没有过渡，需要实现过渡
- 关联相应的`BehaviorTree`
- 有`Dead`相关函数，方便通过`AIcontroller`停止`AI`的行为



示例：

- 定义：

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "B_01_TPS/Player/STUCharacterBase.h"
  #include "STUAI.generated.h"
  
  class UBehaviorTree;
  
  UCLASS()
  class B_01_TPS_API ASTUAI : public ASTUCharacterBase
  {
  	GENERATED_BODY()
  
  protected:
  	ASTUAI(const FObjectInitializer& ObjectInit);
  
  /* My Code */
  	// Property
  private:
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="AI", meta=(AllowPrivateAccess=true))
  	UBehaviorTree* BehaviorTreeAsset;
  
  public:
  	UFUNCTION()
  	FORCEINLINE UBehaviorTree* GetBehaviorTreeAsset() const {return BehaviorTreeAsset;}
  
  	virtual void OnDead() override;
  };
  ```

  

- 实现：

  ```c++
  #include "STUAI.h"
  #include "BrainComponent.h"
  #include "STUAIController.h"
  #include "B_01_TPS/Component/STUAIWeaponActorComponent.h"
  #include "B_01_TPS/Component/STUCharacterMovementComponent.h"
  
  ASTUAI::ASTUAI(const FObjectInitializer& ObjectInit) :
  Super(ObjectInit.SetDefaultSubobjectClass<USTUAIWeaponActorComponent>("WeaponActorComponent"))
  {
      // 初始化 AI 和 AIController
  	AutoPossessAI = EAutoPossessAI::PlacedInWorldOrSpawned;
  	AIControllerClass = ASTUAIController::StaticClass();
  	
      // 利用 CharacterMovementComponent 设置 AI 旋转面向 最近Actor 的过渡效果
  	if (GetCharacterMovement())
  	{
  		bUseControllerRotationYaw = false;
  		GetCharacterMovement()->bUseControllerDesiredRotation = true;
  		GetCharacterMovement()->RotationRate = FRotator(0.f, 200.f, 0.f);
  	}
  }
  
  void ASTUAI::OnDead()
  {
  	Super::OnDead();
  
      // 通知 AIController 停止 AI行为
  	const auto STUController = Cast<AAIController>(Controller);
  	if (STUController && STUController->BrainComponent)
  	{
  		STUController->BrainComponent->Cleanup();
  	}
  }
  ```

  





### 32. BeHaviorTree Task



情景：

- `AI Task`的关键信息是`AIController`和`BlackBoardComp`
- 通过`AIController`获得绑定的`AI Pawn`
- 通过`UNavigationSystemV1`获得场景的`NavMesh`导航
- 利用`BlackBoardComp`获得和设置`Key`值
- 以玩家随机进入下一个场景随机点为例



示例：

- 定义：

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "BehaviorTree/BTTaskNode.h"
  #include "STUNextLocationTask.generated.h"
  
  struct FBlackboardKeySelector;
  
  UCLASS()
  class B_01_TPS_API USTUNextLocationTask : public UBTTaskNode
  {
  	GENERATED_BODY()
  
  public:
  	USTUNextLocationTask();
  
  protected:
  	virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
  
  /* My Code */
  	// Property
  private:
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="AI|Task", meta=(AllowPrivateAccess=true))
  	float Radius;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="AI|Task", meta=(AllowPrivateAccess=true))
  	FBlackboardKeySelector AimLocationKey;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="AI|Task", meta=(AllowPrivateAccess=true))
  	bool IsSelfCenter;
  
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="AI|Task",
  		meta=(AllowPrivateAccess=true, EditCondition="!IsSelfCenter"))
  	FBlackboardKeySelector CenterActorKey;
  };
  ```



- 实现：

  ```c++
  #include "STUNextLocationTask.h"
  
  #include "AIController.h"
  #include "NavigationSystem.h"
  #include "BehaviorTree/BlackboardComponent.h"
  
  USTUNextLocationTask::USTUNextLocationTask()
  {
  	Radius = 1000.f;
  	NodeName = "Next Location";
  	IsSelfCenter = true;
  }
  
  EBTNodeResult::Type USTUNextLocationTask::ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory)
  {
      // 获得 AIController 和 BlackBoardComp
  	const auto Controller = OwnerComp.GetAIOwner();
  	const auto BlackBoard = OwnerComp.GetBlackboardComponent();
  
      // 通过 AIController 获得 AI Pawn
  	if (!Controller || !BlackBoard){return EBTNodeResult::Failed;}
  	const auto Pawn = Controller->GetPawn();
  
      // 获得场景中的 NavMeshSystem
  	if (!Pawn){return EBTNodeResult::Failed;}
  	const auto NavSys = UNavigationSystemV1::GetCurrent(Pawn);
  
      // 先获得 AI 当前的位置
  	if (!NavSys){return EBTNodeResult::Failed;}
  	FNavLocation NavLocation;
  	auto Location = Pawn->GetActorLocation();
  
      // 判断 AI 是否已经到达 随机的位置
      // 通过 黑板组件获得 指定位置的 AI 的 key值
  	if (IsSelfCenter){return EBTNodeResult::Failed;}
  	const auto CenterActor = Cast<AActor>(BlackBoard->GetValueAsObject(CenterActorKey.SelectedKeyName));
  
      // 但 AI到达随机位置，设置下个随机点的 位置
  	if (!CenterActor){return EBTNodeResult::Failed;}
  	Location = CenterActor->GetActorLocation();
  	const bool IsFound = NavSys->GetRandomReachablePointInRadius(Location, Radius, NavLocation);
  
      // 设定好下个 随机位置，需要用 黑板组件 设置 新的位置 key值
  	if (!IsFound){return EBTNodeResult::Failed;}
  	BlackBoard->SetValueAsVector(AimLocationKey.SelectedKeyName, NavLocation.Location);
  	return EBTNodeResult::Succeeded;
  }
  ```

  



### 33. BehaviorTree Service



情景：

- 利用前面`AIPerceptionComponent`，设置新的目标`Actor`
- 以通过`AIPerceptionComponent`获得新目标，设置`黑板组件`为例



示例：

- 定义：

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "BehaviorTree/BTService.h"
  #include "STUFindEnemyBTService.generated.h"
  
  struct FBlackboardKeySelector;
  
  UCLASS()
  class B_01_TPS_API USTUFindEnemyBTService : public UBTService
  {
  	GENERATED_BODY()
  
  public:
  	USTUFindEnemyBTService();
  
  protected:
  	virtual void TickNode(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds) override;
  
  /* My Code */
  	// Property
  private:
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="AI", meta=(AllowPrivateAccess=true))
  	FBlackboardKeySelector EnemyActorKey;
  };
  ```

  

- 实现：

  ```c++
  #include "STUFindEnemyBTService.h"
  #include "AIController.h"
  #include "BehaviorTree/BlackboardComponent.h"
  #include "B_01_TPS/Component/STUAIPerceptionComponent.h"
  #include "B_01_TPS/Dev/STUUtils.h"
  
  USTUFindEnemyBTService::USTUFindEnemyBTService()
  {
  	NodeName = "Find Enemy";
  }
  
  void USTUFindEnemyBTService::TickNode(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds)
  {
  	Super::TickNode(OwnerComp, NodeMemory, DeltaSeconds);
  
      // 获得 黑板 和 AIController
  	const auto BlackBoard = OwnerComp.GetBlackboardComponent();
  
  	if (!BlackBoard){return;}
  	const auto Controller = OwnerComp.GetAIOwner();
  
      // 获得 PerceptionComponent
  	if (!Controller){return;}
  	const auto PerceptionComponent = STUUtils::GetSTUPlayerComponent<USTUAIPerceptionComponent>(Controller);
  
      // 从 PerceptionComponent 得到新的 Actort 设置到 黑板 的 key值
  	if (!PerceptionComponent){return;}
  	BlackBoard->SetValueAsObject(EnemyActorKey.SelectedKeyName, PerceptionComponent->GetClosetEnemy());
  }
  ```

  





### 34. HUD生成Widget



情景：

- 设置自己的`HUD`生成指定`Widget`



示例：

- 定义：

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "GameFramework/HUD.h"
  #include "STUGameHUD.generated.h"
  
  UCLASS()
  class B_01_TPS_API ASTUGameHUD : public AHUD
  {
  	GENERATED_BODY()
  
  protected:
  	virtual void BeginPlay() override;
  	
  /* My Code */
  	// Function
  public:
  	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="UI", meta=(AllowPrivateAccess=true))
  	TSubclassOf<UUserWidget> PlayerHUDWidgetClass;
  };
  ```



- 实现：

  ```c++
  #include "STUGameHUD.h"
  #include "Blueprint/UserWidget.h"
  #include "Engine/Canvas.h"
  
  void ASTUGameHUD::BeginPlay()
  {
  	Super::BeginPlay();
  
  	const auto PlayerHUDWidget = CreateWidget<UUserWidget>(GetWorld(), PlayerHUDWidgetClass);
  
  	if (!PlayerHUDWidget){return;}
  	PlayerHUDWidget->AddToViewport();
  }
  ```





### 35. HUD绘制准星



情景：

- 直接通过`HUD`生成静态准星



示例：

- 定义：

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "GameFramework/HUD.h"
  #include "STUGameHUD.generated.h"
  
  UCLASS()
  class B_01_TPS_API ASTUGameHUD : public AHUD
  {
  	GENERATED_BODY()
  
  protected:
  	virtual void DrawHUD() override;
  public:
  	UFUNCTION()
  	void DrawCrossHair();
  };
  ```



- 实现：

  ```c++
  #include "STUGameHUD.h"
  #include "Blueprint/UserWidget.h"
  #include "Engine/Canvas.h"
  
  void ASTUGameHUD::DrawHUD()
  {
  	Super::DrawHUD();
  
  	DrawCrossHair();
  }
  
  void ASTUGameHUD::DrawCrossHair()
  {
  	const TInterval<float> Center(Canvas->SizeX * 0.5f, Canvas->SizeY * 0.5f);
  	constexpr float HalfLineSize = 10.f;
  	constexpr float LineThickness = 2.f;
  	const FColor LineColor = FColor::Green;
  
  	DrawLine(
  		Center.Min - HalfLineSize,
  		Center.Max,
  		Center.Min + HalfLineSize,
  		Center.Max,
  		LineColor,
  		LineThickness);
  	DrawLine(
  		Center.Min,
  		Center.Max - HalfLineSize,
  		Center.Min,
  		Center.Max + HalfLineSize,
  		LineColor,
  		LineThickness);
  }
  ```





### 36. Widget创建蓝图可用函数



情景：

- `C++`创建的`Widget`
- 创建`蓝图可调用`函数



示例：

- 定义：

  ```c++
  UCLASS()
  class B_01_TPS_API USTUPlayerHUDWidget : public UUserWidget
  {
  	GENERATED_BODY()
  
  /* My Code */
  	// Function
  public:
  	UFUNCTION(BlueprintCallable)
  	float GetHealthPercent() const;
  
  	UFUNCTION(BlueprintCallable)
  	bool IsPlayerAlive() const;
  
  	UFUNCTION(BlueprintCallable)
  	bool IsPlayerSpectating() const;
  };
  ```



- 实现：

  ```c++
  #include "STUPlayerHUDWidget.h"
  #include "B_01_TPS/Dev/STUUtils.h"
  #include "B_01_TPS/Component/STUHealthActorComponent.h"
  #include "B_01_TPS/Component/STUWeaponActorComponent.h"
  
  float USTUPlayerHUDWidget::GetHealthPercent() const
  {
  	const auto HealthComp = STUUtils::GetSTUPlayerComponent<USTUHealthActorComponent>(GetOwningPlayerPawn());
  	if (!HealthComp){return 0.f;}
  	return HealthComp->GetHPPercent();
  }
  
  bool USTUPlayerHUDWidget::IsPlayerSpectating() const
  {
  	const auto Controller = GetOwningPlayer();
  	return Controller && Controller->GetStateName() == NAME_Spectating;
  }
  
  bool USTUPlayerHUDWidget::IsPlayerAlive() const
  {
  	const auto HealthComp = STUUtils::GetSTUPlayerComponent<USTUHealthActorComponent>(GetOwningPlayerPawn());
  	return HealthComp && !HealthComp->IsDead();
  }
  ```

  





### 37. UPROPERTY()



情景：

> `private`：需要在蓝图中`可读可写`，任意处编辑
>
> > UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))



> 以 `bool IsTrue;`为判断标准，为`false`时，编辑器处为`不可编辑`状态
>
> > UPROPERTY(EditAnywhere, meta=(EditCondition="IsTrue"))



> `float HP;`为例，在编辑器中设置的值要符合一个固定的范围
>
> > UPROPERTY(EditAnywhere, meta=(ClampMin=0, ClampMax=100))





### 38. check-checkf-checkNoEntry



情景：

- 需要在`BeginPlay()`检测组件是否有效

  `check(GetMesh());`：如果`GetMesh()`无效，程序会中断

  

- 还需要通过条件判断是否有效，同时打印指定语句到日志

- 以`float HP;`为例，默认`BeginPlay()`时，`HP`应该`大于0`

  `checkf(HP > 0.f, TEXT("HP Shound Great 0"));`



- 应用于不可到达的代码片段，当出现不应该出现的情况时，程序中断

  ```c++
  void Function(AActor* Actor)
  {
      if (Actor)
      {
          ....
      }
      else
      {
          checkNoEntry();
      }
  }
  ```






### 39. UFUNCTION




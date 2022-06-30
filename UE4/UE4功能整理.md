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
  	const FVector Direction = FRotationMatrix(GetController()->GetControlRotation()).GetScaledAxis(EAxis::X);
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

  

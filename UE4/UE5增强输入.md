# EnhancedInput



使用增强输入完成Character的输入



### 1. 创建InputAction

创建`IA_Jump`：

- 默认值类型：数字（布尔）

创建`IA_Move`：

- 默认值类型：Axis2D（Vector2D）

创建`IA_Look`：

- 默认值类型：Axis2D（Vector2D）



### 2. 创建InputMappingContext

创建`IMC_Action`：

- 添加映射：`IA_Jump`
  - 映射按键：`Space`

创建`IMC_Movement`：

- 添加映射：`IA_Move`
  - 映射按键：`W`
    - 修改器：`拌合输入轴值：YXZ`
  - 映射按键：`S`
    - 修改器：`否定：XYZ`、`拌合输入轴值：YXZ`
  - 映射按键：`D`
  - 映射按键：`A`
    - 修改器：`否定：XYZ`
- 添加映射：`IA_Look`
  - 映射按键：`鼠标XY 2D轴`
    - 修改器：`否定：Y`



### 3. Build.cs

```c#
PublicDependencyModuleNames.AddRange(new string[] { "EnhancedInput" });
```



### 4. PlayerController

`PlayerController.h`

```c++
class UInputMappingContext;
class UInputAction;

public:
	// Input
	// Mapping
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Input|IMC", meta=(AllowPrivateAccess=true))
	UInputMappingContext* IMC_Movement;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Input|IMC", meta=(AllowPrivateAccess=true))
	UInputMappingContext* IMC_Action;

	// Action
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Input|IA", meta=(AllowPrivateAccess=true))
	UInputAction* IA_Move;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Input|IA", meta=(AllowPrivateAccess=true))
	UInputAction* IA_Look;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Input|IA", meta=(AllowPrivateAccess=true))
	UInputAction* IA_Jump;
```

`PlayerController.cpp`

```c++
#include "EnhancedInputLibrary.h"

void AMyPlayerControlller::BeginPlay()
{
	Super::BeginPlay();

	const auto EInputSubSys = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(GetLocalPlayer());
	if (EInputSubSys)
	{
		EInputSubSys->AddMappingContext(IMC_Movement, 0);
		EInputSubSys->AddMappingContext(IMC_Action, 0);
	}
}
```



### 5. Character

`Character.h`

```c++
class AMyPlayerControlller;
class USpringArmComponent;
class UCameraComponent;

private:
	UPROPERTY()
	AMyPlayerControlller* PC;

	UPROPERTY(VisibleDefaultsOnly, BlueprintReadWrite, Category="Player", meta=(AllowPrivateAccess=true))
	USpringArmComponent *PlayerSpringArmComponent;

	UPROPERTY(VisibleDefaultsOnly, BlueprintReadWrite, Category="Player", meta=(AllowPrivateAccess=true))
	UCameraComponent *PlayerCameraComponent;

	UFUNCTION()
	void Move(const FInputActionValue& InputValue);
	UFUNCTION()
	void Look(const FInputActionValue& InputValue);
```

`Character.cpp`

```c++
#include "EnhancedInputComponent.h"

AMyCharacter::AMyCharacter()
{
	PrimaryActorTick.bCanEverTick = true;

	GetCapsuleComponent()->InitCapsuleSize(36.f, 92.f);
	
	PlayerSpringArmComponent = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
	PlayerSpringArmComponent->SetupAttachment(RootComponent);
	PlayerSpringArmComponent->bUsePawnControlRotation = true;
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
	GetCharacterMovement()->RotationRate = FRotator(0.f, 720.f, 0.f);
	GetCharacterMovement()->GravityScale = 1.5f;
	GetCharacterMovement()->MaxAcceleration = 980.f;
	GetCharacterMovement()->JumpZVelocity = 600.f;
	GetCharacterMovement()->AirControl = 0.2f;

	bUseControllerRotationPitch = false;
	bUseControllerRotationRoll = false;
	bUseControllerRotationYaw = false;
}

void AMyCharacter::BeginPlay()
{
	Super::BeginPlay();

	PC = Cast<AMyPlayerControlller>(GetController());
}

void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	const auto EInputComp = Cast<UEnhancedInputComponent>(PlayerInputComponent);
	if (!EInputComp) return;
	
	PC = PC == nullptr ? Cast<AMyPlayerControlller>(GetController()) : PC;
	if (PC)
	{
		if (PC->IA_Move)
			EInputComp->BindAction(PC->IA_Move, ETriggerEvent::Triggered, this, &AMyCharacter::Move);
		if (PC->IA_Look)
			EInputComp->BindAction(PC->IA_Look, ETriggerEvent::Triggered, this, &AMyCharacter::Look);
		if (PC->IA_Jump)
			EInputComp->BindAction(PC->IA_Jump, ETriggerEvent::Started, this, &AMyCharacter::Jump);
			EInputComp->BindAction(PC->IA_Jump, ETriggerEvent::Completed, this, &ThisClass::StopJumping);
	}
}

void AMyCharacter::Move(const FInputActionValue& InputValue)
{
	PC = PC == nullptr ? Cast<AMyPlayerControlller>(GetController()) : PC;
	if (!PC) return;
	
	const auto MovementVector = InputValue.Get<FVector2D>();
	const auto Rotation = PC->GetControlRotation();
	const FRotator YawRotation{0, Rotation.Yaw, 0};
	const auto ForwardDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
	const auto RightDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);

	AddMovementInput(ForwardDirection, MovementVector.Y);
	AddMovementInput(RightDirection, MovementVector.X);
}

void AMyCharacter::Look(const FInputActionValue& InputValue)
{
	PC = PC == nullptr ? Cast<AMyPlayerControlller>(GetController()) : PC;
	if (!PC) return;

	const auto LookAxisVector = InputValue.Get<FVector2D>();

	AddControllerYawInput(LookAxisVector.X);
	AddControllerPitchInput(LookAxisVector.Y);
}
```


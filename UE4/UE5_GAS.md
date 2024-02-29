# UE5 GAS



[toc]



### 0. 整体概念

:::tip

一切都来自`AbilitySystemComponent`，简称：`ASC`

:::



:::warning 相关简称

`AbilitySystemComponent` -> `ASC`

`GameplayAbility` -> `GA`

`GameplayAbilitySpec` -> `GASpec`

`GameplayEffect` -> `GE`

`GameplayEffectSpec` -> `GESpec`

`GameplayEffectContext` -> `GEContext`

`GameplayEffectExecutionCalculation` -> `GEEC`，`GE_ExeCalc`

`GameplayEffectCooldown` -> `GE_CD`

`GameplayEffectCost` -> `GE_Cost`

`ActiveGameplayEffect` -> `ActiveGE`

`AttributeSet` -> `AS`

`GameplayCue` -> `GC`

`GameplayTask` -> `GT`

`GameplayModMagnitudeCalculation` -> `GMMC`，`GameplayModMagniCalc`

`GameplayEffectCustomApplicationRequirement` -> `GE_CAR`

:::



:::danger 通信原理

1. 凡是使用`GAS`或受到`GAS`影响的`Actor`都应该添加`ASC`
2. 拥有`ASC`的两个`Actor`之间所有的通信，都使用`ASC`发起和接受
3. `ASC`会将接受的响应回调给自身拥有的`GA`或`GE`
4. 发起信息前，需要初始化`ASC`
5. 拥有`ASC`的`Actor`也可确认需要使用的属性，抽象到`AS`中
6. `ASC`初始化完毕后，需要将`GA`赋予它
7. `GA`中可以添加相关`GE`
8. `GE`产生时，可以预先创建`GE`的`Context`和`Spec`，也可以指定相关`GameplayTag`筛选作用对象
9. `GameplayTag`独立于`GAS`，但`GAS`会有大量的逻辑通信使用到`GameplayTag`

:::



### 1. 使用GAS

1. 在编辑器中启用`GameplayAbilitySystem`插件

2. 编辑`YourProjectName.Build.cs`, 添加`"GameplayAbilities"`, `"GameplayTags"`, `"GameplayTasks"`到你的`PrivateDependencyModuleNames`

   ```C#
   using UnrealBuildTool;
   
   public class GAS_02 : ModuleRules
   {
   	public GAS_02(ReadOnlyTargetRules Target) : base(Target)
   	{
   		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
   
   		PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", 
                                                              "HeadMountedDisplay", "EnhancedInput" });
   		PrivateDependencyModuleNames.AddRange(new string[] { "GameplayAbilities", "GameplayTags", "GameplayTasks" 
                                                              });
   	}
   }
   ```

   

### 2. 添加和初始化

#### 2.1 添加ASC

- 无论是单机还是多人，考虑到玩家重生，并继续使用重生前的GE，AS的持久化内容，ASC应该添加到PlayerState
- 如果不考虑GE，AS持久化，单机中ASC可以直接挂在Character或PlayerController
- 否则ASC最好挂载到PlayerState上



:::warning 挂载PlayerState的注意事项

1. 提高PlayerState的`NetUpdateFrequency`, 其默认是一个很低的值, 因此在客户端上发生`Attribute`和`GameplayTag`改变时会造成延迟或卡顿. 确保启用[Adaptive Network Update Frequency](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/Properties/index.html#adaptivenetworkupdatefrequency)

:::



#### 2.2 初始化ASC

:::tip

这里演示普通多人的初始化方式

:::



在OwnerActor的构造函数中使用：

```c++
AGAS_02Character::AGAS_02Character()
{        
	// Init Ability System Component
	ASComponent = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
	ASComponent->SetIsReplicated(true);
	ASComponent->SetReplicationMode(EGameplayEffectReplicationMode::Minimal);
}
```



ASC设置同步模式：

| 同步模式  | 何时使用              | 描述                                                         |
| --------- | --------------------- | ------------------------------------------------------------ |
| `Full`    | 单人                  | 所有`GameplayEffect`都同步到客户端.                          |
| `Mixed`   | 多人, 玩家控制的Actor | `GameplayEffect`只同步到其所属客户端, 只有`GameplayTag`和`GameplayCue`同步到所有客户端. |
| `Minimal` | 多人, AI控制的Actor   | `GameplayEffect`从不同步到任何客户端, 只有`GameplayTag`和`GameplayCue`同步到所有客户端. |



:::warning 初始化细节

1. `Mixed`同步模式需要`OwnerActor`的`Owner`是`Controller`. 
2. `PlayerState`的`Owner`默认是`Controller`但是`Character`不是. 
3. 如果`OwnerActor`不是`PlayerState`时使用`Mixed`同步模式, 那么需要在`OwnerActor`中调用`SetOwner()`设置`Controller`.
4. 需要使用`PossessedBy()`设置新的`Controller`为`Pawn`的Owner.
5. `ASC`在`FActiveGameplayEffectContainer ActiveGameplayEffect`中保存其当前活跃的`GameplayEffect`.
6. `ASC`在`FGameplayAbilitySpecContainer ActivatableAbility`中保存其授予的`GameplayAbility`. 
7. 遍历`ActivatableAbility.Items`时, 确保在循环体之上添加`ABILITYLIST_SCOPE_LOCK();`来锁定列表以防其改变(比如移除一个Ability). 
8. 每个域中的`ABILITYLIST_SCOPE_LOCK();`会增加`AbilityScopeLockCount`, 之后出域时会减量. 
9. 不要尝试在`ABILITYLIST_SCOPE_LOCK();`域中移除某个Ability(Ability删除函数会在内部检查`AbilityScopeLockCount`以防在列表锁定时移除Ability).

:::



初始化ASC的位置：

- `OwnerActor`和`AvatarActor`的`ASC`在服务端和客户端上均需初始化, 你应该在`Pawn`的`Controller`设置之后初始化(Possess之后), 单人游戏只需参考服务端的做法.

- 对于玩家控制的Character且`ASC`位于`Pawn`, 一般在服务端`Pawn`的`PossessedBy()`函数中初始化, 在客户端`PlayerController`的`AcknowledgePossession()`函数中初始化.

- 对于玩家控制的Character且`ASC`位于`PlayerState`, 一般在服务端`Pawn`的`PossessedBy()`函数中初始化, 在客户端PlayerController的`OnRep_PlayerState()`函数中初始化, 这确保了`PlayerState`存在于客户端上

- `OwnerActor`是`Character`，客户端也可以在`OnRep_PlayerState()`中初始化

  ```c++
  void AGAS_02Character::PossessedBy(AController* NewController)
  {
  	Super::PossessedBy(NewController);
  
  	if (ASComponent)
  	{
  		ASComponent->InitAbilityActorInfo(this, this);
  
          // 此处可以在服务端，分别赋予OwnerActor的GA和初始GE
  		InitGameplayAbilities();
  		InitGameplayEffects();
  	}
      // ASC MixedMode replication requires that the ASC Owner's Owner be the Controller.
  	// SetOwner(NewController);
  }
  
  void AGAS_02Character::OnRep_PlayerState()
  {
  	Super::OnRep_PlayerState();
  
  	if (ASComponent)
  	{
  		ASComponent->InitAbilityActorInfo(this, this);
  
          // 客户端绑定按键输入和ASC的链接关系
  		BindAbilityInput();
  		InitGameplayEffects();
  	}
  }
  ```

  

  一下是`InitGameplayAbilities()`和`InitGameplayEffects()`，`BindAbilityInput()`

  ```c++
  UPROPERTY(BlueprintReadOnly, EditDefaultsOnly, Category="AFH|GAS|Ability", meta=(AllowPrivateAccess="true"))
  TArray<TSubclassOf<class UGGAbility>> DefaultAbilities;
  
  UPROPERTY(BlueprintReadOnly, EditDefaultsOnly, Category="AFH|GAS|Effect", meta=(AllowPrivateAccess="true"))
  TArray<TSubclassOf<class UGameplayEffect>> DefaultEffects;
  
  
  void AGAS_02Character::InitGameplayAbilities()
  {
  	if (HasAuthority() && ASComponent)
  	{
  		for (const auto& Ability : DefaultAbilities)
  		{
  			ASComponent->GiveAbility(
  				FGameplayAbilitySpec(
  					Ability, 1, static_cast<int32>(Ability.GetDefaultObject()->AbilityInputID), this));
  		}
  	}
  }
  
  void AGAS_02Character::InitGameplayEffects()
  {
  	if (!ASComponent) return;
  
  	auto EffectContext = ASComponent->MakeEffectContext();
  	EffectContext.AddSourceObject(this);
  	
  	for (const auto& Effect : DefaultEffects)
  	{
  		const auto SpecHandle = ASComponent->MakeOutgoingSpec(Effect, 1, EffectContext);
  		if (SpecHandle.IsValid())
  		{
  			ASComponent->ApplyGameplayEffectSpecToSelf(*SpecHandle.Data.Get());
  		}
  	}
  }
  
  void AGAS_02Character::BindAbilityInput()
  {
  	if (!IsAbilityBindOver && ASComponent && InputComponent)
  	{
  		const auto EnumAssetPath = FTopLevelAssetPath(
  			FName("/Script/GAS_02"), FName("EAbilityInputID"));
  		
  		ASComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(
  			FString("Confirm"),
  			FString("Cancel"),
  			EnumAssetPath,
  			static_cast<int32>(EAbilityInputID::Confirm),
  			static_cast<int32>(EAbilityInputID::Cancel)));
  
  		IsAbilityBindOver = true;
  	}
  }
  ```



:::danger 绑定Enum的路径问题

```c++
const auto EnumAssetPath = FTopLevelAssetPath(
		FName("/Script/GAS_02"), FName("EAbilityInputID"));
```



```
"/Script/GAS_02" 中的 GAS_02 更换为自己的项目名称
"EAbilityInputID" 更换为需要于ASC建立绑定的Enum名称
```



`GameplayAbilityInput.h`

```c++
#pragma once

#include "CoreMinimal.h"

UENUM(BlueprintType)
enum class EAbilityInputID : uint8
{
    None            	UMETA(Displayname="None"),
    Confirm             UMETA(Displayname="Confirm"),
    Cancel          	UMETA(Displayname="Cancel"),
    FireAbility         UMETA(Displayname="FireAbility"),
    ActionBar01Ability 	UMETA(Displayname="Ability01"),
    ActionBar02Ability 	UMETA(Displayname="Ability02"),
    ThrowGrenade      	UMETA(Displayname="ThrowGrenade")
};
```

:::



#### 2.3 添加访问接口

ASC添加到OwnerActor后，需要在OwnerActor继承`IAbilitySystemInterface`，并重载一个外部访问的接口`GetAbilitySystemComponent()`

```c++
virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;

UAbilitySystemComponent* AGAS_02Character::GetAbilitySystemComponent() const
{
	return ASComponent;
}
```



#### 2.4 增强输入绑定

当增强输入是bool类型，即存在`Pressed`和`Released`状态

1. 在客户端的`SetupPlayerInputComponent()`中绑定`InputAction`

   ```c++
   // .h
   UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
   class UInputAction* FireAction;
   
   UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
   class UInputAction* ThrowAction;
   
   void Fire(const FInputActionValue& Value);
   void Throw(const FInputActionValue& Value);
   
   UFUNCTION()
   void BindAbilityInput();
   
   // .cpp
   void AGAS_02Character::BindAbilityInput()
   {
   	if (!IsAbilityBindOver && ASComponent && InputComponent)
   	{
   		const auto EnumAssetPath = FTopLevelAssetPath(
   			FName("/Script/GAS_02"), FName("EAbilityInputID"));
   		
   		ASComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(
   			FString("Confirm"),
   			FString("Cancel"),
   			EnumAssetPath,
   			static_cast<int32>(EAbilityInputID::Confirm),
   			static_cast<int32>(EAbilityInputID::Cancel)));
   
   		IsAbilityBindOver = true;
   	}
   }
   
   void AGAS_02Character::SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent)
   {
   	if (UEnhancedInputComponent* EnhancedInputComponent = CastChecked<UEnhancedInputComponent>
           (PlayerInputComponent))
   	{
   		.....
   		
           // 额外绑定，用于发起GA
           EnhancedInputComponent->BindAction(FireAction, ETriggerEvent::Triggered, this, &AGAS_02Character::Fire);
   		EnhancedInputComponent->BindAction(ThrowAction, ETriggerEvent::Triggered, this, &AGAS_02Character::Throw);
   	}
   
       // 此次再次绑定，防止客端 OnRep_PlayerState() 中绑定失败
   	BindAbilityInput();
   }
   ```

2. 实现增强输入绑定的回调事件和在`SendAbilityLocalInput()`中处理按键和ASC的绑定

   ```c++
   void AGAS_02Character::Fire(const FInputActionValue& Value)
   {
   	SendAbilityLocalInput(Value, static_cast<int32>(EAbilityInputID::FireAbility));
   }
   
   void AGAS_02Character::Throw(const FInputActionValue& Value)
   {
   	SendAbilityLocalInput(Value, static_cast<int32>(EAbilityInputID::ThrowGrenade));
   }
   
   void AGAS_02Character::SendAbilityLocalInput(const FInputActionValue& Value, int32 InputID)
   {
   	if (!ASComponent) return;
   	
   	if (Value.Get<bool>())
   	{
   		ASComponent->AbilityLocalInputPressed(InputID);
   	}
   	else
   	{
   		ASComponent->AbilityLocalInputReleased(InputID);
   	}
   }
   ```





### 3. Attribute和AS

#### 3.1 Attribute定义

1. `Attribute`是由[FGameplayAttributeData](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayAttributeData/index.html)结构体定义的浮点值, 其可以表示从角色生命值到角色等级再到一瓶药水的剂量的任何事物
2. 如果某项数值是属于某个Actor且游戏相关的, 应该考虑使用`Attribute`. 
3. `Attribute`一般应该只能由[GameplayEffect](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ge)修改, 这样`ASC`才能[预测(Predict)](https://github.com/FHangH/GASDocumentation_Chinese#concepts-p)其改变.
4. `Attribute`也可以由`AttributeSet`定义并存于其中. [AttributeSet](https://github.com/FHangH/GASDocumentation_Chinese#concepts-as)用于同步那些标记为replication的`Attribute`.



:::tip

如果你不想某个`Attribute`显示在编辑器的`Attribute`列表, 可以使用`Meta = (HideInDetailsView)`属性宏.

:::



#### 3.2 Attribute概念

##### 3.2.1 Base和Current

- `Attribute`是由两个值：`BaseValue`和`CurrentValue`组成
  - `BaseValue`是`Attribute`的永久值
  - `CurrentValue`是`BaseValue`加上`GameplayEffect`给的临时修改值后得到的



:::warning 修改`CurrentValue` 和 `BaseValue`

1. 关于限制(Clamp)`Attribute`值的问题在[PreAttributeChange()](https://github.com/FHangH/GASDocumentation_Chinese#concepts-as-preattributechange)中讨论了CurrentValue的修改,

2. 在[PostGameplayEffectExecute()](https://github.com/FHangH/GASDocumentation_Chinese#concepts-as-postgameplayeffectexecute)中讨论了`GameplayEffect`对`BaseValue`的修改.
3. `即刻(Instant)GameplayEffect`可以永久性的修改`BaseValue`
4. `持续(Duration)`和`无限(Infinite)GameplayEffect`可以修改CurrentValue. 
5. 周期性(Periodic)`GameplayEffect`被视为`即刻(Instant)GameplayEffect`并且可以修改`BaseValue`.

:::



##### 3.2.2 MetaAttribute

一些`Attribute`被视为占位符, 其是用于预计和`Attribute`交互的临时值, 这些`Attribute`被叫做`Meta Attribute`.

例：

- 我们通常定义伤害值为`Meta Attribute`, 使用伤害值`Meta Attribute`作为占位符, 而不是使用`GameplayEffect`直接修改生命值`Attribute`
- 使用这种方法, 伤害值就可以在[GameplayEffectExecutionCalculation](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ge-ec)中由buff和debuff修改
- 并且可以在`AttributeSet`中进一步操作
- 在最终将生命值减去伤害值之前, 要将伤害值减去当前的护盾值
- 伤害值`Meta Attribute`在`GameplayEffect`之间不是持久化的, 并且可以被任何一方重写
- `Meta Attribute`一般是不可同步的.



#### 3.3 AttributeSet

- 简单来说，`AttributeSet`是`Attribute`的集合
- 一个`AS`可以含有多个`Attribute`
- 一个`Character`可以包含多个`AS`



##### 3.3.1 AS定义

:::tip AS中使用Attribute的方式

```
/**
 *  这是一个帮助程序宏，可在 RepNotify 函数中用于处理客户端将预测性修改的属性。
 *  
 *  void UMyHealthSet::OnRep_Health(const FGameplayAttributeData& OldValue)
 *  {
 *     GAMEPLAYATTRIBUTE_REPNOTIFY(UMyHealthSet, Health, OldValue);
 *  }
 */

/**
 * 这定义了一组用于访问和初始化属性的辅助函数，以避免手动编写这些函数。
 * 它将为属性 Health 创建以下函数
 *
 *  static FGameplayAttribute UMyHealthSet::GetHealthAttribute();
 *  FORCEINLINE float UMyHealthSet::GetHealth() const;
 *  FORCEINLINE void UMyHealthSet::SetHealth(float NewVal);
 *  FORCEINLINE void UMyHealthSet::InitHealth(float NewVal);
 *
 * 要在游戏中使用它，您可以定义如下内容，然后根据需要添加特定于游戏的函数：
 * 
 *  #define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
 *  GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
 *  GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
 *  GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
 *  GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
 * 
 *  ATTRIBUTE_ACCESSORS(UMyHealthSet, Health)
 */
```

:::



假设定义一个Player和Enemy通用的属性集合（AS），定义：`GGAttributeSet.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "AbilitySystemComponent.h"
#include "AttributeSet.h"
#include "GGAttributeSet.generated.h"

#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)

// 自定义的委托，用于拥有AS的OwnerActor响应这些委托
DECLARE_MULTICAST_DELEGATE_FourParams(
	FGGAttributeEvent, AActor* /*Instigator*/, AActor* /*Causer*/, const FGameplayEffectSpec&, float /*Effect_Magnitude*/);

UCLASS()
class GAS_02_API UGGAttributeSet : public UAttributeSet
{
	GENERATED_BODY()

public:
	UGGAttributeSet();

	UPROPERTY(BlueprintReadOnly, ReplicatedUsing="OnRep_Health", Category="AFH|GAS|AttributeSet", meta=(AllowPrivateAccess="true"))
	FGameplayAttributeData Health {100.f};
	ATTRIBUTE_ACCESSORS(UGGAttributeSet, Health)

	UPROPERTY(BlueprintReadOnly, ReplicatedUsing="OnRep_MaxHealth", Category="AFH|GAS|AttributeSet", meta=(AllowPrivateAccess="true"))
	FGameplayAttributeData MaxHealth {100.f};
	ATTRIBUTE_ACCESSORS(UGGAttributeSet, MaxHealth)

	UPROPERTY(BlueprintReadOnly, ReplicatedUsing="OnRep_Armor", Category="AFH|GAS|AttributeSet", meta=(AllowPrivateAccess="true"))
	FGameplayAttributeData Armor {100.f};
	ATTRIBUTE_ACCESSORS(UGGAttributeSet, Armor)

	UPROPERTY(BlueprintReadOnly, ReplicatedUsing="OnRep_MaxArmor", Category="AFH|GAS|AttributeSet", meta=(AllowPrivateAccess="true"))
	FGameplayAttributeData MaxArmor {100.f};
	ATTRIBUTE_ACCESSORS(UGGAttributeSet, MaxArmor)
        
    // InDamage在服务端处理，无需复制到客户端
    UPROPERTY(BlueprintReadOnly, Category="AFH|GAS|AttributeSet", meta=(AllowPrivateAccess="true"))
	FGameplayAttributeData InDamage {};
	ATTRIBUTE_ACCESSORS(UGGAttributeSet, InDamage)
	
	mutable FGGAttributeEvent OnOutOfHealth;
	mutable FGGAttributeEvent OnOutOfArmor;


protected:	
	// Function
	// Override
	virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;
	virtual void PreAttributeBaseChange(const FGameplayAttribute& Attribute, float& NewValue) const override;
	virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) override;
	virtual void PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data) override;

	// New Define
	UFUNCTION()
	virtual void ClampAttributeOnChange(const FGameplayAttribute& Attribute, float& NewValue) const;
	
	UFUNCTION()
	virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);

	UFUNCTION()
	virtual void OnRep_MaxHealth(const FGameplayAttributeData& OldMaxHealth);

	UFUNCTION()
	virtual void OnRep_Armor(const FGameplayAttributeData& OldArmor);

	UFUNCTION()
	virtual void OnRep_MaxArmor(const FGameplayAttributeData& OldMaxArmor);
};

```



实现：`GGAttributeSet.cpp`

```c++
#include "GGAttributeSet.h"
#include "GameplayEffectExtension.h"
#include "Net/UnrealNetwork.h"

UGGAttributeSet::UGGAttributeSet()
{
}

// Override
void UGGAttributeSet::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME_CONDITION_NOTIFY(UGGAttributeSet, Health, COND_None, REPNOTIFY_Always);
	DOREPLIFETIME_CONDITION_NOTIFY(UGGAttributeSet, MaxHealth, COND_None, REPNOTIFY_Always);
	DOREPLIFETIME_CONDITION_NOTIFY(UGGAttributeSet, Armor, COND_None, REPNOTIFY_Always);
	DOREPLIFETIME_CONDITION_NOTIFY(UGGAttributeSet, MaxArmor, COND_None, REPNOTIFY_Always);
}

void UGGAttributeSet::PreAttributeBaseChange(const FGameplayAttribute& Attribute, float& NewValue) const
{
	Super::PreAttributeBaseChange(Attribute, NewValue);

	ClampAttributeOnChange(Attribute, NewValue);
}

void UGGAttributeSet::PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)
{
	Super::PreAttributeChange(Attribute, NewValue);

	ClampAttributeOnChange(Attribute, NewValue);
}

void UGGAttributeSet::PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data)
{
	Super::PostGameplayEffectExecute(Data);

	if (Data.EvaluatedData.Attribute == GetInDamageAttribute())
	{
        // 此处进行些 Attribute的修改
        // 也可以回调一些方法，使得拥有的OwnerActor可以响应
        if (OnOutOfArmor.IsBound())
        {
            OnOutofArmor.Broadcast();
        }
    }
}

// New Define
void UGGAttributeSet::ClampAttributeOnChange(const FGameplayAttribute& Attribute, float& NewValue) const
{
	if (Attribute == GetHealthAttribute())
	{
		NewValue =  FMath::Clamp(NewValue, 0.f, GetMaxHealth());
	}
	if (Attribute == GetArmorAttribute())
	{
		NewValue =  FMath::Clamp(NewValue, 0.f, GetMaxArmor());
	}
}

void UGGAttributeSet::OnRep_Health(const FGameplayAttributeData& OldHealth)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGGAttributeSet, Health, OldHealth);
}

void UGGAttributeSet::OnRep_MaxHealth(const FGameplayAttributeData& OldMaxHealth)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGGAttributeSet, MaxHealth, OldMaxHealth);
}

void UGGAttributeSet::OnRep_Armor(const FGameplayAttributeData& OldArmor)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGGAttributeSet, Armor, OldArmor);
}

void UGGAttributeSet::OnRep_MaxArmor(const FGameplayAttributeData& OldMaxArmor)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGGAttributeSet, MaxArmor, OldMaxArmor);
}
```



##### 3.3.2 AS初始化

```c++
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="AFH|GAS|AttributeSet", meta=(AllowPrivateAccess="true"))
UGGAttributeSet* AttributeSet;

AGAS_02Character::AGAS_02Character()
{
	// Init AttributeSet
	AttributeSet = CreateDefaultSubobject<UGGAttributeSet>(TEXT("AttributeSet"));
}
```



##### 3.3.3 Attribute改变回调

- `AS`内的`Attribute`发生改变：`Set()`，`Init()`；
- 都会触发`Attribute`的变化回调；
- 为了监听`Attribute`何时变化以便更新UI和其他游戏逻辑, 可以使用`UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute `Attribute`)`,
- 该函数返回一个委托(Delegate), 你可以将其绑定一个当`Attribute`变化时需要自动调用的函数. 
- 该委托提供一个`FOnAttributeChangeData`参数, 其中有`NewValue`, `OldValue`和`FGameplayEffectModCallbackData`.



:::warning

`FGameplayEffectModCallbackData`只能在服务端上设置

:::



在OwnerActor的进行AS初始化后，进行回调绑定：

```c++
virtual void OnHealthAttributeChanged(const FOnAttributeChangeData& Data);
virtual void OnArmorAttributeChanged(const FOnAttributeChangeData& Data);

UFUNCTION(BlueprintNativeEvent, Category="GAS")
void OnHealthChanged(float OldHealth, float NewHealth);

UFUNCTION(BlueprintNativeEvent, Category="GAS")
void OnArmorChanged(float OldArmor, float NewArmor);

void AGAS_02Character::BeginPlay()
{
	Super::BeginPlay();

	if (ASComponent)
	{
		ASComponent->GetGameplayAttributeValueChangeDelegate(
			AttributeSet->GetHealthAttribute()).AddUObject(this, &AGAS_02Character::OnHealthAttributeChanged);

		ASComponent->GetGameplayAttributeValueChangeDelegate(
			AttributeSet->GetArmorAttribute()).AddUObject(this, &AGAS_02Character::OnArmorAttributeChanged);
	}
}

void AGAS_02Character::OnHealthAttributeChanged(const FOnAttributeChangeData& Data)
{
	OnHealthChanged(Data.OldValue, Data.NewValue);
}

void AGAS_02Character::OnArmorAttributeChanged(const FOnAttributeChangeData& Data)
{
	OnArmorChanged(Data.OldValue, Data.NewValue);
}

void AGAS_02Character::OnHealthChanged_Implementation(float OldHealth, float NewHealth)
{
	// TODO Blueprint Override Function
}

void AGAS_02Character::OnArmorChanged_Implementation(float OldArmor, float NewArmor)
{
	// TODO Blueprint Override Function
}
```



:::tip `AttributeSet`内实现自定义委托绑定

ASC提供的回调不包含更多信息，自定义的委托可以额外回调更详细的内容

前面`AS`定义的时候，内部自定义了委托：

```c++
DECLARE_MULTICAST_DELEGATE_FourParams(
	FGGAttributeEvent, AActor* /*Instigator*/, AActor* /*Causer*/, const FGameplayEffectSpec&, float 
	/*Effect_Magnitude*/);
	
mutable FGGAttributeEvent OnOutOfHealth;
mutable FGGAttributeEvent OnOutOfArmor;
```



在`ASC`所在的`OwnerActor`内，初始化`AS`后，可以添加：

```c++
virtual void OnOutOfHealthChanged(AActor* DamageInstigator, AActor* DamageCauser, const FGameplayEffectSpec& 
                                  DamageEffectSpec, float DamageMagnitude);

virtual void OnOutOfArmorChanged(AActor* DamageInstigator, AActor* DamageCauser, const FGameplayEffectSpec& 
                                 DamageEffectSpec, float DamageMagnitude);
```



完整绑定方式：`AGAS_02Character.h`

```c++
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="AFH|GAS|AttributeSet", meta=(AllowPrivateAccess="true"))
UGGAttributeSet* AttributeSet;

UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="AFH|GAS", meta=(AllowPrivateAccess="true"))
UAbilitySystemComponent* ASComponent;

virtual void OnOutOfHealthChanged(AActor* DamageInstigator, AActor* DamageCauser, const FGameplayEffectSpec& 
                                  DamageEffectSpec, float DamageMagnitude);
virtual void OnOutOfArmorChanged(AActor* DamageInstigator, AActor* DamageCauser, const FGameplayEffectSpec& 
                                 DamageEffectSpec, float DamageMagnitude);

UFUNCTION(BlueprintNativeEvent, Category="AFH|GAS")
void OnOutOfHealth(AActor* DamageInstigator, AActor* DamageCauser, const FGameplayEffectSpec& DamageEffectSpec, float 
                   DamageMagnitude);
	
UFUNCTION(BlueprintNativeEvent, Category="AFH|GAS")
void OnOutOfArmor(AActor* DamageInstigator, AActor* DamageCauser, const FGameplayEffectSpec& DamageEffectSpec, float 
                  DamageMagnitude);
```



`AGAS_02Character.cpp`

```c++
AGAS_02Character::AGAS_02Character()
{
	// Init Ability System Component
	ASComponent = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
	ASComponent->SetIsReplicated(true);
	ASComponent->SetReplicationMode(EGameplayEffectReplicationMode::Minimal);

	// Init AttributeSet
	AttributeSet = CreateDefaultSubobject<UGGAttributeSet>(TEXT("AttributeSet"));
}

void AGAS_02Character::BeginPlay()
{
	Super::BeginPlay();

	if (ASComponent)
	{
		ASComponent->GetGameplayAttributeValueChangeDelegate(
			AttributeSet->GetHealthAttribute()).AddUObject(this, &AGAS_02Character::OnHealthAttributeChanged);

		ASComponent->GetGameplayAttributeValueChangeDelegate(
			AttributeSet->GetArmorAttribute()).AddUObject(this, &AGAS_02Character::OnArmorAttributeChanged);
	}
	if (AttributeSet)
	{
		AttributeSet->OnOutOfHealth.AddUObject(this, &AGAS_02Character::OnOutOfHealthChanged);
		AttributeSet->OnOutOfArmor.AddUObject(this, &AGAS_02Character::OnOutOfArmorChanged);
	}
}

void AGAS_02Character::OnOutOfHealthChanged(AActor* DamageInstigator, AActor* DamageCauser,
                                            const FGameplayEffectSpec& DamageEffectSpec, float DamageMagnitude)
{
	OnOutOfHealth(DamageInstigator, DamageCauser, DamageEffectSpec, DamageMagnitude);
}

void AGAS_02Character::OnOutOfArmorChanged(AActor* DamageInstigator, AActor* DamageCauser,
	const FGameplayEffectSpec& DamageEffectSpec, float DamageMagnitude)
{
	OnOutOfArmor(DamageInstigator, DamageCauser, DamageEffectSpec, DamageMagnitude);
}

void AGAS_02Character::OnOutOfHealth_Implementation(AActor* DamageInstigator, AActor* DamageCauser,
	const FGameplayEffectSpec& DamageEffectSpec, float DamageMagnitude)
{
	// TODO Blueprint Override Function
}

void AGAS_02Character::OnOutOfArmor_Implementation(AActor* DamageInstigator, AActor* DamageCauser,
	const FGameplayEffectSpec& DamageEffectSpec, float DamageMagnitude)
{
	// TODO Blueprint Override Function
}
```

:::



##### 3.3.4 PreAttributeChange

- `PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)`是`AttributeSet`中的主要函数之一
- 其在修改发生前响应`Attribute`的CurrentValue变化
- 其是通过引用参数NewValue限制(Clamp)CurrentValue即将进行的修改的理想位置
- `PreAttributeChange()`可以被`Attribute`的任何修改触发, 无论是使用`Attribute`的setter(由`AttributeSet.h`中的宏块定义([定义Attribute](https://github.com/FHangH/GASDocumentation_Chinese#concepts-as-attributes)))还是使用[GameplayEffect](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ge).



:::warning

- 在这里做的任何限制都不会永久性地修改`ASC`中的`Modifier`, 只会修改查询`Modifier`的返回值,
- 这意味着像[GameplayEffectExecutionCalculations](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ge-ec)和[ModifierMagnitudeCalculations](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ge-mmc)这种自所有`Modifier`重新计算CurrentValue的函数需要再次执行限制(Clamp)操作.

- Epic对于`PreAttributeChange()`的注释说明不要将该函数用于游戏逻辑事件, 而主要在其中做限制操作. 
- 对于修改`Attribute`的游戏逻辑事件的建议位置是`UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`([响应Attribute变化](https://github.com/FHangH/GASDocumentation_Chinese#concepts-a-changes)).

:::



##### 3.3.5 PostGameplayEffectExecute

- `PostGameplayEffectExecute(const FGameplayEffectModCallbackData & Data)`仅在`即刻(Instant)GameplayEffect`对`Attribute`的BaseValue修改之后触发
- 当[GameplayEffect](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ge)对其修改时, 这就是一个处理更多`Attribute`操作的有效位置.
- 其他只会由`即刻(Instant)GameplayEffect`修改BaseValue的`Attribute`, 像魔法值和耐力值, 也可以在这里被限制为其相应的最大值`Attribute`.



:::warning

- 当PostGameplayEffectExecute()被调用时, 对`Attribute`的修改已经发生, 但是还没有被同步回客户端
- 因此在这里限制值不会造成对客户端的二次同步, 客户端只会接收到限制后的值.

:::



### 4. GameplayAbility

#### 4.1 GA概念

[GameplayAbility(GA)](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/UGameplayAbility/index.html)是Actor在游戏中可以触发的一切行为和技能. 多个`GameplayAbility`可以在同一时刻激活, 例如奔跑和射击. 其可由蓝图或C++完成.

`GameplayAbility`示例:

- 跳跃
- 奔跑
- 射击
- 每X秒被动地阻挡一次攻击
- 使用药剂
- 开门
- 收集资源
- 建造

不应该使用`GameplayAbility`的场景:

- 基础移动输入
- 一些与UI的交互 - 不要使用`GameplayAbility`从商店中购买物品



1. `GameplayAbility`运行在所属(Owning)客户端还是服务端取决于[网络执行策略(Net Execution Policy)](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ga-net)而不是Simulated Proxy. 
2. `网络执行策略(Net Execution Policy)`决定某个`GameplayAbility`是否是客户端可[预测](https://github.com/FHangH/GASDocumentation_Chinese#concepts-p)的, 其对于[可选的Cost和`Cooldown GameplayEffect`](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ga-commit)包含有默认行为.
3.  `GameplayAbility`使用[AbilityTask](https://github.com/FHangH/GASDocumentation_Chinese#concepts-at)用于随时间推移而发生的行为, 例如等待某个事件, 等待某个Attribute改变, 等待玩家选择一个目标或者使用`Root Motion Source`移动某个`Character`. 
4. **Simulated Client不会运行`GameplayAbility`,** 而是当服务端执行`Ability`时, 任何需要在Simulated Proxy上展现的视觉效果(像动画蒙太奇)将会被同步(Replicate)或者通过`AbilityTask`进行RPC或者对于像声音和粒子这样的装饰效果使用[GameplayCue](https://github.com/FHangH/GASDocumentation_Chinese#concepts-gc).



:::warning

所有的`GameplayAbility`都会有它们各自由你的游戏逻辑重写的`ActivateAbility()`函数, 附加的逻辑可以添加到`EndAbility()`, 其会在`GameplayAbility`完成或取消时执行.

:::



一个简单的`GameplayAbility`流程图: 

![Simple GameplayAbility Flowchart](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/abilityflowchartsimple.png)

一个更复杂`GameplayAbility`流程图: 

![Complex GameplayAbility Flowchart](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/abilityflowchartcomplex.png)



#### 4.2 GA创建

- 一般情况下，存在输入通过ASC绑定GA的情况；
- 所以除了建立一个输入枚举之外，最好是自定义一个新的GameplayAbility



```c++
#pragma once

#include "CoreMinimal.h"
#include "Abilities/GameplayAbility.h"
#include "GAS_02/Utils/GameAbilityInput.h"
#include "GGAbility.generated.h"

//	The important functions:
//	
//		CanActivateAbility()	- const 函数来查看能力是否可激活。可通过UI等调用
//
//		TryActivateAbility()	- 尝试激活该技能。调用 CanActivateAbility（）。输入事件可以直接调用它。
//								- 还处理每次执行实例化逻辑和复制/预测调用。
//		
//		CallActivateAbility()	- 受保护的非虚拟功能。做一些样板“预激活”的东西，然后调用 ActivateAbility（）
//
//		ActivateAbility()		- 能力*做什么*。这是子类想要覆盖的内容。
//	
//		CommitAbility()			- 提交资源/冷却时间等。ActivateAbility（） 必须调用这个！
//		
//		CancelAbility()			- 中断技能（来自外部来源）。
//
//		EndAbility()			- 能力已经结束。这是为了通过结束自身的能力来调用。

UCLASS()
class GAS_02_API UGGAbility : public UGameplayAbility
{
	GENERATED_BODY()

public:
	UPROPERTY(BlueprintReadOnly, EditAnywhere, Category="AFH|GAS|Ability")
	EAbilityInputID AbilityInputID { EAbilityInputID::None };
};
```



#### 4.3 绑定输入

绑定流程：(增强输入)

1. 定义IA
2. 添加IM
3. 客户端输入初始化，绑定输入回调函数
4. 输入回调函数，通过ASC绑定输入ID



#### 4.4 绑定时不激活GA

可以在`UGameplayAbility`子类中添加一个新的布尔变量, `bActivateOnInput`, 其默认值为`true`并重写`UAbilitySystemComponent::AbilityLocalInputPressed()`



```c++
void UGSAbilitySystemComponent::AbilityLocalInputPressed(int32 InputID)
{
	// Consume the input if this InputID is overloaded with GenericConfirm/Cancel and the GenericConfim/Cancel callback is bound
	if (IsGenericConfirmInputBound(InputID))
	{
		LocalInputConfirm();
		return;
	}

	if (IsGenericCancelInputBound(InputID))
	{
		LocalInputCancel();
		return;
	}

	// ---------------------------------------------------------

	ABILITYLIST_SCOPE_LOCK();
	for (FGameplayAbilitySpec& Spec : ActivatableAbilities.Items)
	{
		if (Spec.InputID == InputID)
		{
			if (Spec.Ability)
			{
				Spec.InputPressed = true;
				if (Spec.IsActive())
				{
					if (Spec.Ability->bReplicateInputDirectly && IsOwnerActorAuthoritative() == false)
					{
						ServerSetInputPressed(Spec.Handle);
					}

					AbilitySpecInputPressed(Spec);

					// Invoke the InputPressed event. This is not replicated here. If someone is listening, they may replicate the InputPressed event to the server.
					InvokeReplicatedEvent(EAbilityGenericReplicatedEvent::InputPressed, Spec.Handle, Spec.ActivationInfo.GetActivationPredictionKey());
				}
				else
				{
					UGSGameplayAbility* GA = Cast<UGSGameplayAbility>(Spec.Ability);
					if (GA && GA->bActivateOnInput)
					{
						// Ability is not active, so try to activate it
						TryActivateAbility(Spec.Handle);
					}
				}
			}
		}
	}
}
```



#### 4.5 授予GA

- 向`ASC`授予`GameplayAbility`会将其添加到`ASC`的`ActivatableAbilities`列表, 从而允许其在满足[`GameplayTag`需求](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ga-tags)时激活该`GameplayAbility`.
- 我们在服务端授予`GameplayAbility`, 之后其会自动同步[GameplayAbilitySpec](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ga-spec)到所属(Owning)客户端, 其他客户端/Simulated proxy不会接受到`GameplayAbilitySpec`.
- 游戏开始时将`TArray<TSubclassOf<UGDGameplayAbility>>`保存在它读取和授予的`Character`类中.



```c++
void AGDCharacterBase::AddCharacterAbilities()
{
	// Grant abilities, but only on the server	
	if (Role != ROLE_Authority || !AbilitySystemComponent.IsValid() || AbilitySystemComponent->CharacterAbilitiesGiven)
	{
		return;
	}

	for (TSubclassOf<UGDGameplayAbility>& StartupAbility : CharacterAbilities)
	{
		AbilitySystemComponent->GiveAbility(
			FGameplayAbilitySpec(
                StartupAbility, 
                GetAbilityLevel(StartupAbility.GetDefaultObject()->AbilityID), 
                static_cast<int32>(StartupAbility.GetDefaultObject()->AbilityInputID), 
                this));
	}

	AbilitySystemComponent->CharacterAbilitiesGiven = true;
}
```



#### 4.6 被动GA

- 实现自动激活和持续运行的被动`GameplayAbility`, 需要重写`UGameplayAbility::OnAvatarSet()`, 该函数在授予`GameplayAbility`并设置`AvatarActor`且调用`TryActivateAbility()`时自动调用.
- 添加一个`布尔值`到你的自定义`UGameplayAbility`类来表明其在授予时是否应该被激活
- 被动`GameplayAbility`一般有一个`仅服务器(Server Only)`的[网络执行策略(Net Execution Policy)](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ga-net).



```c++
void UGDGameplayAbility::OnAvatarSet(const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilitySpec & Spec)
{
	Super::OnAvatarSet(ActorInfo, Spec);

	if (ActivateAbilityOnGranted)
	{
		bool ActivatedAbility = ActorInfo->AbilitySystemComponent->TryActivateAbility(Spec.Handle, false);
	}
}
```



:::warning

Epic描述该函数为初始化被动Ability的正确位置和应该做一些类似`BeginPlay`的事情.

:::



#### 4.7 取消GA

- 从内部取消`GameplayAbility`, 可以调用`CancelAbility()`, 其会调用`EndAbility()`并设置它的`WasCancelled`参数为`true`.
- 从外部取消`GameplayAbility`, `ASC`提供了一些函数:



```c++
/** Cancels the specified ability CDO. */
void CancelAbility(UGameplayAbility* Ability);	

/** Cancels the ability indicated by passed in spec handle. If handle is not found among reactivated abilities nothing happens. */
void CancelAbilityHandle(const FGameplayAbilitySpecHandle& AbilityHandle);

/** Cancel all abilities with the specified tags. Will not cancel the Ignore instance */
void CancelAbilities(const FGameplayTagContainer* WithTags=nullptr, const FGameplayTagContainer* WithoutTags=nullptr, UGameplayAbility* Ignore=nullptr);

/** Cancels all abilities regardless of tags. Will not cancel the ignore instance */
void CancelAllAbilities(UGameplayAbility* Ignore=nullptr);

/** Cancels all abilities and kills any remaining instanced abilities */
virtual void DestroyActiveState();
```



:::warning

- 如果存在一个`非实例(Non-Instanced)GameplayAbility`时, `CancelAllAbilities`似乎不能正常运行
- 它似乎会命中这个`非实例(Non-Instanced)GameplayAbility`并放弃继续处理. 
- `CancelAbility`可以更好地处理`非实例(Non-Instanced)GameplayAbility`

:::



#### 4.8 获取激活的GA

1. `GetActivatableAbilities()`
   - 搜索`ASC`的`ActivatableAbility`列表(`ASC`拥有的已授予`GameplayAbility`)并找到一个与你正在寻找的[资源或授予的GameplayTag](https://github.com/FHangH/GASDocumentation_Chinese#concepts-ga-tags)相匹配的Ability.
   - `UAbilitySystemComponent::GetActivatableAbilities()`会返回一个用于遍历的`TArray<FGameplayAbilitySpec>`.
2. `GetActivatableGameplayAbilitySpecsByAllMatchingTags()`
   - `ASC`还有另一个有用的函数, 它将一个`GameplayTagContainer`作为参数来协助搜索, 而无需手动遍历`GameplayAbilitySpec`列表
   - `bOnlyAbilitiesThatSatisfyTagRequirements`参数只会返回那些`GameplayTag`满足需求且可以立刻激活的`GameplayAbilitySpecs`
   - 获取到了寻找的`FGameplayAbilitySpec`, 那么就可以调用它的`IsActive()`.



```c++
// 返回所有可以激活的GA
if (ASComponent)
{
    const auto& ActivatedAbilities = ASComponent->GetActivatableAbilities();
    for (const auto& AbilitySpec : ActivatedAbilities)
    {
       
    }
}

// 返回所有依据GameplayTag匹配的，可以任何形式激活的GA
if (ASComponent)
{
    const FGameplayTagContainer GameplayTagContainer;
    TArray<FGameplayAbilitySpec*> MatchingGameplayAbilities; 
    ASComponent->GetActivatableGameplayAbilitySpecsByAllMatchingTags(GameplayTagContainer, MatchingGameplayAbilities);

    for (const auto& AbilitySpec :MatchingGameplayAbilities)
    {
       if (AbilitySpec->IsActive())
       {
          
       }
    }
}
```



#### 4.9 实例化策略

`GameplayAbility`的实例化策略决定了当`GameplayAbility`激活时是否和如何实例化.

| 实例化策略                            | 描述                                                         | 何时使用的例子                                               |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 按Actor实例化(Instanced Per Actor)    | 每个`ASC`只能有一个在激活之间复用的`GameplayAbility`实例.    | 这可能是你使用最频繁的实例化策略. 你可以对任一`Ability`使用并在激活之间提供持久化. 设计者可以在激活之间手动重设任意变量. |
| 按操作实例化(Instanced Per Execution) | 每有一个`GameplayAbility`激活, 就有一个新的`GameplayAbility`实例创建. | 这些`GameplayAbility`的好处是每次激活时变量都会重置, 其性能要比`Instanced Per Actor`差, 因为每次激活时都会生成新的`GameplayAbility`. 样例项目没有使用该方式. |
| 非实例化(Non-Instanced)               | `GameplayAbility`操作其`ClassDefaultObject`, 没有实例创建.   | 它是三种方式中性能最好的, 但是使用它是最受限制的. `非实例化(Non-Instanced)GameplayAbility`不能存储状态, 这意味着没有动态变量和不能绑定到`AbilityTask`委托. 使用它的最佳场景就是需要频繁使用的简单Ability, 像MOBA或RTS游戏中小兵的基础攻击. 样例项目中的跳跃`GameplayAbility`就是`非实例化(Non-Instanced)`的. |



#### 4.10 网络执行策略(Net Execution Policy)

`GameplayAbility`的`网络执行策略(Net Execution Policy)`决定了谁该以什么顺序运行该`GameplayAbility`.

| 网络执行策略(Net Execution Policy) | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| Local Only                         | `GameplayAbility`只运行在所属(Owning)客户端. 这对那些只需做本地视觉变化的Ability很有用. 单人游戏应该使用`Server Only`. |
| Local Predicted                    | `Local Predicted GameplayAbility`首先在所属(Owning)客户端激活, 之后在服务端激活. 服务端版本会纠正客户端预测的所有不正确的地方. 参见Prediction. |
| Server Only                        | `GameplayAbility`只运行在服务端. 被动`GameplayAbility`一般是`Server Only`. 单人游戏应该使用该项. |
| Server Initiated                   | `Server Initiated GameplayAbility`首先在服务端激活, 之后在所属(Owning)客户端激活. 我个人使用的不多(如果有的话). |



#### 4.11 GA标签

`GameplayAbility`自带有内建逻辑的`GameplayTagContainer`. 这些`GameplayTag`都不进行同步.

| GameplayTagContainer      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Ability Tags              | `GameplayAbility`拥有的`GameplayTag`, 这只是用来描述`GameplayAbility`的`GameplayTag`. |
| Cancel Abilities with Tag | 当该`GameplayAbility`激活时, 其他`Ability Tags`中拥有这些`GameplayTag`的`GameplayAbility`将会被取消. |
| Block Abilities with Tag  | 当该`GameplayAbility`激活时, 其他`Ability Tags`中拥有这些`GameplayTag`的`GameplayAbility`将会阻塞激活. |
| Activation Owned Tags     | 当该`GameplayAbility`激活时, 这些`GameplayTag`会交给该`GameplayAbility`的拥有者. |
| Activation Required Tags  | 该`GameplayAbility`只有在其拥有者拥有所有这些`GameplayTag`时才会激活. |
| Activation Blocked Tags   | 该`GameplayAbility`在其拥有者拥有任意这些标签时不能被激活.   |
| Source Required Tags      | 该`GameplayAbility`只有在`Source`拥有所有这些`GameplayTag`时才会激活. `Source GameplayTag`只有在该`GameplayAbility`由Event触发时设置. |
| Source Blocked Tags       | 该`GameplayAbility`在`Source`拥有任意这些标签时不能被激活. `Source GameplayTag`只有在该`GameplayAbility`由Event触发时设置. |
| Target Required Tags      | 该`GameplayAbility`只有在`Target`拥有所有这些`GameplayTag`时才会激活. `Target GameplayTag`只有在该`GameplayAbility`由Event触发时设置. |
| Target Blocked Tags       | 该`GameplayAbility`在`Target`拥有任意这些标签时不能被激活. `Target GameplayTag`只有在该`GameplayAbility`由Event触发时设置. |



#### 4.12 GASpec

- `GameplayAbilitySpec`会在`GameplayAbility`授予后存在于`ASC`中并定义可激活`GameplayAbility` - `GameplayAbility`类, 等级, 输入绑定和必须与`GameplayAbility`类分开保存的运行时状态.
- 当`GameplayAbility`在服务端授予时, 服务端会同步`GameplayAbilitySpec`到所属(Owning)客户端, 因此可以激活它.
- 激活`GameplayAbilitySpec`会根据它的`实例化策略(Instancing Policy)`创建一个`GameplayAbility`实例(`Non-Instanced GameplayAbility`除外).



#### 4.13 GA回调

在ASC中，绑定GA的相关回调

```c++
/** 每当激活（启动）功能时，通用回调 */
FGenericAbilityDelegate AbilityActivatedCallbacks;

/** 每当技能结束时回调 */
FAbilityEnded AbilityEndedCallbacks;

/** 每当能力结束时回调，并提供额外信息 */
FGameplayAbilityEndedDelegate OnAbilityEnded;

/** 每当提交技能时，都会进行通用回调（应用消耗/冷却） */
FGenericAbilityDelegate AbilityCommittedCallbacks;

/** 当功能执行失败时，调用失败原因 */
FAbilityFailedDelegate AbilityFailedCallbacks;

/** 当能力规范的内部结构发生更改时调用 */
FAbilitySpecDirtied AbilitySpecDirtiedCallbacks;
```





### 5. GameplayEffect

#### 5.1 GE概念

- [GameplayEffect(GE)](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html)是Ability修改其自身和其他[Attribute](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-a)和[GameplayTag](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-gt)的容器
- 其可以立即修改`Attribute`(像伤害或治疗)或应用长期的状态buff/debuff(像移动速度加速或眩晕)
- UGameplayEffect只是一个定义单一游戏效果的数据类, 不应该在其中添加额外的逻辑
- `GameplayEffect`通过[Modifier](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-mods)和[Execution(GameplayEffectExecutionCalculation)](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-ec)修改`Attribute`
- `GameplayEffect`有三种持续类型: `即刻(Instant)`, `持续(Duration)`和`无限(Infinite)`.



:::tip GE额外功能

- `GameplayEffect`可以添加/执行[GameplayCue](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-gc)
- `即刻(Instant)GameplayEffect`可以调用`GameplayCue GameplayTag`的Execute
- 而`持续(Duration)`或`无限(Infinite)`可以调用`GameplayCue GameplayTag`的Add和Remove

:::



GE的修改类型：

| 类型           | GameplayCue事件 | 何时使用                                                     |
| -------------- | --------------- | ------------------------------------------------------------ |
| 即刻(Instant)  | Execute         | 对`Attribute`中BaseValue立即进行的永久性修改. 其不会应用`GameplayTag`, 哪怕是一帧. |
| 持续(Duration) | Add & Remove    | 对`Attribute`中CurrentValue的临时修改和当`GameplayEffect`过期或手动移除时, 应用将要被移除的`GameplayTag`. 持续时间是在UGameplayEffect类/蓝图中明确的. |
| 无限(Infinite) | Add & Remove    | 对`Attribute`中CurrentValue的临时修改和当`GameplayEffect`移除时, 应用将要被移除的`GameplayTag`. 该类型自身永不过期且必须由某个Ability或`ASC`手动移除. |



GE的修改方式：

- `持续(Duration)`和`无限(Infinite)GameplayEffect`可以选择应用周期性的Effect
- 其每过X秒(由周期定义)就应用一次`Modifier`和Execution
- 当周期性的Effect修改`Attribute`的BaseValue和执行`GameplayCue`时就被视为`即刻(Instant)GameplayEffect`
- 这种类型的Effect对于像随时间推移的持续伤害(damage over time, DOT)很有用
- 周期性的Effect不能被[预测](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-p)



持续GE的条件：

- 如果`持续(Duration)`和`无限(Infinite)GameplayEffect`的Ongoing Tag Requirements未满足/满足的话([Gameplay Effect Tags](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-tags))
- 那么它们在应用后就可以被暂时的关闭和打开, 关闭`GameplayEffect`会移除其`Modifier`和已应用`GameplayTag`效果
- 但是不会移除该`GameplayEffect`, 重新打开`GameplayEffect`会重新应用其`Modifier`和`GameplayTag`



重新计算持续GE：

- 重新计算某个`持续(Duration)`或`无限(Infinite)GameplayEffect`的`Modifier`(假设有一个使用非`Attribute`数据的`MMC`)
- 可以使用和`UAbilitySystemComponent::ActiveGameplayEffect.GetActiveGameplayEffect(ActiveHandle).Spec.GetLevel()`相同的Level
- 调用`UAbilitySystemComponent::ActiveGameplayEffect.SetActiveGameplayEffectLevel(FActiveGameplayEffectHandle ActiveHandle, int32 NewLevel)`
- 当`Backing Attribute`更新时, 基于`Backing Attribute`的`Modifier`会自动更新



:::warning

SetActiveGameplayEffectLevel()更新`Modifier`的关键函数是:

```c++
MarkItemDirty(Effect);
Effect.Spec.CalculateModifierMagnitudes();
// Private function otherwise we'd call these three functions without needing to set the level to what it already is
UpdateAllAggregatorModMagnitudes(Effect);
```



- `GameplayEffect`一般是不实例化的
- 当Ability或`ASC`想要应用`GameplayEffect`时, 其会从`GameplayEffect`的`ClassDefaultObject`创建一个`GameplayEffectSpec`
- 之后成功应用的`GameplayEffectSpec`会被添加到一个名为`FActiveGameplayEffect`的新结构体
- 其是`ASC`在名为`ActiveGameplayEffect`的特殊结构体容器中追踪的内容

:::



#### 5.2 应用GE

##### 5.2.1 GE的使用流程

1. `OwnerActor`的`ASC`发起`GA` -> `GA`获取应用`Target` -> `Target`应用`GE`
2. `OwnerActor`的`ASC`发起`GA` -> `GA`生成某个`Actor`，生成`GESpec` -> `Actor`携带`GESpec`，交互`Target` -> `Target`应用`GE`



##### 5.2.2 GE的执行

GE的执行通常都是ASC调用：

- `ApplyGameplayEffectSpecToTarget()`：将先前创建的游戏效果规范应用于目标
- `ApplyGameplayEffectSpecToSelf()`：将之前创建的游戏效果规范应用于此组件



##### 5.2.3 GE绑定回调



1. 添加GE触发回调：
   - 一般用于绑定：`持续(Duration)`或`无限(Infinite)GameplayEffect`的委托来监听其应用到`ASC`
   - 当`持续(Duration)`或`无限(Infinite)GameplayEffect`（可激活）被添加到`ASC`触发回调

```c++
/** 每当添加基于 duraton 的 GE 时，都会在客户端和服务器上调用（例如，即时 GE 不会触发此操作） */
FOnGameplayEffectAppliedDelegate OnActiveGameplayEffectAddedDelegateToSelf;
```

:::tip 使用方式

```c++
AbilitySystemComponent->
    OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(
    	this, &APACharacterBase::OnActiveGameplayEffectAddedCallback);

virtual void OnActiveGameplayEffectAddedCallback(
    UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);
```

:::



2. GE被触发后回调：
   - 包含所有类型的GE被应用

```c++
/** 每当将 GE 应用于自身时，都会在服务器上调用。这包括即时和基于持续时间的 GE。 */
FOnGameplayEffectAppliedDelegate OnGameplayEffectAppliedDelegateToSelf;

/** 每当 GE 应用于其他人时，都会在服务器上调用。这包括即时和基于持续时间的 GE。 */
FOnGameplayEffectAppliedDelegate OnGameplayEffectAppliedDelegateToTarget;

/** 每当定期 GE 自行执行时，都会在服务器上调用 */
FOnGameplayEffectAppliedDelegate OnPeriodicGameplayEffectExecuteDelegateOnSelf;

/** 每当定期 GE 在目标上执行时，都会在服务器上调用 */
FOnGameplayEffectAppliedDelegate OnPeriodicGameplayEffectExecuteDelegateOnTarget;
```



3. 持续性GE的持续时间发生变化
   - 待重载的虚函数方法

```c++
/** 当游戏效果的持续时间发生变化时调用 */
virtual void OnGameplayEffectDurationChange(struct FActiveGameplayEffect& ActiveEffect);
```



:::warning

- 服务端总是会调用该函数而不管同步模式是什么
- `Autonomous Proxy`只会在`Full`和`Mixed`同步模式下对于同步的`GameplayEffect`调用该函数
- `Simulated Proxy`只会在`Full`同步模式下调用该函数

:::



#### 5.3 移除GE

- `GameplayEffect`可以被[GameplayAbility](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ga)和`ASC`中的多个函数移除
- 其通常是`RemoveActiveGameplayEffect`的形式
- 不同的函数本质上都是最终在目标上调用`FActiveGameplayEffectContainer::RemoveActiveEffects()`的方便函数



:::warning `GameplayAbility`之外移除`GameplayEffect`

需要获取到该目标的`ASC`并使用它的函数之一来`RemoveActiveGameplayEffect`

:::



```c++
/** 在删除任何游戏效果时调用 */
FOnGivenActiveGameplayEffectRemoved& OnAnyGameplayEffectRemovedDelegate();

AbilitySystemComponent->
    OnAnyGameplayEffectRemovedDelegate().AddUObject(this, &APACharacterBase::OnRemoveGameplayEffectCallback);

virtual void OnRemoveGameplayEffectCallback(const FActiveGameplayEffect& EffectRemoved);
```



#### 5.4 Modifier

- `Modifier`可以修改`Attribute`并且是唯一可以[预测性](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-p)修改`Attribute`的方法
- 一个`GameplayEffect`可以有0个或多个`Modifier`, 每个`Modifier`通过某个指定的操作只能修改一个`Attribute`



| 操作     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| Add      | 将`Modifier`指定的`Attribute`加上计算结果. 使用负数以实现减法操作. |
| Multiply | 将`Modifier`指定的`Attribute`乘以计算结果.                   |
| Divide   | 将`Modifier`指定的`Attribute`除以计算结果.                   |
| Override | 使用计算结果覆盖`Modifier`指定的`Attribute`.                 |



- `Attribute`的CurrentValue是其所有`Modifier`与其BaseValue计算并总合后的结果

- 像下面这样的`Modifier`总合公式被定义在`GameplayEffectAggregator.cpp`中的`FAggregatorModChannel::EvaluateWithBase`:

  ```c++
  ((InlineBaseValue + Additive) * Multiplicitive) / Division
  ```

- Override`Modifier`会优先覆盖最后应用的`Modifier`得出的最终值

  ```c++
  float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& 
                                                Parameters) const
  {
  	for (const FAggregatorMod& Mod : Mods[EGameplayModOp::Override])
  	{
  		if (Mod.Qualifies())
  		{
  			return Mod.EvaluatedMagnitude;
  		}
  	}
  
  	float Additive = 
          SumMods(
          	Mods[EGameplayModOp::Additive], 	
          	GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Additive), Parameters);
      
  	float Multiplicitive = 
          SumMods(
          	Mods[EGameplayModOp::Multiplicitive], 
          	GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Multiplicitive), Parameters);
  	
      float Division = 
          SumMods(
          	Mods[EGameplayModOp::Division], 
          	GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Division), Parameters);
  
  	if (FMath::IsNearlyZero(Division))
  	{
  		ABILITY_LOG(Warning, TEXT("Division summation was 0.0f in FAggregatorModChannel."));
  		Division = 1.f;
  	}
  
  	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
  }
  ```



:::warning

- 对于基于百分比的修改, 确保使用`Multiply`操作以使其在加法操作之后
- [预测(Prediction)](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-p)对于百分比修改有些问题

:::



##### 5.4.1 Modifier种类

- 有四种类型的`Modifier`: 
  - `ScalableFloat`
  - `AttributeBased`
  - `CustomCalculationClass`
  - `SetByCaller`
- 它们全都生成一些浮点数, 用于之后基于各自的操作修改指定`Modifier`的`Attribute`



| Modifier类型             | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| Scalable Float           | FScalableFloat结构体可以指向某个横向为变量, 纵向为等级的Data Table, `Scalable Float`会以Ability的当前等级自动读取指定Data Table的某行值(或者在[GameplayEffectSpec](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-spec)中重写的不同等级), 该值还可以进一步被系数处理, 如果没有指定Data Table/Row, 那么就会将其视为1, 因此该系数就可以在所有等级都硬编码为一个值.[![ScalableFloat](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/scalablefloats.png)](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/scalablefloats.png) |
| Attribute Based          | `Attribute Based Modifier`将Source(`GameplayEffectSpec`的创建者)或Target(`GameplayEffectSpec`的接收者)上的CurrentValue或BaseValue视为`Backing Attribute`, 可以使用系数和Pre与Post系数和来修改它. `Snapshotting`意味着当`GameplayEffectSpec`创建时捕获该`Attribute`, 而`No Snapshotting`意味着当`GameplayEffectSpec`应用时捕获该`Attribute`. |
| Custom Calculation Class | `Custom Calculation Class`为复杂的`Modifier`提供了最大的灵活性, 该`Modifier`使用了[ModifierMagnitudeCalculation](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-mmc)类, 且可以使用系数和Pre与Post系数和来处理浮点值结果. |
| Set By Caller            | `SetByCaller`Modifier是运行时由Ability或`GameplayEffectSpec`的创建者于`GameplayEffect`之外设置的值, 例如, 如果你想让伤害值随玩家蓄力技能的长短而变化, 那么就需要使用`SetByCaller`. `SetByCaller`本质上是存于`GameplayEffectSpec`中的`TMap<FGameplayTag, float>`, `Modifier`只是告知`Aggregator`去寻找与提供的`GameplayTag`相关联的`SetByCaller`值. `Modifier`使用的`SetByCaller`只能使用该概念的`GameplayTag`形式, `FName`形式在此处不适用. 如果`Modifier`被设置为`SetByCaller`, 但是带有正确`GameplayTag`的`SetByCaller`在`GameplayEffectSpec`中不存在, 那么游戏会抛出一个运行时错误并返回0, 这可能在`Divide`操作中造成问题. 参阅[SetByCallers](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-spec-setbycaller)获取更多关于如何使用`SetByCaller`的信息. |



##### 5.4.2 Multiply和Divide Modifier

- 默认情况下, 所有的`Multiply`和`Divide`Modifier在对`Attribute`的BaseValue乘除前都会先加到一起



引擎默认计算细节：

```c++
float FAggregatorModChannel::SumMods(const TArray<FAggregatorMod>& InMods, float Bias, const 
                                     FAggregatorEvaluateParameters& Parameters)
{
	float Sum = Bias;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Sum += (Mod.EvaluatedMagnitude - Bias);
		}
	}

	return Sum;
}

float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) 
    const
{
	...

	float Additive = 
        SumMods(
        	Mods[EGameplayModOp::Additive], 	
        	GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Additive), Parameters);
    
	float Multiplicitive = 
        SumMods(
        	Mods[EGameplayModOp::Multiplicitive], 
        	GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Multiplicitive), Parameters);
	
    float Division = 
        SumMods(
        	Mods[EGameplayModOp::Division], 
        	GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Division), Parameters);

	...

	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
}
```



修改计算细节：

- 很多游戏会想要它们的`Modify`和`Divide`Modifier在应用到BaseValue之前先乘或除到一起
- 为了实现这种需求, 你需要修改`FAggregatorModChannel::EvaluateWithBase()`的引擎代码



```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) 
    const
{
	...
	float Multiplicitive = MultiplyMods(Mods[EGameplayModOp::Multiplicitive], Parameters);
	...

	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
}

// 需要添加到引擎源码中的 GameplayEffectAggregator.cpp 中
float FAggregatorModChannel::MultiplyMods(const TArray<FAggregatorMod>& InMods, const FAggregatorEvaluateParameters& 
                                          Parameters)
{
	float Multiplier = 1.0f;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Multiplier *= Mod.EvaluatedMagnitude;
		}
	}

	return Multiplier;
}
```



##### 5.4.3 Modifier_GameplayTag

- 每个[Modifier](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-mods)都可以设置`SourceTag`和`TargetTag`
- 它们的作用就像`GameplayEffect`的[Application Tag requirements](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-tags)
- 因此只有当Effect应用后才会考虑标签, 对于周期性(Periodic)的`无限(Infinite)`Effect, 这些标签只会在第一次应用Effect时才会被考虑, 而不是在每次周期执行时.



- `Attribute Based Modifier`也可以设置`SourceTagFilter`和`TargetTagFilter`
- 当确定`Attribute Based Modifier`的源(Source)`Attribute`的Magnitude时
- 这些过滤器就会用来将某些`Modifier`排除在该Attribute之外
- 源(Source)或目标(Target)中没有过滤器所有标签的`Modifier`也会被排除在外



这更详尽的意思是: 

- 源(Source)`ASC`和目标(Target)`ASC`的标签都被`GameplayEffect`所捕获
- 当`GameplayEffectSpec`创建时, 源(Source)`ASC`的标签被捕获
- 当执行Effect时, 目标(Target)`ASC`的标签被捕获
- 当确定`无限(Infinite)`或`持续(Duration)`Effect的`Modifier`是否满足条件可以被应用(也就是聚合器条件(Aggregator Qualify))并且过滤器已经设置时
- 被捕获的标签就会和过滤器进行比对



#### 5.5 GE堆栈

- `GameplayEffect`默认会应用新的`GameplayEffectSpec`实例
- 而不明确或不关心之前已经应用过的尚且存在的`GameplayEffectSpec`实例
- `GameplayEffect`可以设置到堆栈中, 新的`GameplayEffectSpec`实例不会添加到堆栈中
- 而是修改当前已经存在的`GameplayEffectSpec`堆栈数
- 堆栈只适用于`持续(Duration)`和`无限(Infinite)GameplayEffect`



有两种类型的堆栈: `Aggregate by Source`和`Aggregate by Target`：

| 堆栈类型            | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| Aggregate by Source | 目标(Target)上的每个源(Source)`ASC`都有一个单独的堆栈实例, 每个源(Source)可以应用堆栈中的X个`GameplayEffect`. |
| Aggregate by Target | 目标(Target)上只有一个堆栈实例而不管源(Source)如何, 每个源(Source)都可以在共享堆栈限制(Shared Stack Limit)内应用堆栈. |



样例项目：

- 包含一个用于监听`GameplayEffect`堆栈变化的自定义蓝图节点
- HUD UMG Widget使用它来更新玩家拥有的被动护盾堆栈(层数)
- 该`AsyncTask`将会一直响应直到手动调用`EndTask()`
- 就像在UMG Widget的`Destruct`事件中调用那样. 参阅`AsyncTaskAttributeChanged.h/cpp`

[![Listen for GameplayEffect Stack Change BP Node](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/gestackchange.png)](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/gestackchange.png)



#### 5.6 GE授予ASC新的GA

- 只有`持续(Duration)`和`无限(Infinite)GameplayEffect`可以授予Ability
- 一个普遍用法是当想要强制另一个玩家做某些事的时候, 像击退或拉取时移动他们
- 就会对它们应用一个`GameplayEffect`来授予其一个自动激活的Ability(查看[被动Ability](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ga-activating-passive)来了解如何在Ability被授予时自动激活它)
- 从而使其做出相应的动作



设计师可以决定一个`GameplayEffect`能够授予哪些Ability, 授予的Ability等级, 将其[绑定](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ga-input)在什么输入键上以及该Ability的移除策略

| 移除策略          | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| 立即取消Ability   | 当授予Ability的`GameplayEffect`从目标移除时, 授予的Ability就会立即取消并移除. |
| 结束时移除Ability | 允许授予的Ability完成, 之后将其从目标移除.                   |
| 无                | 授予的Ability不受从目标移除的授予`GameplayEffect`的影响, 目标将会一直拥有该Ability直到之后被手动移除. |



#### 5.7 GE标签

- `GameplayEffect`可以带有多个[GameplayTagContainer](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-gt)
- 设计师可以编辑每个类别的`Added`和`Removed`GameplayTagContainer
- 结果会在编译后显示在`Combined GameplayTagContainer`中
- `Added`标签是该`GameplayEffect`新增的之前其父类没有的标签
- `Removed`标签是其父类拥有但该类没有的标签



| Gameplay Effect Asset Tags        | `GameplayEffect`拥有的标签, 它们自身没有任何功能且只用于描述`GameplayEffect`. |
| --------------------------------- | ------------------------------------------------------------ |
| Granted Tags                      | 存于`GameplayEffect`中且又用于`GameplayEffect`所应用`ASC`的标签. 当`GameplayEffect`移除时它们也会从`ASC`中移除. 该标签只作用于`持续(Duration)`和`无限(Infinite)GameplayEffect`. |
| Ongoing Tag Requirements          | 一旦`GameplayEffect`应用后, 这些标签将决定`GameplayEffect`是开启还是关闭. `GameplayEffect`可以是关闭但仍然是应用的. 如果某个`GameplayEffect`由于不符合`Ongoing Tag Requirements`而关闭, 但是之后又满足需求了, 那么该`GameplayEffect`会重新打开并重新应用它的`Modifier`. 该标签只作用于`持续(Duration)`和`无限(Infinite)GameplayEffect`. |
| Application Tag Requirements      | 位于目标上决定某个`GameplayEffect`是否可以应用到该目标的标签, 如果不满足这些需求, 那么`GameplayEffect`就不可应用. |
| Remove Gameplay Effects with Tags | 当`GameplayEffect`成功应用后, 如果位于目标上的该`GameplayEffect`在其`Asset Tags`或`Granted Tags`中有任意一个本标签的话, 其就会自目标上移除. |



#### 5.8 GE免疫

- `GameplayEffect`可以基于[GameplayTag](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-gt)实现免疫, 有效阻止其他`GameplayEffect`应用
- 尽管免疫可以由`Application Tag Requirements`等方式有效地实现
- 但是使用该系统可以在`GameplayEffect`被免疫阻止时提供`UAbilitySystemComponent::OnImmunityBlockGameplayEffectDelegate`委托(Delegate)
- `GrantedApplicationImmunityTags`会检查源(Source)`ASC`(包括源Ability的AbilityTag如果有的话)是否包含特定的标签
- 这是一种基于确定Character或源(Source)的标签对其所有`GameplayEffect`提供免疫的方法
- `Granted Application Immunity Query`会检查传入的`GameplayEffectSpec`是否与其查询条件相匹配, 从而阻止或允许其应用



`AbilitySystemComponent.h`

```c++
/** 当 GameplayEffectSpec 因免疫而被 ActiveGameplayEffect 阻止时发出通知  */
DECLARE_MULTICAST_DELEGATE_TwoParams(FImmunityBlockGE, const FGameplayEffectSpec& /*BlockedSpec*/, const FActiveGameplayEffect* /*ImmunityGameplayEffect*/);

/** Immunity notification support */
FImmunityBlockGE OnImmunityBlockGameplayEffectDelegate;
```



#### 5.9 GESpec

##### 5.9.1 GESpec概念

1. 作用：
   - [GameplayEffectSpec(GESpec)](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectSpec/index.html)可以看作是`GameplayEffect`的实例
   - 它保存了一个其所代表的`GameplayEffect`类引用, 创建时的等级和创建者, 它在应用之前可以在运行时(Runtime)自由的创建和修改
   - 不像`GameplayEffect`应该由设计师在运行前创建
   - 当应用`GameplayEffect`时, `GameplayEffectSpec`会自`GameplayEffect`创建并且会实际应用到目标(Target)
2. 创建和使用：
   - `GameplayEffectSpec`是由`UAbilitySystemComponent::MakeOutgoingSpec()(BlueprintCallable)`自`GameplayEffect`创建的
   - `GameplayEffectSpec`不必立即应用
   - 通常是将`GameplayEffectSpec`传递给创建自Ability的投掷物, 该投掷物可以应用到它之后击中的目标
   - 当`GameplayEffectSpec`成功应用后, 就会返回一个名为`FActiveGameplayEffect`的新结构体
3. `GameplayEffectSpec`的重要内容:
   - 创建该`GameplayEffectSpec`的`GameplayEffect`类.
   - 该`GameplayEffectSpec`的等级. 通常和创建`GameplayEffectSpec`的Ability的等级一样, 但是可以是不同的.
   - `GameplayEffectSpec`的持续时间. 默认是`GameplayEffect`的持续时间, 但是可以是不同的.
   - 对于周期性Effect中`GameplayEffectSpec`的周期, 默认是`GameplayEffect`的周期, 但是可以是不同的.
   - 该`GameplayEffectSpec`的当前堆栈数. 堆栈限制取决于`GameplayEffect`.
   - [GameplayEffectContextHandle](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-context)表明该`GameplayEffectSpec`由谁创建.
   - `Attribute`在`GameplayEffectSpec`创建时由`Snapshot`捕获.
   - 除了`GameplayEffect`授予的`GameplayTags`, `GameplayEffectSpec`还会授予目标(Target)`DynamicGrantedTags`.
   - 除了`GameplayEffect`拥有的`AssetTags`, `GameplayEffectSpec`还会拥有`DynamicAssetTags`.
   - `SetByCaller TMaps`.



##### 5.9.2 SetByCaller

- `SetByCaller`允许`GameplayEffectSpec`拥有和`GameplayTag`或`FName`相关联的浮点值
- 它们存储在`GameplayEffectSpec`上其各自的`TMaps: TMap<FGameplayTag, float>`和`TMap<FName, float>`中
- 可以作为`GameplayEffect`的`Modifier`或者传递浮点值的通用方法使用
- 其普遍用法是经由`SetByCaller`传递某个Ability内部生成的数值数据到[GameplayEffectExecutionCalculations](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-ec)或[ModifierMagnitudeCalculations](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-mmc).



| SetByCaller使用 | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `Modifier`      | 必须提前在`GameplayEffect`类中定义. 只能使用`GameplayTag`形式. 如果在`GameplayEffect`类中定义而`GameplayEffectSpec`中没有相应的标签/浮点值对, 那么游戏在`GameplayEffectSpec`应用时会抛出运行时错误并返回0, 这对于`Divide`操作是个潜在问题, 参阅[Modifier](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-mods). |
| 其他            | 无需提前定义. 读取`GameplayEffectSpec`中不存在的`SetByCaller`会返回一个由开发者定义的可带有警告信息的默认值. |



蓝图中指定SetByCaller：

![Assigning SetByCaller](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/setbycaller.png)



C++中指定SetByCaller：`GameplayEffect.h` -> `FGameplayEffectSpec`

```c++
/** 设置 SetByCaller 修饰符的幅度 */
void SetSetByCallerMagnitude(FName DataName, float Magnitude);
void SetSetByCallerMagnitude(FGameplayTag DataTag, float Magnitude);

/** 返回 SetByCaller 修饰符的大小。如果尚未设置量级，将返回 0.f 并发出警告。 */
float GetSetByCallerMagnitude(FName DataName, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
float GetSetByCallerMagnitude(FGameplayTag DataTag, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```



:::tip 使用建议

- 建议使用`GameplayTag`形式而不是`FName`形式
- 这可以避免蓝图中的拼写错误
- 并且当`GameplayEffectSpec`同步时, `GameplayTag`比`FName`在网络传输中更有效率, 因为`TMap`也会同步

:::



#### 5.10 GEContext

- [GameplayEffectContext](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectContext/index.html)结构体存有关于`GameplayEffectSpec`创建者(Instigator)和[TargetData](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-targeting-data)的信息
- 这也是一个很好的可继承结构体以在[ModifierMagnitudeCalculation](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-mmc)/[GameplayEffectExecutionCalculation](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-ec), [AttributeSet](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-as)和[GameplayCue](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-gc)之间传递任意数据



继承`GameplayEffectContext`:

1. 继承`FGameplayEffectContext`.
2. 重写`FGameplayEffectContext::GetScriptStruct()`.
3. 重写`FGameplayEffectContext::Duplicate()`.
4. 如果新数据需要同步的话, 重写`FGameplayEffectContext::NetSerialize()`.
5. 对子结构体实现`TStructOpsTypeTraits`, 就像父结构体`FGameplayEffectContext`有的那样.
6. 在[AbilitySystemGlobals](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-asg)类中重写`AllocGameplayEffectContext()`以返回一个新的子结构体对象.



#### 5.11 MMC

`GameplayModMagnitudeCalculation`

- [ModifierMagnitudeCalculations](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayModMagnitudeCalculation/index.html)(`ModMagniCalc`或`MMC`)是在`GameplayEffect`中作为[Modifier](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-mods)使用的强有力的类
- 它的功能类似[GameplayEffectExecutionCalculation](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-ec)但是要逊色一些
- `MMC`可以用于各种持续时间的`GameplayEffect` - `即刻(Instant)`, `持续(Duration)`, `无限(Infinite)`和`周期性(Periodic)`.



:::warning

1. 最重要的是它是可预测的
2. 唯一要做的就是自`CalculateBaseMagnitude_Implementation()`返回浮点值
3. 可以在C++和蓝图中继承并重写该函数

:::



- `MMC`的优势在于能够完全访问`GameplayEffectSpec`来读取`GameplayTag`和`SetByCaller`
- 从而能够捕获`GameplayEffect`的`源(Source)`或`目标(Target)`上任意数量的`Attribute`值
- `Attribute`可以被Snapshot也可以不被Snapshot
- `Snapshotted Attribute`在`GameplayEffectSpec`创建时被捕获而非`Snapshotted Attribute`在`GameplayEffectSpec`应用时被捕获
- 并且该`Attribute`被`无限(Infinite)`或`持续(Duration)`GameplayEffect修改时会自动更新
- 捕获`Attribute`会自`ASC`现有的`Modifier`重新计算它们的`CurrentValue`



:::warning 最终的结果

1. 该重新计算**不会**执行`AbilitySet`中的[PreAttributeChange()](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-as-preattributechange), 因此所有的限制操作(Clamp)必须在这里重新处理
2. `MMC`的结果浮点值可以进一步由系数和前后系数之和在`GameplayEffect`的`Modifier`中修改

:::



| Snapshot | 源(Source)或目标(Target) | 在GameplayEffectSpec中捕获 | Attribute被无限(Infinite)或持续(Duration)GameplayEffect修改时自动更新 |
| -------- | ------------------------ | -------------------------- | ------------------------------------------------------------ |
| 是       | Source                   | 创建                       | 否                                                           |
| 是       | Target                   | 应用                       | 否                                                           |
| 否       | Source                   | 应用                       | 是                                                           |
| 否       | Target                   | 应用                       | 是                                                           |



示例：

- 该`MMC`会捕获目标的魔法值`Attribute`并因为毒药Effect而将其减少, 其减少量的变化取决于目标所拥有的魔法值和可能拥有的某个标签

```c++
UPAMMC_PoisonMana::UPAMMC_PoisonMana()
{

	//ManaDef defined in header FGameplayEffectAttributeCaptureDefinition ManaDef;
	ManaDef.AttributeToCapture = UPAAttributeSetBase::GetManaAttribute();
	ManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	ManaDef.bSnapshot = false;

	//MaxManaDef defined in header FGameplayEffectAttributeCaptureDefinition MaxManaDef;
	MaxManaDef.AttributeToCapture = UPAAttributeSetBase::GetMaxManaAttribute();
	MaxManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	MaxManaDef.bSnapshot = false;

	RelevantAttributesToCapture.Add(ManaDef);
	RelevantAttributesToCapture.Add(MaxManaDef);
}

float UPAMMC_PoisonMana::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	// Gather the tags from the source and target as that can affect which buffs should be used
	const FGameplayTagContainer* SourceTags = Spec.CapturedSourceTags.GetAggregatedTags();
	const FGameplayTagContainer* TargetTags = Spec.CapturedTargetTags.GetAggregatedTags();

	FAggregatorEvaluateParameters EvaluationParameters;
	EvaluationParameters.SourceTags = SourceTags;
	EvaluationParameters.TargetTags = TargetTags;

	float Mana = 0.f;
	GetCapturedAttributeMagnitude(ManaDef, Spec, EvaluationParameters, Mana);
	Mana = FMath::Max<float>(Mana, 0.0f);

	float MaxMana = 0.f;
	GetCapturedAttributeMagnitude(MaxManaDef, Spec, EvaluationParameters, MaxMana);
	MaxMana = FMath::Max<float>(MaxMana, 1.0f); // Avoid divide by zero

	float Reduction = -20.0f;
	if (Mana / MaxMana > 0.5f)
	{
		// Double the effect if the target has more than half their mana
		Reduction *= 2;
	}
	
	if (TargetTags->HasTagExact(FGameplayTag::RequestGameplayTag(FName("Status.WeakToPoisonMana"))))
	{
		// Double the effect if the target is weak to PoisonMana
		Reduction *= 2;
	}
	
	return Reduction;
}
```



:::warning

- 如果你没有在`MMC`的构造函数中将`FGameplayEffectAttributeCaptureDefinition`添加到`RelevantAttributesToCapture`中
- 并且尝试捕获`Attribute`, 那么将会得到一个关于捕获时缺失Spec的错误
- 如果不需要捕获`Attribute`, 那么就不必添加什么到`RelevantAttributesToCapture`

:::



#### 5.12 GE_ExeCalc

##### 5.12.1 概念定义

- 像[ModifierMagnitudeCalculation](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-mmc)一样, 它也可以捕获`Attribute`并选择性地为其创建Snapshot
- 和`MMC`不同的是, 它可以修改多个`Attribute`并且基本上可以处理程序员想要做的任何事
- `ExecutionCalculation`只能由`即刻(Instant)`和`周期性(Periodic)`GameplayEffect使用, 插件中所有和"Execute"相关的一般都引用到这两种类型的`GameplayEffect`



:::warning

它是不可[预测](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-p)的且必须在C++中实现

:::



- 当`GameplayEffectSpec`创建时, Snapshot会捕获`Attribute`
- 而当`GameplayEffectSpec`应用时, 非Snapshot会捕获`Attribute`
- 捕获`Attribute`会自`ASC`现有的`Modifier`重新计算它们的`CurrentValue`



:::warning

- 该重新计算**不会**执行`AbilitySet`中的[PreAttributeChange()](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-as-preattributechange)
- 因此所有的限制操作(Clamp)必须在这里重新处理

:::



| 快照 | Source或Target | 在GameplayEffectSpec中捕获 |
| ---- | -------------- | -------------------------- |
| 是   | Source         | 创建                       |
| 是   | Target         | 应用                       |
| 否   | Source         | 应用                       |
| 否   | Target         | 应用                       |



##### 5.12.2 创建使用

- 为了设置`Attribute`捕获, 我们采用Epic的ActionRPG样例项目使用的方式
- 定义一个保存和声明如何捕获`Attribute`的结构体, 并在该结构体的构造函数中创建一个它的副本(Copy)
- 每个`ExecCalc`都需要有一个这样的结构体



:::warning

- 每个结构体需要一个独一无二的名字, 因为它们共享同一个命名空间
- 多个结构体使用相同名字在捕获`Attribute`时会造成错误(大多是捕获到错误的`Attribute`值)

:::



代码示例：

`GGE_ExeCalc.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "GameplayEffectExecutionCalculation.h"
#include "GGE_ExeCalc.generated.h"

UCLASS()
class GAS_02_API UGGE_ExeCalc : public UGameplayEffectExecutionCalculation
{
	GENERATED_BODY()

public:
	UGGE_ExeCalc();

	virtual void Execute_Implementation(const FGameplayEffectCustomExecutionParameters& ExecutionParams, 
                                        FGameplayEffectCustomExecutionOutput& OutExecutionOutput) const override;
};
```



`GGE_ExeCalc.cpp`

```c++
#include "GGE_ExeCalc.h"
#include "GAS_02/GAS/AS/GGAttributeSet.h"
#include "GAS_02/GAS/GEContext/GG_GEContext.h"

struct FDamageStatics
{
	DECLARE_ATTRIBUTE_CAPTUREDEF(InDamage);
	DECLARE_ATTRIBUTE_CAPTUREDEF(CritChance);
	DECLARE_ATTRIBUTE_CAPTUREDEF(CritMulti);
	DECLARE_ATTRIBUTE_CAPTUREDEF(LuckyChance);

	FDamageStatics(): CritChanceProperty(nullptr), CritMultiProperty(nullptr), LuckyChanceProperty(nullptr)
	{
		DEFINE_ATTRIBUTE_CAPTUREDEF(UGGAttributeSet, InDamage, Source, false);
		DEFINE_ATTRIBUTE_CAPTUREDEF(UGGAttributeSet, CritChance, Source, false);
		DEFINE_ATTRIBUTE_CAPTUREDEF(UGGAttributeSet, CritMulti, Source, false);
		DEFINE_ATTRIBUTE_CAPTUREDEF(UGGAttributeSet, LuckyChance, Source, false);
	}
};

static FDamageStatics DamageStatics()
{
	static FDamageStatics DmgStatics;
	return DmgStatics;
}

UGGE_ExeCalc::UGGE_ExeCalc()
{
	RelevantAttributesToCapture.Add(DamageStatics().InDamageDef);
	RelevantAttributesToCapture.Add(DamageStatics().CritChanceDef);
	RelevantAttributesToCapture.Add(DamageStatics().CritMultiDef);
	RelevantAttributesToCapture.Add(DamageStatics().LuckyChanceDef);
}

void UGGE_ExeCalc::Execute_Implementation(const FGameplayEffectCustomExecutionParameters& ExecutionParams,
	FGameplayEffectCustomExecutionOutput& OutExecutionOutput) const
{
	auto SourceASC = ExecutionParams.GetSourceAbilitySystemComponent();
	auto TargetASC = ExecutionParams.GetTargetAbilitySystemComponent();
	auto TargetActor = TargetASC ? TargetASC->GetAvatarActor() : nullptr;

	const auto Spec = ExecutionParams.GetOwningSpec();
	const auto SourceTags = Spec.CapturedSourceTags.GetAggregatedTags();
	const auto TargetTags = Spec.CapturedTargetTags.GetAggregatedTags();

	FAggregatorEvaluateParameters EvaluateParameters;
	EvaluateParameters.SourceTags = SourceTags;
	EvaluateParameters.TargetTags = TargetTags;

	auto InDamage = 0.f;
		/*FMath::Max(
			Spec.GetSetByCallerMagnitude(
				FGameplayTag::RequestGameplayTag(
					FName("Damage.SetByCaller")), false, -1.f), 0.f);*/
	ExecutionParams.AttemptCalculateCapturedAttributeMagnitude
        (DamageStatics().InDamageDef, EvaluateParameters, InDamage);

	auto IsCrit = false;
	auto IsLucky = false;
	auto CritChance = 0.f;
	auto CritMulti = 0.f;
	auto LuckyChance = 0.f;

	ExecutionParams.AttemptCalculateCapturedAttributeMagnitude
        (DamageStatics().CritChanceDef, EvaluateParameters, CritChance);
    
	const auto Hit = Spec.GetContext().GetHitResult();
	if (Hit && Hit->BoneName == "head")
	{
		CritChance += 100.f;
	}

	ExecutionParams.AttemptCalculateCapturedAttributeMagnitude
        (DamageStatics().CritMultiDef, EvaluateParameters, CritMulti);
    
	if (CritMulti > 1.f)
	{
		IsCrit = FMath::RandRange(0.01f, 100.f) <= CritChance;
		if (IsCrit)
		{
			InDamage *= CritMulti;
		}
	}

	ExecutionParams.AttemptCalculateCapturedAttributeMagnitude
        (DamageStatics().LuckyChanceDef, EvaluateParameters, LuckyChance);
    
	if (LuckyChance > 0.f)
	{
		auto LuckyMulti = 1.f;
		
		while (FMath::RandRange(0.01f, 100.f) <= LuckyChance)
		{
			LuckyChance -= 100.f;
			LuckyMulti += 1.f;
			IsLucky = true;
		}

		if (IsLucky)
		{
			InDamage *= LuckyMulti;
		}
	}

	OutExecutionOutput.AddOutputModifier
        (FGameplayModifierEvaluatedData(DamageStatics().InDamageProperty, EGameplayModOp::Additive, InDamage));

	const auto MutableSpec = ExecutionParams.GetOwningSpecForPreExecuteMod();
	if (const auto Context = static_cast<FGG_GEContext*>(MutableSpec->GetContext().Get()))
	{
		Context->SetIsCriticalHit(IsCrit);
		Context->SetIsLuckyHit(IsLucky);
	}
	
	// TODO Debug Last Damage
	GEngine->AddOnScreenDebugMessage(-1, 3.f, FColor::Blue, FString::Printf(TEXT("Damage: %f"), InDamage));
}
```



- 对于`Local Predicted`, `Server Only`和`Server Initiated`的[GameplayAbility](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ga), `ExecCalc`只在服务端调用
- `ExecCalc`最普遍的应用场景是计算一个来自很多`源(Source)`和`目标(Target)`中`Attribute`伤害值的复杂公式
- 样例项目中有一个简单的`ExecCalc`用于计算伤害值, 其从`GameplayEffectSpec`的[SetByCaller](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-spec-setbycaller)中读取伤害值, 之后基于从`目标(Target)`捕获的护盾`Attribute`来减少该伤害值



##### 5.12.3 发送数据到GE_ExeCalc

- 除了捕获`Attribute`, 还有几种方法可以发送数据到`ExecutionCalculation`



1. SetByCaller：
   - 任何[设置在`GameplayEffectSpec`中的`SetByCaller`](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-spec-setbycaller)都可以直接在`ExecutionCalculation`中读取

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
float Damage = 
    FMath::Max<float>(
    	Spec.GetSetByCallerMagnitude(
        	FGameplayTag::RequestGameplayTag(FName("Data.Damage")), false, -1.0f), 0.0f);
```



2. Backing数据Attribute计算Modifier
   - 硬编码值到`GameplayEffect`, 可以使用`CalculationModifier`传递
   - 其使用捕获的`Attribute`之一作为Backing数据

示例：

- 给捕获的伤害值`Attribute`增加了50, 也可以将其设为`Override`来直接传入硬编码值

[![Backing Data Attribute Calculation Modifier](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/calculationmodifierbackingdataattribute.png)](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/calculationmodifierbackingdataattribute.png)

当`ExecutionCalculation`捕获该`Attribute`时会读取这个值

```c++
float Damage = 0.0f;
// Capture optional damage value set on the damage GE as a CalculationModifier under the ExecutionCalculation
ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(DamageStatics().DamageDef, EvaluationParameters, Damage);
```



3. Backing数据临时变量计算Modifier
   - 硬编码值到`GameplayEffect`, 可以在C++中使用`CalculationModifier`传递
   - 其使用一个`临时变量`或`暂时聚合器(Transient Aggregator)`, 该`临时变量`与`GameplayTag`相关联

示例：

- 使用`Data.Damage GameplayTag`增加50到一个临时变量

[![Backing Data Temporary Variable Calculation Modifier](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/calculationmodifierbackingdatatempvariable.png)](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/calculationmodifierbackingdatatempvariable.png)

添加Backing临时变量到你的`ExecutionCalculation`构造函数：

```c++
ValidTransientAggregatorIdentifiers.AddTag(FGameplayTag::RequestGameplayTag("Data.Damage"));
```

`ExecutionCalculation`会使用和`Attribute`捕获函数相似的特殊捕获函数来读取这个值

```c++
float Damage = 0.0f;
ExecutionParams.AttemptCalculateTransientAggregatorMagnitude
    (FGameplayTag::RequestGameplayTag("Data.Damage"), EvaluationParameters, Damage);
```



4. GameplayEffectContext
   - 通过[`GameplayEffectSpec`中的自定义`GameplayEffectContext`](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-context)发送数据到`ExecutionCalculation`

示例：

在`ExecutionCalculation`中, 你可以自`FGameplayEffectCustomExecutionParameters`访问`EffectContext`

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(Spec.GetContext().Get());
```



如果需要修改`GameplayEffectSpec`中的什么或者`EffectContext`:

```c++
FGameplayEffectSpec* MutableSpec = ExecutionParams.GetOwningSpecForPreExecuteMod();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(MutableSpec->GetContext().Get());
```



在`ExecutionCalculation`中修改`GameplayEffectSpec`时要小心. 参看`GetOwningSpecForPreExecuteMod()`的注释.

```c++
/** Non const access. Be careful with this, especially when modifying a spec after attribute capture. */
FGameplayEffectSpec* GetOwningSpecForPreExecuteMod() const;
```



#### 5.13 GE_CAR

- `GameplayEffectCustomApplicationRequirement`类为设计师提供对于`GameplayEffect`是否可以应用的高阶控制
- 而不是对`GameplayEffect`进行简单的`GameplayTag`检查
- 这可以通过在蓝图中重写`CanApplyGameplayEffect()`和在C++中重写`CanApplyGameplayEffect_Implementation()`实现



`UGameplayEffectCustomApplicationRequirement.h`

```c++
/** 用于通过蓝图或本机代码执行自定义游戏效果修改器计算的类 */ 
UCLASS(BlueprintType, Blueprintable, Abstract)
class GAMEPLAYABILITIES_API UGameplayEffectCustomApplicationRequirement : public UObject
{

public:
	GENERATED_UCLASS_BODY()
	
	/** 返回是否应用游戏效果 */
	UFUNCTION(BlueprintNativeEvent, Category="Calculation")
	bool CanApplyGameplayEffect(const UGameplayEffect* GameplayEffect, const FGameplayEffectSpec& Spec, 
                                UAbilitySystemComponent* ASC) const;
};
```



应用条件和场景：

- 目标需要有一定数量的`Attribute`
- 目标需要有一定数量的`GameplayEffect`堆栈



:::tip 补充说明

- `GE_CAR`还有很多高阶功能, 像检查`GameplayEffect`实例是否已经位于目标上
- 修改当前实例的[持续时间](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-duration)而不是应用一个新实例(对于`CanApplyGameplayEffect()`返回false)

:::



#### 5.14 GE_Cost

##### 5.14.1 Cost概念

- [GameplayAbility](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ga)有一个特别设计用来作为Ability花费(Cost)的可选`GameplayEffect`
- 花费(Cost)是指`ASC`激活`GameplayAbility`所必需的`Attribute`量
- 如果某个`GA`不能提供`Cost GE`, 那么它就不能被激活
- 该`Cost GE`应该是某个带有一个或多个自`Attribute`中减值Modifier的`即刻(Instant)GameplayEffect`
- 默认情况下, `Cost GE`是可以被预测的, 建议保留该功能, 也就是不要使用`ExecutionCalculations`
- `MMC`对于复杂的花费计算是完美适配并且鼓励使用的



##### 5.14.2 Cost复用

- 一个更高阶的技巧是对多个`GA`复用一个`Cost GE`
- 只需修改自`GA`的`Cost GE`创建的`GameplayEffectSpec`中指定的数据(花费值是在`GA`上定义的)



:::warning

这只作用于`实例化(Instanced)`的Ability

:::



复用技巧：

1. 使用`MMC`. 这是最简单的方式. 创建一个从`GameplayAbility`实例读取花费值的[MMC](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-mmc), 你可以从`GameplayEffectSpec`中获取到该实例

```c++
float UPGMMC_HeroAbilityCost::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->Cost.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

在这个例子中, 花费值是一个添加到`GameplayAbility`子类上的`FScalableFloat`.

```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cost")
FScalableFloat Cost;
```

[![Cost GE With MMC](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/costmmc.png)](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/costmmc.png)



2. **重写`UGameplayAbility::GetCostGameplayEffect()`.** 重写该函数并在[运行时](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-dynamic)创建一个用来读取`GameplayAbility`中花费值的`GameplayEffect`.



#### 5.15 GE_CD

##### 5.15.1 Cooldown概念

- [GameplayAbility](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ga)有一个特别设计用来作为Ability冷却(Cooldown)的可选`GameplayEffect`
- 冷却时间决定了激活Ability之后多久可以再次激活
- 如果某个`GA`在冷却中, 那么它就不能被激活
- 该`Cooldown GE`应该是一个不带有`Modifier`的`持续(Duration)GameplayEffect`
- 并且在`GameplayEffect`的`GrantedTags`中每个`GameplayAbility`或Ability插槽(Slot)(如果你的游戏有分配到插槽的可交换Ability且共享同一个冷却)都有一个独一无二的`GameplayTag`(`Cooldown Tag`)
- `GA`实际上会检查`Cooldown Tag`的存在而不是`Cooldown GE`的存在
- 默认情况下, `Cooldown GE`是可以被预测的, 建议保留该功能, 也就是不要使用`ExecutionCalculations`, `MMC`对于复杂的冷却计算是完美适配并且鼓励使用的



##### 5.15.2 Cooldown复用

- 一个更高阶的技巧是对多个`GA`复用一个`Cooldown GE`
- 只需修改自`GA`的`Cooldown GE`创建的`GameplayEffectSpec`中指定的数据(冷却时间和`Cooldown Tag`是在`GA`上定义的)



:::warning

这只作用于`实例化(Instanced)`的Ability

:::



复用技巧：

1. 使用[SetByCaller](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-spec-setbycaller)：
   - 这是最简单的方式
   - 使用`GameplayTag`设置`SetByCaller`为共享`Cooldown GE`的持续时间
   - 在`GameplayAbility`子类中, 为持续时间定义一个浮点/`FScalableFloat`变量
   - 为独一无二的`Cooldown Tag`定义一个`FGameplayTagContainer`
   - 除此之外还要定义一个临时`FGameplayTagContainer`
   - 其用来作为`Cooldown Tag`与`Cooldown GE`标签并集的返回指针

```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// Temp container that we will return the pointer to in GetCooldownTags().
// This will be a union of our CooldownTags and the Cooldown GE's cooldown tags.
UPROPERTY()
FGameplayTagContainer TempCooldownTags;
```

之后重写`UGameplayAbility::GetCooldownTags()`以返回`Cooldown Tag`和所有现有`Cooldown GE`标签的并集

```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

最后, 重写`UGameplayAbility::ApplyCooldown()`以注入我们自己的`Cooldown Tag`, 并将`SetByCaller`添加到`Cooldown GameplayEffectSpec`

```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * 
                                       ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = 
            MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
        
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
        
		SpecHandle.Data.Get()->SetSetByCallerMagnitude(
            FGameplayTag::RequestGameplayTag(FName(OurSetByCallerTag)), 
            CooldownDuration.GetValueAtLevel(GetAbilityLevel()));
        
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

下面图片中, 冷却时间`Modifier`被设置为`SetByCaller`, 其`Data Tag`为`Data.Cooldown`. `Data.Cooldown`就是上面代码中的`OurSetByCallerTag`

[![Cooldown GE with SetByCaller](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/cooldownsbc.png)](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/cooldownsbc.png)



2. 使用[MMC](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-ge-mmc)
   - 它的设置与上文所提的一致, 除了不需要在`Cooldown GE`和`ApplyCost`中设置`SetByCaller`作为持续时间
   - 相反, 我们需要将持续时间设置为`Custom Calculation类`并将其指向新创建的`MMC`

```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// Temp container that we will return the pointer to in GetCooldownTags().
// This will be a union of our CooldownTags and the Cooldown GE's cooldown tags.
UPROPERTY()
FGameplayTagContainer TempCooldownTags;
```

之后重写`UGameplayAbility::GetCooldownTags()`以返回`Cooldown Tag`和所有现有`Cooldown GE`标签的并集.

```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

最后, 重写`UGameplayAbility::ApplyCooldown()`以将我们的`Cooldown Tag`注入`Cooldown GameplayEffectSpec`.

```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo *
                                       ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = 
            MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
        
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```



```c++
float UPGMMC_HeroAbilityCooldown::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->CooldownDuration.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

[![Cooldown GE with MMC](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/cooldownmmc.png)](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/cooldownmmc.png)



##### 5.15.3 Cooldown剩余时间

```c++
bool APGPlayerState::GetCooldownRemainingForTag(FGameplayTagContainer CooldownTags, float & TimeRemaining, float & 
                                                CooldownDuration)
{
	if (AbilitySystemComponent && CooldownTags.Num() > 0)
	{
		TimeRemaining = 0.f;
		CooldownDuration = 0.f;

		FGameplayEffectQuery const Query = FGameplayEffectQuery::MakeQuery_MatchAnyOwningTags(CooldownTags);
        
		TArray< TPair<float, float> > DurationAndTimeRemaining = 
            AbilitySystemComponent->GetActiveEffectsTimeRemainingAndDuration(Query);
        
		if (DurationAndTimeRemaining.Num() > 0)
		{
			int32 BestIdx = 0;
			float LongestTime = DurationAndTimeRemaining[0].Key;
			for (int32 Idx = 1; Idx < DurationAndTimeRemaining.Num(); ++Idx)
			{
				if (DurationAndTimeRemaining[Idx].Key > LongestTime)
				{
					LongestTime = DurationAndTimeRemaining[Idx].Key;
					BestIdx = Idx;
				}
			}

			TimeRemaining = DurationAndTimeRemaining[BestIdx].Key;
			CooldownDuration = DurationAndTimeRemaining[BestIdx].Value;

			return true;
		}
	}

	return false;
}
```



:::warning

在客户端上查询剩余冷却时间要求其可以接收同步的`GameplayEffect`, 这依赖于它们`ASC`的[同步模式](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-asc-rm)

:::



##### 5.15.4 Cooldown回调



监听某个冷却何时开始：

```c++
AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf()

AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)
```

- 分别在`Cooldown GE`应用和`Cooldown Tag`添加时作出响应
- 建议监听`Cooldown GE`何时应用, 因为这时还可以访问应用它的`GameplayEffectSpec`
- 可以确定当前`Cooldown GE`是客户端预测的还是由服务端校正的



监听某个冷却何时结束：

```c++
AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate()
    
AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)
```

- 分别在`Cooldown GE`移除和`Cooldown Tag`移除时作出响应
- 建议监听`Cooldown Tag`何时移除, 因为当服务端校正的`Cooldown GE`到来时, 会移除客户端预测的`Cooldown GE`
- 这会响应`OnAnyGameplayEffectRemovedDelegate()`, 即使仍处于冷却过程中
- 预测的`Cooldown GE`在移除时和服务端校正的`Cooldown GE`在应用时`Cooldown Tag`都不会改变



:::warning

在客户端上监听某个`GameplayEffect`添加或移除要求其可以接收同步的`GameplayEffect`, 这依赖于它们`ASC`的[同步模式](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-asc-rm)

:::



- 样例项目包含一个用于监听冷却开始和结束的自定义蓝图节点
- HUD UMG Widget使用它来更新陨石技能的剩余冷却时间
- 该`AsyncTask`会一直响应直到手动调用`EndTask()`, 就像在UMG Widget的`Destruct`事件中调用那样
- 参阅`AsyncTaskAttributeChanged.h/cpp`.

[![Listen for Cooldown Change BP Node](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/cooldownchange.png)](https://raw.githubusercontent.com/BillEliot/GASDocumentation_Chinese/main/Images/cooldownchange.png)



#### 5.16 修改已激活GE持续时间

- 为了修改`Cooldown GE`或其他任何`持续(Duration)`GameplayEffect的剩余时间
- 需要修改`GameplayEffectSpec`的持续时间
- 更新它的`StartServerWorldTime`, `CachedStartServerWorldTime`, `StartWorldTime`
- 并且使用`CheckDuration()`重新检查持续时间
- 在服务端上完成这些操作并将`FActiveGameplayEffect`标记为dirty, 其会将这些修改同步到客户端



:::warning

该操作包含一个`const_cast`, 这可能不是`Epic`希望的修改持续时间的方法, 但是迄今为止它看起来运行得很好

:::



```c++
bool UPAAbilitySystemComponent::SetGameplayEffectDurationHandle(FActiveGameplayEffectHandle Handle, float NewDuration)
{
	if (!Handle.IsValid())
	{
		return false;
	}

	const FActiveGameplayEffect* ActiveGameplayEffect = GetActiveGameplayEffect(Handle);
	if (!ActiveGameplayEffect)
	{
		return false;
	}

	FActiveGameplayEffect* AGE = const_cast<FActiveGameplayEffect*>(ActiveGameplayEffect);
	if (NewDuration > 0)
	{
		AGE->Spec.Duration = NewDuration;
	}
	else
	{
		AGE->Spec.Duration = 0.01f;
	}

	AGE->StartServerWorldTime = ActiveGameplayEffects.GetServerWorldTime();
	AGE->CachedStartServerWorldTime = AGE->StartServerWorldTime;
	AGE->StartWorldTime = ActiveGameplayEffects.GetWorldTime();
	ActiveGameplayEffects.MarkItemDirty(*AGE);
	ActiveGameplayEffects.CheckDuration(Handle);

	AGE->EventSet.OnTimeChanged.Broadcast(AGE->Handle, AGE->StartWorldTime, AGE->GetDuration());
	OnGameplayEffectDurationChange(*AGE);

	return true;
}
```



#### 5.17 动态创建GE

- 在运行时创建动态`GameplayEffect`是一个高阶技术, 不必经常使用它



:::warning

- 只有`即刻(Instant)GameplayEffect`可以在运行时由C++创建
- `持续(Duration)`和`无限(Infinite)`GameplayEffect不能在运行时动态创建
- 因为它们在同步时会寻找并不存在的`GameplayEffect`类定义
- 为了实现该功能, 你应该创建一个原型`GameplayEffect`类
- 就像平时在编辑器中做的那样, 之后根据运行时所需来定制化`GameplayEffectSpec`

:::



- 运行时创建的`即刻(Instant)GameplayEffect`也可以在客户端[预测](https://github.com/FHangH/GASDocumentation_Chinese?tab=readme-ov-file#concepts-p)的`GameplayAbility`中调用
- 然而, 目前还不明确动态创建是否有副作用



示例1：

- 在角色`AttributeSet`中的值受到致命一击时创建该`GameplayEffect`来将金币和经验点数返还给击杀者

```c++
// Create a dynamic instant Gameplay Effect to give the bounties
UGameplayEffect* GEBounty = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Bounty")));
GEBounty->DurationPolicy = EGameplayEffectDurationType::Instant;

int32 Idx = GEBounty->Modifiers.Num();
GEBounty->Modifiers.SetNum(Idx + 2);

FGameplayModifierInfo& InfoXP = GEBounty->Modifiers[Idx];
InfoXP.ModifierMagnitude = FScalableFloat(GetXPBounty());
InfoXP.ModifierOp = EGameplayModOp::Additive;
InfoXP.Attribute = UGDAttributeSetBase::GetXPAttribute();

FGameplayModifierInfo& InfoGold = GEBounty->Modifiers[Idx + 1];
InfoGold.ModifierMagnitude = FScalableFloat(GetGoldBounty());
InfoGold.ModifierOp = EGameplayModOp::Additive;
InfoGold.Attribute = UGDAttributeSetBase::GetGoldAttribute();

Source->ApplyGameplayEffectToSelf(GEBounty, 1.0f, Source->MakeEffectContext());
```



示例2：

- 一个客户端预测的`GameplayAbility`中创建运行时`GameplayEffect`, 使用风险自负

```c++
UGameplayAbilityRuntimeGE::UGameplayAbilityRuntimeGE()
{
	NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;
}

void UGameplayAbilityRuntimeGE::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
	if (HasAuthorityOrPredictionKey(ActorInfo, &ActivationInfo))
	{
		if (!CommitAbility(Handle, ActorInfo, ActivationInfo))
		{
			EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
		}

		// Create the GE at runtime.
		UGameplayEffect* GameplayEffect = NewObject<UGameplayEffect>(GetTransientPackage(), TEXT("RuntimeInstantGE"));
		GameplayEffect->DurationPolicy = EGameplayEffectDurationType::Instant; // Only instant works with runtime GE.

		// Add a simple scalable float modifier, which overrides MyAttribute with 42.
		// In real world applications, consume information passed via TriggerEventData.
		const int32 Idx = GameplayEffect->Modifiers.Num();
		GameplayEffect->Modifiers.SetNum(Idx + 1);
		FGameplayModifierInfo& ModifierInfo = GameplayEffect->Modifiers[Idx];
		ModifierInfo.Attribute.SetUProperty(UMyAttributeSet::GetMyModifiedAttribute());
		ModifierInfo.ModifierMagnitude = FScalableFloat(42.f);
		ModifierInfo.ModifierOp = EGameplayModOp::Override;

		// Apply the GE.

		// Create the GESpec here to avoid the behavior of ASC to create GESpecs from the GE class default object.
		// Since we have a dynamic GE here, this would create a GESpec with the base GameplayEffect class, so we
		// would lose our modifiers. Attention: It is unknown, if this "hack" done here can have drawbacks!
		// The spec prevents the GE object being collected by the GarbageCollector, since the GE is a UPROPERTY on the spec.
		FGameplayEffectSpec* GESpec = new FGameplayEffectSpec(GameplayEffect, {}, 0.f); // "new", since lifetime is managed by a shared ptr within the handle
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, FGameplayEffectSpecHandle(GESpec));
	}
	EndAbility(Handle, ActorInfo, ActivationInfo, false, false);

```

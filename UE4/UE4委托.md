# UE4 委托



### 1. 委托定义



介绍：在C++对象上引用和执行成员函数的数据类型



定义：

- 是一种泛型但类型安全的方式，可在C++对象上调用成员函数
- 可使用委托动态绑定到任意对象的成员函数，之后在该对象上调用函数，即使调用程序不知对象类型也可进行操作
- 复制委托对象很安全。你也可以利用值传递委托，但这样操作需要在堆上分配内存，因此通常并不推荐
- 尽量通过引用传递委托



种类：

1. 单点委托
2. [组播委托](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Multicast)
   - [事件](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Events)
3. [动态(UObject, serializable)](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Dynamic)





### 2. 基本使用



#### 2.1 声明委托



前提：

- 根据与委托相绑定的函数（或多个函数）的函数签名来选择宏
- 每个宏都为新的委托类型名称、函数返回类型（如果不是 `void` 函数）及其参数提供了参数



提供的函数签名：

- 返回一个值的函数
- 声明为 `const` 函数
- 最多4个"载荷"变量
- 最多9个函数参数（可自定义）



参照表格查找用于声明委托的生命宏：

| 函数签名                                     | 声明宏                                                       |
| -------------------------------------------- | ------------------------------------------------------------ |
| `void Function()`                            | `DECLARE_DELEGATE(DelegateName)`                             |
| `void Function(Param1)`                      | `DECLARE_DELEGATE_OneParam(DelegateName, Param1Type)`        |
| `void Function(Param1, Param2)`              | `DECLARE_DELEGATE_TwoParams(DelegateName, Param1Type, Param2Type)` |
| `void Function(Param1, Param2, ...)`         | `DECLARE_DELEGATE_<Num>Params(DelegateName, Param1Type, Param2Type, ...)` |
| `<RetValType> Function()`                    | `DECLARE_DELEGATE_RetVal(RetValType, DelegateName)`          |
| `<RetValType> Function(Param1)`              | `DECLARE_DELEGATE_RetVal_OneParam(RetValType, DelegateName, Param1Type)` |
| `<RetValType> Function(Param1, Param2)`      | `DECLARE_DELEGATE_RetVal_TwoParams(RetValType, DelegateName, Param1Type, Param2Type)` |
| `<RetValType> Function(Param1, Param2, ...)` | `DECLARE_DELEGATE_RetVal_<Num>Params(RetValType, DelegateName, Param1Type, Param2Type, ...)` |



注意：

- 委托函数支持与[UFunctions](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/GameplayArchitecture/Functions)的[说明符](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/GameplayArchitecture/Functions/Specifiers)相同，但使用`UDELEGATE`而不是宏`UFUNCTION`

- 代码`BlueprintAuthorityOnly`说明添加到`FInstigatedAnyDamageSignature`委托符中

  ```c++
  UDELEGATE(BlueprintAuthorityOnly)
  DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(FInstigatedAnyDamageSignature, float, Damage, const UDamageType*, DamageType, AActor*, DamagedActor, AActor*, DamageCauser);
  ```





组播委托、动态委托和封装委托，上述宏的变体如下：

- DECLARE_MULTICAST_DELEGATE...
- DECLARE_DYNAMIC_DELEGATE...
- DECLARE_DYNAMIC_MULTICAST_DELEGATE...
- DECLARE_DYNAMIC_DELEGATE...
- DECLARE_DYNAMIC_MULTICAST_DELEGATE...

委托签名声明可存在于全局范围内、命名空间内、甚至类声明内，此类声明可能不在于函数体内

委托函数支持与[UFunctions](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/GameplayArchitecture/Functions)相同的[说明符](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/GameplayArchitecture/Functions/Specifiers)，但使用 `UDELEGATE` 宏而不是 `UFUNCTION`





#### 2.2 绑定委托



委托系统理解某些类型的对象，使用此类对象时将启用附加功能

将委托绑定到UObject或共享指针类的成员， 委托系统可保留对该对象的弱引用，因此对象在委托下方被销毁时，可通过调用 `IsBound()` 或 `ExecuteIfBound()` 函数进行处理



注意各类受支持对象的特殊绑定语法：

| 函数          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| `Bind`        | 绑定到现有委托对象                                           |
| `BindStatic`  | 绑定原始C++指针全局函数委托                                  |
| `BindRaw`     | 绑定原始C++指针委托<br />由于原始指针不使用任何类型的引用，因此在删除目标对象后调用`Execute` 或 `ExecuteIfBound` 会不安全 |
| `BindLambda`  | 绑定一个函子<br />这通常用于Lambda函数                       |
| `BindSP`      | 绑定基于指针的共享成员函数委托<br />共享指针委托会保留对对象的弱引用<br />可使用 `ExecuteIfBound()` 进行调用 |
| `BindUObject` | 绑定 `UObject` 的成员函数委托<br />`UObject` 委托会保留对你的对象 `UObject` 的弱引用<br />可使用 `ExecuteIfBound()` 进行调用 |
| `UnBind`      | 取消绑定此委托                                               |



参见 `DelegateSignatureImpl.inl`（位于 `..\UE4\Engine\Source\Runtime\Core\Public\Templates\`），了解此类函数的变体、参数和实现





#### 2.3 载荷数据



说明：

- 绑定到委托时，可同时传递载荷数据
- 其为调用时被直接传到绑定函数的任意变量
- 此操作是为了使代码更安全，因为有时委托可能含有未初始化且被后续访问的返回值和输出参数
- 执行未绑定的委托在某些情况下确实可能导致内存混乱



使用：

- 可调用 `IsBound()` 检查是否可安全执行委托
- 同时，对于无返回值的委托，可调用 `ExecuteIfBound()`，但需注意输出参数可能未初始化



注意输出参数可能未初始化：

| 执行函数         | 描述                                                         |
| :--------------- | :----------------------------------------------------------- |
| `Execute`        | 不检查其绑定情况即执行一个委托                               |
| `ExecuteIfBound` | 检查一个委托是否已绑定，如是，则调用Execute                  |
| `IsBound`        | 检查一个委托是否已绑定，经常出现在包含 `Execute` 调用的代码前 |







#### 2.4 用法示例



1. 设类拥有可在任何地方随意调用的方法：

```c++
class FLogWriter
{
    void WriteToLog(FString);
};
```



2. 要调用WriteToLog函数，需创建该函数签名的委托类型：

```c++
DECLARE_DELEGATE_OneParam(FStringDelegate, FString);

// 此将创建名为 FStringDelegate 的委托类型，该类型使用 FString 类型的单个参数
```



3. 在类中使用此 `FStringDelegate` 的方法范例：

```c++
class FMyClass
{
    FStringDelegate WriteToLogDelegate;
};
```

- 利用此操作，类可保有指向任意类中的方法的指针
- 该类唯一真正了解的信息就是，此委托是其的函数签名



4. 如要分配委托，现在只需创建委托类的实例，将拥有该方法的类作为模板参数传递
5. 同时还需传递对象的实例和方法的实际函数地址
6. 创建 `FLogWriter` 类的实例， 然后创建该对象实例 `WriteToLog` 方法的委托：

```c++
TSharedRef<FLogWriter> LogWriter(new FLogWriter());

WriteToLogDelegate.BindSP(LogWriter, &FLogWriter::WriteToLog);
```

- 此操作可将委托动态绑定到类的方法



注意：

- 绑定到的对象由共享指针拥有，因此 `BindSP` 的SP部分代表共享指针
- 还有不同对象类型的版本，例如BindRaw()和BindUObject()
- FMyClass现在可调用 `WriteToLog` 方法，甚至无需了解 `FLogWriter` 类的任何信息



7. 要调用委托，只需使用 `Execute()` 方法：

```c++
WriteToLogDelegate.Execute(TEXT("Delegates are great!"));
```



8. 如将函数绑定到网络前调用Execute()，将触发断言，多数情况下，建议进行以下操作：

```c++
WriteToLogDelegate.ExecuteIfBound(TEXT("Only executes if a function was bound!"));
```







### 3. 动态委托





- 可序列化且支持反射的委托
- 动态委托可序列化，其函数可按命名查找，但其执行速度比常规委托慢





#### 3.1 声明动态委托



介绍：动态委托的声明方式与[声明标准委托](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/)相同， 只是前者使用动态委托专属的宏变体

| 声明宏                                                       | 描述                 |
| :----------------------------------------------------------- | :------------------- |
| `DECLARE_DYNAMIC_DELEGATE[_RetVal, ...]( DelegateName )`     | 创建一个动态委托     |
| `DECLARE_DYNAMIC_MULTICAST_DELEGATE[_RetVal, ...]( DelegateName )` | 创建一个动态组播委托 |





#### 3.2 动态委托绑定



| 辅助宏                                  | 说明                                                         |
| :-------------------------------------- | :----------------------------------------------------------- |
| `BindDynamic( UserObject, FuncName )`   | 用于在动态委托上调用BindDynamic()的辅助宏，自动生成函数命名字符串 |
| `AddDynamic( UserObject, FuncName )`    | 用于在动态组播委托上调用AddDynamic()的辅助宏，自动生成函数命名字符串 |
| `RemoveDynamic( UserObject, FuncName )` | 用于在动态组播委托上调用RemoveDynamic()的辅助宏，自动生成函数命名字符串 |





#### 3.3 执行动态委托



使用：通过调用委托的 `Execute()` 函数执行绑定到委托的函数



注意：

- 执行前须检查委托是否已绑定
- 此操作是为了使代码更安全，因为有时委托可能含有未初始化且被后续访问的返回值和输出参数
- 执行未绑定的委托在某些情况下确实可能导致内存混乱
- 可调用 `IsBound()` 检查是否可安全执行委托
- 同时，对于无返回值的委托，可调用 `ExecuteIfBound()`，但需注意输出参数可能未初始化



| 执行函数         | 描述                                                         |
| :--------------- | :----------------------------------------------------------- |
| `Execute`        | 不检查其绑定情况即执行一个委托                               |
| `ExecuteIfBound` | 检查一个委托是否已绑定，如是，则调用Execute                  |
| `IsBound`        | 检查一个委托是否已绑定，经常出现在包含 `Execute` 调用的代码前 |









### 4. 事件



- 可绑定到多个函数并同时全部执行的委托



说明：

- 虽然任意类均可绑定事件，但只有声明事件的类可以调用事件 的 `Broadcast`、`IsBound` 和 `Clear` 函数
- 意味着事件对象可在公共接口中公开，而无需让外部类访问这些敏感度函数



事件使用情况有：

- 在纯抽象类中包含回调、限制外部类调用 `Broadcast`、`IsBound` 和 `Clear` 函数





#### 4.1 声明事件



- 事件的声明和 [组播委托声明](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Multicast) 方式几乎相同，唯一的区别是它们使用事件特有的宏变体

| 声明宏                                                       | 描述                  |
| :----------------------------------------------------------- | :-------------------- |
| `DECLARE_EVENT( OwningType, EventName )`                     | 创建一个事件          |
| `DECLARE_EVENT_OneParam( OwningType, EventName, Param1Type )` | 创建带一个参数的事件  |
| `DECLARE_EVENT_TwoParams( OwningType, EventName, Param1Type, Param2Type )` | 创建带两个参数的事件  |
| `DECLARE_EVENT_<Num>Params( OwningType, EventName, Param1Type, Param2Type, ...)` | 创建带 N 个参数的事件 |



注意：`DECLARE_EVENT` 宏的首个参数是"拥有"此事件的类，因此可调用 `Broadcast()` 函数





#### 4.2 绑定事件



- 事件绑定与 [组播委托绑定](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Multicast) 方式相同





#### 4.3 事件执行



说明：

- 事件允许附带多个函数委托，然后调用事件的 `Broadcast()` 函数将它们一次性全部执行
- 事件签名不允许使用返回值。对于事件而言，只有定义事件的类才能调用 `Broadcast()` 函数



使用：

- 即使不存在绑定，在事件上调用 `Broadcast()` 也是安全操作
- 唯一需要注意的情况是使用事件初始化输出变量，通常不建议执行此操作



注意：

- 调用 `Broadcast()` 函数时，被绑定函数的执行顺序尚未定义
- 有可能不按照函数的添加顺序执行



| 函数          | 描述                                         |
| :------------ | :------------------------------------------- |
| `Broadcast()` | 将此事件广播到所有绑定对象，已失效的对象除外 |







#### 4.4 实现范例



##### 4.4.1 简单事件



```c++
public:
/** Broadcasts whenever the layer changes */
DECLARE_EVENT( FLayerViewModel, FChangedEvent )
FChangedEvent& OnChanged() { return ChangedEvent; }

private:
/** Broadcasts whenever the layer changes */
FChangedEvent ChangedEvent;
```



注意：事件的访问器应该依照 OnXXX 模式，而非常规的 GetXXX 模式







##### 4.4.2 继承的抽象事件



**基础类实现**

```c++
/** Register/Unregister a callback for when assets are added to the registry */
DECLARE_EVENT_OneParam( IAssetRegistry, FAssetAddedEvent, const FAssetData&);
virtual FAseetAddedEvent& OnAssetAdded() = 0;
```



**派生类实现**

```c++
DECLARE_DERIVED_EVENT( FAssetRegistry, IAssetRegistry::FAssetAddedEvent, FAssetAddedEvent);
virtual FassetAddedEvent& OnAssetAdded() override { return AssetAddedEvent; }
```



注意：

- 在派生类中声明一个派生事件时，不要在 `DECLARE_DERIVED_EVENT` 宏中重复函数签名
- 此外，`DECLARE_DERIVED_EVENT` 宏的最后一个参数是事件的新命名，通常与基础类型相同





##### 4.4.3 继承事件



- 派生类不会继承对基础类敏感事件成员的访问
- 允许派生类广播其事件的基础类需要公开事件受保护的广播函数



**基础类**

```c++
public:
/** Broadcasts whenever the layer changes */
DECLARE_EVENT( FLayerViewModel, FChangedEvent )
FChangedEvent& OnChanged() { return ChangedEvent; }

protected:
void BroadcastChanged()
{
    ChangedEvent.Broadcast();
}

private:
/** Broadcasts whenever the layer changes */
FChangedEvent ChangedEvent;
```







### 5. 多播委托



说明：

- 可以绑定到多个函数并一次性同时执行它们的委托



功能：

- 多播委托拥有大部分与单播委托相同的功能
- 它们只拥有对对象的弱引用，可以与结构体一起使用，可以四处轻松复制



注意：

- 像常规委托一样，多播委托可以远程加载/保存和触发；但多播委托函数不能使用返回值
- 它们最适合用来 四处轻松传递一组委托
- [事件](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Events)是一种特殊类型的多播委托，它在访问`Broadcast()`、`IsBound()`和`Clear()`函数时会受到限制







#### 5.1 声明多播委托



多播委托在声明方式上与[声明标准委托](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/)相同，只是前者使用特定于多播委托的宏变体。

| 声明宏                                                       | 说明                   |
| :----------------------------------------------------------- | :--------------------- |
| `DECLARE_MULTICAST_DELEGATE[_RetVal, ...]( DelegateName )`   | 创建一个多播委托。     |
| `DECLARE_DYNAMIC_MULTICAST_DELEGATE[_RetVal, ...]( DelegateName )` | 创建一个动态多播委托。 |





#### 5.2 绑定多播委托



- 多播委托可以绑定多个函数，当委托触发时，将调用所有这些函数
- 因此，绑定函数在语义上与数组更加类似

| 函数           | 说明                                                         |
| :------------- | :----------------------------------------------------------- |
| "Add()"        | 将函数委托添加到该多播委托的调用列表中                       |
| "AddStatic()"  | 添加原始C++指针全局函数委托                                  |
| "AddRaw()"     | 添加原始C++指针委托<br />原始指针不使用任何类型的引用，因此如果从委托下面删除了对象，则调用此函数可能不安全<br />调用Execute()时请小心！ |
| "AddSP()"      | 添加基于共享指针的（快速、非线程安全）成员函数委托<br />共享指针委托保留对对象的弱引用 |
| "AddUObject()" | 添加基于UObject的成员函数委托<br />UObject委托保留对对象的弱引用 |
| "Remove()"     | 从该多播委托的调用列表中删除函数（性能为O(N)）<br />注意，委托的顺序可能不会被保留 |
| "RemoveAll()"  | 从该多播委托的调用列表中删除绑定到指定UserObject的所有函数<br />注意，委托的顺序可能不会被保留 |



注意：

- "RemoveAll()"将删除绑定到所提供指针的所有已注册委托
- 未绑定到对象指针的原始委托不会被该函数所删除



参阅"DelegateSignatureImpl.inl"（位于"..\UE4\Engine\Source\Runtime\Core\Public\Delegates\"中）了解这些函数的变体、参数和实现





#### 5.3 多播执行



说明：

- 多播委托允许您附加多个函数委托，然后通过调用多播委托的"Broadcast()"函数一次性同时执行它们
- 多播委托签名不得使用返回值



注意：

- 在多播委托上调用"Broadcast()"总是安全的，即使是在没有任何绑定时也是如此
- 唯一需要注意的是，如果您使用委托来初始化输出变量，通常会带来非常不利的后果
- 调用"Broadcast()"时绑定函数的执行顺序尚未定义
- 执行顺序可能与函数的添加顺序不相同



| 函数          | 说明                                                 |
| :------------ | :--------------------------------------------------- |
| "Broadcast()" | 将该委托广播给所有绑定的对象，但可能已过期的对象除外 |









### 6. 委托示例



#### 6.1 单播委托



**定义单播委托**

1. 新建一个测试类`MyActor`

2. `MyActor.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "GameFramework/Actor.h"
   #include "MyActor.generated.h"
   
   /*
    * 定义单播委托
    */
   // 无返回值，无参数
   DECLARE_DELEGATE(FTestDelegateNoParam);
   
   // 无返回值，一个参数
   DECLARE_DELEGATE_OneParam(FTestDelegateOneParam, float);
   
   // 无返回值，多个参数（两个参数为例）
   DECLARE_DELEGATE_TwoParams(FTestDelegateTwoParams, float, const FString&);
   
   // 有返回值，多个参数（两个参数为例）
   DECLARE_DELEGATE_RetVal_TwoParams(int32, FTestDelegateRetValTwoParams, float, const FString&);
   
   UCLASS()
   class A_03_DELEGATE_API AMyActor : public AActor
   {
   	GENERATED_BODY()
   
   public:
   	AMyActor();
   
   protected:
   	virtual void BeginPlay() override;
   
   /*
    * 声明委托，定义函数，绑定委托
    */
   protected:
   	// 声明委托
   	FTestDelegateRetValTwoParams DelegateRetValTwoParams;
   	
   public:
   	// 测试单播委托，有返回值，两个参数
   	UFUNCTION()
   	int32 Func(float a, const FString& s);
   
       // 测试单播委托，绑定静态函数
   	UFUNCTION()
   	static int32 Func_Static(float a, const FString &s);
   };
   ```

3. `MyActor.cpp`

   ```c++
   #include "MyActor.h"
   
   class FTest
   {
   public:
   	int32 Func(float a, const FString &s)
   	{
   		return 1;
   	}
   };
   
   AMyActor::AMyActor(){}
   
   void AMyActor::BeginPlay()
   {
   	Super::BeginPlay();
   
   	// 委托-通过对象-绑定函数
   	DelegateRetValTwoParams.BindUObject(this, &AMyActor::Func);
   	
   	// 委托-直接绑定Lambda函数
   	DelegateRetValTwoParams.BindLambda([this](float a, const FString &s)->int32
   	{
   		return 1;
   	});
   
   	// 委托-通过原生Cpp类-绑定函数
   	FTest TestA;
   	DelegateRetValTwoParams.BindRaw(&TestA, &FTest::Func);
   
   	// 委托-通过共享指针-绑定函数
   	const TSharedPtr<FTest> TestB = MakeShareable(new FTest);
   	DelegateRetValTwoParams.BindSP(TestB.ToSharedRef(), &FTest::Func);
   
   	// 委托-绑定静态函数
   	DelegateRetValTwoParams.BindStatic(&Func_Static);
   
   	// 委托-通过线程安全共享指针-绑定函数
   	const TSharedPtr<FTest, ESPMode::ThreadSafe> TestC = MakeShareable(new FTest);
   	DelegateRetValTwoParams.BindThreadSafeSP(TestC.ToSharedRef(), &FTest::Func);
   
   	// 委托-通过函数名称的反射-绑定UFUNCTION()修饰的函数
   	DelegateRetValTwoParams.BindUFunction(this, "Func");
   
   	// 顺手通过委托调用绑定的函数
   	if (DelegateRetValTwoParams.IsBound())
   	{
   		// ExecuteIfBound 是无参委托的调用方法
   		int a = DelegateRetValTwoParams.Execute(24.f, "FTestDelegateRetValTwoParams");
   	}
   }
   
   /* My Code */
   int32 AMyActor::Func(float a, const FString &s)
   {
   	return 1;
   }
   
   int32 AMyActor::Func_Static(float a, const FString& s)
   {
   	return 1;
   }
   ```

   





#### 6.2 多播委托



**定义多播委托**

1. `MyActor.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "GameFramework/Actor.h"
   #include "MyActor.generated.h"
   
   /*
    * 定义多播委托，绑定多个形式相同的函数，同时执行（顺序随机）
    * 无返回值类型
    */
   // 无返回值，无参数
   DECLARE_MULTICAST_DELEGATE(FTestMulDelegateNoParam);
   
   // 无返回值，一个或多个参数（两个参数为例）
   DECLARE_MULTICAST_DELEGATE_TwoParams(FTestMulDelegateTwoParams, float, const FString &);
   
   UCLASS()
   class A_03_DELEGATE_API AMyActor : public AActor
   {
   	GENERATED_BODY()
   
   public:
   	AMyActor();
   
   protected:
   	virtual void BeginPlay() override;
   
   /*
    * 声明委托，定义函数，绑定委托
    */
   protected:
   	// 声明委托
   	FTestMulDelegateTwoParams MulDelegateTwoParams;
   	
   public:
   	// 测试多播委托，无返回值，两个参数
   	UFUNCTION()
   	void Func_Mul(float a, const FString &s);
   };
   ```

2. `MyActor.cpp`

   ```c++
   #include "MyActor.h"
   
   AMyActor::AMyActor(){}
   
   void AMyActor::BeginPlay()
   {
   	Super::BeginPlay();
   
   	/*
   	 * 多播委托
   	 */
       // 多播委托-通过对象-绑定函数
   	MulDelegateTwoParams.AddUObject(this, &AMyActor::Func_Mul);
       
       // 调用函数
   	MulDelegateTwoParams.Broadcast(24.f, "FTestMulDelegateTwoParams");
   }
   
   /* My Code */
   void AMyActor::Func_Mul(float a, const FString& s)
   {
   }
   ```

   





#### 6.3 动态委托



##### 6.3.1 动态单播委托



**定义动态单播委托**

1. `MyActor.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "GameFramework/Actor.h"
   #include "MyActor.generated.h"
   
   /*
    * 定义动态单播委托
    * 可以用于蓝图中
    * 所以定义时，参数列表中的参数类型后要跟上参数名
    */
   // 无返回值，无参数
   DECLARE_DYNAMIC_DELEGATE(FTestDynamicDelegateNoParam);
   
   // 无返回值，有参数（一个参数为例）
   DECLARE_DYNAMIC_DELEGATE_OneParam(FTestDynamicDelegateOneParam, int32, a);
   
   // 有返回值，有参数（一个参数为例）
   DECLARE_DYNAMIC_DELEGATE_RetVal_OneParam(int32, FTestDynamicDelegateRetValOneParam, int32, a);
   
   UCLASS()
   class A_03_DELEGATE_API AMyActor : public AActor
   {
   	GENERATED_BODY()
   
   public:
   	AMyActor();
   
   protected:
   	virtual void BeginPlay() override;
   
   /*
    * 声明委托，定义函数，绑定委托
    */
   protected:
   	// 声明委托
   	FTestDynamicDelegateRetValOneParam DynamicDelegateRetValOneParam;
       
   public:
   	// 测试动态单播委托，蓝图中使用
   	UFUNCTION(BlueprintCallable)
   	void Func_Dynamic(FTestDynamicDelegateRetValOneParam DynamicDelegateRetValOneParam);
   
   	// 测试动态单播委托，cpp使用
   	UFUNCTION()
   	int32 Func_DynamicCpp(int32 a);
   };
   ```

2. `MyActor.cpp`

   ```c++
   #include "MyActor.h"
   
   AMyActor::AMyActor(){}
   
   void AMyActor::BeginPlay()
   {
   	Super::BeginPlay();
   
   	/*
   	 * 动态单播委托
   	 */
   	DynamicDelegateRetValOneParam.BindDynamic(this, &AMyActor::Func_DynamicCpp);
   	int32 b = DynamicDelegateRetValOneParam.Execute(24);
   }
   
   /* My Code */
   void AMyActor::Func_Dynamic(FTestDynamicDelegateRetValOneParam DynamicDelegateRetValOneParam)
   {
   }
   
   int32 AMyActor::Func_DynamicCpp(int32 a)
   {
   	return 1;
   }
   ```





##### 6.3.2 动态多播委托



**定义动态多播委托**

1. `MyActor.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "GameFramework/Actor.h"
   #include "MyActor.generated.h"
   
   /*
    * 动态多播委托
    * 可以用于蓝图中
    * 等同于蓝图中的事件调度器
    * 多播委托没有返回值
    */
   // 无返回值，无参数
   DECLARE_DYNAMIC_MULTICAST_DELEGATE(FTestDynamicMulDelegate);
   
   // 无返回值，有参数（一个参数为例）
   DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FTestDynamicMulDelegateOnParam, int32, a);
   
   UCLASS()
   class A_03_DELEGATE_API AMyActor : public AActor
   {
   	GENERATED_BODY()
   
   public:
   	AMyActor();
   
   protected:
   	virtual void BeginPlay() override;
   
   /*
    * 声明委托，定义函数，绑定委托
    */
   protected:
   	// 声明委托
   	UPROPERTY(BlueprintAssignable)
   	FTestDynamicMulDelegate DynamicMulDelegate;
   
   	FTestDynamicMulDelegateOnParam DynamicMulDelegateOnParam;
   	
   public:
   	// 测试动态多播委托，cpp使用
   	UFUNCTION()
   	void Func_DynamicMulCpp();
   
   	// 测试动态多播委托，有参数，cpp使用
   	UFUNCTION()
   	void Func_DynamicMul_One(int32 a);
   };
   ```

2. `MyActor.cpp`

   ```cpp
   #include "MyActor.h"
   
   AMyActor::AMyActor(){}
   
   void AMyActor::BeginPlay()
   {
   	Super::BeginPlay();
   
   	/*
   	 * 动态多播委托
   	 */
   	// 无参数
   	DynamicMulDelegate.AddDynamic(this, &AMyActor::Func_DynamicMulCpp);
   	DynamicMulDelegate.Broadcast();
   
   	// 有参数
   	DynamicMulDelegateOnParam.AddDynamic(this, &AMyActor::Func_DynamicMul_One);
   	DynamicMulDelegateOnParam.Broadcast(24);
   }
   
   /* My Code */
   void AMyActor::Func_DynamicMulCpp()
   {
   }
   
   void AMyActor::Func_DynamicMul_One(int32 a)
   {
   }
   ```

   





### 7. 原生Cpp委托



**Delegate.cpp**单播

```c++
#include <iostream>
#include <functional>

// 函数指针的两种定义方式
typedef int (*FuncMethod)(int, int);
// using FuncMethod = int(*)(int, int);

class TestA
{
public:
    int TestAddNum(int a, int b)
    {
        const int c = a + b;
        std::cout << c << std::endl;
        return c;
    }
};
typedef int (TestA::*TestFuncMethod)(int , int);

//using TestFuncMethod = int(TestA::*)(int, int);


// 单播代理
class FDelegateTwoParams
{
private:
    std::function<int()> Func;

public:
    void BindGlobalFunc(FuncMethod FuncPtr, int a, int b)
    {
        Func = std::bind(FuncPtr, a, b);
    }

    void BindRaw(TestA *UserClass, TestFuncMethod FuncPtr, int a, int b)
    {
        Func = std::bind(FuncPtr, UserClass, a, b);
    }

    bool IsBound() const
    {
        return Func ? true : false;
    }

    void Execute()
    {
        Func();
    }

    bool ExecuteIfBound()
    {
        if (IsBound())
        {
            Execute();
            return true;
        }
        return false;
    }
};

int AddNum(int a, int b)
{
    const int c = a + b;
    std::cout << c << std::endl;
    return c;
}

void DelegateDemo()
{
    FDelegateTwoParams Delegate;
    Delegate.BindGlobalFunc(&AddNum, 10, 20);
    Delegate.Execute();
}

void DelegateDemo2()
{
    TestA *testA = new TestA;
    FDelegateTwoParams Delegate;
    Delegate.BindRaw(testA, &TestA::TestAddNum, 20, 30);
    Delegate.Execute();
}

int main()
{
    // // 函数指针调用函数
    // FuncMethod FuncPtr = &AddNum;
    // FuncPtr(1, 2);

    // // 全局绑定调用函数
    // std::function<int()> FuncPtr2 = std::bind(&AddNum, 2, 3);
    // FuncPtr2();

    DelegateDemo();
    DelegateDemo2();

    return 0;
}
```





**DelegateMul.cpp**多播

```c++
#include <iostream>
#include <functional>
#include <vector>

// 函数指针的两种定义方式
typedef int (*FuncMethod)(int, int);

class TestA
{
public:
    int TestAddNum(int a, int b)
    {
        const int c = a + b;
        std::cout << c << std::endl;
        return c;
    }
};
typedef int (TestA::*TestFuncMethod)(int , int);


// 多播代理
class FDelegateTwoParams
{
public:
    void BindGlobalFunc(FuncMethod FuncPtr, int a, int b)
    {
        FuncArray.push_back(std::bind(FuncPtr, a, b));
    }

    void BindRaw(TestA *UserClass, TestFuncMethod FuncPtr, int a, int b)
    {
        FuncArray.push_back(std::bind(FuncPtr, UserClass, a, b));
    }

    void BroadCast()
    {
        for (std::vector<std::function<int()>>::iterator itr = FuncArray.begin();
              itr != FuncArray.end(); ++itr)
        {
            (*itr)();
        }
        
    }

private:
    std::vector<std::function<int()>> FuncArray;
};

int AddNum(int a, int b)
{
    const int c = a + b;
    std::cout << c << std::endl;
    return c;
}

void DelegateDemo()
{
    TestA *testA = new TestA;
    FDelegateTwoParams Delegate;
    Delegate.BindRaw(testA, &TestA::TestAddNum, 10, 20);
    Delegate.BindRaw(testA, &TestA::TestAddNum, 20, 30);
    Delegate.BroadCast();
}

int main()
{
    DelegateDemo();

    return 0;
}
```





**DelegateMarco.cpp**

```c++
#include <iostream>
#include <functional>

class TestA
{
public:
    void FuncNoParam()
    {
        std::cout << "FuncNoParamDelegate" << std::endl;
    }
    
    void FuncTwoParam(int a, int b)
    {
        const int c = a + b;
        std::cout << "FuncTwoParam : " << c << std::endl;
    }
};
// 定义类 作用域下的函数指针
typedef void (TestA::*FuncMethodNoParam)(void);
typedef void (TestA::*FuncMethodTwoParam)(int, int);

// 宏定义 代理
// 无参数，无返回值
#define DECLARE_DELEGATE(DelegateName)\
class My##DelegateName\
{\
public:\
    void BindRaw(TestA *UserClass, FuncMethodNoParam FuncPtr)\
    {\
        Func = std::bind(FuncPtr, UserClass);\
    }\
    bool IsBound() const\
    {\
        return Func ? true : false;\
    }\
    void Execute()\
    {\
        Func();\
    }\
    bool ExecuteIfBound()\
    {\
        if (IsBound())\
        {\
            Execute();\
            return true;\
        }\
        return false;\
    }\
private:\
    std::function<void()> Func;\
};

// 有参数，无返回值
#define DECLARE_DELEGATE_TWO_PARAM(DelegateName, TypeParam1, TypeParam2)\
class My##DelegateName\
{\
public:\
    void BindRaw(TestA *UserClass, FuncMethodTwoParam FuncPtr, TypeParam1 p1, TypeParam2 p2)\
    {\
        Func = std::bind(FuncPtr, UserClass, p1, p2);\
    }\
    bool IsBound() const\
    {\
        return Func ? true : false;\
    }\
    void Execute()\
    {\
        Func();\
    }\
    bool ExecuteIfBound()\
    {\
        if (IsBound())\
        {\
            Execute();\
            return true;\
        }\
        return false;\
    }\
private:\
    std::function<void()> Func;\
};

// 创建自定义宏代理
DECLARE_DELEGATE(TestDelegateNoParam);
DECLARE_DELEGATE_TWO_PARAM(TestDelegateTwoParam, int, int);

void DelegateDemo()
{
    TestA testA;
    MyTestDelegateNoParam Delegate;
    Delegate.BindRaw(&testA, &TestA::FuncNoParam);
    Delegate.Execute();
}

void DelegateDemo2()
{
    TestA testB;
    MyTestDelegateTwoParam Delegate;
    Delegate.BindRaw(&testB, &TestA::FuncTwoParam, 10, 20);
    Delegate.Execute();
}

int main()
{
    DelegateDemo();
    DelegateDemo2();
    return 0;
}
```





**DelegateTemplate.cpp**

```c++
#include <iostream>
#include <functional>

template<typename Class, typename FuncType>
struct TMemFuncPtrType;

template<typename Class, typename RetType, typename... ArgTypes>
struct TMemFuncPtrType<Class, RetType(ArgTypes...)>
{
    typedef RetType(Class::*Type)(ArgTypes...);
};


// 定义一个模板代理
template <typename RetValType, typename... ParamTypes>
class TBaseDelegate
{
private:
    std::function<RetValType()> Func;

public:
    template<typename UserClass>
    void BindRaw(UserClass *MyUserClass, 
            typename TMemFuncPtrType<UserClass, RetValType(ParamTypes...)>::Type FuncPtr, 
            ParamTypes... Vars)
    {
        Func = std::bind(FuncPtr, MyUserClass, Vars...);
    }

    bool IsBound() const
    {
        return Func ? true : false;
    }

    void Execute()
    {
        Func();
    }

    bool ExecuteIfBound()
    {
        if (IsBound())
        {
            Execute();
            return true;
        }
        return false;
    }
};

#define DECLARE_DELEGATE(DelegateName) class DelegateName : public TBaseDelegate<void>{};
#define DECLARE_DELEGATE_TWO_PARAMS(DelegateName, ParamType1, ParamType2) class DelegateName : public TBaseDelegate<void, ParamType1, ParamType2>{};
#define DECLARE_DELEGATE_RETVAL_TWO_PARAMS(RetType, DelegateName, ParamType1, ParamType2) class DelegateName : public TBaseDelegate<RetType, ParamType1, ParamType2>{};


DECLARE_DELEGATE(TestDelegate);
DECLARE_DELEGATE_TWO_PARAMS(TestDelegateTwoParams, int, int);
DECLARE_DELEGATE_RETVAL_TWO_PARAMS(int, TestDelegateRetValTwoParams, int, int);

class TestA
{
public:
    void Print()
    {
        std::cout << "Hello World" << std::endl;
    }

    void Print2(int a, int b)
    {
        const int c = a + b;
        std::cout << c << std::endl;
    }

    int Print3(int a, int b)
    {
        const int c = a + b;
        std::cout << c << std::endl;
        return c;
    }
};
 
void DelegateDemo()
{
    TestDelegate TD;
    TestA testA;
    TD.BindRaw(&testA, &TestA::Print);
    TD.Execute();
}

void DelegateDemo2()
{
    TestDelegateTwoParams TDTwoParams;
    TestA testB;
    TDTwoParams.BindRaw(&testB, &TestA::Print2, 10, 20);
    TDTwoParams.Execute();
}

void DelegateDemo3()
{
    TestDelegateRetValTwoParams TDRTwoParams;
    TestA testC;
    TDRTwoParams.BindRaw(&testC, &TestA::Print3, 99, 100);
    TDRTwoParams.Execute();
}

int main()
{
    DelegateDemo();
    DelegateDemo2();
    DelegateDemo3();
    return 0;
}
```


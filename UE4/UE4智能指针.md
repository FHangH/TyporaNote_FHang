# UE4 智能指针



[toc]



### 1. 虚幻智能指针库



- 为C++11智能指针的自定义实现，旨在减轻内存分配和追踪的负担
- 该实现包括行业标准 **共享指针**、**弱指针** 和 **唯一指针**。其还可添加 **共享引用**，此类引用的行为与不可为空的共享指针相同
- 虚幻Objects使用更适合游戏代码的单独内存追踪系统，因此这些类无法与 `UObject` 系统同时使用





### 2. 智能指针类型



前提：

- 智能指针可影响其包含或引用对象的寿命
- 不同智能指针对对象有不同的限制和影响



下表可用于协助决定各类型智能指针的适用情况：

| 智能指针类型                                                 | 适用情形                                                     |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| **[共享指针](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/SmartPointerLibrary/SharedPointer)**（`TSharedPtr`） | 共享指针拥有其引用的对象，无限防止该对象被删除，并在无共享指针或共享引用（见下文）引用其时，最终处理其的删除<br />共享指针可为空白，意味其不引用任何对象<br />任何非空共享指针都可对其引用的对象生成共享引用 |
| **共享引用<br />(ProgrammingAndScripting<br />/ProgrammingWithCPP<br />/UnrealArchitecture<br />/SmartPointerLibrary<br />/SharedReference)**（`TSharedRef`） | 共享引用的行为与共享指针类似，即其拥有自身引用的对象<br />对于空对象而言，其存在不同；共享引用须固定引用非空对象<br />共享指针无此类限制，因此共享引用可固定转换为共享指针，且该共享指针固定引用有效对象<br />要确认引用的对象是非空，或者要表明共享对象所有权时，请使用共享引用 |
| **弱指针(ProgrammingAndScripting<br />/ProgrammingWithCPP<br />/UnrealArchitecture<br />/SmartPointerLibrary<br />/WeakPointer)**（TWeakPtr`TSharedPtr`） | 弱指针类与共享指针类似，但不拥有其引用的对象，因此不影响其生命周期<br />此属性中断引用循环，因此十分有用，但也意味弱指针可在无预警的情况下随时变为空<br />因此，弱指针可生成指向其引用对象的共享指针，确保程序员能对该对象进行安全临时访问 |
| **唯一指针**（`TUniquePtr`）                                 | 唯一指针仅会显式拥有其引用的对象<br />仅有一个唯一指针指向给定资源，因此唯一指针可转移所有权，但无法共享<br />复制唯一指针的任何尝试都将导致编译错误<br />唯一指针超出范围时，其将自动删除其所引用的对象 |



注意：

- 对唯一指针引用的对象进行共享指针或共享引用的操作十分危险
- 即使其他智能指针继续引用该对象，此操作不会取消唯一指针自身被销毁时删除该对象的行为
- 同样，不应为共享指针或共享引用引用的对象创建唯一指针





### 3. 智能指针



| 优点                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| **防止内存泄漏**       | 共享引用不存在时，智能指针（弱指针除外）会自动删除对象       |
| **弱引用**             | 弱指针会中断引用循环并阻止悬挂指针                           |
| **可选择的线程安全**） | 虚幻智能指针库包括线程安全代码，可跨线程管理引用计数，如无需线程安全，可用其换取更好性能 |
| **运行时安全**         | 共享引用从不为空，可固定随时取消引用                         |
| **授予意图**           | 可轻松区分对象所有者和观察者                                 |
| **内存**               | 智能指针在64位下仅为C++指针大小的两倍（加上共享的16字节引用控制器），唯一指针除外，其与C++指针大小相同 |





### 4. 助手类和函数



| 助手类            | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| `TSharedFromThis` | 在添加 `AsShared` 或 `SharedThis` 函数的 `TSharedFromThis` 中衍生类，利用此类函数可获取对象的 `TSharedRef` |



| 函数                                             | 描述                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| `MakeShared`<br />`MakeShareable`                | 在常规C++指针中创建共享指针<br />`MakeShared` 会在单个内存块中分配新的对象实例和引用控制器，但要求对象提交公共构造函数`MakeShareable` 的效率较低，但即使对象的构造函数为私有，其仍可运行<br />利用此操作可拥有非自己创建的对象，并在删除对象时支持自定义行为 |
| `StaticCastSharedRef`<br />`StaticCastSharedPtr` | 静态投射效用函数，通常用于向下投射到衍生类型                 |
| `ConstCastSharedRef`<br />`ConstCastSharedPtr`   | 将 `const` 智能引用或智能指针分别转换为 `mutable` 智能引用或智能指针 |





### 5. 智能指针实现细节



注：在功能和效率方面，虚幻智能指针库中的智能指针具有一些共同特征





#### 5.1 运行速度



注：

- 要使用智能指针时，始终考虑性能
- 智能指针非常适合某些高级系统、资源管理或工具编程
- 但部分智能指针类型比原始C++指针更慢，这种开销使得其在低级引擎代码（如渲染）中用处不大



智能指针的部分一般性能优势包括：

- 所有运算均为常量时间
- 取消引用多数智能指针的速度和原始C++指针的相同（在发布版本中）
- 复制智能指针永不会分配内存
- 线程安全智能指针是无锁的



智能指针的性能缺陷包括：

- 创建和复制智能指针比创建和复制原始C++指针需要更多开销
- 保持引用计数增加基本运算的周期
- 部分智能指针占用的内存比原始的C++更多
- 引用控制器有两个堆分配。使用 `MakeShared` 代替 `MakeShareable` 可避免二次分配，并可提高性能





#### 5.2 侵入性访问器



定义：

- 共享指针是非侵入性的，意味对象不知道其是否为智能指针拥有

- 此通常是可以接受的，但在某些情况下，可能要将对象作为共享引用或共享指针进行访问

- 为此，使用对象的类作为模板参数，在 `TSharedFromThis` 衍生对象的类

  

- `TSharedFromThis` 提供两个函数：`AsShared` 和 `SharedThis`，可将对象转换为共享引用（并从共享引用转换为共享指针）

- 使用固定返回共享引用的类factory时，或需将对象传到需要共享引用或共享指针的系统时，此操作十分有用

- `AsShared` 会将类返回为最初作为模板参数传到 `TSharedFromThis` 的类型返回，其可能是调用对象的父类型，而 `SharedThis` 将直接从该类型衍生类型，并返回引用该类型对象的智能指针

  

- 以下范例代码中演示这两种函数：

  ```c++
  class FRegistryObject;
  class FMyBaseClass: public TSharedFromThis<FMyBaseClass>
  {
      virtual void RegisterAsBaseClass(FRegistryObject* RegistryObject)
      {
          // 访问对"this"的共享引用。
          // 直接继承自< TSharedFromThis >，因此AsShared()和SharedThis(this)会返回相同的类型。
          TSharedRef<FMyBaseClass> ThisAsSharedRef = AsShared();
          // RegistryObject需要 TSharedRef<FMyBaseClass>，或TSharedPtr<FMyBaseClass>。TSharedRef可被隐式转换为TSharedPtr.
          RegistryObject->Register(ThisAsSharedRef);
      }
  };
  class FMyDerivedClass : public FMyBaseClass
  {
      virtual void Register(FRegistryObject* RegistryObject) override
      {
          // 并非直接继承自TSharedFromThis<>，因此AsShared()和SharedThis(this)不会返回相同类型。
          // 在本例中，AsShared()会返回在TSharedFromThis<> - TSharedRef<FMyBaseClass>中初始指定的类型。
          // 在本例中，SharedThis(this)会返回具备"this"类型的TSharedRef - TSharedRef<FMyDerivedClass>。
          // SharedThis()函数仅在与 'this'指针相同的范围内可用。
          TSharedRef<FMyDerivedClass> AsSharedRef = SharedThis(this);
          // FMyDerivedClass是FMyBaseClass的一种类型，因此RegistryObject将接受TSharedRef<FMyDerivedClass>。
          RegistryObject->Register(ThisAsSharedRef);
      }
  };
  class FRegistryObject
  {
      // 此函数将接受到FMyBaseClass或其子类的TSharedRef或TSharedPtr。
      void Register(TSharedRef<FMyBaseClass>);
  };
  ```



- 注意：要在构造函数中调用 `AsShared` 或 `Shared`，共享引用此时并未初始化，将导致崩溃或断言





#### 5.3 投射



可通过虚幻智能指针库包含的多个支持函数投射共享指针(和共享引用)

Up-casting是隐式的，与C++指针相同

可使用 `ConstCastSharedPtr` 函数进行常量投射，使用 `StaticCastSharedPtr` 进行静态投射（通常是向下投射到衍生类指针）

无run-type类型的信息（RTTI），因此不支持动态转换；应使用静态投射



如以下代码所示：

```c++
// 假设通过其他方式验证了FDragDropOperation实际为FAssetDragDropOp。
TSharedPtr<FDragDropOperation> Operation = DragDropEvent.GetOperation();
//现在可使用StaticCastSharedPtr进行投射。
TSharedPtr<FAssetDragDropOp> DragDropOp = StaticCastSharedPtr<FAssetDragDropOp>(Operation);
```





#### 5.4 线程安全



通常仅在单线程上访问智能指针的操作才是安全的

如需访问多线程，请使用智能指针类的线程安全版本：

- `TSharedPtr<T, ESPMode::ThreadSafe>`
- `TSharedRef<T, ESPMode::ThreadSafe>`
- `TWeakPtr<T, ESPMode::ThreadSafe>`
- `TSharedFromThis<T, ESPMode::ThreadSafe>`



由于原子引用计数，此类线程安全版本比默认版本稍慢，但其行为与常规C++指针一致：

- 读取和复制固定为线程安全
- 写入和重置须同步后才安全



注：

- 如了解多线程永不访问指针，可通过避免使用线程安全版本获得更好性能





### 6. 提示和限制



- 避免将数据作为 `TSharedRef` 或 `TSharedPtr` 参数传到函数，此操作将因取消引用和引用计数而产生开销

- 相反，建议将引用对象作为 `const &` 进行传递

  

- 可将共享指针向前声明为不完整类型

  

- 共享指针与虚幻对象(`UObject` 及其衍生类)不兼容

- 引擎具有 `UObject` 管理的单独内存管理系统（[对象处理](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Objects/Optimizations)文档），两个系统未互相重叠







### 7. 共享指针



定义：

- 一种支持共享拥有、自动失效、弱引用等特性的智能指针
- **共享指针（Shared Pointers）** 是指既健壮、又能为空指针的智能指针
- 共享指针沿袭了普通智能指针的所有优点，它能避免出现内存泄漏、悬挂指针，还能避免指针指向未初始化的内存



其他特点：

- **共享所有权（Shared Ownership）：** 引用计数支持多个共享指针，以确保它们引用的数据对象永远不被删除，前提是它们中的任意一个仍指向数据对象
- **自动失效（Automatic Invalidation）：** 你可安全引用易变对象，无需担心出现悬挂指针
- **弱引用：** [弱指针](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/SmartPointerLibrary/WeakPointer)可中断引用循环
- **意向指示（Indication of Intent）：** 区分拥有者（参见[共享引用](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/SmartPointerLibrary/SharedReference)）和观察者，并提供不可为空的引用



其他特性：

- 语法非常健壮
- 非侵入式（但能反射）
- 线程安全（视情况而定）
- 性能佳，占用内存少



注意：

- 共享指针类似于[共享引用](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/SmartPointerLibrary/SharedReference)，主要区别在于共享引用不可为空，因此会始终引用有效对象
- 在共享引用和共享指针之间进行选择时，除非需要空对象或可为空的对象，否则建议你优先选择共享引用





#### 7.1 声明和初始化



- 因为共享指针可为空，所以无论有无数据对象，都可以对它们进行初始化

- 创建共享指针的一些示例：

  ```c++
  // 创建一个空白的共享指针
  TSharedPtr EmptyPointer;
  // 为新对象创建一个共享指针
  TSharedPtr<FMyObjectType> NewPointer(new FMyObjectType());
  // 从共享引用创建一个共享指针
  TSharedRef<FMyObjectType> NewReference(new FMyObjectType());
  TSharedPtr<FMyObjectType> PointerFromReference = NewReference;
  // 创建一个线程安全的共享指针
  TSharedPtr<FMyObjectType, ESPMode::ThreadSafe> NewThreadsafePointer = MakeShared<FMyObjectType, ESPMode::ThreadSafe>(MyArgs);
  ```

- 在第二个示例中，`NodePtr` 实际上拥有新的 `FMyObjectType` 对象，因为没有其他共享指针引用该对象
- 如果 `NodePtr` 超出范围，并且没有其他共享指针或共享引用指向该对象，那么该对象将被销毁



- 复制共享指针时，系统将向它引用的对象添加一个引用

  ```c++
  // 增加任意对象ExistingSharedPointer引用的引用数。
  TSharedPtr<FMyObjectType> AnotherPointer = ExistingSharedPointer;
  ```

- 对象将持续存在，直到不再有共享指针（或共享引用）引用它为止



- 可以使用 `Reset` 函数、或分配一个空指针来重设共享指针，如下所示：

  ```c++
  PointerOne.Reset();
  PointerTwo = nullptr;
  // PointerOne和PointerTwo现在都引用nullptr。
  ```



- 使用 `MoveTemp`（或 `MoveTempIfPossible`）函数将一个共享指针的内容转移到另一个共享指针，将原始的共享指针保留为空：

  ```c++
  // 将PointerOne的内容移至PointerTwo。在此之后，PointerOne将引用nullptr。
  PointerTwo = MoveTemp(PointerOne);
  // 将PointerTwo的内容移至PointerOne。在此之后，PointerTwo将引用nullptr。
  PointerOne = MoveTempIfPossible(PointerTwo);
  ```

- 注意：`MoveTemp` 和 `MoveTempIfPossible` 的唯一不同之处在于 `MoveTemp` 包含静态断言，强制其只能在非常量左值（lvalue）上执行





#### 7.2 共享指针与引用转换



- 在共享指针与共享引用之间进行转换是一种常见的做法。共享引用隐式地转换为共享指针，并提供新的共享指针将引用有效对象的额外保证

- 转换由普通语法处理：

  ```c++
  TSharedPtr<FMyObjectType> MySharedPointer = MySharedReference;
  ```



- 只要共享指针引用了一个非空对象，你就可以使用 `Shared Pointer` 函数 `ToSharedRef` 从此共享指针创建一个共享引用

- 试图从空共享指针创建共享引用将导致程序断言

  ```c++
  // 在解引用之前，请确保共享指针有效，以避免可能出现的断言。
  if (MySharedPointer.IsValid())
  {
      MySharedReference = MySharedPointer.ToSharedRef();
  }
  ```





#### 7.3 对比



- 目的：测试共享指针彼此间的相等性

  ```c++
  TSharedPtr<FTreeNode> NodeA, NodeB;
  if (NodeA == NodeB)
  {
      // ...
  }
  ```



- `IsValid` 函数和 `bool` 运算符有助于判断共享指针是否引用了有效对象

- 可以调用 `Get`，查看它是否返回有效的（非空）对象指针

  ```c++
  if (Node.IsValid())
  {
      // ...
  }
  if (Node)
  {
      // ...
  }
  if (Node.Get() != nullptr)
  {
      // ...
  }
  ```

  



#### 7.4 解引用和访问



说明：

- 像使用普通C++指针那样解引用，调用方法和访问成员
- 可以像使用其他C++指针那样，通过调用 `IsValid` 函数或使用重载的 `bool` 运算符，在取消引用之前执行空检查



```c++
// 在解引用前，检查节点是否引用了一个有效对象。
if (Node)
{
    // 以下三行代码中的任意一行都能解引用节点，并且对它的对象调用ListChildren：
    Node->ListChildren();
    Node.Get()->ListChildren();
    (*Node).ListChildren();
}
```







#### 7.5 自定义删除器



- 说明：共享指针和共享引用支持对它们引用的对象使用自定义删除器



- 如需运行自定义删除代码，请提供lambda函数，作为创建智能指针时使用的参数，就像这样：

  ```c++
  void DestroyMyObjectType(FMyObjectType* ObjectAboutToBeDeleted)
  {
      // 此处添加删除代码。
  }
  // 这些函数使用自定义删除器创建指南指针。
  TSharedRef<FMyObjectType> NewReference(new FMyObjectType(), [](FMyObjectType* Obj){ DestroyMyObjectType(Obj); });
  TSharedPtr<FMyObjectType> NewPointer(new FMyObjectType(), [](FMyObjectType* Obj){ DestroyMyObjectType(Obj); });
  ```







### 8. 共享引用



- 不能为未初始化或分配为空的智能指针类型



说明：

- **共享引用** 是一类强大且不可为空的 **智能指针**，其被用于引擎的 `Uobject` 系统外的数据对象
- 此意味无法重置共享引用、向其指定空对象，或创建空白引用
- 因此共享引用固定包含有效对象，甚至未拥有 `IsValid` 方法
- 在共享引用和 **[共享指针]（Shared Pointers）** 间选择时，除非需要空白或可为空的对象，否则共享引用为优先选项
- 如需可能空白或可为空的引用，则应使用共享指针



注意：与标准的C++引用不同，可在创建后将共享引用重新指定到另一对象





#### 8.1 声明和初始化



- 共享引用不可为空，因此初始化需要数据对象

- 在无有效对象的情况下尝试创建的共享引用将不会编译，并尝试将共享引用初始化为空指针变量

  ```c++
  //创建新节点的共享引用
  TSharedRef<FMyObjectType> NewReference = MakeShared<FMyObjectType>();
  ```



- 在无有效对象的情况下尝试创建的共享引用将不会编译：

  ```c++
  //以下两者均不会编译：
  TSharedRef<FMyObjectType> UnassignedReference;
  TSharedRef<FMyObjectType> NullAssignedReference = nullptr;
  //以下会编译，但如NullObject实际为空则断言。
  TSharedRef<FMyObjectType> NullAssignedReference = NullObject;
  ```





#### 8.2 共享指针与引用转换



说明：共享引用会隐式转换为共享指针，并为新共享指针引用有效对象提供额外保证



- 使用普通语法处理转换：

  ```c++
  TSharedPtr<FMyObjectType> MySharedPointer = MySharedReference;
  ```



- 如共享指针引用非空对象，即可使用 `共享指针` 函数 `ToSharedRef`，在共享指针中创建共享引用

- 尝试在空共享指针中创建共享引用，将导致程序断言

  ```c++
  //在取消引用前，确保共享指针为有效，避免潜在断言。
  If (MySharedPointer.IsValid())
  {
      MySharedReference = MySharedPointer.ToSharedRef();
  }
  ```





#### 8.3 对比



- 可测试共享引用彼此是否相等。在此情况下，相等表示引用相同对象

  ```c++
  TSharedRef<FMyObjectType> ReferenceA, ReferenceB;
  if (ReferenceA == ReferenceB)
  {
      // ...
  }
  ```






### 9. 弱指针



- 存储弱引用且不阻止其对象被销毁的智能指针



说明：

- **弱指针** 存储对象的弱引用
- 与 **共享指针** 或 **共享引用** 不同，弱指针不会阻止其引用的对象被销毁

- 在访问弱指针引用的对象前，应使用 `Pin` 函数生成共享指针
- 此操作确保使用该对象时其将继续存在
- 如只需要确定弱指针是否引用对象，可将其与 `nullptr` 比较，或在之上调用 `IsValid`



注意：

- 弱指针的使用有助于授予意图——弱指针表明对引用对象的观察，而无需所有权，同时不控制其生命周期





#### 9.1 声明初始化和分配



- 可创建空白弱指针，或在共享引用、共享指针或其他弱指针中进行

  ```c++
  //分配新的数据对象，并创建对其的强引用。
  TSharedRef<FMyObjectType> ObjectOwner = MakeShared<FMyObjectType>();
  //创建指向新数据对象的弱指针。
  TWeakPtr<FMyObjectType> ObjectObserver(ObjectOwner);
  ```



- 弱指针不会阻止对象被销毁

- 无论 `ObjectOwner` 是否在范围内，重置 `ObjectOwner` 都将销毁对象：

  ```c++
  //假设ObjectOwner是其对象的唯一拥有者，ObjectOwner停止引用该对象时，该对象将被销毁。
  ObjectOwner.Reset();
  //ObjectOwner引用空对象，因此Pin()生成的共享指针将也将为空。被视为布尔时，空白共享指针的值为false。
  if (ObjectObserver.Pin())
  {
      //只当ObjectOwner非对象的唯一拥有者时，此代码才会运行。
      check(false);
  }
  ```



- 与共享指针相同，弱指针是否引用有效对象，均可进行安全复制：

  ```c++
  TWeakPtr<FMyObjectType> AnotherObjectObserver = ObjectObserver;
  ```



- 使用完弱指针后，可进行重置

  ```c++
  //可通过将弱指针设为nullptr进行重置。
  ObjectObserver = nullptr;
  //也可使用重置函数。
  AnotherObjectObserver.Reset();
  ```







#### 9.2 转换为共享指针



说明：

- `Pin` 函数将创建指向弱指针对象的共享指针
- 只要共享指针在范围内且引用对象，则该对象将持续有效
- 此外，共享指针（包括由 `Pin` 函数返回的指针）可在条件句中作为 `布尔` 类型进行求值，其中 `true` 表示有效对象
- 以下代码模式检查弱指针是否引用有效对象
- 如是，至少在共享指针（由 `Pin` 函数创建）超出范围或被显式清除前，将保证其持续有效



```c++
//获取弱指针中的共享指针，并检查其是否引用有效对象。
if (TSharedPtr<FMyObjectType> LockedObserver = ObjectObserver.Pin())
{
    //共享指针仅在此范围内有效。
    //该对象已被验证为存在，而共享指针阻止其被删除。
    LockedObserver->SomeFunction();
}
```







#### 9.3 取消引用和访问



- 要访问弱指针的对象，首需使用 `Pin` 函数，将其提升为共享指针
- 然后可通过共享指针或弱指针上的 `Get` 函数进行访问
- 此方法可确保使用该对象时，其将持续有效







#### 9.4 打破引用循环



- 两个或多个对象使用智能指针保持彼此间的强引用时，将出现引用循环
- 在此类情况下，对象间会相互保护以免被删除
- 各对象固定被另一对象引用，因此对象无法在另一对象存在时被删除
- 如外部对象未对引用循环中对象进行引用，其实际上将出现泄漏
- 弱指针不会保留自身引用的对象，因此其可中断此类引用循环
- 要在未拥有对象时对其进行引用，并延长其寿命时，可使用软指针







#### 9.5 使用警告



如不想保证数据对象会持续存在时，弱指针将非常有用，但该属性可能会变得异常危险

在以下情况中请谨慎使用弱指针：

- 在[集](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/TSet)或[映射](https://docs.unrealengine.com/4.27/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/TMap)中用作键。弱指针可能会在未通知容器的情况下随时无效，因此共享指针或共享引用更适用于充当键，可安全地将弱指针用作数值
- 虽然弱指针提供 `IsValid` 函数，但是检查 `IsValid` 无法保证对象在任何时间长度内均可持续有效
- 线程安全共享指针可能会因另一线程上的活动而随时无效，因此使用线程安全共享指针应尤其注意
- `Pin` 返回的共享指针将使对象在代码将其清除或其超出范围前保持活跃状态，因此 `Pin` 函数是用于检查的首选方法，此类检查会导致取消引用或访问存储对象

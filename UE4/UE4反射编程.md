# UE4 反射编程



[toc]



### 1. 准备工作



1. 新建空白`ue4 cpp`项目：`Reflective`

2. 打开`ReflectiveGameModeBase.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "GameFramework/GameModeBase.h"
   #include "ReflectiveGameModeBase.generated.h"
   
   UCLASS()
   class REFLECTIVE_API ReflectiveGameModeBase : public AGameModeBase
   {
   	GENERATED_UCLASS_BODY()
   	
   };
   ```

3. 修改`ReflectiveGameModeBase.cpp`，创建构造函数

   ```c++
   #include "ReflectiveGameModeBase.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Reflection Succeed
   	UE_LOG(LogTemp, Log, TEXT("[ Hello Reflective ]"));
   }
   ```

4. 编译运行：

   ```powershell
   LogTemp: [ Hello Reflective ]
   ```





### 2. 获取类名



1. 创建`UE4 None类型 cpp文件`：`Student`

2. `Student.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   
   class REFLECTIVE_API Student
   {
   public:
   	Student();
       ~Student();
   }
   ```

3. `Student.cpp`

   ```c++
   #include "Student.h"
   
   Student::Student()
   {
   }
   
   Student::~Student()
   {
   }
   ```

4. 需要进行修改，才能拥有虚幻的反射功能

5. `Student.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "Student.generated.h" // 1. #include "文件名.generated.h"
   
   UCLASS() // 2. UCLASS() 是虚幻提供的类反射
   class REFLECTIVE_API UStudent : public UObject // 3. 需要继承UObject才能使用UCALSS(), 类名前面要加U
   {
   	GENERATED_BODY() // 4. 要加入 GENERATED_BODY()
   public:
   	UStudent(); // 5. 类名前要统一加U
       // 6. 因为继承UObject，不需要考虑垃圾回收，~Student()不需要
   };
   ```

6. `Student.cpp`

   ```c++
   #include "Student.h"
   
   UStudent::UStudent() // 1. 类名前加 U
   {
   }
   
   // 2. 不需要 Student::~Student(){}
   ```

7. 开始获取类型

8. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Get Class Name By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Class Name ]"));
   	UStudent *Student = NewObject<UStudent>();
   	const UClass *StudentClass = Student->GetClass();
   	const FName StudentName = StudentClass->GetFName();
   	UE_LOG(LogTemp, Warning, TEXT("<-- Class Name: %s -->"), *StudentName.ToString());
   }
   ```

9. 打印结果：

   ```powershell
   LogTemp: [ Get Class Name ]
   LogTemp: Warning: <-- Class Name: Student -->
   ```







### 3. 获取类标签



1. `Student.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "Student.generated.h"
   
   UCLASS(BlueprintType) // 此次填入 BlueprintType
   class REFLECTIVE_API UStudent : public UObject
   {
   	GENERATED_BODY()
   public:
   	UStudent();
   };
   ```

2. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
       UStudent *Student = NewObject<UStudent>();
   	const UClass *StudentClass = Student->GetClass();
   	const FName StudentName = StudentClass->GetFName();
   
   	// Get Class Flags By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Class Flags ]"));
   	EClassFlags StudentClassFlags = StudentClass->ClassFlags;
   	UE_LOG(LogTemp, Warning, TEXT("<-- Class Name: %s, Class Flags: %x -->"),
   		*StudentName.ToString(), StudentClassFlags);
   }
   ```

3. 打印结果

   ```powershell
   LogTemp: [ Get Class Flags ]
   LogTemp: Warning: <-- Class Name: Student, Class Flags: 305000a0 -->
   ```

   



### 4. 获取类属性



1. `Student.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "Student.generated.h"
   
   UCLASS(BlueprintType)
   class REFLECTIVE_API UStudent : public UObject
   {
   	GENERATED_BODY()
   public:
   	UStudent();
       
   private: // 添加属性，要加入UPROPERTY()，否则无法参与反射
   	UPROPERTY()
   	FString Name;
   
   	UPROPERTY()
   	FString Country;
   };
   ```

2. `Student.cpp`

   ```c++
   #include "Student.h"
   
   UStudent::UStudent()
   {
   	Name = "FHang";
   	Country = "China";
   }
   ```

3. ``ReflectiveGameModeBase.cpp``

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
       UStudent *Student = NewObject<UStudent>();
   	const UClass *StudentClass = Student->GetClass();
   
   	// Get Class Property Value By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Class Property Value ]"))
   	for (FProperty *Property = StudentClass->PropertyLink; Property; Property = Property->PropertyLinkNext)
   	{
   		FString PropertyName = Property->GetName();
   		FString PropertyType = Property->GetCPPType();
   		
   		if (PropertyType == "FString")
   		{
   			const FStrProperty *StringProperty = CastField<FStrProperty>(Property);
   			void *Address = StringProperty->ContainerPtrToValuePtr<void>(Student);
   			FString PropertyValue = StringProperty->GetPropertyValue(Address);
   			UE_LOG(LogTemp, Warning, TEXT("<-- Class Property: %s, Type: %s, Value: %s-->"),
   				*PropertyName, *PropertyType, *PropertyValue);
   		}
   	}
   }
   ```

4. 打印结果：

   ```powershell
   LogTemp: [ Get Class Property Value ]
   LogTemp: Warning: <-- Class Property: Name, Type: FString, Value: FHang -->
   LogTemp: Warning: <-- Class Property: Country, Type: FString, Value: China -->
   ```





### 5. 获取类属性元数据



1. `Student.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "Student.generated.h"
   
   UCLASS(BlueprintType)
   class REFLECTIVE_API UStudent : public UObject
   {
   	GENERATED_BODY()
   public:
   	UStudent();
       
   private:
   	UPROPERTY(VisibleAnywhere, Category="Info") // 添加元数据
   	FString Name;
   
   	UPROPERTY()
   	FString Country;
   };
   ```

2. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
       UStudent *Student = NewObject<UStudent>();
   	const UClass *StudentClass = Student->GetClass();
   
   	// Get Class Property Value By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Class Property Value ]"))
   	for (FProperty *Property = StudentClass->PropertyLink; Property; Property = Property->PropertyLinkNext)
   	{
           // Get Class Property Meta By Reflection
   		UE_LOG(LogTemp, Log, TEXT("[ Get Class Property Meta ]"))
   		FString PropertyMeta = Property->GetMetaData(TEXT("Category"));
   		
   		if (PropertyType == "FString")
   		{
   			const FStrProperty *StringProperty = CastField<FStrProperty>(Property);
   			void *Address = StringProperty->ContainerPtrToValuePtr<void>(Student);
   			FString PropertyValue = StringProperty->GetPropertyValue(Address);
   			UE_LOG(LogTemp, Warning, TEXT("<-- Class Property Meta: %s -->"), *PropertyMeta);
   		}
   	}
   }
   ```

3. 打印结果：

   ```powershell
   LogTemp: [ Get Class Property Value ]
   LogTemp: [ Get Class Property Meta ]
   LogTemp: Warning: <-- Class Property Meta: Info -->
   LogTemp: Warning: <-- Class Property Meta:  -->
   ```

   





### 6. 设置类属性值



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
       UStudent *Student = NewObject<UStudent>();
   	const UClass *StudentClass = Student->GetClass();
   
   	// Get Class Property Value By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Class Property Value ]"))
   	for (FProperty *Property = StudentClass->PropertyLink; Property; Property = Property->PropertyLinkNext)
   	{
   		FString PropertyName = Property->GetName();
   		FString PropertyType = Property->GetCPPType();
   		
   		if (PropertyType == "FString")
   		{
   			const FStrProperty *StringProperty = CastField<FStrProperty>(Property);
   			void *Address = StringProperty->ContainerPtrToValuePtr<void>(Student);
   			FString PropertyValue = StringProperty->GetPropertyValue(Address);
   			UE_LOG(LogTemp, Warning, TEXT("<-- Class Property: %s, Type: %s, Value: %s -->"),
   				*PropertyName, *PropertyType, *PropertyValue);
   			
   			// Set Class Property Value By Reflection
   			UE_LOG(LogTemp, Log, TEXT("[ Set Class Property Value ]"))
   			StringProperty->SetPropertyValue(Address, "XXXX");
   			FString NewStringProperty = StringProperty->GetPropertyValue(Address);
   			UE_LOG(LogTemp, Warning, TEXT("<-- Class Property: %s, Type: %s, Value: %s -->"),
   				*PropertyName, *PropertyType, *NewStringProperty);
   		}
   	}
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Get Class Property Value ]
   LogTemp: Warning: <-- Class Property: Name, Type: FString, Value: FHang -->
   LogTemp: [ Set Class Property Value ]
   LogTemp: Warning: <-- Class Property: Name, Type: FString, Value: XXXX -->
   LogTemp: Warning: <-- Class Property: Country, Type: FString, Value: China -->
   LogTemp: [ Set Class Property Value ]
   LogTemp: Warning: <-- Class Property: Country, Type: FString, Value: XXXX -->
   ```







### 7. 获得类函数名



1. `Student.h`

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "Student.generated.h"
   
   UCLASS(BlueprintType)
   class REFLECTIVE_API UStudent : public UObject
   {
   	GENERATED_BODY()
   public:
   	UStudent();
   
   private:
   	UPROPERTY(VisibleAnywhere, Category="Info")
   	FString Name;
   
   	UPROPERTY()
   	FString Country;
   
   public:
   	UFUNCTION(BlueprintCallable)
   	void Study(){};
   
   	UFUNCTION()
   	void Demo01(int a, bool isA){};
   };
   ```

2. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
       UStudent *Student = NewObject<UStudent>();
   	const UClass *StudentClass = Student->GetClass();
       const FName StudentName = StudentClass->GetFName();
   	
       // Get Class Function Name By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Class Function ]"));
   	for (TFieldIterator<UFunction> IteratorOfFunction(StudentClass); IteratorOfFunction; ++IteratorOfFunction)
   	{
   		const UFunction *Function = *IteratorOfFunction;
   		FString FunctionName = Function->GetName();
   		if (FunctionName == "ExecuteUbergraph"){continue;}
   
   		UE_LOG(LogTemp, Warning, TEXT("<-- Class: %s, Function: %s -->"), *StudentName.ToString(),
   			*FunctionName);
       }
   }
   ```

3. 打印结果：

   ```powershell
   LogTemp: [ Get Class Function ]
   LogTemp: Warning: <-- Class: Student, Function: Study -->
   LogTemp: Warning: <-- Class: Student, Function: Demo01 -->
   ```







### 8. 获取类函数标签



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
       UStudent *Student = NewObject<UStudent>();
   	const UClass *StudentClass = Student->GetClass();
       const FName StudentName = StudentClass->GetFName();
   	
       // Get Class Function Name By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Class Function ]"));
   	for (TFieldIterator<UFunction> IteratorOfFunction(StudentClass); IteratorOfFunction; ++IteratorOfFunction)
   	{
   		const UFunction *Function = *IteratorOfFunction;
   		FString FunctionName = Function->GetName();
   		if (FunctionName == "ExecuteUbergraph"){continue;}
           		
           // Get Class Function Flags
   		UE_LOG(LogTemp, Log, TEXT("[ Get Class Function Flags ]"))
   		EFunctionFlags FunctionFlags = Function->FunctionFlags;
   		UE_LOG(LogTemp, Warning, TEXT("<-- Class: %s, Function: %s, Flags: %x -->"), *StudentName.ToString(),
   			*FunctionName, FunctionFlags);
       }
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Get Class Function ]
   LogTemp: [ Get Class Function Flags ]
   LogTemp: Warning: <-- Class: Student, Function: Study, Flags: 4020401 -->
   LogTemp: [ Get Class Function Flags ]
   LogTemp: Warning: <-- Class: Student, Function: Demo01, Flags: 20401 -->
   ```





### 9. 获取类函数参数



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
       UStudent *Student = NewObject<UStudent>();
   	const UClass *StudentClass = Student->GetClass();
   	
       // Get Class Function Name By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Class Function ]"));
   	for (TFieldIterator<UFunction> IteratorOfFunction(StudentClass); IteratorOfFunction; ++IteratorOfFunction)
   	{
   		const UFunction *Function = *IteratorOfFunction;
   		FString FunctionName = Function->GetName();
   		if (FunctionName == "ExecuteUbergraph"){continue;}
           		
           // Get Class Function Params By Reflection
   		UE_LOG(LogTemp, Log, TEXT("[ Get Function Params ]"));
   		for (TFieldIterator<FProperty> IteratorOfParams(Function); IteratorOfParams; ++IteratorOfParams)
   		{
   			const FProperty *Param = *IteratorOfParams;
   			FString ParamType = Param->GetCPPType();
   			FString ParamName = Param->GetName();
   			UE_LOG(LogTemp, Warning, TEXT("<-- Function: %s, ParamType: %s, ParamName: %s -->"),
   				*FunctionName, *ParamType, *ParamName);
   		}
       }
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Get Class Function ]
   LogTemp: [ Get Function Params ]
   LogTemp: Warning: <-- Function: Demo01, ParamType: int32, ParamName: a -->
   LogTemp: Warning: <-- Function: Demo01, ParamType: bool, ParamName: isA -->
   ```





### 10. 获取类函数参数标签



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
       UStudent *Student = NewObject<UStudent>();
   	const UClass *StudentClass = Student->GetClass();
   	
       // Get Class Function Name By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Class Function ]"));
   	for (TFieldIterator<UFunction> IteratorOfFunction(StudentClass); IteratorOfFunction; ++IteratorOfFunction)
   	{
   		const UFunction *Function = *IteratorOfFunction;
   		FString FunctionName = Function->GetName();
   		if (FunctionName == "ExecuteUbergraph"){continue;}
           		
           // Get Class Function Params By Reflection
   		UE_LOG(LogTemp, Log, TEXT("[ Get Function Params ]"));
   		for (TFieldIterator<FProperty> IteratorOfParams(Function); IteratorOfParams; ++IteratorOfParams)
   		{
   			const FProperty *Param = *IteratorOfParams;
   
               // Get Function Params Flags By Reflection
   			UE_LOG(LogTemp, Log, TEXT("[ Get Params Flags ]"));
   			EPropertyFlags ParamFlag = Param->GetPropertyFlags();
   			UE_LOG(LogTemp, Warning, TEXT("<-- Function: %s, Flags: %x -->"),
   				*FunctionName, ParamFlag);
   		}
       }
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Get Class Function ]
   LogTemp: [ Get Function Params ]
   LogTemp: [ Get Params Flags ]
   LogTemp: Warning: <-- Function: Demo01, ParamType: int32, ParamName: a, Flags: 40000280 -->
   LogTemp: [ Get Params Flags ]
   LogTemp: Warning: <-- Function: Demo01, ParamType: bool, ParamName: isA, Flags: 40000280 -->
   ```







### 11. 获取父类



1. 新建`Student`子类`SubStudent`

2. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "SubStudent.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Get SubClass FatherClass By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get FatherClass ]"));
   	const USubStudent *SubStudent = NewObject<USubStudent>();
   	const UClass *FatherClass = SubStudent->GetClass()->GetSuperClass();
   	const FString SubClassName = SubStudent->GetClass()->GetName();
   	const FString FatherClassName = FatherClass->GetName();
   	UE_LOG(LogTemp, Warning, TEXT("<-- SubClass: %s, SuperClass: %s -->"), *SubClassName, *FatherClassName);
   }
   ```

3. 打印结果：

   ```powershell
   LogTemp: [ Get FatherClass ]
   LogTemp: Warning: <-- SubClass: SubStudent, SuperClass: Student -->
   ```





### 12. 判断是否是子类



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   #include "SubStudent.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Determine if the current class is a subclass of another class By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Determine Is SubClass ]"));
   	const UClass *Class1 = UStudent::StaticClass();
   	const UClass *Class2 = USubStudent::StaticClass();
   	const UClass *Class3 = AActor::StaticClass();
   	if (Class2->IsChildOf(Class1))
   	{
   		UE_LOG(LogTemp, Warning, TEXT("<-- %s Is %s SubClass -->"), 
   			*Class2->GetName(), *Class1->GetName());
   	}
   	if (!Class3->IsChildOf(Class1))
   	{
   		UE_LOG(LogTemp, Warning, TEXT("<-- %s Is Not %s SubClass -->"),
   			*Class3->GetName(), *Class1->GetName());
   	}
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Determine Is SubClass ]
   LogTemp: Warning: <-- SubStudent Is Student SubClass -->
   LogTemp: Warning: <-- Actor Is Not Student SubClass -->
   ```





### 13. 查找类的所有子类



1. 新建`Student`子类`Sub1Student`

2. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	const *NewStudent = NewObject<UStudent>();
   	const UClass *FatherClass = NewStudent->GetClass();
   	
   	// Find Current Class All Of SubClass By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Current Class All Of SubClass ]"));
   	TArray<UClass*> ClassArray;
   	GetDerivedClasses(FatherClass, ClassArray, false);
   	UE_LOG(LogTemp, Warning, TEXT("<-- SuperClass: %s -->"), *FatherClassName);
   	for (const auto &Elem : ClassArray)
   	{
   		UE_LOG(LogTemp, Warning, TEXT("<-- SubClass: %s -->"), *Elem->GetName());
   	}
   }
   ```

3. 打印结果：

   ```powershell
   LogTemp: [ Get Current Class All Of SubClass ]
   LogTemp: Warning: <-- SuperClass: Student -->
   LogTemp: Warning: <-- SubClass: Sub1Student -->
   LogTemp: Warning: <-- SubClass: SubStudent -->
   ```







### 14. 查找类生成的所有对象



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "Sub1Student.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Get Current Class Generated Objects By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get Current Class Generated Objects ]"));
   	TArray<UObject*> ObjectsArray;
   	USub1Student *Sub1Student = NewObject<USub1Student>(this, FName("Sub1Student"));
   	USub1Student *Sub1 = NewObject<USub1Student>(this, FName("Sub1"));
   	GetObjectsOfClass(USub1Student::StaticClass(), ObjectsArray, false);
   	UE_LOG(LogTemp, Warning, TEXT("<-- Current Class: %s -->"), *Sub1Student->GetName());
   	for (const auto &Elem : ObjectsArray)
   	{
   		UE_LOG(LogTemp, Warning, TEXT("<-- Object: %s -->"), *Elem->GetName());
   	}
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Get Current Class Generated Objects ]
   LogTemp: Warning: <-- Current Class: Sub1Student -->
   LogTemp: Warning: <-- Object: Sub1Student -->
   LogTemp: Warning: <-- Object: Sub1 -->
   ```







### 15. 通过字符串查找类



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Use String Find Class By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Use String Find Class ]"));
   	UClass *FindedClass = FindObject<UClass>(ANY_PACKAGE, *FString("Student"), true);
   	if (FindedClass)
   	{
   		UE_LOG(LogTemp, Warning, TEXT("<-- Find %s Succeed -->"), *FindedClass->GetName());
   	}
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Use String Find Class ]
   LogTemp: Warning: <-- Find Student Succeed -->
   ```







### 16. 通过字符查找枚举



1. `Student.h`中定义`EStudentType`枚举

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "Student.generated.h"
   
   UENUM()
   enum class EStudentType : uint8
   {
   	E_GOOD UMETA(DisplayName = "GOOD"),
   	E_BAD UMETA(DisplayName = "BAD")
   };
   
   UCLASS(BlueprintType)
   class REFLECTIVE_API UStudent : public UObject
   {
   	GENERATED_BODY()
   public:
   	UStudent();
   
   private:
   	UPROPERTY(VisibleAnywhere, Category="Info")
   	FString Name;
   
   	UPROPERTY()
   	FString Country;
   
   public:
   	UFUNCTION(BlueprintCallable)
   	void Study(){};
   
   	UFUNCTION()
   	void Demo01(int a, bool isA){};
   };
   ```

2. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Use String Find Enum By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Use String Find Enum ]"));
   	UEnum *FindedEnum = FindObject<UEnum>(ANY_PACKAGE, *FString("EStudentType"), true);
   	if (FindedEnum)
   	{
   		UE_LOG(LogTemp, Warning, TEXT("<-- Find %s Succeed -->"), *FindedEnum->GetName());
   	}
   }
   ```

3. 打印结果：

   ```powershell
   LogTemp: [ Use String Find Enum ]
   LogTemp: Warning: <-- Find EStudentType Succeed -->
   ```







### 17. 获得枚举的所有项



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Use String Find Enum By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Use String Find Enum ]"));
   	UEnum *FindedEnum = FindObject<UEnum>(ANY_PACKAGE, *FString("EStudentType"), true);
   	if (FindedEnum)
   	{        		
           // Get Current Enum All Of Elements By Reflection
   		UE_LOG(LogTemp, Log, TEXT("[ Get Current Enum All Of Elements ]"));
   		UE_LOG(LogTemp, Warning, TEXT("<-- Enum: %s -->"), *FindedEnum->GetName());
   		for (int8 Index = 0; Index < FindedEnum->NumEnums(); ++Index)
   		{
   			UE_LOG(LogTemp, Warning, TEXT("Elem: %s"), *FindedEnum->GetNameStringByIndex(Index));
   		}
   	}
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Get Current Enum All Of Elements ]
   LogTemp: Warning: <-- Enum: EStudentType -->
   LogTemp: Warning: Elem: E_GOOD
   LogTemp: Warning: Elem: E_BAD
   LogTemp: Warning: Elem: E_MAX
   ```







### 18. 通过字符串查找蓝图类



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Use String Find Blueprint Class
   	UE_LOG(LogTemp, Log, TEXT("[ Use String Find Blueprint Class ]"));
   	UBlueprint *FindedBlueprint = FindObject<UBlueprint>(ANY_PACKAGE, *FString("BP_Student"));
   	if (FindedBlueprint)
   	{
   		UE_LOG(LogTemp, Warning, TEXT("<-- Find %s Succeed -->"), *FindedBlueprint->GetName());
   	}
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Use String Find Blueprint Class ]
   LogTemp: <-- Find BP_Student Succeed -->
   ```







### 19. 判断蓝图是否是Native



1. 在`UE`中通过`Student.cpp`类新建`BP_Student`蓝图类

2. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Use String Find Blueprint Class
   	UE_LOG(LogTemp, Log, TEXT("[ Use String Find Blueprint Class ]"));
   	UBlueprint *FindedBlueprint = FindObject<UBlueprint>(ANY_PACKAGE, *FString("BP_Student"));
   	if (FindedBlueprint)
   	{
   		// Determine Is BlueprintClass Or Native(Cpp) Class By Reflection
   		UE_LOG(LogTemp, Log, TEXT("[ Determine Is BlueprintClass Or Native ]"));
   		if (!FindedBlueprint->IsNative())
   		{
   			UE_LOG(LogTemp, Warning, TEXT("<-- %s Is Blueprint Class -->"), *FindedBlueprint->GetName());
   		}
   		else
   		{
   			UE_LOG(LogTemp, Warning, TEXT("<-- %s Is Native Class -->"), *FindedBlueprint->GetName());
   		}
   	}
   }
   ```

3. 打印结果：

   ```powershell
   LogTemp: [ Use String Find Blueprint Class ]
   LogTemp: [ Determine Is BlueprintClass Or Native ]
   LogTemp: <-- BP_Student Is Blueprint Class -->
   ```







### 20. 获取所有类



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Get All Class By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Get All Class ]"));
   	TArray<FString> AllClassNames;
   	for (TObjectIterator<UClass> ClassIt; ClassIt; ++ClassIt)
   	{
   		FString ClassName = ClassIt->GetName();
   		AllClassNames.Emplace(ClassName);
   	}
   	for (const auto &Elem : AllClassNames)
   	{
   		if (Elem == "Student")
   			UE_LOG(LogTemp, Warning, TEXT("<-- %s -->"), *Elem);
   	}
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Get All Class ]
   LogTemp: Warning: <-- Student -->
   ```







### 21. 通过字符串查找类函数



1. `SubStudent.h`定义函数

   ```c++
   #pragma once
   
   #include "CoreMinimal.h"
   #include "Student.h"
   #include "SubStudent.generated.h"
   
   UCLASS()
   class REFLECTIVE_API USubStudent : public UStudent
   {
   	GENERATED_BODY()
   
   public:
   	UFUNCTION()
   	void PlayGame(FString GameName);
   
   	UFUNCTION()
   	int IsBoy();
   };
   ```

2. `SubStudent.cpp`实现函数

   ```c++
   #include "SubStudent.h"
   
   void USubStudent::PlayGame(FString GameName)
   {
   	UE_LOG(LogTemp, Warning, TEXT("<-- SubStudent::PlayGame(FString)>> Play %s -->"), *GameName);
   }
   
   int USubStudent::IsBoy()
   {
   	UE_LOG(LogTemp, Warning, TEXT("<-- SubStudent::IsBoy>> Is Boy -->"));
   	return 1;
   }
   ```

3. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "USubStudent.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Use String Find Class Function By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Use String Find Class Function ]"));
   	USubStudent *SubStudent01 = NewObject<USubStudent>();
   	if (UClass *SubStudent01Class = SubStudent01->GetClass())
   	{
   		UFunction *SubStudent01Function = SubStudent01Class->FindFunctionByName(
   			TEXT("PlayGame"), EIncludeSuperFlag::ExcludeSuper);
   		if (SubStudent01Function)
   		{
   			UE_LOG(LogTemp, Warning, TEXT("<-- Class: %s, Function: %s -->"),
   				*SubStudent01->GetClass()->GetName(), *SubStudent01Function->GetName());
   		}
       }
   }
   ```

4. 打印结果：

   ```powershell
   LogTemp: [ Use String Find Class Function ]
   LogTemp: Warning: <-- Class: SubStudent, Function: PlayGame -->
   ```







### 22. ProcessEvent调用类函数



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "USubStudent.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Use String Find Class Function By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Use String Find Class Function ]"));
   	USubStudent *SubStudent01 = NewObject<USubStudent>();
   	if (UClass *SubStudent01Class = SubStudent01->GetClass())
   	{
   		UFunction *SubStudent01Function = SubStudent01Class->FindFunctionByName(
   			TEXT("PlayGame"), EIncludeSuperFlag::ExcludeSuper);
   		if (!SubStudent01Function){break;}
           
   		// ProcessEvent Transfer Class Function By Reflection
   		UE_LOG(LogTemp, Log, TEXT("[ ProcessEvent Transfer Class Function ]"));
           
   		// 1.Allocate Space For Parameters
   		uint16 *AllParamMemory = static_cast<uint16*>FMemory_Alloca(SubStudent01Function->ParmsSize);
   		FMemory::Memzero(AllParamMemory, SubStudent01Function->ParmsSize);
   		
   		// 2.Parameter assignment
   		for (TFieldIterator<FProperty> IteratorOfParams(SubStudent01Function); IteratorOfParams; ++IteratorOfParams)
   		{
   			const FProperty *Param = *IteratorOfParams;
   			FString ParamName = Param->GetName();
   			if (ParamName == FString("GameName"))
   			{
   				*Param->ContainerPtrToValuePtr<FString>(AllParamMemory) = "CSGO";
   			}
   		}
           		
           // 3.Call Function(Method)
   		SubStudent01->ProcessEvent(SubStudent01Function, AllParamMemory);
       }
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Use String Find Class Function ]
   LogTemp: [ ProcessEvent Transfer Class Function ]
   LogTemp: Warning: <-- SubStudent::PlayGame(FString)>> Play CSGO -->
   ```







### 23. Invoke调用类函数



1. `ReflectiveGameModeBase.cpp`

   ```c++
   #include "ReflectiveGameModeBase.h"
   #include "USubStudent.h"
   
   ReflectiveGameModeBase::ReflectiveGameModeBase(const FObjectInitializer& ObjectInitializer)
   	:AGameModeBase(ObjectInitializer)
   {
   	// Use String Find Class Function By Reflection
   	UE_LOG(LogTemp, Log, TEXT("[ Use String Find Class Function ]"));
   	USubStudent *SubStudent01 = NewObject<USubStudent>();
   	if (UClass *SubStudent01Class = SubStudent01->GetClass())
   	{   
   		// 	Invoke Transfer Class Function By Reflection
   		UE_LOG(LogTemp, Log, TEXT("[ Invoke Transfer Class Function ]"));
   		if (UFunction *SubStudent02Function = SubStudent01Class->FindFunctionByName(
   			TEXT("IsBoy"), EIncludeSuperFlag::ExcludeSuper))
   		{
   
   			// 1.Allocate Space For Parameters
   			uint16 *AllParamMemory02 = static_cast<uint16*>FMemory_Alloca(SubStudent02Function->ParmsSize);
   			FMemory::Memzero(AllParamMemory02, SubStudent02Function->ParmsSize);
   
   			// 2.Create FFrame
   			FFrame Frame(nullptr, SubStudent02Function, &AllParamMemory02);
   
   			// 3.Invoke Function
   			SubStudent02Function->Invoke(SubStudent02Function, Frame,
   				&AllParamMemory02 + SubStudent02Function->ReturnValueOffset);
   
   			// 4. Get Function Return Value
   			int *ReturnValue = reinterpret_cast<int*>(&AllParamMemory02 + SubStudent02Function->ReturnValueOffset);
   			UE_LOG(LogTemp, Warning, TEXT("<-- Return Value: %d -->"), *ReturnValue);
   		}
       }
   }
   ```

2. 打印结果：

   ```powershell
   LogTemp: [ Invoke Transfer Class Function ]
   LogTemp: Warning: <-- SubStudent::IsBoy>> Is Boy -->
   LogTemp: Warning: <-- Return Value: 1 -->
   ```




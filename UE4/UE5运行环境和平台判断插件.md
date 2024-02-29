# UE5运行环境和平台判断插件



[toc]



### 0. github

[UE5运行环境和平台判断插件](https://github.com/FHangH/FH_UnrealPlatform)



### 1. 结构设计

`Platform.h`

```c++
#pragma once

#include "Platform.generated.h"

UENUM(BlueprintType)
enum class ERuntime : uint8
{
	ER_Runtime	UMETA(Displayname="Runtime"),
	ER_Editor	UMETA(Displayname="Editor")
};

UENUM(BlueprintType)
enum class ERuntime_Editor_Debug_Development_Shipping : uint8
{
	ER_Editor		UMETA(Displayname="Editor"),
	ER_Debug		UMETA(Displayname="Debug"),
	ER_Development	UMETA(Displayname="Development"),
	ER_Shipping		UMETA(Displayname="Shipping")
};

UENUM(BlueprintType)
enum class EPlatform : uint8
{
	PC		UMETA(Displayname="PC"),
	Mac		UMETA(Displayname="MAC"),
	Android	UMETA(Displayname="Android"),
	IOS		UMETA(Displayname="IOS"),
	Linux	UMETA(Displayname="Linux")
};

UENUM(BlueprintType)
enum class EPlatform_PC_Linux : uint8
{
	PC		UMETA(Displayname="PC"),
	Linux	UMETA(Displayname="Linux")
};

UENUM(BlueprintType)
enum class EPlatform_PC_Linux_Android : uint8
{
	PC		UMETA(Displayname="PC"),
	Linux	UMETA(Displayname="Linux"),
	Android	UMETA(Displayname="Android")
};

UENUM(BlueprintType)
enum class EPlatform_PC_Linux_Android_IOS : uint8
{
	PC		UMETA(Displayname="PC"),
	Linux	UMETA(Displayname="Linux"),
	Android	UMETA(Displayname="Android"),
	IOS		UMETA(Displayname="IOS")
};
```



### 2. API

`FuncLib_PlatformUtils.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "Platform.h"
#include "Kismet/BlueprintFunctionLibrary.h"
#include "FuncLib_PlatformUtils.generated.h"

UCLASS(Blueprintable)
class FH_PLATFORM_API UFuncLib_PlatformUtils : public UBlueprintFunctionLibrary
{
	GENERATED_BODY()

	UFUNCTION(BlueprintCallable, meta = (ExpandEnumAsExecs = "RUNTIME", ToolTip = "判断当前的环境"), Category = 
              "FH|Runtime")
	static void SwitchRuntime(ERuntime& RUNTIME);

	UFUNCTION(BlueprintCallable, meta = (ExpandEnumAsExecs = "RUNTIME", ToolTip = "判断当前的环境"), Category = 
              "FH|Runtime")
	static void SwitchRuntime_Editor_Debug_Development_Shipping(ERuntime_Editor_Debug_Development_Shipping& RUNTIME);

	UFUNCTION(BlueprintCallable, meta=(ExpandEnumAsExecs="PLATFORM", ToolTip="依旧当前的平台,在蓝图中走不同的分支"), 
              Category="FH|Platform")
	static void SwitchPlatform(EPlatform& PLATFORM);

	UFUNCTION(BlueprintCallable, meta=(ExpandEnumAsExecs="PLATFORM", ToolTip="仅判断是PC, Linux"), 
              Category="FH|Platform")
	static void SwitchPC_Linux(EPlatform_PC_Linux& PLATFORM);

	UFUNCTION(BlueprintCallable, meta=(ExpandEnumAsExecs="PLATFORM", ToolTip="仅判断是PC, Linux, Android"), 
              Category="FH|Platform")
	static void SwitchPC_Linux_Android(EPlatform_PC_Linux_Android& PLATFORM);

	UFUNCTION(BlueprintCallable, meta=(ExpandEnumAsExecs="PLATFORM", ToolTip="仅判断是PC, Linux, Android, IOS"), 
              Category="FH|Platform")
	static void SwitchPC_Linux_Android_IOS(EPlatform_PC_Linux_Android_IOS& PLATFORM);
};
```



`FuncLib_PlatformUtils.cpp`

```c++
#include "FuncLib_PlatformUtils.h"

void UFuncLib_PlatformUtils::SwitchRuntime(ERuntime& RUNTIME)
{
#if UE_EDITOR
	RUNTIME = ERuntime::ER_Editor;
#else
	RUNTIME = ERuntime::ER_Runtime;
#endif
}

void UFuncLib_PlatformUtils::SwitchRuntime_Editor_Debug_Development_Shipping(
	ERuntime_Editor_Debug_Development_Shipping& RUNTIME)
{
#if UE_EDITOR
	RUNTIME = ERuntime_Editor_Debug_Development_Shipping::ER_Editor;
#elif UE_BUILD_DEBUG
	RUNTIME = ERuntime_Editor_Debug_Development_Shipping::ER_Debug;
#elif UE_BUILD_DEVELOPMENT
	RUNTIME = ERuntime_Editor_Debug_Development_Shipping::ER_Development;
#elif UE_BUILD_SHIPPING
	RUNTIME = ERuntime_Editor_Debug_Development_Shipping::ER_Shipping;
#endif
}

void UFuncLib_PlatformUtils::SwitchPlatform(EPlatform& PLATFORM)
{
#if PLATFORM_WINDOWS
	PLATFORM = EPlatform::PC;
#elif PLATFORM_MAC
	PLATFORM = EPlatform::Mac;
#elif PLATFORM_ANDROID
	PLATFORM = EPlatform::Android;
#elif PLATFORM_IOS
	PLATFORM = EPlatform::IOS;
#elif PLATFORM_LINUX
	PLATFORM = EPlatform::Linux;
#endif
}

void UFuncLib_PlatformUtils::SwitchPC_Linux(EPlatform_PC_Linux& PLATFORM)
{
#if PLATFORM_WINDOWS
	PLATFORM = EPlatform_PC_Linux::PC;
#elif PLATFORM_LINUX
	PLATFORM = EPlatform_PC_Linux::Linux;
#endif
}

void UFuncLib_PlatformUtils::SwitchPC_Linux_Android(EPlatform_PC_Linux_Android& PLATFORM)
{
#if PLATFORM_WINDOWS
	PLATFORM = EPlatform_PC_Linux_Android::PC;
#elif PLATFORM_LINUX
	PLATFORM = EPlatform_PC_Linux_Android::Linux;
#elif PLATFORM_ANDROID
	PLATFORM = EPlatform_PC_Linux_Android::Android;
#endif
}

void UFuncLib_PlatformUtils::SwitchPC_Linux_Android_IOS(EPlatform_PC_Linux_Android_IOS& PLATFORM)
{
#if PLATFORM_WINDOWS
	PLATFORM = EPlatform_PC_Linux_Android_IOS::PC;
#elif PLATFORM_LINUX
	PLATFORM = EPlatform_PC_Linux_Android_IOS::Linux;
#elif PLATFORM_ANDROID
	PLATFORM = EPlatform_PC_Linux_Android_IOS::Android;
#elif PLATFORM_IOS
	PLATFORM = EPlatform_PC_Linux_Android_IOS::IOS;
#endif
}
```


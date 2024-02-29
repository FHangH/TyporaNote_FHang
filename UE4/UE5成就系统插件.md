# UE5成就系统插件



[toc]



### 0. github

[UE5成就系统插件](https://github.com/FHangH/FH_Unreal_FH_AchievementSystem)



### 1. 成就数据结构设计

#### 1.1 成就等级

```c++
UENUM(BlueprintType)
enum class EAchievementGrade : uint8
{
	EAG_C	UMETA(Displayname="C"),
	EAG_B	UMETA(Displayname="B"),
	EAG_A	UMETA(Displayname="A"),
	EAG_S	UMETA(Displayname="S")
};
```



#### 1.2 成就类型

```c++
UENUM(BlueprintType)
enum class EAchievementType : uint8
{
	EAT_OneTime		UMETA(Displayname="一次任务"),
	EAT_Cumulative	UMETA(Displayname="累计任务")
};
```



#### 1.3 成就结构体

```c++
USTRUCT(BlueprintType)
struct FH_ACHIEVEMENTSYSTEM_API FAchievement
{
	GENERATED_USTRUCT_BODY()

	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	int32 AchievementID{0};

	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	FString Name{TEXT("")};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	FString Describe{TEXT("")};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	EAchievementGrade EAG{EAchievementGrade::EAG_C};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	UTexture2D* Image{nullptr};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	UMaterialInstance* ImageIns{nullptr};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	EAchievementType EAT{EAchievementType::EAT_OneTime};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	int32 CurProgressCount{0};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	int32 MaxProgressCount{1};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	bool IsAchievement{false};
};
```



#### 1.4 成就结构源码

`AchievementData.h`

```c++
#pragma once

#include <CoreMinimal.h>
#include "AchievementData.generated.h"

UENUM(BlueprintType)
enum class EAchievementGrade : uint8
{
	EAG_C	UMETA(Displayname="C"),
	EAG_B	UMETA(Displayname="B"),
	EAG_A	UMETA(Displayname="A"),
	EAG_S	UMETA(Displayname="S")
};

UENUM(BlueprintType)
enum class EAchievementType : uint8
{
	EAT_OneTime		UMETA(Displayname="一次任务"),
	EAT_Cumulative	UMETA(Displayname="累计任务")
};

USTRUCT(BlueprintType)
struct FH_ACHIEVEMENTSYSTEM_API FAchievement
{
	GENERATED_USTRUCT_BODY()

	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	int32 AchievementID{0};

	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	FString Name{TEXT("")};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	FString Describe{TEXT("")};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	EAchievementGrade EAG{EAchievementGrade::EAG_C};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	UTexture2D* Image{nullptr};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	UMaterialInstance* ImageIns{nullptr};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	EAchievementType EAT{EAchievementType::EAT_OneTime};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	int32 CurProgressCount{0};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	int32 MaxProgressCount{1};
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite)
	bool IsAchievement{false};
};

static TMap<int32, FAchievement> AchievementsList{};
```



### 2. 成就系统API

`FH_FuncLib_AS.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "Kismet/BlueprintFunctionLibrary.h"
#include "AchievementData.h"
#include "FH_FuncLib_AS.generated.h"

UCLASS(BlueprintType)
class FH_ACHIEVEMENTSYSTEM_API UFH_FuncLib_AS : public UBlueprintFunctionLibrary
{
	GENERATED_BODY()

/* My Code */
	/* Function */
public:
	// 判断成就是否存在
	UFUNCTION(BlueprintPure, Category="FH|Achievement")
	static bool IsAchievementExists(const int32 AchievementID);
	
    // 添加新成就
	UFUNCTION(BlueprintCallable, Category="FH|Achievement")
	static bool AddAchievement(const FAchievement& NewAchievement);

    // 移除成就
	UFUNCTION(BlueprintCallable, Category="FH|Achievement")
	static bool RemoveAchievement(const int32 AchievementID);

    // 获取所有存在的成就的列表
	UFUNCTION(BlueprintPure, Category="FH|Achievement")
	static TMap<int32, FAchievement> GetAchievementsList();

    // 清除所有存在成就的列表
	UFUNCTION(BlueprintPure, Category="FH|Achievement")
	static bool ClearAchievementsList();

    // 获取所有成就
	UFUNCTION(BlueprintCallable, Category="FH|Achievement")
	static void GetAllAchievements(TArray<FAchievement>& Achievements);

	

	// 完成一次成就
	UFUNCTION(BlueprintCallable, Category="FH|Achievement")
	static void DoOnceTask(const int32 AchievementID, int32& ID, bool& IsDoOver);

    // 判断成就是否完成
	UFUNCTION(BlueprintPure, Category="FH|Achievement")
	static void IsAchievementComplete(int32 AchievementID, bool& IsComplete);
};
```



:::warning

1. 添加新成就需要保证赋予全局唯一的ID，该成就系统依旧ID辨别成就和获取
2. 移除不需要的成就也是通过ID进行移除
3. 无论是否是一次性成就，在执行`DoOnceTask`后，都应该立刻使用`IsAchievementComplete`判断成就是否完成

:::



`FH_FuncLib_AS.cpp`

```c++
#include "FH_FuncLib_AS.h"

// Add And Remove Achievement List
bool UFH_FuncLib_AS::IsAchievementExists(const int32 AchievementID)
{
	if (AchievementsList.IsEmpty()) return false;
	if (AchievementsList.Find(AchievementID)) return true;
	return false;
}

bool UFH_FuncLib_AS::AddAchievement(const FAchievement& NewAchievement)
{
	if (AchievementsList.Find(NewAchievement.AchievementID)) return false;
	AchievementsList.Add(NewAchievement.AchievementID, NewAchievement);
	return true;
}

bool UFH_FuncLib_AS::RemoveAchievement(const int32 AchievementID)
{
	if (AchievementsList.IsEmpty()) return false;
	if (AchievementsList.Find(AchievementID))
	{
		AchievementsList.Remove(AchievementID);
		return true;
	}
	return false;
}

TMap<int32, FAchievement> UFH_FuncLib_AS::GetAchievementsList()
{
	return AchievementsList;
}

bool UFH_FuncLib_AS::ClearAchievementsList()
{
	AchievementsList.Empty();
	if (AchievementsList.IsEmpty()) return true;
	return false;
}

void UFH_FuncLib_AS::GetAllAchievements(TArray<FAchievement>& Achievements)
{
	AchievementsList.GenerateValueArray(Achievements);
}

// Complete Task
void UFH_FuncLib_AS::DoOnceTask(const int32 AchievementID, int32& ID, bool& IsDoOver)
{
	if (!IsAchievementExists(AchievementID))
	{
		IsDoOver = false;
		return;
	}
	const auto Achievement = AchievementsList.Find(AchievementID);
	ID = AchievementID;
	
	if (Achievement->IsAchievement && Achievement->CurProgressCount == Achievement->MaxProgressCount)
	{
		IsDoOver = false;
		return;
	}

	switch (Achievement->EAT)
	{
	case EAchievementType::EAT_OneTime:
		{
			Achievement->CurProgressCount = 1;
			Achievement->IsAchievement = true;
			IsDoOver = true;
		}
		break;
	case EAchievementType::EAT_Cumulative:
		{
			Achievement->CurProgressCount += 1;
			IsDoOver = true;
			if (Achievement->CurProgressCount == Achievement->MaxProgressCount)
			{
				Achievement->IsAchievement = true;
			}
		}
		break;
	default:
		IsDoOver = false;
		break;
	}
}

void UFH_FuncLib_AS::IsAchievementComplete(const int32 AchievementID, bool& IsComplete)
{
	if (!IsAchievementExists(AchievementID))
	{
		IsComplete = false;
		return;
	}
	const auto Achievement = AchievementsList.Find(AchievementID);
	
	if (Achievement->IsAchievement && Achievement->CurProgressCount == Achievement->MaxProgressCount)
	{
		IsComplete = true;
	}
	else
	{
		IsComplete = false;
	}
}
```


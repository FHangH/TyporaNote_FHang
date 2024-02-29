# UE5字符匹配插件



[toc]



### 0. github

[UE5字符匹配插件](https://github.com/FHangH/FH_StringSimilarity)



### 1. 功能介绍

#### 1.1 字符匹配

- 支持`莱温斯特编辑距离`和`Jaccard`两种字符匹配算法
- 正常情况下，推荐`莱温斯特编辑距离`



#### 1.2 匹配方式

- `StringSimilarity`选择匹配类型，进行两个字符串匹配，返回匹配率
- `StrArraySimilarity`选择匹配类型，将一组字符串和一个字符串进行匹配，，返回匹配率和匹配率最高的字符串



#### 1.3 清除中英文标点符号和空格

- `RemoveSpacesAndSymbolsByString`可以移除字符串中的
  - 英文标点符号和空格
  - 中文标点符号和空格
- `IsChinesePunctuationOrSpace`可以拓展

```c++
bool UFuncLib_StringSimilarity::IsChinesePunctuationOrSpace(const wchar_t C)
{
	if (C == L'，' || C == L'。' || C == L'！' || C == L'？' || C == L' ' || C == L'：')
	{
		return true;
	}
	return false;
}
```



### 2. API

`FuncLib_StringSimilarity.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "Kismet/BlueprintFunctionLibrary.h"
#include "FuncLib_StringSimilarity.generated.h"

UENUM(BlueprintType)
enum class ESimilarityType : uint8
{
	EST_Levenshtein		UMETA(Displayname="莱温斯特编辑距离"),
	EST_Jaccard			UMETA(Displayname="Jaccard")
};

UCLASS(BlueprintType)
class FH_STRINGSIMILARITY_API UFuncLib_StringSimilarity : public UBlueprintFunctionLibrary
{
	GENERATED_BODY()

private:
	UFUNCTION()
	static int LevenshteinDistance(const FString& Str1, const FString& Str2);
	
public:
	UFUNCTION(BlueprintPure, Category="FH|Similarity", meta=(ToolTip="比较两个字符串的相似度，中英文数字都能算，莱温斯特编辑距离精
                                                             准(考虑顺序)， Jaccard(不考虑顺序，只考虑是否包含)"))
	static float StringSimilarity(const ESimilarityType EST, const FString& Str1, const FString& Str2);

	UFUNCTION(BlueprintPure, Category="FH|Similarity", meta=(ToolTip="一组字符串和待比较的字符串，计算所有的相似度，返回最大相似度
                                                             和字符串，中英文数字都能算，莱温斯特编辑距离精准(考虑顺序)， 
                                                             Jaccard(不考虑顺序，只考虑是否包含)"))
	static FString StrArraySimilarity(const ESimilarityType EST, const TArray<FString>& CompareStrArr, const FString& 
                                      NewStr, float& Similarity);

	static bool IsChinesePunctuationOrSpace(const wchar_t C);
	
	UFUNCTION(BlueprintPure, Category="FH|Similarity", meta=(ToolTip="删除字符串中的空格和标点符号"))
	static FString RemoveSpacesAndSymbolsByString(const FString& Str);
};
```



`FuncLib_StringSimilarity.cpp`

```c++
#include "FuncLib_StringSimilarity.h"
#include <string>
#include <ctype.h>
#include <algorithm>
#include <unordered_set>
#include <vector>

int UFuncLib_StringSimilarity::LevenshteinDistance(const FString& Str1, const FString& Str2)
{
	if (Str1 == Str2)
	{
		return 0;
	}
	const int m = Str1.Len();
	const int n = Str2.Len();
	std::vector T(m + 1, std::vector(n + 1, 0));

	for (int i = 1; i <= m; i++)
	{
		T[i][0] = i;
	}
	for (int j = 1; j <= n; j++)
	{
		T[0][j] = j;
	}
	for (int i = 1; i <= m; i++)
	{
		for (int j = 1; j <= n; j++)
		{
			const int Weight = Str1[i - 1] == Str2[j - 1] ? 0 : 1;
			T[i][j] = std::min(std::min(T[i-1][j] + 1, T[i][j-1] + 1), T[i-1][j-1] + Weight);
		}
	}
	return T[m][n];
}

float UFuncLib_StringSimilarity::StringSimilarity(const ESimilarityType EST, const FString& Str1, const FString& Str2)
{
	if (EST == ESimilarityType::EST_Jaccard)
	{
		std::string S1(TCHAR_TO_UTF8(*Str1));
		std::string S2(TCHAR_TO_UTF8(*Str2));
		const std::unordered_set Set_S1(S1.begin(), S1.end());
		const std::unordered_set Set_S2(S2.begin(), S2.end());
		int Intersection = 0;
		for (auto c : Set_S1)
		{
			if (Set_S2.find(c) != Set_S2.end())
			{
				++Intersection;
			}
		}
		return static_cast<float>(Intersection) / (Set_S1.size() + Set_S2.size() - Intersection);
	}
	if (EST == ESimilarityType::EST_Levenshtein)
	{
		const int Distance = LevenshteinDistance(Str1, Str2);
		const int MaxLength = std::max(Str1.Len(), Str2.Len());
		if (MaxLength == 0)
		{
			return 0.0f;
		}
		return 1.0f - static_cast<float>(Distance) / static_cast<float>(MaxLength);
	}
	return 0.f;
}

FString UFuncLib_StringSimilarity::StrArraySimilarity(const ESimilarityType EST, const TArray<FString>& CompareStrArr, const FString& NewStr, float& Similarity)
{
	if (CompareStrArr.Num() == 0) return "";
	
	int Index = 0;
	float Temp = 0.f;
	for (int i = 0; i < CompareStrArr.Num(); ++i)
	{
		if (StringSimilarity(EST, CompareStrArr[i], NewStr) > Temp)
		{
			Index = i;
			Temp = StringSimilarity(EST, CompareStrArr[i], NewStr);
		}
	}
	Similarity = Temp;
	return CompareStrArr[Index];
}

bool UFuncLib_StringSimilarity::IsChinesePunctuationOrSpace(const wchar_t C)
{
	if (C == L'，' || C == L'。' || C == L'！' || C == L'？' || C == L' ' || C == L'：')
	{
		return true;
	}
	return false;
}

FString UFuncLib_StringSimilarity::RemoveSpacesAndSymbolsByString(const FString& Str)
{
	std::wstring NewStr = TCHAR_TO_WCHAR(*Str);
	NewStr.erase(
		std::remove_if(
			NewStr.begin(), NewStr.end(), static_cast<int(*)(int)>(&ispunct)),
			NewStr.end()
	);
	NewStr.erase(
		std::remove_if(
			NewStr.begin(), NewStr.end(), static_cast<int(*)(int)>(&isspace)),
			NewStr.end()
	);
	NewStr.erase(
		std::remove_if(
			NewStr.begin(), NewStr.end(), IsChinesePunctuationOrSpace),
			NewStr.end()
	);
	return WCHAR_TO_TCHAR(NewStr.c_str());
}
```


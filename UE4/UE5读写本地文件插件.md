# UE5读写本地文件插件



[toc]



### 0. github

[UE5读写本地文件插件](https://github.com/FHangH/FH_Unreal_ReadWriteLocalFile)



### 1. API

`ReadWriteLocalFile.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "Kismet/BlueprintFunctionLibrary.h"
#include "ReadWriteLocalFile.generated.h"

UENUM(BlueprintType)
enum class EPathDir : uint8
{
	EPD_RootDir UMETA(Displayname="RootDir"),
	EPD_ProjectDir UMETA(Displayname="ProjectDir"),
	EPD_ProjectConfigDir UMETA(Displayname="ProjectConfigDir"),
	EPD_ProjectContentDir UMETA(Displayname="ProjectContentDir"),
	EPD_ProjectLogDir UMETA(Displayname="ProjectLogDir"),
	EPD_ProjectPluginsDir UMETA(Displayname="ProjectPluginsDir"),
	EPD_ProjectSavedDir UMETA(Displayname="ProjectSavedDir"),
	EPD_ProjectUserDir UMETA(Displayname="ProjectUserDir"),
	EPD_EngineDir UMETA(Displayname="EngineDir"),
	EPD_EngineConfigDir UMETA(Displayname="EngineConfigDir"),
	EPD_EngineContentDir UMETA(Displayname="EngineContentDir"),
	EPD_EnginePluginsDir UMETA(Displayname="EnginePluginsDir"),
	EPD_EngineSavedDir UMETA(Displayname="EngineSavedDir"),
	EPD_EngineSourceDir UMETA(Displayname="EngineSourceDir"),
	EPD_LaunchDir UMETA(Displayname="LaunchDir"),
	EPD_GameSourceDir UMETA(Displayname="GameSourceDir")
};

UCLASS(BlueprintType)
class FH_READWRITELOCALFILE_API UReadWriteLocalFile : public UBlueprintFunctionLibrary
{
	GENERATED_BODY()

/* My Code */
public:	
	// Enum Convert String
	UFUNCTION(BlueprintPure, Category="FH|ReadWriteLocalFile")
	static FString ConvPathToString(const EPathDir EPD);

	// TArray<FString> Convert To FString
	UFUNCTION(BlueprintPure, Category="FH|ReadWriteLocalFile")
	static FString ConvertArrayToString(const TArray<FString>& TextResults);
	
	// Check Local File Exists
	UFUNCTION(BlueprintPure, Category="FH|ReadWriteLocalFile")
	static bool IsLocalFileExistsByStr(const FString& FilePath);

	UFUNCTION(BlueprintPure, Category="FH|ReadWriteLocalFile")
	static bool IsLocalFileExistsByEnum(const EPathDir EPD);

	// Create Local File Path And File
	UFUNCTION(BlueprintCallable, Category="FH|ReadWriteLocalFile")
	static bool CreatePathAndFileByStr(const FString& FilePath, const FString& FileName, FString& Path);

	UFUNCTION(BlueprintCallable, Category="FH|ReadWriteLocalFile")
	static bool CreatePathAndFileByEnum(const EPathDir EPD, const FString& FileName, FString& Path);

	// Read Local File
	UFUNCTION(BlueprintCallable, Category="FH|ReadWriteLocalFile")
	static TArray<FString> ReadLocalFileByStr(const FString& FilePath, const FString& FileName);

	UFUNCTION(BlueprintCallable, Category="FH|ReadWriteLocalFile")
	static TArray<FString> ReadLocalFileByEnum(const EPathDir EPD, const FString& FileName);
	
	// Write Local File
	UFUNCTION(BlueprintCallable, Category="FH|ReadWriteLocalFile")
	static bool WriteLocalFileByStr(const FString& FilePath, const FString& FileName, const FString& Content);

	UFUNCTION(BlueprintCallable, Category="FH|ReadWriteLocalFile")
	static bool WriteLocalFileByEnum(const EPathDir EPD, const FString& FileName, const FString& Content);
};
```



`ReadWriteLocalFile.cpp`

```c++
#include "ReadWriteLocalFile.h"

// Enum Convert String
FString UReadWriteLocalFile::ConvPathToString(const EPathDir EPD)
{
	if (EPD == EPathDir::EPD_RootDir) return FPaths::RootDir();
	if (EPD == EPathDir::EPD_ProjectDir) return FPaths::ProjectDir();
	if (EPD == EPathDir::EPD_ProjectConfigDir) return FPaths::ProjectConfigDir();
	if (EPD == EPathDir::EPD_ProjectContentDir) return FPaths::ProjectContentDir();
	if (EPD == EPathDir::EPD_ProjectLogDir) return FPaths::ProjectLogDir();
	if (EPD == EPathDir::EPD_ProjectPluginsDir) return FPaths::ProjectPluginsDir();
	if (EPD == EPathDir::EPD_ProjectSavedDir) return FPaths::ProjectSavedDir();
	if (EPD == EPathDir::EPD_ProjectUserDir) return FPaths::ProjectUserDir();
	if (EPD == EPathDir::EPD_EngineDir) return FPaths::EngineDir();
	if (EPD == EPathDir::EPD_EngineConfigDir) return FPaths::EngineConfigDir();
	if (EPD == EPathDir::EPD_EngineContentDir) return FPaths::EngineContentDir();
	if (EPD == EPathDir::EPD_EnginePluginsDir) return FPaths::EnginePluginsDir();
	if (EPD == EPathDir::EPD_EngineSavedDir) return FPaths::EngineSavedDir();
	if (EPD == EPathDir::EPD_EngineSourceDir) return FPaths::EngineSourceDir();
	if (EPD == EPathDir::EPD_LaunchDir) return FPaths::LaunchDir();
	if (EPD == EPathDir::EPD_GameSourceDir) return FPaths::GameSourceDir();
	return {};
}

// TArray<FString> Convert To FString
FString UReadWriteLocalFile::ConvertArrayToString(const TArray<FString>& TextResults)
{
	FString TextContent;
	for (const auto& result : TextResults)
	{
		TextContent += FString::Printf(TEXT("%s\n"), *result);
	}
	return TextContent;
}

// Check Local File Exists
bool UReadWriteLocalFile::IsLocalFileExistsByStr(const FString& FilePath)
{
	return FPlatformFileManager::Get().GetPlatformFile().FileExists(*FPaths::ConvertRelativePathToFull(FilePath));
}

bool UReadWriteLocalFile::IsLocalFileExistsByEnum(const EPathDir EPD)
{
	return FPlatformFileManager::Get().GetPlatformFile().FileExists(*ConvPathToString(EPD));
}

// Create Local File Path And File
bool UReadWriteLocalFile::CreatePathAndFileByStr(const FString& FilePath, const FString& FileName, FString& Path)
{
	const FString tempName = FileName == "" ? "" : FString::Printf(TEXT("/%s"), *FileName);
	Path = FString::Printf(TEXT("%s%s"), *FilePath, *tempName);
	FPaths::MakeValidFileName(*Path);

	if (IsLocalFileExistsByStr(Path))
	{
		return true;
	}
	return FFileHelper::SaveStringToFile(TEXT(""), *Path, FFileHelper::EEncodingOptions::ForceUTF8);
}

bool UReadWriteLocalFile::CreatePathAndFileByEnum(const EPathDir EPD, const FString& FileName, FString& Path)
{
	Path = FString::Printf(TEXT("%s%s"), *ConvPathToString(EPD), *FileName);
	FPaths::MakeValidFileName(*Path);
	
	if (IsLocalFileExistsByStr(Path))
	{
		return true;
	}
	return FFileHelper::SaveStringToFile(TEXT(""), *Path, FFileHelper::EEncodingOptions::ForceUTF8);
}

// Read Local File
TArray<FString> UReadWriteLocalFile::ReadLocalFileByStr(const FString& FilePath, const FString& FileName)
{
	const FString tempName = FileName == "" ? "" : FString::Printf(TEXT("/%s"), *FileName);
	const auto Path = FString::Printf(TEXT("%s%s"), *FilePath, *tempName);
	FPaths::MakeValidFileName(*Path);
	TArray<FString> TextResults;

	if (IsLocalFileExistsByStr(Path))
	{
		if (FFileHelper::LoadFileToStringArray(TextResults, *Path)) return TextResults;
		return {};
	}
	return {};
}

TArray<FString> UReadWriteLocalFile::ReadLocalFileByEnum(const EPathDir EPD, const FString& FileName)
{
	const auto Path = FString::Printf(TEXT("%s%s"), *ConvPathToString(EPD), *FileName);
	FPaths::MakeValidFileName(*Path);
	TArray<FString> TextResults;

	if (IsLocalFileExistsByStr(Path))
	{
		if (FFileHelper::LoadFileToStringArray(TextResults, *Path)) return TextResults;
		return {};
	}
	return {};
}

// Write Local File
bool UReadWriteLocalFile::WriteLocalFileByStr(const FString& FilePath, const FString& FileName, const FString& Content)
{
	const FString tempName = FileName == "" ? "" : FString::Printf(TEXT("/%s"), *FileName);
	const auto Path = FString::Printf(TEXT("%s%s"), *FilePath, *tempName);
	FPaths::MakeValidFileName(*Path);

	if (IsLocalFileExistsByStr(Path))
	{
		return FFileHelper::SaveStringToFile(Content, *Path, FFileHelper::EEncodingOptions::ForceUTF8);
	}
	return false;
}

bool UReadWriteLocalFile::WriteLocalFileByEnum(const EPathDir EPD, const FString& FileName, const FString& Content)
{
	const auto Path = FString::Printf(TEXT("%s%s"), *ConvPathToString(EPD), *FileName);
	FPaths::MakeValidFileName(*Path);

	if (IsLocalFileExistsByStr(Path))
	{
		return FFileHelper::SaveStringToFile(Content, *Path, FFileHelper::EEncodingOptions::ForceUTF8);
	}
	return false;
}
```


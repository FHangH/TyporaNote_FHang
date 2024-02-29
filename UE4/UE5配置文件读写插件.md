# UE5配置文件读写插件



[toc]



### 0. github

[UE5配置文件读写插件](https://github.com/FHangH/FH_Unreal_INI)



### 1. API

`FH_FuncLib_INI.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "Kismet/BlueprintFunctionLibrary.h"
#include "FH_FuncLib_INI.generated.h"

UENUM(BlueprintType)
enum class EDirPath : uint8
{
	EDP_RootDir				UMETA(Displayname="RootDir"),
	EDP_ProjectDir			UMETA(Displayname="ProjectDir"),
	EDP_ProjectConfigDir	UMETA(Displayname="ProjectConfigDir"),
	EDP_ProjectContentDir	UMETA(Displayname="ProjectContentDir"),
	EDP_ProjectLogDir		UMETA(Displayname="ProjectLogDir"),
	EDP_ProjectPluginsDir	UMETA(Displayname="ProjectPluginsDir"),
	EDP_ProjectSavedDir		UMETA(Displayname="ProjectSavedDir"),
	EDP_ProjectUserDir		UMETA(Displayname="ProjectUserDir"),
	EDP_EngineDir			UMETA(Displayname="EngineDir"),
	EDP_EngineConfigDir		UMETA(Displayname="EngineConfigDir"),
	EDP_EngineContentDir	UMETA(Displayname="EngineContentDir"),
	EDP_EnginePluginsDir	UMETA(Displayname="EnginePluginsDir"),
	EDP_EngineSavedDir		UMETA(Displayname="EngineSavedDir"),
	EDP_EngineSourceDir		UMETA(Displayname="EngineSourceDir"),
	EDP_LaunchDir			UMETA(Displayname="LaunchDir"),
	EDP_GameSourceDir		UMETA(Displayname="GameSourceDir")
};

UCLASS(BlueprintType)
class FH_INI_API UFH_FuncLib_INI : public UBlueprintFunctionLibrary
{
	GENERATED_BODY()

/* My Code */
public:
	// Enum Convert To String
	UFUNCTION(BlueprintPure, Category="FH|INI|Convert")
	static FString ConvertPathToString(const EDirPath EDP);

	// Check Local File Exists
	UFUNCTION(BlueprintPure, Category="FH|INI|Check")
	static bool IsLocalFileExists(const FString& FilePath);

	// Create INI File
	UFUNCTION(BlueprintPure, Category="FH|INI|Init")
	static FString GenerateFilePath(const FString& RelativePath, const FString& FileName, FString& Path, const EDirPath 
                                    EDP = EDirPath::EDP_ProjectDir);

	// Init INI File
	UFUNCTION(BlueprintPure, Category="FH|INI|Init")
	static bool InitConfigINIByStr(const FString& RelativePath, const FString& FileName, FString& Path);

	UFUNCTION(BlueprintPure, Category="FH|INI|Init")
	static bool InitConfigINIByEnum(const EDirPath EDP, const FString& RelativePath, const FString& FileName, FString& 
                                    Path);

	// Read INI File
	UFUNCTION(BlueprintPure, Category="FH|INI|Read")
	static bool ReadConfigINIByStr(const FString& Path, const FString& Section, const FString& Key, FString& Value);

	UFUNCTION(BlueprintPure, Category="FH|INI|Read")
	static bool ReadConfigINIByEnum(const EDirPath EDP, const FString& RelativePath, const FString& FileName, const 
                                    FString& Section, const FString& Key, FString& Value);

	// Write INI File
	UFUNCTION(BlueprintPure, Category="FH|INI|Write")
	static bool WriteConfigInIByStr(const FString& Path, const FString& Section, const FString& Key, const FString& 
                                    Value);

	UFUNCTION(BlueprintPure, Category="FH|INI|Write")
	static bool WriteConfigInIByEnum(const EDirPath EDP, const FString& RelativePath, const FString& FileName, const 
                                     FString& Section, const FString& Key, const FString& Value);
};
```



`FH_FuncLib_INI.cpp`

```c++
#include "FH_FuncLib_INI.h"

// Enum Convert String
FString UFH_FuncLib_INI::ConvertPathToString(const EDirPath EDP)
{
	if (EDP == EDirPath::EDP_RootDir)			return FPaths::RootDir();
	if (EDP == EDirPath::EDP_ProjectDir)		return FPaths::ProjectDir();
	if (EDP == EDirPath::EDP_ProjectConfigDir)	return FPaths::ProjectConfigDir();
	if (EDP == EDirPath::EDP_ProjectContentDir) return FPaths::ProjectContentDir();
	if (EDP == EDirPath::EDP_ProjectLogDir)		return FPaths::ProjectLogDir();
	if (EDP == EDirPath::EDP_ProjectPluginsDir) return FPaths::ProjectPluginsDir();
	if (EDP == EDirPath::EDP_ProjectSavedDir)	return FPaths::ProjectSavedDir();
	if (EDP == EDirPath::EDP_ProjectUserDir)	return FPaths::ProjectUserDir();
	if (EDP == EDirPath::EDP_EngineDir)			return FPaths::EngineDir();
	if (EDP == EDirPath::EDP_EngineConfigDir)	return FPaths::EngineConfigDir();
	if (EDP == EDirPath::EDP_EngineContentDir)	return FPaths::EngineContentDir();
	if (EDP == EDirPath::EDP_EnginePluginsDir)	return FPaths::EnginePluginsDir();
	if (EDP == EDirPath::EDP_EngineSavedDir)	return FPaths::EngineSavedDir();
	if (EDP == EDirPath::EDP_EngineSourceDir)	return FPaths::EngineSourceDir();
	if (EDP == EDirPath::EDP_LaunchDir)			return FPaths::LaunchDir();
	if (EDP == EDirPath::EDP_GameSourceDir)		return FPaths::GameSourceDir();
	return {};
}

// Check Local File Exists
bool UFH_FuncLib_INI::IsLocalFileExists(const FString& FilePath)
{
	return FPaths::FileExists(*FilePath);
}

// Create INI File
FString UFH_FuncLib_INI::GenerateFilePath(const FString& RelativePath, const FString& FileName, FString& Path, const EDirPath EDP)
{
	Path =
		FileName == "" ?
			*FString::Printf(TEXT("%s%s"),
				*FPaths::ConvertRelativePathToFull(*ConvertPathToString(EDP)),
				*RelativePath) :
			*FString::Printf(TEXT("%s%s%s.ini"),
				*FPaths::ConvertRelativePathToFull(*ConvertPathToString(EDP)),
				*RelativePath,
				*FPaths::MakeValidFileName(*FileName));
	return Path;
}

// Init INI File
bool UFH_FuncLib_INI::InitConfigINIByStr(const FString& RelativePath, const FString& FileName, FString& Path)
{
	if (IsLocalFileExists(*GenerateFilePath(RelativePath, FileName, Path))) return true;
	return FFileHelper::SaveStringToFile(TEXT(""), *Path, FFileHelper::EEncodingOptions::ForceUTF8);
}

bool UFH_FuncLib_INI::InitConfigINIByEnum(const EDirPath EDP, const FString& RelativePath, const FString& FileName, FString& Path)
{
	if (IsLocalFileExists(*GenerateFilePath(RelativePath, FileName, Path, EDP))) return true;
	return FFileHelper::SaveStringToFile(TEXT(""), *Path, FFileHelper::EEncodingOptions::ForceUTF8);
}

// Read INI File
bool UFH_FuncLib_INI::ReadConfigINIByStr(const FString& Path, const FString& Section, const FString& Key, FString& Value)
{
	if (GConfig == nullptr) return false;
	if (IsLocalFileExists(*Path))
	{
		GConfig->Flush(true, *Path);
		return GConfig->GetString(*Section, *Key, Value, *Path);
	}
	return false;
}

bool UFH_FuncLib_INI::ReadConfigINIByEnum(const EDirPath EDP, const FString& RelativePath, const FString& FileName, const FString& Section, const FString& Key, FString& Value)
{
	if (GConfig == nullptr) return false;
	const auto Path =
		FString::Printf(TEXT("%s%s%s.ini"),
			*FPaths::ConvertRelativePathToFull(*ConvertPathToString(EDP)),
			*RelativePath,
			*FPaths::MakeValidFileName(*FileName));
	
	if (IsLocalFileExists(*Path))
	{
		GConfig->Flush(true, *Path);
		return GConfig->GetString(*Section, *Key, Value, *Path);
	}
	return false;
}

// Write INI File
bool UFH_FuncLib_INI::WriteConfigInIByStr(const FString& Path, const FString& Section, const FString& Key, const FString& Value)
{
	if (GConfig == nullptr) return false;
	if (IsLocalFileExists(*Path))
	{
		GConfig->Flush(true, *Path);
		GConfig->SetString(*Section, *Key, *Value, *Path);
		return true;
	}
	return false;
}

bool UFH_FuncLib_INI::WriteConfigInIByEnum(const EDirPath EDP, const FString& RelativePath, const FString& FileName, const FString& Section, const FString& Key, const FString& Value)
{
	if (GConfig == nullptr) return false;
	const auto Path =
		FString::Printf(TEXT("%s%s%s.ini"),
			*FPaths::ConvertRelativePathToFull(*ConvertPathToString(EDP)),
			*RelativePath,
			*FPaths::MakeValidFileName(*FileName));
	
	if (IsLocalFileExists(*Path))
	{
		GConfig->Flush(true, *Path);
		GConfig->SetString(*Section, *Key, *Value, *Path);
		return true;
	}
	return false;
}
```




# UE5腾讯语音插件



[toc]



### 0. github

里面动态库太大了，暂时没传github；



腾讯语音 https://www.alipan.com/s/YQ8AJm7VVoo 提取码: 2dg0



### 1. SDK接口

插件使用的腾讯GME提供的全平台SDK，先用原生C++定义接口，再用UE C++ Subsystem接入到蓝图中

`GMEAudioAdpter.h`

```c++
#pragma once
#include "tmg_sdk.h"
#include "CoreMinimal.h"

class ITMGMessage
{
public:
	virtual ~ITMGMessage() {};
	virtual void OnMessage(const FString& message) = 0;
};

#define SEND_MESSAGE(message) if (nullptr != MessageHandler) { \
	MessageHandler->OnMessage(message);							\
}

class GMEAudioAdpter : public ITMGDelegate
{
public:
	static GMEAudioAdpter* Ins;
	static GMEAudioAdpter* Get();
protected:
	GMEAudioAdpter();
public:
	virtual void OnEvent(ITMG_MAIN_EVENT_TYPE eventType, const char* data) override;

	void SetMessageHandler(ITMGMessage* handler);
	void SetUserID(const FString& pUserID);
	void StartRecord() const;
	void StartRecordingWithStreaming();
	void StopRecord();
	void CancelRecord();
	void Upload() const;
	void Download() const;
	void Play() const;
	void ConvertToText() const;
	int InitGME();
	void UnInit();
	void OnTickGME();
	
private:
	FString GetFilePath() const;
	
private:
	ITMGMessage* MessageHandler;
	FString UserID;
	FString FilePath;
	FString FileUrl;

public:
	FString STT;
	FString SdkAppID;
	FString SdkAppKey;
};
```



`GMEAudioAdpter.cpp`

```c++
#include "GMEAudioAdpter.h"
#include <string>
#include "Containers/StringConv.h"
#include "Dom/JsonObject.h"
#include "Engine/Engine.h"
#include "Misc/Paths.h"
#include "Serialization/JsonReader.h"
#include "Serialization/JsonSerializer.h"
#include "Templates/SharedPointer.h"

GMEAudioAdpter* GMEAudioAdpter::Ins = nullptr;

GMEAudioAdpter* GMEAudioAdpter::Get()
{
	if (nullptr == Ins)
	{
		Ins = new GMEAudioAdpter;
	}
	return Ins;
}

GMEAudioAdpter::GMEAudioAdpter() : MessageHandler(nullptr)
{
}

void GMEAudioAdpter::OnEvent(ITMG_MAIN_EVENT_TYPE eventType, const char* data)
{
	FString jsonData = FString(UTF8_TO_TCHAR(data));
	TSharedPtr<FJsonObject> JsonObject;
	const TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(FString(UTF8_TO_TCHAR(data)));
	FJsonSerializer::Deserialize(Reader, JsonObject);

#if PLATFORM_ANDROID || PLATFORM_IOS || PLATFORM_WINDOWS || PLATFORM_MAC
	if (eventType == ITMG_MAIN_EVNET_TYPE_PTT_RECORD_COMPLETE)
	{
		STT = "";
		FileUrl = "";
		const int32 result = JsonObject->GetIntegerField(TEXT("result"));
		FilePath = JsonObject->GetStringField(TEXT("file_path"));

		const std::string path = TCHAR_TO_UTF8(FilePath.operator*());
		int duration = 0;
		int filesize = 0;
		if (result == 0)
		{
			duration = ITMGContextGetInstance()->GetPTT()->GetVoiceFileDuration(path.c_str());
			filesize = ITMGContextGetInstance()->GetPTT()->GetFileSize(path.c_str());
		}

		const FString msg = FString::Printf(TEXT("info: recording completed.\nreslut=%d\nfilepath=%ls\nduarion=%.2f\nfilesize=%.2fKB"), result, *FilePath, duration / 1000.0f, filesize / 1024.0f);
		SEND_MESSAGE(msg)
		UE_LOG(LogTemp, Warning, TEXT("%s"), *msg);
	}
	else if (eventType == ITMG_MAIN_EVNET_TYPE_PTT_UPLOAD_COMPLETE)
	{
		STT = "";
		const int32 result = JsonObject->GetIntegerField(TEXT("result"));
		
		FilePath = JsonObject->GetStringField(TEXT("file_path"));
		const FString fileid = JsonObject->GetStringField(TEXT("file_id"));
		FileUrl = fileid;
		const FString msg = FString::Printf(TEXT("Upload Complete. URL=%s"), *FileUrl);
		SEND_MESSAGE(msg)
	}
	else if (eventType == ITMG_MAIN_EVNET_TYPE_PTT_DOWNLOAD_COMPLETE)
	{
		STT = "";
		FileUrl = "";
		const int32 result = JsonObject->GetIntegerField(TEXT("result"));
		FilePath = JsonObject->GetStringField(TEXT("file_path"));
		const FString fileid = JsonObject->GetStringField(TEXT("file_id"));
		const FString msg = FString::Printf(TEXT("download Complete. Path=%s"), *FilePath);
		SEND_MESSAGE(msg)
	}
	else if (eventType == ITMG_MAIN_EVNET_TYPE_PTT_SPEECH2TEXT_COMPLETE)
	{
		const int32 result = JsonObject->GetIntegerField(TEXT("result"));
		const FString text = JsonObject->GetStringField(TEXT("text"));
		STT = text;
		UE_LOG(LogTemp, Warning, TEXT("Text == %s"), *text);
		const FString fileid = JsonObject->GetStringField(TEXT("file_id"));
		const FString msg = FString::Printf(TEXT("%s convert to %s"), *fileid, *text);
		UE_LOG(LogTemp, Warning, TEXT("%s"), *msg);
		SEND_MESSAGE(msg)
	}
	else if (eventType == ITMG_MAIN_EVNET_TYPE_PTT_STREAMINGRECOGNITION_IS_RUNNING)
	{
		int32 nResult = JsonObject->GetIntegerField(TEXT("result"));
		FString text = JsonObject->GetStringField(TEXT("text"));
		STT = text;
		UE_LOG(LogTemp, Warning, TEXT("Streaming Text == %s"), *text);
		FString fileid = TEXT("STREAMINGRECOGNITION_IS_RUNNING");
		FString file_path = JsonObject->GetStringField(TEXT("file_path"));
		const FString msg = FString::Printf(TEXT("%s convert to %s"), *fileid, *text);
		UE_LOG(LogTemp, Warning, TEXT("%s"), *msg);
		SEND_MESSAGE(STT)
	}
	else if (eventType == ITMG_MAIN_EVNET_TYPE_PTT_STREAMINGRECOGNITION_COMPLETE)
	{
		int32 nResult = JsonObject->GetIntegerField(TEXT("result"));
		FString text = JsonObject->GetStringField(TEXT("text"));
		STT = text;
		UE_LOG(LogTemp, Warning, TEXT("Streaming Completed Text == %s"), *text);
		FString fileid = JsonObject->GetStringField(TEXT("file_id"));
		FString file_path = JsonObject->GetStringField(TEXT("file_path"));
		const FString msg = FString::Printf(TEXT("%s convert to %s"), *fileid, *text);
		UE_LOG(LogTemp, Warning, TEXT("%s"), *msg);
		SEND_MESSAGE(STT)
	}
#endif
}

void GMEAudioAdpter::SetMessageHandler(ITMGMessage* handler)
{
	MessageHandler = handler;
}

void GMEAudioAdpter::SetUserID(const FString& pUserID)
{
	UserID = pUserID;
}

void GMEAudioAdpter::StartRecord() const
{
	ITMGContextGetInstance()->GetAudioCtrl()->EnableMic(true);
	ITMGContextGetInstance()->GetAudioCtrl()->EnableSpeaker(true);
	
	const FString path = GetFilePath() + TEXT("/unreal_ptt_local.file");
	ITMGContextGetInstance()->GetPTT()->StartRecording(TCHAR_TO_UTF8(*path));
}

void GMEAudioAdpter::StartRecordingWithStreaming()
{
	ITMGContextGetInstance()->GetPTT()->StartRecordingWithStreamingRecognition(TCHAR_TO_UTF8(*FilePath));
}

void GMEAudioAdpter::StopRecord()
{
	ITMGContextGetInstance()->GetPTT()->StopRecording();
}

void GMEAudioAdpter::CancelRecord()
{
	ITMGContextGetInstance()->GetPTT()->CancelRecording();
}

void GMEAudioAdpter::Upload() const
{
	ITMGContextGetInstance()->GetPTT()->UploadRecordedFile(TCHAR_TO_UTF8(*FilePath));
}

void GMEAudioAdpter::Download() const
{
	const FString path = GetFilePath() + TEXT("/unreal_ptt_downloadl.file");
	ITMGContextGetInstance()->GetPTT()->DownloadRecordedFile(TCHAR_TO_UTF8(*FileUrl),TCHAR_TO_UTF8(*path));
}

void GMEAudioAdpter::Play() const
{
	ITMGContextGetInstance()->GetPTT()->PlayRecordedFile(TCHAR_TO_UTF8(*FilePath));
}

void GMEAudioAdpter::ConvertToText() const
{
	ITMGContextGetInstance()->GetPTT()->SpeechToText(TCHAR_TO_UTF8(*FileUrl));
}

int GMEAudioAdpter::InitGME()
{
	ITMGContextGetInstance()->SetAdvanceParams("DisableEncryptLog","1");
	ITMGContextGetInstance()->SetAppVersion("fangh.space");		// Just For Test
	
	#ifdef __NX__
	ITMGContextGetInstance()->SetLogPath("gme_log_cache:/GME");
#else
	FString logDir = FPaths::ProjectSavedDir();
	ITMGContextGetInstance()->SetLogPath(TCHAR_TO_UTF8(*logDir));
#endif

	const std::string sdkAppId = TCHAR_TO_UTF8(*SdkAppID);
	const std::string sdkAppKey = TCHAR_TO_UTF8(*SdkAppKey);
	const int nAppid = atoi(sdkAppId.c_str());
	//const char* userId = TCHAR_TO_UTF8(*UserID);
	const int ret = ITMGContextGetInstance()->Init(sdkAppId.c_str(), TCHAR_TO_UTF8(*UserID));
	ITMGContextGetInstance()->SetTMGDelegate(this);

	const int RetCode = (int) ITMGContextGetInstance()->CheckMicPermission();
	const FString msg = FString::Printf(TEXT("check Permission retcode =%d"), RetCode);
	GEngine->AddOnScreenDebugMessage(INDEX_NONE, 10.0f, FColor::Yellow, *msg);

	constexpr char strSig[128] = {0};
	unsigned int nLength = 128;
	nLength = QAVSDK_AuthBuffer_GenAuthBuffer(nAppid, "0", TCHAR_TO_UTF8(*UserID), sdkAppKey.c_str(), (unsigned char *)strSig, nLength);
	ITMGContextGetInstance()->GetPTT()->ApplyPTTAuthbuffer(strSig, nLength);
	ITMGContextGetInstance()->SetAdvanceParams("ForceHighQuanlityState","0");
	
	ITMGContextGetInstance()->GetAudioCtrl()->EnableMic(true);
	ITMGContextGetInstance()->GetAudioCtrl()->EnableSpeaker(true);

	ITMGContextGetInstance()->SetAdvanceParams("SetStreamingRecTimeOut","5");
	
	return ret;
}

void GMEAudioAdpter::UnInit()
{
	ITMGContextGetInstance()->Uninit();
}

void GMEAudioAdpter::OnTickGME()
{
	ITMGContextGetInstance()->Poll();
}

FString GMEAudioAdpter::GetFilePath() const
{
#if PLATFORM_ANDROID
	extern FString GExternalFilePath;
	FString path = GExternalFilePath + FString("/");
	return path;
#elif PLATFORM_IOS
	char buffer[256]={0};
	snprintf(buffer, sizeof(buffer), "%s%s", getenv("HOME"),"/Documents/");
	return buffer;
#endif
	return FPaths::ProjectSavedDir();
}
```



### 2. UE接口

`TencentSTT_Subsystem.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "Subsystems/GameInstanceSubsystem.h"
#include "GMEAudioAdpter.h"
#include "Engine/EngineTypes.h"
#include "TimerManager.h"
#include "TencentSTT_Subsystem.generated.h"

DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMessageHandler, FString, Value);

UCLASS(Blueprintable)
class GMESDK_API UTencentSTT_Subsystem : public UGameInstanceSubsystem, public ITMGMessage
{
	GENERATED_BODY()

public:
	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void InitSDK(const FString& AppID);

	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void StartListening();

	UFUNCTION()
	void OnTimer() const;
public:
	UPROPERTY(BlueprintAssignable)
	FMessageHandler MessageHandler;
private:
	FTimerHandle TimerHandle;

public:
	
	virtual void Deinitialize() override;
	virtual void OnMessage(const FString& message) override;

	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void SetUserID(const FString& pUserID);
	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void StartRecord();
	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void StartRecordStreaming();
	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void StopRecord();
	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void CancelRecord();
	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void Upload();
	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void Download();
	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void Play();
	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void ConvertToText();
	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	int InitGME();
	UFUNCTION()
	void OnTickGME();
public:
	UPROPERTY(BlueprintAssignable)
	FMessageHandler TXMessageNotify;
private:
	FString UserID;
	FString FilePath;
	FString FileUrl;
	FTimerHandle TXTimerHandle;

public:
	UFUNCTION(BlueprintPure, Category="FH|Tencent")
	void GetSTT(FString& STT)
	{
		STT = GMEAudioAdpter::Get()->STT;
	}

	UFUNCTION(BlueprintCallable, Category="FH|Tencent")
	void SetSdkIdAndKey(const FString& ID, const FString& Key)
	{
		GMEAudioAdpter::Get()->SdkAppID = ID;
		GMEAudioAdpter::Get()->SdkAppKey = Key;
	}

	UFUNCTION(BlueprintPure, Category="FH|Tencent|Utils")
	static UPARAM(Displayname="ResultStr") FString FixSTT_TrimAndChop(const FString& OriginStr, const int32 Count = 1);
};
```



`TencentSTT_Subsystem.cpp`

```c++
// Fill out your copyright notice in the Description page of Project Settings.


#include "TencentSTT_Subsystem.h"
#include "XFAdpter.h"
#include "Engine/World.h"
#include "Kismet/KismetStringLibrary.h"

void UTencentSTT_Subsystem::InitSDK(const FString& AppID)
{
#if PLATFORM_ANDROID
	XFAdpter::InitSDK(AppID);
#endif
	GetWorld()->GetTimerManager().SetTimer(TimerHandle, this, &UTencentSTT_Subsystem::OnTimer, 0.1, true);
}

void UTencentSTT_Subsystem::StartListening()
{
#if PLATFORM_ANDROID
	XFAdpter::StartListening();
#endif
}

void UTencentSTT_Subsystem::OnTimer() const
{
	if (XFAdpter::CurrentMessage == TEXT(""))
	{
		return;
	}
	MessageHandler.Broadcast(XFAdpter::CurrentMessage);
	XFAdpter::CurrentMessage = TEXT("");
}

void UTencentSTT_Subsystem::Deinitialize()
{
	GMEAudioAdpter::Get()->UnInit();
	Super::Deinitialize();
}

void UTencentSTT_Subsystem::OnMessage(const FString& message)
{
	TXMessageNotify.Broadcast(message);
}

void UTencentSTT_Subsystem::SetUserID(const FString& pUserID)
{
	GMEAudioAdpter::Get()->SetUserID(pUserID);
}

void UTencentSTT_Subsystem::StartRecord()
{
	GMEAudioAdpter::Get()->StartRecord();
}

void UTencentSTT_Subsystem::StartRecordStreaming()
{
	GMEAudioAdpter::Get()->StartRecordingWithStreaming();
}

void UTencentSTT_Subsystem::StopRecord()
{
	GMEAudioAdpter::Get()->StopRecord();
}

void UTencentSTT_Subsystem::CancelRecord()
{
	GMEAudioAdpter::Get()->CancelRecord();
}

void UTencentSTT_Subsystem::Upload()
{
	GMEAudioAdpter::Get()->Upload();
}

void UTencentSTT_Subsystem::Download()
{
	GMEAudioAdpter::Get()->Download();
}

void UTencentSTT_Subsystem::Play()
{
	GMEAudioAdpter::Get()->Play();
}

void UTencentSTT_Subsystem::ConvertToText()
{
	GMEAudioAdpter::Get()->ConvertToText();
}

int UTencentSTT_Subsystem::InitGME()
{
	GMEAudioAdpter::Get()->SetMessageHandler(this);
	GetWorld()->GetTimerManager().SetTimer(TXTimerHandle, this, &UTencentSTT_Subsystem::OnTickGME, 0.2, true);
	return GMEAudioAdpter::Get()->InitGME();
}

void UTencentSTT_Subsystem::OnTickGME()
{
	GMEAudioAdpter::Get()->OnTickGME();
}

FString UTencentSTT_Subsystem::FixSTT_TrimAndChop(const FString& OriginStr, const int32 Count)
{
	return UKismetStringLibrary::LeftChop(UKismetStringLibrary::TrimTrailing(UKismetStringLibrary::Trim(OriginStr)), Count);
}
```


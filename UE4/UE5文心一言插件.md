# UE5文心一言插件



[toc]



### 0. github

[UE5文心一言插件](https://github.com/FHangH/WenXinYiYan)



### 1. 结构设计

`WXYY.h`

```c++
#pragma once

#include "WXYY.generated.h"

DECLARE_LOG_CATEGORY_EXTERN(LogWenXin, Log, All);
DEFINE_LOG_CATEGORY(LogWenXin);

UENUM(BlueprintType)
enum class ERole : uint8
{
	ER_User			UMETA(Displayname="user"),
	ER_Assistant	UMETA(Displayname="assistant")
};

USTRUCT(BlueprintType)
struct FChatMessage
{
	GENERATED_USTRUCT_BODY()

	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
	ERole Role;
	
	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
	FString Content;

	FChatMessage() : Role(ERole::ER_User), Content(""){}

	explicit FChatMessage(const ERole NewRole)
	{
		Role = NewRole;
		Content = "";
	}
};

USTRUCT(BlueprintType)
struct FWXConfigKey
{
	GENERATED_USTRUCT_BODY()

	FString AppID;
	FString AppSecret;

	FWXConfigKey() : AppID(""), AppSecret(""){}
};

UCLASS()
class UWXYY : public UObject
{
	GENERATED_BODY()
	
public:
	static FString AccessToken;
	
	static FORCEINLINE void SetAccessToken(const FString& Token)
	{
		AccessToken = Token;
	}

	UFUNCTION(BlueprintPure, Category="FH|WXYY")
	static FORCEINLINE FString ConverToStr(const ERole& Role)
	{
		return Role == ERole::ER_User ? FString("user") : Role == ERole::ER_Assistant ? FString("assistant") : 
        FString("user");
	}
};

FString UWXYY::AccessToken{};
```



### 2. 获取AccessToken

`WXYY_AccessToken.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "HttpModule.h"
#include "Interfaces/IHttpRequest.h"
#include "Kismet/BlueprintAsyncActionBase.h"
#include "WXYY_AccessToken.generated.h"

DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FResponseAccessTokenDelegate, FString, Result);

UCLASS()
class FH_WENXINYIIYAN_API UGetAccessToken : public UBlueprintAsyncActionBase
{
	GENERATED_BODY()

public:
	UPROPERTY(BlueprintAssignable)
	FResponseAccessTokenDelegate OnSuccess;

	UPROPERTY(BlueprintAssignable)
	FResponseAccessTokenDelegate OnFail;
	
	UFUNCTION(BlueprintCallable, Category="FH|WXYY", meta=(BlueprintInternalUseOnly="true"))
	static UGetAccessToken* GetAccessToken(FString API, FString Secret);

private:
	TSharedRef<IHttpRequest, ESPMode::ThreadSafe> RequestToken = FHttpModule::Get().CreateRequest();
	
	void OnHttpsRequestToken(const FString& API, const FString& Secret);

	void OnDeserializeResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, const bool Success);

	FORCEINLINE bool IsRequestProcessing() const {return RequestToken->GetStatus() == EHttpRequestStatus::Processing;}
};
```



`WXYY_AccessToken.cpp`

```c++
#include "WXYY_AccessToken.h"
#include "HttpManager.h"
#include "HttpModule.h"
#include "Interfaces/IHttpResponse.h"
#include "WXYY.h"

UGetAccessToken* UGetAccessToken::GetAccessToken(FString API, FString Secret)
{
	const auto RequestTokenObject = NewObject<UGetAccessToken>();
	RequestTokenObject->OnHttpsRequestToken(API, Secret);
	return RequestTokenObject;
}

void UGetAccessToken::OnHttpsRequestToken(const FString& API, const FString& Secret)
{
	AddToRoot();
	
	const auto URL =
		FString::Printf(
			TEXT("https://aip.baidubce.com/oauth/2.0/token?client_id=%s&client_secret=%s&grant_type=client_credentials"),
			*API, *Secret);

	RequestToken = FHttpModule::Get().CreateRequest();
	RequestToken->SetVerb(TEXT("POST"));
	RequestToken->SetURL(URL);
	RequestToken->SetHeader(TEXT("Content-Type"), TEXT("application/json"));
	RequestToken->SetHeader(TEXT("Accept"), TEXT("application/json"));

	RequestToken->OnProcessRequestComplete().BindUObject(this, &UGetAccessToken::OnDeserializeResponse);

	if (RequestToken->ProcessRequest() || IsRequestProcessing())
	{
		UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan Start ======"));
		UE_LOG(LogWenXin, Warning, TEXT("Request Process"));
		FHttpModule::Get().GetHttpManager().Tick(0.f);
		
		if (RequestToken->GetStatus() == EHttpRequestStatus::Failed)
		{
			OnFail.Broadcast("null");
			UE_LOG(LogWenXin, Warning, TEXT("Request Fail"));
			UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));
		}
	
		RemoveFromRoot();
	}
	UE_LOG(LogWenXin, Warning, TEXT("Request Complete"));
}

void UGetAccessToken::OnDeserializeResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, const bool Success)
{
	FString Result;
	if (Success && Response->GetResponseCode() == EHttpResponseCodes::Ok)
	{
		UE_LOG(LogWenXin, Warning, TEXT("Response Success"));
		const auto ResponseContent = Response->GetContentAsString();
		const auto JsonReader = TJsonReaderFactory<>::Create(ResponseContent);
    
		if (TSharedPtr<FJsonObject> JsonObject; FJsonSerializer::Deserialize(JsonReader, JsonObject))
		{
			UE_LOG(LogWenXin, Warning, TEXT("Deserialize Response"));
			Result = JsonObject->GetStringField("access_token");
			OnSuccess.Broadcast(Result);
			UWXYY::SetAccessToken(Result);
			UE_LOG(LogWenXin, Warning, TEXT("Request Result = %s"), *Result);
			UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));
		}
	}
	else
	{
		OnFail.Broadcast("null");
		UE_LOG(LogWenXin, Warning, TEXT("Request Result = %s"), *Result);
		UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));
	}

	RemoveFromRoot();
}
```



### 3. 获取对话内容

`WXYY_Chat.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "Kismet/BlueprintAsyncActionBase.h"
#include "WXYY.h"
#include "Interfaces/IHttpRequest.h"
#include "WXYY_Chat.generated.h"

DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FResponseChatMessage, FChatMessage, ChatMessage);

UCLASS()
class FH_WENXINYIIYAN_API USendChatMessage : public UBlueprintAsyncActionBase
{
	GENERATED_BODY()

public:
	UPROPERTY(BlueprintAssignable)
	FResponseChatMessage OnCompleted;

	UPROPERTY(BlueprintAssignable)
	FResponseChatMessage OnFail;

	UFUNCTION(BlueprintCallable, Category="FH|WXYY", meta=(BlueprintInternalUseOnly="true"))
	static USendChatMessage* SendChatMessage(FChatMessage ChatMessage);
	
	UFUNCTION(BlueprintCallable, Category="FH|WXYY")
	static void CancelMessageRequest();

private:
	inline static USendChatMessage* SendChatHandler = nullptr;
	TSharedPtr<IHttpRequest, ESPMode::ThreadSafe> RequestChat;
	
	void OnHttpsRequestChatMessage(const FChatMessage& ChatMessage);
	
	void OnDeserializeResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, const bool Success);

	static bool IsRequestProcessing();
};

UCLASS()
class FH_WENXINYIIYAN_API USendChatMessageByStream : public UBlueprintAsyncActionBase
{
	GENERATED_BODY()

public:
	UPROPERTY(BlueprintAssignable)
	FResponseChatMessage OnUpdate;
	
	UPROPERTY(BlueprintAssignable)
	FResponseChatMessage OnCompleted;

	UPROPERTY(BlueprintAssignable)
	FResponseChatMessage OnFail;

	UFUNCTION(BlueprintCallable, Category="FH|WXYY", meta=(BlueprintInternalUseOnly="true"))
	static USendChatMessageByStream* SendChatMessageByStream(FChatMessage ChatMessage);

	UFUNCTION(BlueprintCallable, Category="FH|WXYY")
	static void CancelStreamMessageRequest();

private:
	FString AllResult;
	FString AllResponse;
	FString TempResponse;
	
	inline static USendChatMessageByStream* SendStreamChatHandler = nullptr;
	TSharedPtr<IHttpRequest, ESPMode::ThreadSafe> RequestChat;
	
	void OnHttpsRequestChatMessage(const FChatMessage& ChatMessage);

	void OnUpdateResponse(FHttpRequestPtr HttpRequest, int32 BytesSent, int32 BytesReceived);

	void OnDeserializeResponseUpdate();
	
	void OnCompletedResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, const bool Success);

	static bool IsRequestProcessing();
};
```



`WXYY_Chat.cpp`

```c++
#include "WXYY_Chat.h"
#include "HttpManager.h"
#include "HttpModule.h"
#include "Interfaces/IHttpResponse.h"

// Send Chat Message
USendChatMessage* USendChatMessage::SendChatMessage(FChatMessage ChatMessage)
{
	const auto RequestObject = NewObject<USendChatMessage>();
	RequestObject->OnHttpsRequestChatMessage(ChatMessage);
	SendChatHandler = RequestObject;
	return RequestObject;
}

void USendChatMessage::OnHttpsRequestChatMessage(const FChatMessage& ChatMessage)
{
	AddToRoot();

	const auto URL =
		FString::Printf(
			TEXT("https://aip.baidubce.com/rpc/2.0/ai_custom/v1/wenxinworkshop/chat/eb-instant?access_token=%s"),
			*UWXYY::AccessToken);

	RequestChat = FHttpModule::Get().CreateRequest();
	RequestChat->SetVerb(TEXT("POST"));
	RequestChat->SetURL(URL);
	RequestChat->SetHeader(TEXT("Content-Type"), TEXT("application/json"));
	RequestChat->SetHeader(TEXT("Accept"), TEXT("application/json"));
	RequestChat->SetContentAsString(
		FString::Printf(
			TEXT("{\"messages\": [{\"role\":\"%s\",\"content\":\"%s\"}]}"),
			*UWXYY::ConverToStr(ChatMessage.Role), *ChatMessage.Content));

	RequestChat->OnProcessRequestComplete().BindUObject(this, &USendChatMessage::OnDeserializeResponse);

	if (RequestChat->ProcessRequest() || IsRequestProcessing())
	{
		UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan Start ======"));
		UE_LOG(LogWenXin, Warning, TEXT("Request Process"));
		FHttpModule::Get().GetHttpManager().Tick(0.f);

		if (RequestChat->GetStatus() == EHttpRequestStatus::Failed)
		{
			FChatMessage Message;
			Message.Role = ERole::ER_Assistant;
			Message.Content = "";
			OnFail.Broadcast(Message);
			UE_LOG(LogWenXin, Warning, TEXT("Request Fail"));
			UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));
		}
	}
	UE_LOG(LogWenXin, Warning, TEXT("Request Complete"));
}

void USendChatMessage::OnDeserializeResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, const bool Success)
{
	FChatMessage ChatMessage;
	
	if (Success && Response->GetResponseCode() == EHttpResponseCodes::Ok)
	{
		UE_LOG(LogWenXin, Warning, TEXT("Response Success"));
		const auto ResponseContent = Response->GetContentAsString();
		const auto JsonReader = TJsonReaderFactory<>::Create(ResponseContent);
    
		if (TSharedPtr<FJsonObject> JsonObject; FJsonSerializer::Deserialize(JsonReader, JsonObject))
		{
			UE_LOG(LogWenXin, Warning, TEXT("Deserialize Response"));
			ChatMessage.Role = ERole::ER_Assistant;
			ChatMessage.Content = JsonObject->GetStringField("result");
			if (ChatMessage.Content == "")
			{
				OnFail.Broadcast(ChatMessage);
				UE_LOG(LogWenXin, Warning, TEXT("Request Result = %s : null"), *UWXYY::ConverToStr(ChatMessage.Role));
				UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));
				
				RemoveFromRoot();
				return;
			}
			OnCompleted.Broadcast(ChatMessage);
			UE_LOG(LogWenXin, Warning, TEXT("Request Result = %s : %s"), *UWXYY::ConverToStr(ChatMessage.Role), *ChatMessage.Content);
			UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));
		}
	}
	else
	{
		ChatMessage.Role = ERole::ER_Assistant;
		ChatMessage.Content = "";
		OnFail.Broadcast(ChatMessage);
		UE_LOG(LogWenXin, Warning, TEXT("Request Result = %s : null"), *UWXYY::ConverToStr(ChatMessage.Role));
		UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));
	}
    
	RemoveFromRoot();
}

bool USendChatMessage::IsRequestProcessing()
{
	if (!SendChatHandler || !SendChatHandler->RequestChat) return false;
	return SendChatHandler->RequestChat->GetStatus() == EHttpRequestStatus::Processing;
}

void USendChatMessage::CancelMessageRequest()
{
	if (!SendChatHandler || !SendChatHandler->RequestChat) return;
	if (SendChatHandler->RequestChat->GetStatus() == EHttpRequestStatus::Processing)
	{
		SendChatHandler->RequestChat->CancelRequest();
		UE_LOG(LogWenXin, Warning, TEXT("Request Cancle"))
	}
}

// Send Chat Message By Stream
USendChatMessageByStream* USendChatMessageByStream::SendChatMessageByStream(FChatMessage ChatMessage)
{
	const auto RequestObject = NewObject<USendChatMessageByStream>();
	RequestObject->OnHttpsRequestChatMessage(ChatMessage);
	SendStreamChatHandler = RequestObject;
	return RequestObject;
}

void USendChatMessageByStream::OnHttpsRequestChatMessage(const FChatMessage& ChatMessage)
{
	AddToRoot();

	AllResult = "";
	TempResponse = "";
	AllResponse = "";
	
	const auto URL =
		FString::Printf(
			TEXT("https://aip.baidubce.com/rpc/2.0/ai_custom/v1/wenxinworkshop/chat/eb-instant?access_token=%s"),
			*UWXYY::AccessToken);

	RequestChat = FHttpModule::Get().CreateRequest();
	RequestChat->SetVerb(TEXT("POST"));
	RequestChat->SetURL(URL);
	RequestChat->SetHeader(TEXT("Content-Type"), TEXT("application/json"));
	RequestChat->SetHeader(TEXT("Accept"), TEXT("application/json"));
	RequestChat->SetContentAsString(
		FString::Printf(
			TEXT("{\"messages\": [{\"role\":\"%s\",\"content\":\"%s\"}],\"stream\":true}"),
			*UWXYY::ConverToStr(ChatMessage.Role), *ChatMessage.Content));

	RequestChat->OnRequestProgress().BindUObject(this, &USendChatMessageByStream::OnUpdateResponse);
	RequestChat->OnProcessRequestComplete().BindUObject(this, &USendChatMessageByStream::OnCompletedResponse);

	if (RequestChat->ProcessRequest() || IsRequestProcessing())
	{
		UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan Start ======"));
		UE_LOG(LogWenXin, Warning, TEXT("Request Stream Process"));
		FHttpModule::Get().GetHttpManager().Tick(0.f);

		if (RequestChat->GetStatus() == EHttpRequestStatus::Failed)
		{
			FChatMessage Message;
			Message.Role = ERole::ER_Assistant;
			Message.Content = "";
			OnFail.Broadcast(Message);
			UE_LOG(LogWenXin, Warning, TEXT("Request Stream Fail"));
			UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));
		}
	}
	UE_LOG(LogWenXin, Warning, TEXT("Request Stream Complete"));
}

void USendChatMessageByStream::OnUpdateResponse(FHttpRequestPtr HttpRequest, int32 BytesSent, int32 BytesReceived)
{
	UE_LOG(LogWenXin, Warning, TEXT("Request Process By Stream"));
	const auto Response = HttpRequest->GetResponse()->GetContentAsString();

	if (AllResponse.IsEmpty())
	{
		AllResponse = TempResponse = Response;
		OnDeserializeResponseUpdate();
	}
	else
	{
		TempResponse = Response;
		TempResponse.RemoveFromStart(AllResponse);
		OnDeserializeResponseUpdate();
		AllResponse = Response;
	}
}

void USendChatMessageByStream::OnDeserializeResponseUpdate()
{
	if (TempResponse.StartsWith("data:"))
	{
		auto JsonData = TempResponse.RightChop(6);
		JsonData.TrimStartAndEndInline();

		const auto JsonReader = TJsonReaderFactory<>::Create(JsonData);
		if (TSharedPtr<FJsonObject> JsonObject; FJsonSerializer::Deserialize(JsonReader, JsonObject))
		{
			UE_LOG(LogWenXin, Warning, TEXT("Deserialize Stream Response"));
			FChatMessage ChatMessage;
			ChatMessage.Role = ERole::ER_Assistant;
			ChatMessage.Content = JsonObject->GetStringField("result");
			OnUpdate.Broadcast(ChatMessage);
			AllResult += ChatMessage.Content;
			UE_LOG(LogWenXin, Warning, TEXT("Stream Result = %s"), *ChatMessage.Content);
		}
	}
}

void USendChatMessageByStream::OnCompletedResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, const bool Success)
{
	FChatMessage ChatMessage;
	if (Success && Response->GetResponseCode() == EHttpResponseCodes::Ok)
	{
		UE_LOG(LogWenXin, Warning, TEXT("Response Stream Success"));
		ChatMessage.Role = ERole::ER_Assistant;
		ChatMessage.Content = AllResult;
		if (ChatMessage.Content == "")
		{
			OnFail.Broadcast(ChatMessage);
			UE_LOG(LogWenXin, Warning, TEXT("Stream Result = %s : null"), *UWXYY::ConverToStr(ChatMessage.Role));
			UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));

			RemoveFromRoot();
			return;
		}
		OnCompleted.Broadcast(ChatMessage);
		UE_LOG(LogWenXin, Warning, TEXT("Request Stream Result = %s : %s"), *UWXYY::ConverToStr(ChatMessage.Role), *ChatMessage.Content);
		UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));
	}
	else
	{
		ChatMessage.Role = ERole::ER_Assistant;
		ChatMessage.Content = "";
		OnFail.Broadcast(ChatMessage);
		UE_LOG(LogWenXin, Warning, TEXT("Request Stream Result = %s : null"), *UWXYY::ConverToStr(ChatMessage.Role));
		UE_LOG(LogWenXin, Warning, TEXT("====== WenXinYiYan End ======"));
	}

	RemoveFromRoot();
}

bool USendChatMessageByStream::IsRequestProcessing()
{
	if (!SendStreamChatHandler || !SendStreamChatHandler->RequestChat) return false;
	return SendStreamChatHandler->RequestChat->GetStatus() == EHttpRequestStatus::Processing;
}

void USendChatMessageByStream::CancelStreamMessageRequest()
{
	if (!SendStreamChatHandler || !SendStreamChatHandler->RequestChat) return;
	if (SendStreamChatHandler->RequestChat->GetStatus() == EHttpRequestStatus::Processing)
	{
		SendStreamChatHandler->RequestChat->CancelRequest();
		UE_LOG(LogWenXin, Warning, TEXT("Request Cancle"))
	}
}
```


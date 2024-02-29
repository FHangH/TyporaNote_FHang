# UE5腾讯AI绘画插件



[toc]



### 0. github

[UE5腾讯AI绘画插件](https://github.com/FHangH/TencentAIArt)



### 1. 腾讯HTTP V3验证

`TencentAIArtV3.h`

```c++
#pragma once

#include "TencentAIArtConfig.h"

using namespace std;

DECLARE_LOG_CATEGORY_EXTERN(LogTencentAIArt, Log, All);
inline DEFINE_LOG_CATEGORY(LogTencentAIArt);

class TencentAIArtV3
{
private:
	TencentAIArtV3() = default;

	string SECRET_ID;
    string SECRET_KEY;
    string service;
    string host;
    string region;
    string action;
    string version;

	string payload;
    int64_t timestamp;
    string date;

    string hashedRequestPayload;
    string httpRequestMethod;
    string canonicalUri;
    string canonicalQueryString;
    string lower;
    string canonicalHeaders;
    string signedHeaders;
    string canonicalRequest;

    string algorithm;
    string RequestTimestamp;
    string credentialScope;
    string hashedCanonicalRequest;
    string stringToSign;


    string kKey;
    string kDate;
    string kService;
    string kSigning;
    string signature;
    string authorization;

	EReqImgType ReqImageType;

public:
	static TencentAIArtV3& Get()
	{
		static TencentAIArtV3 TAAV3;
		return TAAV3;
	}

	void initConfig(const FTencentAIArtConfig& AIArtConfig);
	void initTTI_Or_ITI(const FPromptTTIConfig& PromptTTIConfig);
	void getReqImageType(EReqImgType& ReqImgType) const;
	
	string get_data(const int64_t &timestamp);

	string int2str(int64_t n);

	string sha256Hex(const string &str);
	
	string HmacSha256(const string &key, const string &input);

	string HexEncode(const string &input);

	string GetPayload() {return payload;}
	string Gettimestamp() {return int2str(timestamp);}
	string GetHost() {return host;}
	string GetAction() {return action;}
	string GetVersion() {return version;}
	string GetRegion() {return region;}
	string GetAuthorization();
};
```



`TencentAIArtV3.cpp`

```c++
#include "TencentAIArtV3.h"
#include <iomanip>
#include <sstream>
#include <stdio.h>
#include <time.h>

#define UI UI_ST
THIRD_PARTY_INCLUDES_START
#include "openssl/sha.h"
#include "openssl/hmac.h"
THIRD_PARTY_INCLUDES_END
#undef UI

void TencentAIArtV3::initConfig(const FTencentAIArtConfig& AIArtConfig)
{
	SECRET_ID = TCHAR_TO_UTF8(*AIArtConfig.Secret_ID);
	SECRET_KEY =  TCHAR_TO_UTF8(*AIArtConfig.Secret_Key);
	service =  TCHAR_TO_UTF8(*AIArtConfig.Service);
	host =  TCHAR_TO_UTF8(*UTencentConfig::ConvStr_EHost(AIArtConfig.Host));
	region =  TCHAR_TO_UTF8(*UTencentConfig::ConvStr_Region(AIArtConfig.Region));
	action =  TCHAR_TO_UTF8(*UTencentConfig::ConvStr_EAction(AIArtConfig.Action));
	version =  TCHAR_TO_UTF8(*AIArtConfig.Version);
}

void TencentAIArtV3::initTTI_Or_ITI(const FPromptTTIConfig& PromptTTIConfig)
{
	const FString TTI_Prompt = PromptTTIConfig.Prompt;
	const FString TTI_Styles = UTencentConfig::ConvStr_EStylesTTI(PromptTTIConfig.Styles);
	const FString TTI_Resolution = UTencentConfig::ConvStr_EResolution(PromptTTIConfig.Resolution);
	const int TTI_LogoAdd = UTencentConfig::ConvStr_ELogoAdd(PromptTTIConfig.LogoAdd);
	ReqImageType = PromptTTIConfig.ReqImgType;
	const FString TTI_ReqImgType = UTencentConfig::ConvStr_ERspImgType(ReqImageType);
	const FString Text =
		FString::Printf(
			TEXT("{\"Prompt\":\"%s\",\"Styles\":[\"%s\"],\"ResultConfig\":{\"Resolution\":\"%s\"},\"LogoAdd\":%d,\"RspImgType\":\"%s\"}"),
			*TTI_Prompt, *TTI_Styles, *TTI_Resolution, TTI_LogoAdd, *TTI_ReqImgType);
	
	payload = string(TCHAR_TO_UTF8(*Text));
	UE_LOG(LogTencentAIArt, Warning, TEXT("payload = %s"), *FString(UTF8_TO_TCHAR(payload.c_str())));
}

void TencentAIArtV3::getReqImageType(EReqImgType& ReqImgType) const
{
	ReqImgType = ReqImageType;
}

string TencentAIArtV3::get_data(const int64_t& Timestamp)
{
	char buff[20] = {0};
	tm sttime;
	gmtime_s(&sttime, &Timestamp);
	strftime(buff, sizeof(buff), "%Y-%m-%d", &sttime);
	string utcDate = string(buff);
	return utcDate;
}

string TencentAIArtV3::int2str(const int64_t n)
{
	std::stringstream ss;
	ss << n;
	return ss.str();
}

string TencentAIArtV3::sha256Hex(const string& str)
{
	unsigned char hash[SHA256_DIGEST_LENGTH];
	SHA256_CTX sha256;
	SHA256_Init(&sha256);
	SHA256_Update(&sha256, str.c_str(), str.size());
	SHA256_Final(hash, &sha256);
	std::string NewString = "";
	for(int i = 0; i < SHA256_DIGEST_LENGTH; i++)
	{
		char buf[3];
		snprintf(buf, sizeof(buf), "%02x", hash[i]);
		NewString = NewString + buf;
	}
	return NewString;
}

string TencentAIArtV3::HmacSha256(const string& key, const string& input)
{
	unsigned char hash[32];

	HMAC_CTX *h;
#if OPENSSL_VERSION_NUMBER < 0x10100000L
	HMAC_CTX hmac;
	HMAC_CTX_init(&hmac);
	h = &hmac;
#else
	h = HMAC_CTX_new();
#endif

	HMAC_Init_ex(h, &key[0], key.length(), EVP_sha256(), nullptr);
	HMAC_Update(h, ( unsigned char* )&input[0], input.length());
	unsigned int len = 32;
	HMAC_Final(h, hash, &len);

#if OPENSSL_VERSION_NUMBER < 0x10100000L
	HMAC_CTX_cleanup(h);
#else
	HMAC_CTX_free(h);
#endif

	std::stringstream ss;
	ss << std::setfill('0');
	for (unsigned int i = 0; i < len; i++)
	{
		ss  << hash[i];
	}
	return (ss.str());
}

string TencentAIArtV3::HexEncode(const string& input)
{
	const size_t len = input.length();

	string output;
	output.reserve(2 * len);
	for (size_t i = 0; i < len; ++i)
	{
		static const char* const lut = "0123456789abcdef";
		const unsigned char c = input[i];
		output.push_back(lut[c >> 4]);
		output.push_back(lut[c & 15]);
	}
	return output;
}

string TencentAIArtV3::GetAuthorization()
{
	timestamp = FDateTime::UtcNow().ToUnixTimestamp();
	date = get_data(timestamp);
	
	// ************* 步骤 1：拼接规范请求串 *************
	hashedRequestPayload = sha256Hex(payload);
	UE_LOG(LogTencentAIArt, Warning, TEXT("hashedRequestPayload = %s"), *FString(hashedRequestPayload.c_str()))

	httpRequestMethod = "POST";
	canonicalUri = "/";
	canonicalQueryString = "";
	lower = action;
	std::transform(action.begin(), action.end(), lower.begin(), ::tolower);
	canonicalHeaders = string("content-type:application/json\n")
			+ "host:" + host + "\n"
			+ "x-tc-action:" + lower + "\n";
	signedHeaders = "content-type;host;x-tc-action";
	canonicalRequest = httpRequestMethod + "\n"
			+ canonicalUri + "\n"
			+ canonicalQueryString + "\n"
			+ canonicalHeaders + "\n"
			+ signedHeaders + "\n"
			+ hashedRequestPayload;

	UE_LOG(LogTencentAIArt, Warning, TEXT("canonicalRequest = %s"), *FString(canonicalRequest.c_str()))

	// ************* 步骤 2：拼接待签名字符串 *************
	algorithm = "TC3-HMAC-SHA256";
	RequestTimestamp = int2str(timestamp);
	credentialScope = date + "/" + service + "/" + "tc3_request";
	hashedCanonicalRequest = sha256Hex(canonicalRequest);
	stringToSign = algorithm + "\n" + RequestTimestamp + "\n" + credentialScope + "\n" + hashedCanonicalRequest;

	UE_LOG(LogTencentAIArt, Warning, TEXT("stringToSign = %s"), *FString(stringToSign.c_str()))

	// ************* 步骤 3：计算签名 ***************
	kKey = "TC3" + SECRET_KEY;
	kDate = HmacSha256(kKey, date);
	kService = HmacSha256(kDate, service);
	kSigning = HmacSha256(kService, "tc3_request");
	signature = HexEncode(HmacSha256(kSigning, stringToSign));

	// ************* 步骤 4：拼接 Authorization *************
	authorization = algorithm + " " + "Credential=" + SECRET_ID + "/" + credentialScope + ", "
			+ "SignedHeaders=" + signedHeaders + ", " + "Signature=" + signature;

	UE_LOG(LogTencentAIArt, Warning, TEXT("authorization = %s"), *FString(authorization.c_str()))
	
	return authorization;
}
```



### 2. API

`TencentAiArt.h`

```c++
#pragma once

#include "CoreMinimal.h"
#include "TencentAIArtConfig.h"
#include "Interfaces/IHttpRequest.h"
#include "Kismet/BlueprintAsyncActionBase.h"
#include "TencentAiArt.generated.h"

DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FResponseDelegate, FString, Result);

UCLASS(Blueprintable)
class FH_TENCENTAIART_API UTencentAiArt : public UBlueprintAsyncActionBase
{
	GENERATED_BODY()
	
public:
	UPROPERTY(BlueprintAssignable)
	FResponseDelegate OnCompleted;
	
	UPROPERTY(BlueprintAssignable)
	FResponseDelegate OnFail;

	UFUNCTION(BlueprintCallable, Category="FH|Tencent|AI Art")
	static void InitConfig(const FTencentAIArtConfig& AIArtConfig);

	UFUNCTION(BlueprintCallable, Category="FH|Tencent|AI Art")
	static void InitTTI_Or_ITI(const FPromptTTIConfig& PromptTTIConfig);
	//void InitITI(){}

	UFUNCTION(BlueprintCallable, Category="FH|Tencent|AI Art", meta=(BlueprintInternalUseOnly="true", WorldContext = 
                                                                     "WorldContextObject"))
	static UTencentAiArt* ResultImageByRequest(UObject* WorldContextObject);

	UFUNCTION(BlueprintPure, Category="FH|Tencent|AI Art")
	static UTexture2D* Base64ToTexture2D(const FString& Base64Str);

	UFUNCTION(BlueprintPure, Category="FH|Tencent|AI Art")
	static void GetRequestImageType(EReqImgType& ReqImgType);
	
	UFUNCTION(BlueprintCallable, Category="FH|Tencent|AI Art")
	static void CancelRequestImage();

private:
	inline static UTencentAiArt* AIArtHandler = nullptr;
	TSharedPtr<IHttpRequest, ESPMode::ThreadSafe> HttpRequestImage/* = FHttpModule::Get().CreateRequest()*/;
	
	void OnResultImageByRequest();
	void OnRequestCompleted(FHttpRequestPtr HttpRequest, FHttpResponsePtr HttpResponse, bool bWasSuccessful);
};
```



`TencentAiArt.cpp`

```c++
#include "TencentAiArt.h"
#include "Interfaces/IHttpResponse.h"
#include "HttpManager.h"
#include "HttpModule.h"
#include "TencentAIArtV3.h"

void UTencentAiArt::InitConfig(const FTencentAIArtConfig& AIArtConfig)
{
	TencentAIArtV3::Get().initConfig(AIArtConfig);
}

void UTencentAiArt::InitTTI_Or_ITI(const FPromptTTIConfig& PromptTTIConfig)
{
	TencentAIArtV3::Get().initTTI_Or_ITI(PromptTTIConfig);
}

UTencentAiArt* UTencentAiArt::ResultImageByRequest(UObject* WorldContextObject)
{
	const auto RequestObject = NewObject<UTencentAiArt>();
	RequestObject->OnResultImageByRequest();
	RequestObject->RegisterWithGameInstance(WorldContextObject);
	AIArtHandler = RequestObject;
	return RequestObject;
}

UTexture2D* UTencentAiArt::Base64ToTexture2D(const FString& Base64Str)
{
	return UTencentConfig::Base64ToTexture2D(Base64Str);
}

void UTencentAiArt::GetRequestImageType(EReqImgType& ReqImgType)
{
	TencentAIArtV3::Get().getReqImageType(ReqImgType);
}

void UTencentAiArt::OnResultImageByRequest()
{
	AddToRoot();
	
	const FString url = "https://" + FString(TencentAIArtV3::Get().GetHost().c_str());

	HttpRequestImage = FHttpModule::Get().CreateRequest();
	HttpRequestImage->SetVerb("POST");
	HttpRequestImage->SetURL(url);
	HttpRequestImage->SetHeader("Authorization", TencentAIArtV3::Get().GetAuthorization().c_str());
	HttpRequestImage->SetHeader("Content-Type", "application/json");
	HttpRequestImage->SetHeader("Host", TencentAIArtV3::Get().GetHost().c_str());

	if (FString(TencentAIArtV3::Get().GetAction().c_str()) == "ImageToImage")
	{
		HttpRequestImage->SetHeader("X-TC-Action", FString("ImageToImage"));
	}
	HttpRequestImage->SetHeader("X-TC-Action", FString("TextToImage"));
	HttpRequestImage->SetHeader("X-TC-Timestamp", TencentAIArtV3::Get().Gettimestamp().c_str());
	HttpRequestImage->SetHeader("X-TC-Version", TencentAIArtV3::Get().GetVersion().c_str());
	HttpRequestImage->SetHeader("X-TC-Region", TencentAIArtV3::Get().GetRegion().c_str());
	HttpRequestImage->SetContentAsString(FString(UTF8_TO_TCHAR(TencentAIArtV3::Get().GetPayload().c_str())));
	
	HttpRequestImage->OnProcessRequestComplete().BindUObject(this, &UTencentAiArt::OnRequestCompleted);
	HttpRequestImage->OnRequestProgress().BindLambda([](FHttpRequestPtr HttpRequest, int32 BytesSent, int32 BytesReceived)
	{
		UE_LOG(LogTencentAIArt, Warning, TEXT("Is Request Process, wait"))
	});
	
	if (HttpRequestImage->ProcessRequest())
	{
		UE_LOG(LogTencentAIArt, Warning, TEXT("====== TencentAIArt Start ======"));
		UE_LOG(LogTencentAIArt, Warning, TEXT("Request Process"));
		FHttpModule::Get().GetHttpManager().Tick(0.f);

		if (HttpRequestImage->GetStatus() == EHttpRequestStatus::Failed)
		{
			OnFail.Broadcast(FString("null"));
			UE_LOG(LogTencentAIArt, Warning, TEXT("Request Failed"));
			UE_LOG(LogTencentAIArt, Warning, TEXT("====== TencentAIArt End ======"));
		}
	}
}

void UTencentAiArt::OnRequestCompleted(FHttpRequestPtr HttpRequest, FHttpResponsePtr HttpResponse, bool bWasSuccessful)
{
	UE_LOG(LogTencentAIArt, Warning, TEXT("Request Completed"));
	if (bWasSuccessful && HttpResponse.IsValid())
	{
		UE_LOG(LogTencentAIArt, Warning, TEXT("Request Successed"));
		const FString ResponseContent = HttpResponse->GetContentAsString();

		UE_LOG(LogTencentAIArt, Warning, TEXT("Response Content = %s"), *ResponseContent);
		// 解析响应内容
		TSharedPtr<FJsonObject> JsonObject;
		const TSharedRef<TJsonReader<>> JsonReader = TJsonReaderFactory<>::Create(ResponseContent);
		if (FJsonSerializer::Deserialize(JsonReader, JsonObject))
		{
			if (JsonObject->HasField("Response"))
			{
				UE_LOG(LogTencentAIArt, Warning, TEXT("Get Response"));
				const TSharedPtr<FJsonObject> ResponseObject = JsonObject->GetObjectField("Response");
				if (ResponseObject->HasField("ResultImage"))
				{
					UE_LOG(LogTencentAIArt, Warning, TEXT("Get ResultImage"));
					const FString ImageResult = ResponseObject->GetStringField("ResultImage");

					OnCompleted.Broadcast(ImageResult);
					UE_LOG(LogTencentAIArt, Warning, TEXT("Result = %s"), *ImageResult);
					UE_LOG(LogTencentAIArt, Warning, TEXT("====== TencentAIArt End ======"));
				}
				else
				{
					OnFail.Broadcast(FString("null"));
					UE_LOG(LogTencentAIArt, Warning, TEXT("Request Error"));
					UE_LOG(LogTencentAIArt, Warning, TEXT("====== TencentAIArt End ======"));
				}
			}
			else
			{
				OnFail.Broadcast(FString("null"));
				UE_LOG(LogTencentAIArt, Warning, TEXT("Deserialize Error"));
				UE_LOG(LogTencentAIArt, Warning, TEXT("====== TencentAIArt End ======"));
			}
		}
		
		else
		{
			OnFail.Broadcast(FString("null"));
			UE_LOG(LogTencentAIArt, Warning, TEXT("Deserialize Error"));
			UE_LOG(LogTencentAIArt, Warning, TEXT("====== TencentAIArt End ======"));
		}
	}
	else
	{
		OnFail.Broadcast(FString("null"));
		UE_LOG(LogTencentAIArt, Warning, TEXT("Request Error"));
		UE_LOG(LogTencentAIArt, Warning, TEXT("====== TencentAIArt End ======"));
	}

	RemoveFromRoot();
}

void UTencentAiArt::CancelRequestImage()
{
	if (!AIArtHandler || !AIArtHandler->HttpRequestImage) return;
	const auto Status = AIArtHandler->HttpRequestImage->GetStatus();

	if (Status == EHttpRequestStatus::Processing)
	{
		AIArtHandler->HttpRequestImage->CancelRequest();
		AIArtHandler->HttpRequestImage.Reset();
		UE_LOG(LogTencentAIArt, Warning, TEXT("Request Cancle"))
		UE_LOG(LogTencentAIArt, Warning, TEXT("====== TencentAIArt End ======"));
	}
}
```




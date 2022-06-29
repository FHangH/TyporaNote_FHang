# UE4 功能整理





### 1. SpawnActor



使用情景：

- 我有一个`Cpp类`
- 这个`Cpp类`要生成一个其他`Cpp`或`蓝图`类
- 可以使用`TSubOfClass<>`



示例：

- 定义

  ```c++
  private:
  	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta=(AllowPrivateAccess=true))
  	TSubclassOf<AActor> SpawnActorTest;
  ```

- 实现

  ```c++
  void AActor_SpawnTest::BeginPlay()
  {
  	Super::BeginPlay();
  
  	FActorSpawnParameters SpawnParameters;
  	SpawnParameters.Name = FName("SpawnActorTest");
  	SpawnParameters.Owner = this;
  	SpawnParameters.Instigator = nullptr;
      
  	GetWorld()->SpawnActor<AActor>(SpawnActorTest, GetActorTransform(), SpawnParameters);
  }
  ```



说明：

- 定义中的`SpawnActorTest`是私有变量，若要保持私有，其蓝图可以`Get`, `Set`和参数列表中显示继承中：
  - `UPOPERTY()`中加入`BlueprintReadWrite`，`meta=(AllowPrivateAccess=true)`

- `FActorSpawnParameters`是可以自定义生成参数，默认可以不写
- `GetActorTransform()`可以是`GetActorLocation`和`GetActorRotation`







### 2. LineTraceSingle


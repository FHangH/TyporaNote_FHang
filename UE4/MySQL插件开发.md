# UE4 MySQL插件开发





### 1. 项目简介



- 个人`UrealEngine MySQL Plugin`开发流程
- 方便`UnrealEngine`项目`链接`和`操作`MySQL数据库





### 2. 项目环境



```
IDE -> JetBrains Rider 2022.1 EAP7 内部版本号 221.4906.10
UnrealEngine -> 4.27.2
Compiler Environment -> Visual Studio 2022 提供编译组件
MySQL -> 8.0.28 MySQL Community Server - GPL
```





### 3. 项目资源



#### 3.1 资源文件结构



```
ConnectorLibs
	- ConnectorLibs
		+ include
		+ lib
	- ConnectorLibs.Build.cs
```





#### 3.2 资源链接



「ConnectorLibs」https://www.aliyundrive.com/s/wTrvzpVdfxU 提取码: 0be0





### 4. 项目创建



#### 4.1 创建UE4项目



1. 默认选择`游戏项目`
2. 默认选择`空项目`
3. 创建`C++项目`，包含`初学者内容包`
4. 选择项目的创建目录
5. 创建项目名`FH_TestMySQL`



#### 4.2 创建新插件



1. 找到`Edit`或`Settings`
2. 选择`plugins`进入插件界面
3. 选择`New Plugin`
4. 选择`Blank`
5. 设置插件名称`FH_MySQL`
6. `Descriptor Data`随意填写
7. `Create Plugin`



#### 4.3 创建MySQL C++类



##### 4.3.1 FH_ConnectionObject



1. 创建`Object`C++类，类名`FH_ConnectionObject`
2. 选择添加到插件目录`FH_MySQL(Runtime)`，默认`public .h，private .cpp`
3. `Create Class`
4. 会弹出一个`Message`窗口，内容大致为`创建成功，但需要重新编译`，选择`No`



##### 4.3.2 BPFuncLib_FHSQL



1. 创建`Blueprint Function Library`C++类，类名`BPFuncLib_FHSQL`
2. 剩下步骤同上





### 5. 项目配置



#### 5.1 引入MySQL环境库



##### 5.1.1 导入环境库



1. 准备好`ConnectorLibs`
2. 来到`FH_TestMySQL\Plugins\FH_MySQL\Source`目录下
3. 创建目录`ThirdParty`
4. 将`ConnectorLibs`放入`ThirdParty`内





##### 5.1.2 配置项目属性



1. `Rider`打开创建好的UE4 C++项目
2. 在资源栏内找到项目名`FH_TestMySQL`，在`Games`目录下
3. 右键，选择`Properties`
4. 在`Configurations`内选择`Development_Editor | x64`
5. 选择`VC++目录`
6. 在`包含目录(include)`中填入绝对路径`项目文件路径\FH_TestMySQL\Plugins\FH_MySQL\Source\ThirdParty\ConnectorLibs\include`
7. 在`库目录(lib)`中填入绝对路径`项目文件路径\FH_TestMySQL\Plugins\FH_MySQL\Source\ThirdParty\ConnectorLibs\lib`
8. `OK`





##### 5.1.3 配置项目文件



1. 修改`FH_MySQL.uplugin`，文件位置`项目文件路径\FH_TestMySQL\Plugins\FH_MySQL`，只需要添加`"WhitelistPlatforms": ["Win64"]`

   ```json
   {
   	"FileVersion": 3,
   	"Version": 1,
   	"VersionName": "1.0",
   	"FriendlyName": "FH_MySQL",
   	"Description": "Using UE4 Connecting MySQL Plugin",
   	"Category": "FhPlugin",
   	"CreatedBy": "FangH",
   	"CreatedByURL": "https://fhangh.gitee.io/",
   	"DocsURL": "",
   	"MarketplaceURL": "",
   	"SupportURL": "",
   	"CanContainContent": true,
   	"IsBetaVersion": true,
   	"IsExperimentalVersion": false,
   	"Installed": false,
   	"Modules": [
   		{
   			"Name": "FH_MySQL",
   			"Type": "Runtime",
   			"LoadingPhase": "Default",
   			"WhitelistPlatforms": ["Win64"]
   		}
   	]
   }
   ```

2. 修改`FH_MySQL.Build.cs`，文件位置`项目文件路径\FH_TestMySQL\Plugins\FH_MySQL\Source\FH_MySQL`，修改如下内容

   ```c#
   PublicDependencyModuleNames.AddRange(
   	new string[]
       {
   		"Core",
   		"ConnectorLibs",
   		// ... add other public dependencies that you statically link with here ...
   	}
   );
   ```





### 6. MySQL C++文件



#### 6.1 MySQL连接对象



- `FH_ConnectionObject.h`

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "UObject/NoExportTypes.h"
  #include "mysql.h"
  #include "FH_ConnectionObject.generated.h"
  
  UCLASS(BlueprintType)
  class FH_MYSQL_API UFH_ConnectionObject : public UObject
  {
  	GENERATED_BODY()
  
  private:
  	UFH_ConnectionObject();
  
  public:
  	MYSQL *Fh_ConnMysql;
  };
  ```

- `FH_ConnectionObject.cpp`

  ```c++
  #include "FH_ConnectionObject.h"
  
  UFH_ConnectionObject::UFH_ConnectionObject()
  {
  	Fh_ConnMysql = nullptr;
  }
  ```





#### 6.2 MySQL函数工具库



- `BPFuncLib_FHSQL.h`

  ```c++
  #pragma once
  
  #include "CoreMinimal.h"
  #include "Kismet/BlueprintFunctionLibrary.h"
  #include "FH_ConnectionObject.h"
  #include "BPFuncLib_FHSQL.generated.h"
  
  USTRUCT(BlueprintType)
  struct FQueryResultRow
  {
  	GENERATED_BODY()
  
  	UPROPERTY(BlueprintReadWrite, Category="MySQL|Result Row Value")
  	TArray<FString> RowValue;
  };
  
  USTRUCT(BlueprintType)
  struct FQueryResultRows
  {
  	GENERATED_BODY()
  
  	UPROPERTY(BlueprintReadWrite, Category="MySQL|Result Rows Value")
  	TArray<FQueryResultRow> RowsValue;
  };
  
  UCLASS(BlueprintType)
  class FH_MYSQL_API UBPFuncLib_FHSQL : public UBlueprintFunctionLibrary
  {
  	GENERATED_BODY()
  
  public:
  	/*
  	 * Connection == MySQL Object
  	 * @return *UFH_ConnectionObject == MySQL Connector
  	 */
  	UFUNCTION(BlueprintCallable, Category="MySQL|Utils")
  	static UFH_ConnectionObject *ConnectToMySQL(FString Host, FString UserName, FString PassWord, FString DBName,
  												int32 Port, FString &ConnectMessage);
  
  	/*
  	 * ConnectionObject == MySQL Object
  	 * @return bool == ConnectionState
  	 */
  	UFUNCTION(BlueprintCallable, Category="MySQL|Utils")
  	static bool GetConnectionState(UFH_ConnectionObject *ConnectionObject);
  
  	/*
  	 * ConnectionObject == MySQL Object
  	 * @return bool == ConnectionState
  	 */
  	UFUNCTION(BlueprintCallable, Category="MySQL|Utils")
  	static bool CloseConnection(UFH_ConnectionObject *ConnectionObject);
  
  	/*
  	 * ConnectionObject == MySQL Object
  	 * @return bool == Insert, Update, Delete Data Is Succeed Or Failed
  	 */
  	UFUNCTION(BlueprintCallable, Category="MySQL|Utils")
  	static bool ActionOnTableData(UFH_ConnectionObject *ConnectionObject, FString SqlQuery);
  	
  	/*
  	 * TableName = DataBase TableName
  	 * InsertValues = MySQL Insert Values to Table
  	 * @return FString = MySQL Insert Query -> Insert
  	 */
  	UFUNCTION(BlueprintPure, Category="MySQL|Utils")
  	static FString InsertFormatSqlQuery(FString TableName, FString InsertValues);
  
  	/*
  	 * TableName = DataBase TableName
  	 * RowName = Need Update Row
  	 * @return FString = MySQL Update Query -> Update
  	 */	
  	UFUNCTION(BlueprintPure, Category="MySQL|Utils")
  	static FString UpdateAllFormatSqlQuery(FString TableName, FString RowName, FString UpdateValue);
  
  	/*
  	 * TableName = DataBase TableName
  	 * RowName = Need Update Row
  	 * WhereName = Update Where
  	 * WhereSymbol = Operator Or Symbol
  	 * WhereValue = Condition Name
  	 * UpdateValue = Need Update Date Value
  	 * @return FString = MySQL Update Query -> Update
  	 */	
  	UFUNCTION(BlueprintPure, Category="MySQL|Utils")
  	static FString UpdateByWhereFormatSqlQuery(FString TableName, FString RowName, FString WhereName, FString WhereSymbol, FString WhereValue, FString UpdateValue);
  
  	/*
  	 * TableName = DataBase TableName
  	 * @return FString = MySQL Delete Query -> Delete
  	 */
  	UFUNCTION(BlueprintPure, Category="MySQL|Utils")
  	static FString DeleteAllFormatSqlQuery(FString TableName);
  
  	/*
  	 * TableName = DataBase TableName
  	 * WhereName = Update Where
  	 * WhereSymbol = Operator Or Symbol
  	 * WhereValue = Condition Name
  	 * @return FString = MySQL Delete Query -> Delete
  	 */
  	UFUNCTION(BlueprintPure, Category="MySQL|Utils")
  	static FString DeleteByWhereFormatSqlQuery(FString TableName, FString WhereName, FString WhereSymbol, FString WhereValue);
  
  	/*
  	 * ConnectionObject == MySQL Object
  	 * @return bool == Select Data Is Succeed Or Failed
  	 */	
  	UFUNCTION(BlueprintCallable, Category="MySQL|Utils")
  	static bool SelectOnTableData(UFH_ConnectionObject *ConnectionObject, FString SqlQuery, FQueryResultRows &ResultRows);
  	
  	/*
  	 * TableName = DataBase TableName
  	 * @return FString = MySQL Select Query -> Select
  	 */
  	UFUNCTION(BlueprintPure, Category="MySQL|Utils")
  	static FString SelectAllFormatSqlQuery(FString TableName);
  
  	/*
  	 * TableName = DataBase TableName
  	 * @return FString = MySQL Select Query -> Select
  	 */	
  	UFUNCTION(BlueprintPure, Category="MySQL|Utils")
  	static FString SelectByColumnsFormatSqlQuery(FString TableName, FString Columns);
  
  	/*
  	 * TableName = DataBase TableName
  	 * @return TArray<FString> = Get All Rows -> In All Columns Values
  	 */		
  	UFUNCTION(BlueprintPure, Category="MySQL|Utils")
  	static FQueryResultRow GetRowByIndex(const FQueryResultRows &ResultRows, int32 RowIndex);
  };
  ```

- `BPFuncLib_FHSQL.cpp`

  ```c++
  #include "BPFuncLib_FHSQL.h"
  #include "mysql.h"
  #include <string>
  
  UFH_ConnectionObject* UBPFuncLib_FHSQL::ConnectToMySQL(FString Host, FString UserName, FString PassWord, FString DBName,
  	int32 Port, FString &ConnectMessage)
  {
  	const std::string m_Host(TCHAR_TO_UTF8(*Host));
  	const std::string m_UserName(TCHAR_TO_UTF8(*UserName));
  	const std::string m_PassWord(TCHAR_TO_UTF8(*PassWord));
  	const std::string m_DBName(TCHAR_TO_UTF8(*DBName));
  	const uint32 m_Port = Port; 
  
  	// Create MySQL Connection Object
  	UFH_ConnectionObject *ConnectionObject = NewObject<UFH_ConnectionObject>();
  	// Init DataBase Connection Object
  	ConnectionObject->Fh_ConnMysql = mysql_init(nullptr);
  
  	// Judge Connection Status And Return ConnectMessage
  	if (mysql_real_connect(ConnectionObject->Fh_ConnMysql, m_Host.c_str(), m_UserName.c_str(), m_PassWord.c_str(),
  	                       m_DBName.c_str(), m_Port, nullptr, 0))
  	{
  		ConnectMessage = TEXT("Connect Succeed");
  	}
  	else
  	{
  		ConnectMessage = TEXT("Connect Failed");
  	}
  
  	// Return MySQL Connection Object
  	return ConnectionObject;
  }
  
  bool UBPFuncLib_FHSQL::GetConnectionState(UFH_ConnectionObject* ConnectionObject)
  {
  	// Judge Current MySQL Connection State
  	if (ConnectionObject)
  	{
  		if (ConnectionObject->Fh_ConnMysql != nullptr)
  		{
  			return true;
  		}
  		return false;
  	}
  	return false;
  }
  
  bool UBPFuncLib_FHSQL::CloseConnection(UFH_ConnectionObject* ConnectionObject)
  {
  	// If MySQL Connected -> Close MySQL Connection; Return True
  	// Else -> Return True
  	if (GetConnectionState(ConnectionObject))
  	{
  		mysql_close(ConnectionObject->Fh_ConnMysql);
  		ConnectionObject->Fh_ConnMysql = nullptr;
  		ConnectionObject = nullptr;
  		return true;
  	}
  	return true;
  }
  
  bool UBPFuncLib_FHSQL::ActionOnTableData(UFH_ConnectionObject* ConnectionObject, FString SqlQuery)
  {
  	const std::string m_SqlQuery(TCHAR_TO_UTF8(*SqlQuery));
  
  	// Judge MySQL Is Connected
  	if (!ConnectionObject)
  	{
  		return false;
  	}
  	// Judge SqlQuery Is Apply Succeed
  	if (mysql_query(ConnectionObject->Fh_ConnMysql, m_SqlQuery.c_str()) == 0)
  	{
  		return true;
  	}
  	return true;
  }
  
  FString UBPFuncLib_FHSQL::InsertFormatSqlQuery(FString TableName, FString InsertValues)
  {
  	// INSERT INTO TableName VALUES(InsertValues);
  	FString SqlQuery = "INSERT INTO " + TableName + " VALUES(" + InsertValues + ");";
  	return SqlQuery;
  }
  
  FString UBPFuncLib_FHSQL::UpdateAllFormatSqlQuery(FString TableName, FString UpdateRowName, FString UpdateValue)
  {	// UPDATE TableName SET RowName=UpdateValue;
  	FString SqlQuery = "UPDATE " + TableName + " SET " + UpdateRowName + "=" + UpdateValue + ";";
  	return SqlQuery;
  }
  
  FString UBPFuncLib_FHSQL::UpdateByWhereFormatSqlQuery(FString TableName, FString UpdateRowName, FString WhereName, FString WhereSymbol, FString WhereValue, FString UpdateValue)
  {
  	// UPDATE TableName SET UpdateRowName=UpdateValue WHERE WhereName=WhereValue;
  	FString SqlQuery = "UPDATE " + TableName + " SET " + UpdateRowName + "=" + UpdateValue + " WHERE " + WhereName + WhereSymbol + WhereValue + ";";
  	return SqlQuery;
  }
  
  FString UBPFuncLib_FHSQL::DeleteAllFormatSqlQuery(FString TableName)
  {
  	// DELETE FROM TableName;
  	FString SqlQuery = "DELETE FROM " + TableName + ";";
  	return SqlQuery;
  }
  
  FString UBPFuncLib_FHSQL::DeleteByWhereFormatSqlQuery(FString TableName, FString WhereName, FString WhereSymbol, FString WhereValue)
  {
  	// DELETE FROM TableName WHERE WhereName=‘WhereValue’;
  	FString SqlQuery = "DELETE FROM " + TableName + " WHERE " + WhereName + WhereSymbol + "'" + WhereValue + "';";
  	return SqlQuery;
  }
  
  bool UBPFuncLib_FHSQL::SelectOnTableData(UFH_ConnectionObject* ConnectionObject, FString SqlQuery, FQueryResultRows &ResultRows)
  {
  	MYSQL_RES *m_Res = nullptr;
  	MYSQL_ROW m_Column;
  	TArray<FString> m_ColumnNames;
  	FQueryResultRows m_Rows;
  	const std::string m_SqlQuery(TCHAR_TO_UTF8(*SqlQuery));
  
  	if (!ConnectionObject){return false;}
  	if (!ConnectionObject->Fh_ConnMysql){return false;}
  
  	if (!mysql_query(ConnectionObject->Fh_ConnMysql, m_SqlQuery.c_str()))
  	{
  		ResultRows = {};
  		m_Res = mysql_store_result(ConnectionObject->Fh_ConnMysql);
  		const int m_Columns = mysql_num_fields(m_Res);
  
  		while ((m_Column = mysql_fetch_row(m_Res)) != nullptr)
  		{
  			FQueryResultRow m_Row;
  			for (int i = 0; i < m_Columns; ++i)
  			{
  				m_Row.RowValue.Add(UTF8_TO_TCHAR(m_Column[i]));
  			}
  			ResultRows.RowsValue.Add(m_Row);
  		}
  	}
  	
  	mysql_free_result(m_Res);
  	return true;
  }
  
  FString UBPFuncLib_FHSQL::SelectAllFormatSqlQuery(FString TableName)
  {
  	// SELECT * FROM TableName;
  	FString SqlQuery = "SELECT * FROM " + TableName + ";";
  	return SqlQuery;
  }
  
  FString UBPFuncLib_FHSQL::SelectByColumnsFormatSqlQuery(FString TableName, FString Columns)
  {
  	// SELECT CustomerName, City, Country FROM Customers;
  	FString SqlQuery = "SELECT " + Columns + " FROM " + TableName + ";";
  	return SqlQuery;
  }
  
  FQueryResultRow UBPFuncLib_FHSQL::GetRowByIndex(const FQueryResultRows &ResultRows, int32 RowIndex)
  {
  	const FQueryResultRow m_Row = ResultRows.RowsValue[RowIndex];
  	return m_Row;
  }
  ```





### 7. 编译项目



- 在`Rider`内进行项目编译
- 编译完成，可以在`Rider`内直接`运行`启动`UE4项目`





### 8. 打包插件



#### 8.1 打包前设置编译工具



- 前提：如果安装的`Visual Studio 2017`可能不需要进行这个步骤，直接进入下一步
- 打包出现`RunUAT`相关错误信息，再进行此步骤
- 如果使用的是`Visual Studio 2019`或`Visual Studio 2022`或其他版本



- 找到`RunUAT.bat`文件，用`记事本`打开或编辑
- 文件位置`UE4引擎的安装目录\UE_4.27\Engine\Build\BatchFiles`
- `编辑 -> 查找`，`%UATExecutable%`
- `%UATExecutable% %* %UATCompileArg%`修改为`%UATExecutable% %* -VS2019=true %UATCompileArg%`
- `我用的是 VS2022，修改成2022好像不行`，但修改成`2019`就可以正常打包了 



#### 8.2 打包MySQL插件



1. 找到`Edit`或`Settings`
2. 选择`plugins`进入插件界面
3. 找到插件`FH_MySQL`
4. 可选步骤`Edit`，进行插件的相关信息的编辑，包括插件的图标
5. 点击`Package(打包)`，选择打包目录`自定义`
6. 等待打包完成即可
7. 我自己的电脑上的环境打包，完全没有任何`Warning`和`Error`，所以理论上打包不存在问题（写这个插件的时候遇到了很多问题，但用我提供的环境库文件和配置步骤，问题都一个个解决了）




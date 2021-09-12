# JavaNote



### 1. Java环境配置



#### 1.1 手动配置Java环境



##### 1.1.1 Oracle官网获得JDK

- [Java-JDK](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)

- 获得JDK，默认安装C盘即可



##### 1.1.2 配置环境变量

1. 在`系统变量`，设置三项属性：
   - JAVA_HOME
   - PATH
   - CLASSPATH（JDK1.5版本以上，无需此项）
   
2. 配置：

   - JAVA_HOME:`C:\Program Files\Java\[实际情况而定]`

   - PATH:

     1. `%JAVA_HOME%\bin`

     2. `%JAVA_HOME%\jre\bin`

3. CMD测试：

   - `java`
   - `javac`
   - `java -version`



##### 1.1.3 测试Java编译

1. 在text中写入Java程序：`Demo.text`

   ```java
   public class Demo {
       public static void main(String[] args){
           int appleNum = 10;
           int day;
           for (day = 1; day <= 3; ++day)
           {
               appleNum -= 2;
           }
           System.out.println("Day " + (day - 1) + " , " + "Apple Count : " + appleNum);
       }
   }
   ```

2. 将text文件文件类型后缀名改为：`Demo.java`

3. 在文件所在地址，打开`CMD`或者`Powershell`

4. 通过命令来编译`Demo.java`--默认是在同地址创建同名的`Demo.class`，`Demo.exe`在系统文件`System32`中

   ```powershell
   javac Demo.java
   ```

5. 通过命令来执行`Demo.exe`

   ```powershell
   java Demo
   ```



#### 1.2 通过Intellij IDEA配置



##### 1.2.1 安装IDEA

- [Interllij IDEA](https://www.jetbrains.com/lp/intellijidea-forrester-tei/)



##### 1.2.2 关键设置

1. `Add Path`
2. 选择安装`Openjdk`



### 2. Java基础



#### 2.1 Java标识符

1. 标识符包括：字母，数字，`_`，`$`；
2. 标识符必须：字母，`_`，`$`开头，不能以数字开头；
3. 标识符不能使用关键字



#### 2.2 常量

常量初始化后，不能再被修改；

```java
final double PI = 3.14;
```



#### 2.3 数据类型



##### 2.3.1 数据类型的分类

- 基本数据类型
  - 数值型
    - 整数类型：byte, short, int, long
    - 浮点类型：float, double
  - 字符型：char
  - 布尔型：bool
- 引用数据类型
  - 类
  - 接口
  - 数组



##### 2.3.2 整型

- Java默认的整型是`int`

| 数据类型 | 名称   | 字节 | 数值范围                    |
| -------- | ------ | ---- | --------------------------- |
| byte     | 字节型 | 1    | -128 ~ 127                  |
| short    | 短整型 | 2    | -32768 ~ 32767              |
| int      | 整型   | 4    | -2147483648 ~ 2147483647    |
| long     | 长整型 | 8    | -2(63次方) ~ -2(63次方) - 1 |

```java
long a = 12345678; // 编译成功
long b = 12345678999; // 编译失败，超出int长度
long c = 12345678999L; // 编译成功，标记 L ，数据是长整型
```



##### 2.3.3 浮点型

- Java默认的浮点型是`double`

| 数据类型 | 名称   | 字节 | 数值范围               |
| -------- | ------ | ---- | ---------------------- |
| float    | 单精度 | 4    | -3.403E38 ~ 3.403E38   |
| double   | 双精度 | 8    | -1.798E308 ~ 1.798E308 |



##### 2.3.4 字符型

- 占用2个字节
- 可用于转义

| 转义符 | 含有   |
| ------ | ------ |
| \b     | 退格   |
| \n     | 换行   |
| \r     | 回车   |
| \t     | 制表符 |
| \\"    | 双引号 |
| \\'    | 单引号 |
| \\\    | 反斜杠 |

```java
char ec = 'a';
char cc = '中';
char c = '\n';
```

- char只能放一个字符
- string可以方一串字符



##### 2.3.4 布尔型

- 布尔型只有两个常量值：`true`，`false`

```java
boolean flag;
flag = true;
```


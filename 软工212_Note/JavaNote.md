# Java基础



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



#### 2.4 Java输入

- 语法

```java
import java.util.Scanner;

public class Demo()
{
    public static void main(String[] args)
    {
        Scanner scan = new Scanner(System.in);
        
        int a = scan.nextInt();
    }
}
```

- Scanner对象的方法说明

  | 方法         | 说明                       |
  | ------------ | -------------------------- |
  | nextByte()   | 读取byte类型的整数         |
  | nextShort()  | 读取short类型的整数        |
  | nextInt()    | 读取int类型的整数          |
  | nextLong()   | 读取long类型的整数         |
  | nextFloat()  | 读取float类型的数          |
  | nextDouble() | 读取double类型的数         |
  | next()       | 读取字符串，遇到空白符结束 |
  | nextLine()   | 读取文本，回车结束         |



#### 2.5 三目运算符

- 语法：`a > b ? a : b;`

- 实例：在三个数中，找到最大数；

  ```java
  import java.util.Scanner;
  
  public class Demo()
  {
      public static void main(String[] args)
      {
          int a = 10, b = 20, c = 30, temp, max;
          temp = a > b ? a : b;
          max = temp > c ? temp : c;
      }
  }
  ```



#### 2.6 Java例题

##### 2.6.1 输入三位数，将各个位数取出

```java
import java.util.Scanner;

public class Demo2
{
    public static void main(String[] args)
    {
        Scanner scanner = new Scanner(System.in);

        System.out.print("输入三位数：");
        int EnterNum = scanner.nextInt();

        int GW, SW, BW;

        GW = (EnterNum % 10);
        SW = (EnterNum % 100) / 10;
        BW = (EnterNum / 100);

        System.out.println("个位：" + GW + " 十位：" + SW + " 百位：" + BW);
    }
}
```



##### 2.6.2 将两个变量值互换

###### 2.6.2.1 设临时变量

```java
public class Demo
{
    public static void main(String[] args)
    {
        int a = 10, b = 20, temp;
        temp = a;
        a = b;
        b = temp;
    }
}
```



###### 2.6.2.2 两数加减

```java
public class Demo
{
    public static void main(String[] args)
    {
        int a = 10, b = 20;
        b = a + b;
        a = b - a;
        b = b - a;
    }
}
```



###### 2.6.2.3 异或

```java
public class Demo
{
    public static void main(String[] args)
    {
        int a = 1, b = 2;
        a = a ^ b;
        b = a ^ b;
        a = a ^ b;
    }
}

//  a = 1 = 01, b = 2 = 10;
# a = a ^ b; //  01 - 10 => 00 == a = 0;
# b = a ^ b; // 00 - 10 => 01 == b = 1;
# a = a ^ b; // 00 - 01 => 10 == a = 2
```





#### 2.7 switch选择语句



##### 2.7.1 例题1

- 学号-姓名-爱好-出生日期，以及是否是闰年

```java
import java.util.Scanner;

class MySelf
{
   private final String stu_ID;
   private final String stu_Name;
   private final String stu_Hobby;
   private final int stu_Date;

   MySelf()
   {
       stu_ID = "82104322666";
       stu_Name = "FHang";
       stu_Hobby = "发呆";
       stu_Date = 1998;
   }

    public String getStu_ID() {
        return stu_ID;
    }

    public String getStu_Name() {
        return stu_Name;
    }

    public String getStu_Hobby() {
        return stu_Hobby;
    }

    public int getStu_Date() {
        return stu_Date;
    }

    public String getIsLeapYear()
    {
        return (stu_Date % 4 == 0 && stu_Date % 100 != 0) || (stu_Date % 400 == 0) ? "是" : "不是";
    }
}

public class Demo3
{
    public static void main(String[] args)
    {
        Scanner scanner = new Scanner(System.in);
        MySelf mySelf = new MySelf();

        while (true)
        {
            System.out.println("1_学号,2_姓名,3_爱好,4_出生日期,<其它_退出>");
            System.out.print("选择: ");
            int num = scanner.nextInt();

            switch (num) {
                case 1 -> System.out.println("学号: " + mySelf.getStu_ID());
                case 2 -> System.out.println("姓名: " + mySelf.getStu_Name());
                case 3 -> System.out.println("爱好: " + mySelf.getStu_Hobby());
                case 4 -> System.out.println("出生日期: " + mySelf.getStu_Date() + " - 闰年: " + mySelf.getIsLeapYear());
                default -> System.exit(0);
            }
        }
    }
}
```



##### 2.7.2 例题2

- 输入一个年份，判断是否是闰年以及每月的天数

```java
import java.util.Scanner;

class Prophet
{
    int year;
    boolean isLeapYear;

    public void CalLeapYear(int year)
    {
        this.year = year;
        isLeapYear = (year % 4 == 0 && year % 100 != 0) || year % 400 == 0;
        String leapYear = isLeapYear ? "是闰年" : "不是闰年";
        System.out.println(year + leapYear);
    }

    public void CalDaysEveryMonth()
    {
        System.out.println(this.year + "的每月天数：");
        int[] days = {31,28,31,30,31,30,31,31,30,31,30,31};
        for (int i = 0; i < days.length; ++i)
        {
            if (this.isLeapYear && i == 1)
            {
                System.out.println("第" + (i + 1) + "月: " + (days[i] + 1));
            }
            else
            {
                System.out.println("第" + (i + 1) + "月: " + days[i]);
            }
        }
    }
}

public class Demo4
{
    public static void main(String[] args)
    {
        Scanner scanner = new Scanner(System.in);
        Prophet pro = new Prophet();

        while (true)
        {
            System.out.print("输入年份<0_退出>：");
            int num = scanner.nextInt();
            if (num == 0)
            {
                System.exit(0);
            }
            else
            {
                pro.CalLeapYear(num);
                pro.CalDaysEveryMonth();
            }
        }
    }
}
```



##### 2.7.3 例题3

- 设计一个加减计算的答题系统，随机生成题目，最少1题，最多10题，并最终返回答题正确数

```java
import java.util.Scanner;
import java.util.Random;

class CalculatorSystem
{
    private int questionCount;
    private int scoreCount;

    private enum CalculateOperation
    {
        ADD, SUB, /*MULTI, DIVIDE*/
    }

    Scanner scanner = new Scanner(System.in);
    Random random = new Random(System.currentTimeMillis());

    CalculatorSystem()
    {
        this.scoreCount = 0;
    }

    public void CalStart()
    {
        System.out.println("<<-- 输入题数(最少1题--最多10题) -->>");
        System.out.print("输入>>");
        questionCount = scanner.nextInt();

        if (questionCount < 1 || questionCount > 10)
        {
            questionCount = questionCount < 1 ? 1 : 10;
        }

        CalRandomQuestion();
    }

    public void CalRandomQuestion()
    {
        int tempA, tempB, inputC, a, b, c;
        CalculateOperation calOperation;

        for (int i = 1; i <= questionCount; ++i)
        {
            System.out.print("第" + i + "题: ");
            int curRand = random.nextInt(2);

            calOperation = CalculateOperation.class.getEnumConstants()[curRand];
            tempA = random.nextInt(101);
            tempB = random.nextInt(101);
            a = Math.max(tempA, tempB);
            b = Math.min(tempB, tempA);

            switch (calOperation)
            {
                case ADD ->
                        {
                            c = a + b;
                            System.out.print(a + "+" + b + "=");
                            inputC = scanner.nextInt();
                            CalCheckAnswer(c, inputC);
                        }
                case SUB ->
                        {
                            c = a - b;
                            System.out.print(a + "-" + b + "=");
                            inputC = scanner.nextInt();
                            CalCheckAnswer(c, inputC);
                        }
            }
        }

        CalSumScore();
    }

    public void CalCheckAnswer(int c, int inputC)
    {
        String printLog = (c == inputC) ? "正确" : "错误";
        boolean isRight = (c == inputC);

        if (isRight)
        {
            this.scoreCount++;
        }
        System.out.println(printLog);
    }

    public void CalSumScore()
    {
        System.out.println("答对" + scoreCount + "题");
        scoreCount = 0;
        System.out.println();
    }

    public void CalEnd()
    {
        System.out.println("<<-- 已退出答题系统 -->>");
    }
}

public class Demo5
{
    public static void main(String[] args)
    {
        Scanner scanner = new Scanner(System.in);
        CalculatorSystem calSys = new CalculatorSystem();

        while (true)
        {
            System.out.println("<<-- (Y/y_开始)--(N/n_结束) -->>");
            System.out.print("输入>>");
            String inputChar = scanner.next();

            if (inputChar.equalsIgnoreCase("Y"))
            {
                calSys.CalStart();
            }
            else if (inputChar.equalsIgnoreCase("N"))
            {
                calSys.CalEnd();
                System.exit(1);
            }
            else
            {
                System.out.println("输入正确的选项");
            }
        }
    }
}
```





##### 2.7.4 例题4

- 提示输入信息—显示信息-退出，显示信息可返回上一级信息提示

```java
import java.util.Scanner;

class PersonInfo
{
    private String fh_ID;
    private String fh_Name;
    private String fh_Hobby;
    private int fh_Date;

    Scanner scanner = new Scanner(System.in);

    public String getFh_ID() {
        return fh_ID;
    }
    public void setFh_ID(String fh_ID) {
        this.fh_ID = fh_ID;
    }

    public String getFh_Name() {
        return fh_Name;
    }
    public void setFh_Name(String fh_Name) {
        this.fh_Name = fh_Name;
    }

    public String getFh_Hobby() {
        return fh_Hobby;
    }
    public void setFh_Hobby(String fh_Hobby) {
        this.fh_Hobby = fh_Hobby;
    }

    public int getFh_Date() {
        return fh_Date;
    }
    public void setFh_Date(int fh_Date) {
        this.fh_Date = fh_Date;
    }

    public void StartPersonInfo()
    {
        while (true)
        {
            System.out.println("<-- 1_输入信息,2_显示信息,3_退出 -->");
            System.out.print("选择>>");
            int num = scanner.nextInt();
            switch (num)
            {
                case 1 -> InputPersonInfo();
                case 2 -> ShowPersonInfoMenu();
                case 3 -> ExitPersonInfo();
            }
        }
    }

    public void ShowPersonInfoMenu()
    {
        System.out.println("<-- 1_学号,2_姓名,3_爱好,4_年份,5_返回上一级 -->");

        while (true)
        {
            System.out.print("选择>>");
            int num = scanner.nextInt();

            switch (num)
            {
                case 1 -> System.out.println("ID: " + getFh_ID());
                case 2 -> System.out.println("姓名: " + getFh_Name());
                case 3 -> System.out.println("爱好: " + getFh_Hobby());
                case 4 -> System.out.println("生日: " + getFh_Date());
                case 5 -> StartPersonInfo();
            }
        }
    }

    public void InputPersonInfo()
    {
        System.out.print("输入ID>>");
        setFh_ID(scanner.next());

        System.out.print("输入姓名>>");
        setFh_Name(scanner.next());

        System.out.print("输入爱好>>");
        setFh_Hobby(scanner.next());

        System.out.print("输入生日>>");
        setFh_Date(scanner.nextInt());
    }

    public void ExitPersonInfo()
    {
        System.out.println("<-- 已退出 -->");
        System.exit(1);
    }
}

public class Demo6
{
    public static void main(String[] args)
    {
        new Scanner(System.in);
        PersonInfo personInfo = new PersonInfo();

        personInfo.StartPersonInfo();
    }
}
```





#### 2.8 for循环



##### 2.8.1 例题1

- 1 - 1000内的水仙花数

```java
class FhMath
{
    int B, S, G;

    public void CalShuiXianHua()
    {
        for (int i = 100; i <= 900; ++i)
        {
            B = i / 100;
            S = (i / 10) % 10;
            G = i % 10;
            if (Math.pow(B, 3) + Math.pow(S, 3) + Math.pow(G, 3) == i)
            {
                System.out.println("水仙花数: " + i);
            }
        }
    }
}

public class Demo8
{
    public static void main(String[] args)
    {
        FhMath fhMath = new FhMath();
        fhMath.CalShuiXianHua();
    }
}
```





##### 2.8.2 例题2

- 公鸡5元一个，母鸡3元一个，三只小鸡一元钱，现有100元，要买100只鸡，怎么买？

```java
class Swap
{
    public void swap(int x, int y, int z)
    {
        for (x = 1; x <= 20; ++x)
        {
            for (y = 1; y <= 33; ++y)
            {
                z = 100 - x - y;
                if (x * 5 + y * 3 + z / 3.0 == 100.0)
                {
                    System.out.println("X= " + x + " Y= " + y + " Z= " + z);
                }
            }
        }
    }
}

public class Demo9
{
    public static void main(String[] args)
    {
        Swap swap = new Swap();
        swap.swap(1, 2, 3);
    }
}
```



##### 2.8.3 例题3

- 计算5的阶乘值的和（递归）

```java
class MathJie
{
    public int fact(int n)
    {
        if (n == 0 || n == 1)
        {
            return 1;
        }
        else
        {
            return n * fact(n - 1);
        }
    }

    public int sum(int n)
    {
        if (n == 1)
        {
            return fact(n);
        }
        else
        {
            return fact(n) + fact(n - 1);
        }
    }
}

public class Demo10
{
    public static void main(String[] args)
    {
        MathJie mathJie = new MathJie();
        int S;
        S = mathJie.sum(5);
        System.out.println(S);
    }
}
```



#### 2.9 函数重载题目



##### 2.9.1 例题1

- 输入n个数，利用重载，打印最大值

```java
class Calculator
{
    void getMax(int...nums)
    {
        int maxNum = nums[0];

        for (int num : nums)
        {
            maxNum = Math.max(maxNum, num);
        }
        System.out.println("Max Num: " + maxNum);
    }
}

public class Demo11
{
    public static void main(String[] args)
    {
        Calculator calculator = new Calculator();

        calculator.getMax(1, 2);
        calculator.getMax(1, 2, 3);
        calculator.getMax(1, 2, 3, 4);
    }
}
```





#### 3.0 Java对象



##### 3.0.1 例题1

- 创建一个宠物类，按提示输入相关信息，并最终打印出来

```java
import java.util.Scanner;

class Pet
{
    private String name;
    private String type;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }

    public String getType() {
        return type;
    }
    public void setType(String type) {
        this.type = type;
    }

    Scanner scanner = new Scanner(System.in);

    Pet()
    {
        TipInfo();

        System.out.print("输入名称>>");
        setName(scanner.next());

        System.out.print("输入类别>>");
        setType(scanner.next());

        PrintPetInfo();
    }

    void TipInfo()
    {
        System.out.println("<-- 输入宠物相关信息 -->");
    }

    void PrintPetInfo()
    {
        System.out.println("<-- 打印宠物相关信息 -->");
        System.out.println("宠物名称: " + getName());
        System.out.println("宠物类别: " + getType());
    }
}

public class Demo10
{
    public static void main(String[] args)
    {
        new Pet();
    }
}
```





#### 3.1 对象继承



##### 3.1.1 继承概念

- 继承是java面向对象编程技术的一块基石，因为它允许创建分等级层次的类
- 继承就是子类继承父类的特征和行为，使得子类对象（实例）具有父类的实例域和方法或子类从父类继承方法，使得子类具有父类相同的行为



- 继承的格式：

  ```java
  class 父类 {
  }
   
  class 子类 extends 父类 {
  }
  ```



##### 3.1.2 代码示例

- Demo.java

  ```java
  package project2;
  
  public class Demo
  {
      public static void main(String[] args)
      {
          Human student = new Student();
          Human teacher = new Teacher();
  
          Getter getter = new Getter();
  
          getter.getInfo((Student) student);
          System.out.println();
          getter.getInfo((Teacher) teacher);
      }
  }
  ```

- Human.java

  ```java
  package project2;
  
  public class Human
  {
  }
  ```

- Student.java

  ```java
  package project2;
  
  public class Student extends Human
  {
      private String name;
      private int stuID;
      private int roomID;
  
      Student()
      {
          name = "QQ";
          stuID = 1;
          roomID = 304;
      }
  
      public String getName()
      {
          return name;
      }
      public void setName(String name)
      {
          this.name = name;
      }
  
      public int getStuID()
      {
          return stuID;
      }
      public void setStuID(int stuID)
      {
          this.stuID = stuID;
      }
  
      public int getRoomID()
      {
          return roomID;
      }
      public void setRoomID(int roomID)
      {
          this.roomID = roomID;
      }
  }
  ```

- Teacher.java

  ```java
  package project2;
  
  public class Teacher extends Human
  {
      private String name;
      private String job;
      private int salary;
  
      Teacher()
      {
          name = "WW";
          job = "MathTeacher";
          salary = 1000;
      }
  
      public String getName()
      {
          return name;
      }
      public void setName(String name)
      {
          this.name = name;
      }
  
      public String getJob()
      {
          return job;
      }
      public void setJob(String job)
      {
          this.job = job;
      }
  
      public int getSalary()
      {
          return salary;
      }
      public void setSalary(int salary)
      {
          this.salary = salary;
      }
  }
  ```

- Getter.java

  ```java
  package project2;
  
  public class Getter
  {
      void getInfo(Student student)
      {
          System.out.println("Name: " + student.getName() + "\nStuID: " + student.getStuID() + "\nRoomID: " + student.getRoomID());
      }
  
      void getInfo(Teacher teacher)
      {
          System.out.println("Name: " + teacher.getName() + "\nJob: " + teacher.getJob() + "\nSalary: " + teacher.getSalary());
      }
  }
  ```





#### 3.2 抽象类



- 概念：
  - 抽象类除了不能实例化对象之外，类的其它功能依然存在，成员变量、成员方法和构造方法的访问方式和普通类一样
  - 由于抽象类不能实例化对象，所以抽象类必须被继承，才能被使用。也是因为这个原因，通常在设计阶段决定要不要设计抽象类
  - 父类包含了子类集合的常见的方法，但是由于父类本身是抽象的，所以不能使用这些方法
  - 在 Java 中抽象类表示的是一种继承关系，一个类只能继承一个抽象类，而一个类却可以实现多个接口





#### 3.3 接口



- 概念：
  - 接口并不是类，编写接口的方式和类很相似，但是它们属于不同的概念
  - 类描述对象的属性和方法，接口则包含类要实现的方法，除非实现接口的类是抽象类，否则该类要定义接口中的所有方法
  - 接口无法被实例化，但是可以被实现



##### 3.3.1 接口的声明

```java
[可见度] interface 接口名称 [extends 其他的接口名] {
        // 声明变量
        // 抽象方法
}
```



##### 3.3.2 接口的特性

- 接口是隐式抽象的，当声明一个接口的时候，不必使用**abstract**关键字
- 接口中每一个方法也是隐式抽象的，声明时同样不需要**abstract**关键字
- 接口中的方法都是公有的



##### 3.3.3 接口的实现

```java
...implements 接口名称[, 其他接口名称, 其他接口名称..., ...] ...
```





##### 3.3.4 代码示例--文件结构

- project3
  - appliance
    - ApplianceBase.java
    - Computer.java
    - Light.java
    - TelePhone.java
    - WashMachine
  - function
    - call.java
    - charge.java
    - playGame.java
  - Demo.java



- 代码源码：

- Demo.java

  ```java
  package project3;
  
  import project3.appliance.Computer;
  import project3.appliance.Light;
  import project3.appliance.TelePhone;
  import project3.appliance.WashMachine;
  
  public class Demo
  {
      public static void main(String[] args)
      {
          // Light
          Light light = new Light("Light");
          light.runStart();
          light.startCharge();
          light.overCharge();
          light.runOver();
  
          // WashMachine
          WashMachine washMachine = new WashMachine("WashMachine");
          washMachine.runStart();
          washMachine.startCharge();
          washMachine.runOver();
  
          // TelePhone
          TelePhone telePhone = new TelePhone("TelePhone");
          telePhone.runStart();
          telePhone.startCall();
          telePhone.overCall();
          telePhone.runOver();
  
          // Computer
          Computer computer = new Computer("Computer");
          computer.runStart();
          computer.playGames();
          computer.overGames();
          computer.runOver();
      }
  }
  ```

- ApplianceBase.java

  ```java
  package project3.appliance;
  
  public class ApplianceBase
  {
      private boolean isRunning;
  
      ApplianceBase(String name)
      {
          isRunning = false;
          System.out.println("<-- " + name + " -->");
      }
  
      public boolean isRunning()
      {
          return isRunning;
      }
  
      public void runStart()
      {
          isRunning = true;
          System.out.println("Is Run Start");
          getCurState();
      }
      public void runOver()
      {
          isRunning = false;
          System.out.println("Is Run Over");
          getCurState();
          System.out.println();
      }
  
      public void getCurState()
      {
          String state = isRunning ? "Running" : "Over";
          System.out.println("CurState>> " + state);
      }
  }
  ```

- Computer.java

  ```java
  package project3.appliance;
  
  import project3.function.charge;
  import project3.function.playGame;
  
  public class Computer extends ApplianceBase implements charge, playGame
  {
      public Computer(String name)
      {
          super(name);
      }
  
      @Override
      public void runStart()
      {
          super.runStart();
      }
  
      @Override
      public void runOver()
      {
          super.runOver();
      }
  
      @Override
      public void getCurState()
      {
          super.getCurState();
      }
  
      @Override
      public void startCharge()
      {
          System.out.println("Computer Is Start Charge");
      }
  
      @Override
      public void overCharge()
      {
          System.out.println("Computer Is Over Charge");
      }
  
      @Override
      public void playGames()
      {
          System.out.println("Computer Play Games");
      }
  
      @Override
      public void overGames()
      {
          System.out.println("Computer Over Games");
      }
  }
  ```

- Light.java

  ```java
  package project3.appliance;
  
  import project3.function.charge;
  
  public class Light extends ApplianceBase implements charge
  {
      public Light(String name)
      {
          super(name);
      }
  
      @Override
      public void runStart()
      {
          super.runStart();
      }
  
      @Override
      public void runOver()
      {
          super.runOver();
      }
  
      @Override
      public void getCurState()
      {
          super.getCurState();
      }
  
      @Override
      public void startCharge()
      {
          System.out.println("Light Is Start Charge");
      }
  
      @Override
      public void overCharge()
      {
          System.out.println("Light Is Over Charge");
      }
  }
  ```

- TelePhone.java

  ```java
  package project3.appliance;
  
  import project3.function.call;
  import project3.function.charge;
  
  public class TelePhone extends ApplianceBase implements charge, call
  {
      public TelePhone(String name)
      {
          super(name);
      }
  
      @Override
      public void runStart()
      {
          super.runStart();
      }
  
      @Override
      public void runOver()
      {
          super.runOver();
      }
  
      @Override
      public void getCurState()
      {
          super.getCurState();
      }
  
      @Override
      public void startCharge()
      {
          System.out.println("TelePhone Is Start Charge");
      }
  
      @Override
      public void overCharge()
      {
          System.out.println("TelePhone Is Over Charge");
      }
  
      @Override
      public void startCall()
      {
          System.out.println("TelePhone Start Call");
      }
  
      @Override
      public void overCall()
      {
          System.out.println("TelePhone Over Call");
      }
  }
  ```

- WashMachine.java

  ```java
  package project3.appliance;
  
  import project3.function.charge;
  
  public class WashMachine extends ApplianceBase implements charge
  {
      public WashMachine(String name)
      {
          super(name);
      }
  
      @Override
      public void runStart()
      {
          super.runStart();
      }
  
      @Override
      public void runOver()
      {
          super.runOver();
      }
  
      @Override
      public void getCurState()
      {
          super.getCurState();
      }
  
      @Override
      public void startCharge()
      {
          System.out.println("WashMachine Is Start Charge");
      }
  
      @Override
      public void overCharge()
      {
          System.out.println("WashMachine Is Over Charge");
      }
  }
  ```

- call.java

  ```java
  package project3.function;
  
  public interface call
  {
      void startCall();
      void overCall();
  }
  ```

- charge.java

  ```java
  package project3.function;
  
  public interface charge
  {
      void startCharge();
      void overCharge();
  }
  ```

- playGame.java

  ```java
  package project3.function;
  
  public interface playGame
  {
      void playGames();
      void overGames();
  }
  ```







#### 3.4 数组和常用类



- 案例描述：在一个九个元素的数组中，通过两组的三位数相加等于第三组组成的三位数，九个元素不重复，取值范围：[0 - 9]

- 代码：

  ```java
  package project4;
  
  import java.util.Arrays;
  
  public class Demo1
  {
      int x,y,z;
      int[] flagNo=new int[]{0,0,0, 0,0,0, 0,0,0};
      public static void main(String[] args)
      {
          Demo1 abc=new Demo1();
          abc.findRes();
      }
  
      public void stau(int i)
      {
          for (int j = 0; j <3 ; j++)
          {
              if (i%10!=0)
                  flagNo[i%10-1]=1;
              i=i/10;
          }
      }
  
      public boolean isRepeat()
      {
          boolean tmp=true;
          for (int j:flagNo)
          {
              if (j == 0)
              {
                  tmp = false;
                  break;
              }
          }
          return tmp;
      }
  
      public void findRes()
      {
          int counter=0;
          for (x=123;x<=987;x++)
              for (y=x+1;y<987;y++)
              {
                  Arrays.fill(flagNo, 0);
  
                  z=x+y;
                  if (z>123&&z<=987)
                  {
                      if (isRepeatXYZ())
                      {
                          counter++;
                          System.out.println(x+"+"+y+"="+z+"【"+counter+"】");
                      }
                  }
              }
  
      }
  
      public boolean isRepeatXYZ()
      {
          boolean tmp;
          stau(x);
          stau(y);
          stau(z);
          tmp=isRepeat();
          return tmp;
      }
  }
  ```







#### 3.5 集合框架



- 集合框架的概念：
  - `接口`：代表集合的抽象数据类型。例如 Collection、List、Set、Map 等。之所以定义多个接口，是为了以不同的方式操作集合对象
  - `实现类`：是集合接口的具体实现。从本质上讲，它们是可重复使用的数据结构，例如：ArrayList、LinkedList、HashSet、HashMap
  - `算法`：是实现集合接口的对象里的方法执行的一些有用的计算，例如：搜索和排序



- `Set` 和 `List`的区别
  - Set 接口实例存储的是无序的，不重复的数据。List 接口实例存储的是有序的，可以重复的元素
  - Set检索效率低下，删除和插入效率高，插入和删除不会引起元素位置改变` <实现类有HashSet,TreeSet>`
  -  List和数组类似，可以动态增长，根据实际存储的数据的长度自动增长List的长度。查找元素效率高，插入删除效率低，因为会引起其他元素位置改变` <实现类有ArrayList,LinkedList,Vector>`





- 代码示例：ArrayList

  ```java
  import java.util.*;
   
  public class Test{
   public static void main(String[] args) {
       List<String> list=new ArrayList<String>();
       list.add("Hello");
       list.add("World");
       list.add("HAHAHAHA");
       //第一种遍历方法使用 For-Each 遍历 List
       for (String str : list) {            //也可以改写 for(int i=0;i<list.size();i++) 这种形式
          System.out.println(str);
       }
   
       //第二种遍历，把链表变为数组相关的内容进行遍历
       String[] strArray=new String[list.size()];
       list.toArray(strArray);
       for(int i=0;i<strArray.length;i++) //这里也可以改写为  for(String str:strArray) 这种形式
       {
          System.out.println(strArray[i]);
       }
       
      //第三种遍历 使用迭代器进行相关遍历
       
       Iterator<String> ite=list.iterator();
       while(ite.hasNext())//判断下一个元素之后有值
       {
           System.out.println(ite.next());
       }
   }
  }
  ```

- 代码解析：第三种方法是采用迭代器的方法，该方法可以不用担心在遍历的过程中会超出集合的长度



- 代码示例：Map

  ```java
  import java.util.*;
   
  public class Test{
       public static void main(String[] args) {
        Map<String, String> map = new HashMap<String, String>();
        map.put("1", "value1");
        map.put("2", "value2");
        map.put("3", "value3");
        
        //第一种：普遍使用，二次取值
        System.out.println("通过Map.keySet遍历key和value：");
        for (String key : map.keySet()) {
         System.out.println("key= "+ key + " and value= " + map.get(key));
        }
        
        //第二种
        System.out.println("通过Map.entrySet使用iterator遍历key和value：");
        Iterator<Map.Entry<String, String>> it = map.entrySet().iterator();
        while (it.hasNext()) {
         Map.Entry<String, String> entry = it.next();
         System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
        }
        
        //第三种：推荐，尤其是容量大时
        System.out.println("通过Map.entrySet遍历key和value");
        for (Map.Entry<String, String> entry : map.entrySet()) {
         System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
        }
      
        //第四种
        System.out.println("通过Map.values()遍历所有的value，但不能遍历key");
        for (String v : map.values()) {
         System.out.println("value= " + v);
        }
       }
  }
  ```



##### 3.5.1 集合排序

```java
package project8;

import java.util.Comparator;
import java.util.Vector;

class Student
{
    int id;
    String name;
    String sex;
    int computerGrade;
    int cGrade;
    int mathGrade;
    int physicalGrade;

    Student(int id, String name, String sex, int computerGrade, int cGrade, int mathGrade, int physicalGrade)
    {
        this.id = id;
        this.name = name;
        this.sex = sex;
        this.computerGrade = computerGrade;
        this.cGrade = cGrade;
        this.mathGrade = mathGrade;
        this.physicalGrade = physicalGrade;
    }
}

class DownSort implements Comparator<Student>
{
    public int compare(Student s1, Student s2)
    {
        int g1 = s1.computerGrade + s1.cGrade + s1.mathGrade + s1.physicalGrade;
        int g2 = s2.computerGrade + s2.cGrade + s2.mathGrade + s2.physicalGrade;

        return Integer.compare(g2, g1);
    }
}

public class Demo
{
    public static void main(String[] args)
    {
        Vector<Student> vS = new Vector<>();
        Comparator<Student> comparator = new DownSort();
        Student s1 = new Student(1001, "mary", "女", 90, 80, 78, 83);
        Student s2 = new Student(1002, "tom", "男", 80, 81, 79, 84);
        Student s3 = new Student(1003, "jerry", "男", 93, 82, 80, 85);
        Student s4 = new Student(1004, "john", "男", 90, 83, 81, 86);

        vS.add(s1);
        vS.add(s2);
        vS.add(s3);
        vS.add(s4);

        System.out.println("全部显示》》");
        printInfo(vS);

        vS.sort(comparator);

        System.out.println("按总分排序》》");
        printInfo(vS);

        System.out.println("查询》》");
        searcher(vS, 1001);
    }

    static void printInfo(Vector<Student> vS)
    {
        for (Student s : vS)
        {
            System.out.println(s.id + " " + s.name + " " + s.sex + " " + s.computerGrade + " " + s.cGrade + " " + s.mathGrade + " " + s.physicalGrade);
        }
        System.out.println();
    }

    static void searcher(Vector<Student> vs, int id)
    {
        for (Student s : vs)
        {
            if (s.id == id)
            {
                System.out.println(s.id + " " + s.name + " " + s.sex + " " + s.computerGrade + " " + s.cGrade + " " + s.mathGrade + " " + s.physicalGrade);
            }
        }
        System.out.println();
    }
}

```





#### 3.6 异常处理



- 异常概念：
  - **检查性异常：**最具代表的检查性异常是用户错误或问题引起的异常，这是程序员无法预见的
  - **运行时异常：** 运行时异常是可能被程序员避免的异常
  - **错误：** 错误不是异常，而是脱离程序员控制的问题



- Exception类的层次
  - IOException 类
  - RuntimeException 类



##### 3.6.1 异常方法

| 序号 | 方法及说明                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | **public String getMessage()**<br/>返回关于发生的异常的详细信息。这个消息在Throwable 类的构造函数中初始化了。 |
| 2    | **public Throwable getCause()**<br/>返回一个Throwable 对象代表异常原因。 |
| 3    | **public String toString()**<br/>使用getMessage()的结果返回类的串级名字。 |
| 4    | **public void printStackTrace()**<br/>打印toString()结果和栈层次到System.err，即错误输出流。 |
| 5    | **public StackTraceElement [] getStackTrace()**<br/>返回一个包含堆栈层次的数组。下标为0的元素代表栈顶，最后一个元素代表方法调用堆栈的栈底。 |
| 6    | **public Throwable fillInStackTrace()**<br/>用当前的调用栈层次填充Throwable 对象栈层次，添加到栈层次任何先前信息中。 |



##### 3.6.2 捕获异常

- 使用 try 和 catch 关键字可以捕获异常。try/catch 代码块放在异常可能发生的地方

```java
try
{
   // 程序代码
}catch(ExceptionName e1)
{
   //Catch 块
}
```

```java
import java.io.*;
public class ExcepTest{
 
   public static void main(String args[]){
      try{
         int a[] = new int[2];
         System.out.println("Access element three :" + a[3]);
      }catch(ArrayIndexOutOfBoundsException e){
         System.out.println("Exception thrown  :" + e);
      }
      System.out.println("Out of the block");
   }
}
```



##### 3.6.3 多重捕获块

- 一个 try 代码块后面跟随多个 catch 代码块的情况就叫多重捕获

```java
try{
   // 程序代码
}catch(异常类型1 异常的变量名1){
  // 程序代码
}catch(异常类型2 异常的变量名2){
  // 程序代码
}catch(异常类型3 异常的变量名3){
  // 程序代码
}
```

```java
try {
    file = new FileInputStream(fileName);
    x = (byte) file.read();
} catch(FileNotFoundException f) { // Not valid!
    f.printStackTrace();
    return -1;
} catch(IOException i) {
    i.printStackTrace();
    return -1;
}
```





##### 3.6.4 throws / throw 关键字

- 如果一个方法没有捕获到一个检查性异常，那么该方法必须使用 throws 关键字来声明

```java
import java.io.*;
public class className
{
  public void deposit(double amount) throws RemoteException
  {
    // Method implementation
    throw new RemoteException();
  }
  //Remainder of class definition
}
```



##### 3.6.5 finally 关键字

- finally 关键字用来创建在 try 代码块后面执行的代码块
- 无论是否发生异常，finally 代码块中的代码总会被执行
- 在 finally 代码块中，可以运行清理类型等收尾善后性质的语句

```java
try{
  // 程序代码
}catch(异常类型1 异常的变量名1){
  // 程序代码
}catch(异常类型2 异常的变量名2){
  // 程序代码
}finally{
  // 程序代码
}
```

```java
public class ExcepTest{
  public static void main(String args[]){
    int a[] = new int[2];
    try{
       System.out.println("Access element three :" + a[3]);
    }catch(ArrayIndexOutOfBoundsException e){
       System.out.println("Exception thrown  :" + e);
    }
    finally{
       a[0] = 6;
       System.out.println("First element value: " +a[0]);
       System.out.println("The finally statement is executed");
    }
  }
}
```

- 注意：
  - catch 不能独立于 try 存在
  - 在 try/catch 后面添加 finally 块并非强制性要求的
  - try 代码后不能既没 catch 块也没 finally 块
  - try, catch, finally 块之间不能添加任何代码



##### 3.6.6 声明自定义异常



- Java 中可以自定义异常

  - 所有异常都必须是 Throwable 的子类
  - 如果写一个检查性异常类，则需要继承 Exception 类
  - 如果写一个运行时异常类，那么需要继承 RuntimeException 类

  ```java
  class MyException extends Exception{}
  ```

- 代码示例：

- BankDemo.java

  ```java
  public class BankDemo
  {
     public static void main(String [] args)
     {
        CheckingAccount c = new CheckingAccount(101);
        System.out.println("Depositing $500...");
        c.deposit(500.00);
        try
        {
           System.out.println("\nWithdrawing $100...");
           c.withdraw(100.00);
           System.out.println("\nWithdrawing $600...");
           c.withdraw(600.00);
        }catch(InsufficientFundsException e)
        {
           System.out.println("Sorry, but you are short $"
                                    + e.getAmount());
           e.printStackTrace();
        }
      }
  }
  ```

- InsufficientFundsException.java

  ```java
  import java.io.*;
   
  //自定义异常类，继承Exception类
  public class InsufficientFundsException extends Exception
  {
    //此处的amount用来储存当出现异常（取出钱多于余额时）所缺乏的钱
    private double amount;
    public InsufficientFundsException(double amount)
    {
      this.amount = amount;
    } 
    public double getAmount()
    {
      return amount;
    }
  }
  ```

- CheckingAccount.java

  ```java
  import java.io.*;
   
  //此类模拟银行账户
  public class CheckingAccount
  {
    //balance为余额，number为卡号
     private double balance;
     private int number;
     public CheckingAccount(int number)
     {
        this.number = number;
     }
    //方法：存钱
     public void deposit(double amount)
     {
        balance += amount;
     }
    //方法：取钱
     public void withdraw(double amount) throws
                                InsufficientFundsException
     {
        if(amount <= balance)
        {
           balance -= amount;
        }
        else
        {
           double needs = amount - balance;
           throw new InsufficientFundsException(needs);
        }
     }
    //方法：返回余额
     public double getBalance()
     {
        return balance;
     }
    //方法：返回卡号
     public int getNumber()
     {
        return number;
     }
  }
  ```



- 代码示例：自定义异常处理--验证码

  ```java
  package project5;
  
  import java.util.Random;
  import java.util.Scanner;
  
  class MyException extends Exception
  {
      MyException(String s)
      {
          super(s);
      }
  }
  
  class VerifyDemo
  {
      int[] verifyArr = new int[6];
  
      void printArr(int[] arr)
      {
          for (int j : arr)
          {
              System.out.print(Character.toString(j) + " ");
          }
      }
  
      void createVerifyCode()
      {
          int verifySeed = 65;
          Random random = new Random(System.currentTimeMillis());
  
          for (int i = 0; i < 6; ++i)
          {
              int rand = random.nextInt(26);
              verifyArr[i] = verifySeed + rand;
          }
  
          printArr(verifyArr);
      }
  
      int[] transformStringToArrInt(String s)
      {
          int[] arr = new int[6];
          char[] charArr = s.toCharArray();
          for (int i = 0; i < charArr.length; ++i)
          {
              arr[i] = charArr[i];
          }
          return arr;
      }
  
      boolean checkVerifyCode(int[] arr)
      {
          boolean isTrue = true;
          for (int i = 0; i < arr.length; ++i)
          {
              if (arr[i] != verifyArr[i])
              {
                  isTrue = false;
                  break;
              }
          }
          return isTrue;
      }
  }
  
  public class Demo
  {
      public static void main(String[] args)
      {
          Scanner scanner = new Scanner(System.in);
          VerifyDemo verifydemo = new VerifyDemo();
          String inputStr;
  
          System.out.print("Verify Code: ");
          verifydemo.createVerifyCode();
          System.out.println();
          System.out.print(">> ");
          inputStr = scanner.next();
  
          int[] verifyArr = verifydemo.transformStringToArrInt(inputStr);
          boolean isVerifyTrue = verifydemo.checkVerifyCode(verifyArr);
  
          try
          {
              if (isVerifyTrue)
              {
                  throw new MyException("<-- Verification Code True! -->");
              }
              else
              {
                  throw new MyException("<-- Verification Code Error! -->");
              }
          }
          catch (MyException exception)
          {
              System.out.println(exception.getMessage());
          }
      }
  }
  ```

  



#### 3.7 多线程



##### 3.7.1 线程的生命周期

- **新建状态:**
  - 使用 **new** 关键字和 **Thread** 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 **start()** 这个线程
- **就绪状态:**
  - 当线程对象调用了start()方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度
- **运行状态:**
  - 如果就绪状态的线程获取 CPU 资源，就可以执行 **run()**，此时线程便处于运行状态
  - 处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态
- **阻塞状态:**
  - 如果一个线程执行了sleep（睡眠）、suspend（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态
  - 在睡眠时间已到或获得设备资源后可以重新进入就绪状态，可以分为三种：
    - 等待阻塞：运行状态中的线程执行 wait() 方法，使线程进入到等待阻塞状态
    - 同步阻塞：线程在获取 synchronized 同步锁失败(因为同步锁被其他线程占用)
    - 其他阻塞：
      - 通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态
      - 当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态
- **死亡状态:**
  - 一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态





##### 3.7.2 线程的优先级



- 作用：每一个 Java 线程都有一个优先级，这样有助于操作系统确定线程的调度顺序
- 方法：其取值范围是 1 （Thread.MIN_PRIORITY ） - 10 （Thread.MAX_PRIORITY ）





##### 3.7.3 线程的创建



- 创建方法：
  - 通过实现 Runnable 接口
  - 通过继承 Thread 类本身
  - 通过 Callable 和 Future 创建线程





- 代码示例：两个各自跑完100m

  ```java
  package project6;
  
  public class Demo
  {
      public static void main(String[] args)
      {
          FH_Thread fh_thread_1 = new FH_Thread();
          FH_Thread fh_thread_2 = new FH_Thread();
  
          Thread thread_1 = new Thread(fh_thread_1, "<Fang> 跑>>");
          Thread thread_2 = new Thread(fh_thread_2, "<张三> 跑>>");
  
          thread_1.start();
          thread_2.start();
      }
  }
  
  class FH_Thread implements Runnable
  {
      int all = 100;
  
      public void run()
      {
          while (true)
          {
              if (all > 0)
              {
                  try
                  {
                      Thread.sleep((int)(100+Math.random()*500));
                  }
                  catch (InterruptedException e)
                  {
                      e.printStackTrace();
                  }
  
                  System.out.println(Thread.currentThread().getName() + " 5m");
                  all -= 5;
              }
              else
              {
                  System.out.println(Thread.currentThread().getName() + " 完100m");
                  break;
              }
          }
      }
  }
  ```





#### 3.8 文件IO



- 文件IO类：
  - [File Class(类)](https://www.runoob.com/java/java-file.html)
  - [FileReader Class(类)](https://www.runoob.com/java/java-filereader.html)
  - [FileWriter Class(类)](https://www.runoob.com/java/java-filewriter.html)



##### 3.8.1 代码示例



- 案例描述：通过文件IO，创建排行榜和排序

- 文件结构：
  - project7
    - Demo.java
    - LeaderBoard.java
    - Player.java
    - UpSort.java
    - LeaderBoard.txt



- 源码：

- Demo.java

  ```java
  package project7;
  
  public class Demo
  {
      public static void main(String[] args)
      {
          LeaderBoard leaderBoard = new LeaderBoard();
          leaderBoard.initPage();
          leaderBoard.writePage();
          leaderBoard.readPage();
          leaderBoard.updatePage();
          leaderBoard.readPage();
      }
  }
  ```

- LeaderBoard.java

  ```java
  package project7;
  
  import java.io.FileReader;
  import java.io.FileWriter;
  import java.io.IOException;
  import java.util.*;
  
  public class LeaderBoard
  {
      Scanner scanner =new Scanner(System.in);
      Vector<Player> vPlayer = new Vector<>();
      Comparator<Player> comparator = new UpSort();
  
      public void initPage()
      {
          System.out.println("<-- Origin Leader Board -->");
          Player player1 = new Player("A", 1);
          Player player2 = new Player("B", 4);
          Player player3 = new Player("C", 5);
          Player player4 = new Player("D", 8);
  
          vPlayer.add(player1);
          vPlayer.add(player2);
          vPlayer.add(player3);
          vPlayer.add(player4);
  
          vPlayer.sort(comparator);
      }
  
      public void readPage()
      {
          FileReader fileReader = null;
          try
          {
              fileReader = new FileReader("src\\project7\\leaderBoard.txt");
              int i;
              while ((i = fileReader.read()) != -1)
              {
                  System.out.print((char)i);
              }
          }
          catch (Exception e)
          {
              System.out.println("Page Can't Read");
          }
          finally
          {
              try
              {
                  assert fileReader != null;
                  fileReader.close();
              }
              catch (IOException e)
              {
                  System.out.println("Page Not Found");
              }
          }
      }
  
      public void writePage()
      {
          FileWriter fileWriter = null;
          try
          {
              fileWriter = new FileWriter("src\\project7\\leaderBoard.txt");
              for (Player player : vPlayer)
              {
                  fileWriter.write(player.grade + ": " + player.name + "\r\n");
              }
          }
          catch (Exception e)
          {
              System.out.println("Page Can't Write");
          }
          finally
          {
              try
              {
                  assert fileWriter != null;
                  fileWriter.close();
              }
              catch (IOException e)
              {
                  System.out.println("Page Not Found");
              }
          }
      }
  
      public void updatePage()
      {
          System.out.println("Add New Player >>");
  
          System.out.print("Input Name: ");
          String newName = scanner.next();
  
          System.out.print("Input Grade: ");
          int newGrade = scanner.nextInt();
  
          Player newPlayer = new Player(newName, newGrade);
          vPlayer.add(newPlayer);
          vPlayer.sort(comparator);
  
          System.out.println();
          System.out.println("### Updated Leader Board ###");
          System.out.println();
          System.out.println("<-- New Leader Board -->");
  
          writePage();
      }
  }
  ```

- Player.java

  ```java
  package project7;
  
  public class Player
  {
      String name;
      int grade;
  
      Player(String name, int grade)
      {
          this.name = name;
          this.grade = grade;
      }
  }
  ```

- UpSort.java

  ```java
  package project7;
  
  import java.util.Comparator;
  
  public class UpSort implements Comparator<Player>
  {
      public int compare(Player player1, Player player2)
      {
          return Integer.compare(player1.grade, player2.grade);
      }
  }
  ```

- LeaderBoard.txt

  ```txt
  1: A
  2: E
  4: B
  5: C
  8: D
  ```

  

# C语言



### 1. 编译预处理命令

三种：

1. 宏定义
2. 文件包含
3. 条件编译



#### 1.1 宏定义

宏是根据一系列定义的规则替换一段文本的一种模式

- 格式：#define 标识符 字符串

- 标识符：宏名，大写书写（区别与普通变量名）
- 字符串：宏体，赋值给标识符（编写时，直接写标识符，编译预处理时替换成字符串的值）
- 宏展开：宏名置换成宏体

定义宏（是否带参数）

- 无参

  直接用宏名代替字符串，称为 符号常量

- 有参

  替换宏名时，对参数进行转换



#### 1.2 文件包含命令

将已有的源文件，通过文件包含引入其他程序中，后续编程直接引用文件包含所有的定义对象，提高代码复用性和编写效率



### 2. 编译过程

源文件 .c 编译后，得到目标代码文件 .obj （0 和1组成），经过链接（组装）成可执行文件 .exe

 	



### 3. 运算量

表达式由运算量和运算符，正确使用运算量是正确编写表达式语句的前提

运算量：

1. 常量
2. 变量
3. 函数



#### 3.1 常量

分类：值常量，符号常量

##### 3.1.1 值常量

- 整形常量

- 实型常量

- 字符常量

  字符型常量和对应的ASCII码通用，可以把A理解成65，参与算术运算



##### 3.1.2 符号常量

用一个符号代替一个值常量，使用前，先用宏定义（无参宏）



#### 3.2 变量

命名要求：

- 标识符不能是关键字
- 标识符只能是字母、数字、下划线组成
- 标识符第一个字符必须是下划线或字母
- 标识符区分大小写



###### 3.2.1 类型占内存大小

- 整型

| 数据类型              | 占用空间                                            | 取值范围           |
| --------------------- | --------------------------------------------------- | ------------------ |
| short（短整型）       | 2字节                                               | （-2^15 - 2^15-1） |
| int（整型）           | 2字节                                               | （-2^31 - 2^31-1） |
| long（长整型）        | windows为4字节，Linux为4字节（32位），8字节（64位） | （-2^31 - 2^31-1） |
| long long（长长整型） | 8字节                                               | （-2^63 - 2^63-1） |

- 浮点型

| 数据类型 | 占用空间 | 有效数字范围      |
| -------- | -------- | ----------------- |
| float    | 4字节    | 7位有效数字       |
| double   | 8字节    | 15 - 16位有效数字 |

- 字符型

- 作用：显示单个字符

- 语法：char 变量名 = '变量值'

- 注意：char定义的变量只能用单引号；只能有一个字符

- 占用：char占用1个字节；将字符对应的ASCII码放入存储单元

- ASCII：	

  0 - 31 分配个控制字符

  32 - 126 分配个键盘上能找到的字符



#### 3.3 函数

和数学一样，返回数值的函数可以作为运算量参与表达式的运算

需要加入头文件 math.h

- sqrt(x)：求x的平方根
- fabs(x)：求x的绝对值
- exp(x)：求e的x次方





### 4. 运算符

分类：

- 算术运算符
- 关系运算符
- 逻辑运算符
- 位运算符
- 赋值运算符
- 条件运算符
- 求字节运算符
- 逗号运算符
- 指针运算符
- 其他

加减乘除运算为：二目运算符（目数取决于参与运算量的个数）

% ：求余或求模，返回两个整形数的余数



#### 4.1 算术运算符的结合性

当表达式中，所有运算符同级，从左往右算；否则，优先算高级。

特点表达式，从右往左算

自增（++），自减（--）：从右往左算 （只对变量做运算，不对表达式）

++x：先改变量值，再参与表达式运算

x++：先参与表达式运算，再改变量值

参数求值顺序位：从右往左



#### 4.2 关系及逻辑运算符

 

##### 4.2.1 关系运算符

实现同类型的运算量之间进行关系比较，用关系运算符连接的式子位关系表达式，值：逻辑真（1），逻辑假（0）



##### 4.2.2 逻辑运算符和表达式

逻辑与（&&），逻辑或（||），逻辑非（！）



#### 4.3 赋值、逗号、求字节



##### 4.3.1 赋值

形式：变量 = 表达式；

右结合性（14级）



##### 4.3.2 逗号

形式：表达式1，表达式2，... ，表达式n；

左结合性（最低优先级，15级）



##### 4.3.3 求字节

形式：sizeof 变量名 或 sizeof(类型名)

功能：求变量或数据类型内存空间占用的字节数

运算结果：整型数；

优先级：单目运算符，2级



#### 4.4 位运算

表达式的值，转二进制，一位一位的逻辑运算

1. 按位与
2. 按位或
3. 按位异或（真 异或 ？= ！？；假 异或 ？= ？）
4. 按位取反

![](C.assets\按位运算.png)

![](C.assets\位运算02.png)





### 5. 数据类型转换

转换方式：

- 赋值转换
- 自动转换
- 强制转换



#### 5.1 赋值转换

赋值时，将右边的值的类型转换成左边变量类型一致的类型



#### 5.2 自动转换

两种类型的数组在表达式中进行算术运算时，低精度会自动转成高精度

float 转成 double

char 转成 string

short/int 转成 long



#### 5.3 强制转换

系统无法自动转换，又必须与另一个类型的数据进行计算，用户需要手动指定转换类型

形式：（类型名）表达式；

注意：强制转换是临时转换，变量值不变，只在需要转换的表达式中有效





### 6. 算法

1. 概念

   为解决某一个具体问题采取明确，有限的操作步骤

2. 特性

   - 确定性：每一个步骤语义明确
   - 有穷性：任务可以复杂，多，但不能无限执行
   - 可行性：每一步都是C语言能够完成的
   - 输入：一个算法可以有0个或n个输入
   - 输出：一个算法至少有一个输出

3. 描述工具

   - 伪代码表示法
   - 传统流程图表示法
   - N-S结构化表示法

4. 结构化思想

   - 顺序结构
   - 选择结构
   - 循环结构

5. 语句种类

   - 表达式语句
   - 空语句
   - 函数调用语句
   - 控制语句
   - 复核语句



#### 6.1 字符输入输出函数



##### 6.1.1 字符输入函数

getchar()：从输入设备中，读入1个字符，返回字符的ASCII码，并通过程序流显示在屏幕上

例：char a；a = getchar()；

注：getch() 后，无需回车键进入下一步



##### 6.1.2 字符输出函数

putchar()：输出一个字符

例：char ch = 'A';

putchar(ch);

putchar('A');

putchar(65);

putchar('\101');

putchar('\x41');

注：putch()：也只输出一个字符



#### 6.2 格式输入输出函数



##### 6.2.1 格式输出函数

例：printf(“格式控制字符串”， 输出字符列表)；

![](C.assets\格式输出.png)



##### 6.2.2 格式输入函数

scanf("格式控制字符"， 字符地址列表)；

例：scanf("a=%d, b=%d", &a, &b);



#### 6.3 三目运算

- 作用：通过三目运算符实现简单的判断
- 语法：表达式a ？表达式b ：表达式c
- 解释：
  1. a为真，执行b，并返回b的结果；
  2. a为假，执行c，并返回c的结果；

```c
#include <stdio>

void main()
{
	int a = 10, b = 20;
	int c;
	
	c = (a > b ? a : b);
	cout << "c = " << c << endl;

	//三目运算符表达式返回的是 变量，所以可以直接作为左值被赋值；
	(a < b ? a : b) = 100;
	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
 }
```





#### 6.4 switch

- 语法：

```C++
switch (表达式)
{
    case 结果1:
    执行语句;
    break；
    case 结果1:
    执行语句;
    break；
    ...
    default:
    执行语句;
    break；
}
```

- 注意：

  switch语句的表达式类型只能是整型或字符型；

  case后没有break语句，程序会一直向下执行；

```C++
#include <iostream>

using namespace std;

void main()
{
	int value;

	cout << "Enter int value : " << endl;
	cin >> value;

	switch (value)
	{
	case 10:
		cout << "Return S" << endl;
		break;
	case 9:
		cout << "Return A" << endl;
		break;
	case 8:
		cout << "Return A" << endl;
		break;
	case 7:
		cout << "Return B" << endl;
		break;
	case 6:
		cout << "Return B" << endl;
		break;
	case 5:
		cout << "Return C" << endl;
		break;
	case 4:
		cout << "Return C" << endl;
		break;
	default:
		cout << "Return D" << endl;
		break;
	}
}
```





### 7. 循环结构程序

1. while
2. do-while
3. for



##### 7.1 while循环

- 作用：满足判断条件，执行循环语句

- 语法：

  ```C++
  while (判断条件)
  {
  	循环语句;
  }
  ```

- 案列：

  ```C++
  #include <iostream>
  
  using namespace std;
  
  void main()
  {
  	int num = 0;
  
  	while (num < 10)
  	{
  		cout << "num = " << num << endl;
  		num++;
  	}
  }
  ```



##### 7.2 do-while

- 作用：先执行循环语句，再满足判断条件，执行循环语句

- 注意：do...while 与 while的区别在于，do...while 先执行一次循环语句，再判断条件是否满足继续执行

- 语法：

  ```c++
  do
  {
      循环语句;
  }while (循环条件);
  ```

  ```c++
  #include <iostream>
  
  using namespace std;
  
  void main()
  {
  	int num = 0;
  
  	do
  	{
  		cout << "num = " << num << endl;
  		num++;
  	} while (num < 10);
  }
  
  ```

  

水仙花案列

- 说明：一个三位的整数满足每一位的三次方的和依旧等于这个三位数 （do ... while）

  ```c++
  #include <iostream>
  
  using namespace std;
  
  void main()
  {
  	int dNum = 100; //最小的三位数
  
  	do
  	{
  		double a, b, c, d; // a为百位，b为十位，c为个位，d为 a b c三次方的和
  
  		//pow(x, y) == x 的 y 次方
  		a = pow(dNum / 100, 3);
  		b = pow(dNum / 10 % 10, 3);
  		c = pow(dNum % 10, 3);
  		d = a + b + c;
  
  		//判断当前的三位数是不是水仙花数
  		if (d == dNum)
  		{
  			//条件为真时 打印水仙花数
  			cout << dNum << endl;
  		}
  		dNum++; //每当while条件满足都执行一遍
  	} while (dNum < 1000); //判断当前是否是三位数
  }
  ```





##### 7.3 for

- 作用：满足条件，执行语句

- 语法：

  ```c++
  for (起始表达式; 循环条件; 循环体)
  {
  	循环语句;
  }
  ```

  ```c++
  #include <iostream>
  
  using namespace std;
  
  void main()
  {
  	for (int i = 0; i < 10; i++)
  	{
  		cout << i << endl;
  	}
  }
  ```

  

for 循环案列-1

- 说明：1 - 100 的区间数字，满足 各位 或 十位 或 倍数 与 7 有关，打印 yes，其余直接打印数字；

  ```c++
  #include <iostream>
  
  using namespace std;
  
  void main()
  {
  	for (int i = 1; i <= 100; i++) //循环打印 1 - 100 
  	{
  		//判断遍历的数字是否满足条件
  		if (i % 10 == 7 || i / 10 % 10 == 7 || i % 7 == 0)
  		{
  			cout << "Yes" << endl; //满足调件打印 yes
  		}
  		else
  		{
  			cout << i << endl; //不满足条件打印 原数字
  		}
  	}
  }
  ```



for 循环案列-2

- 判断一个整数是否为素数（【2， n-1】内没有n的因子）

  ```c
  #include <stdio.h>
  
  #define Log printf
  #define Enter scanf_s
  
  int main()
  {
      int a = 0;  // 素数的个数
      int num = 0;  // 输入的整数
  
      Log("输入一个整数：");
      Enter("%d", &num);
  
      for (int i = 2; i < num; i++)
      {
          if (num % i == 0)
          {
              a++;  // 素数个数加1
          }
      }
  
      if (a == 0)
      {
          Log("%d是素数。\n", num);
      }
      else
      {
          Log("%d不是素数。\n", num);
      }
  
      return 1;
  }
  ```





##### 7.4 goto

- 作用：无条件跳转语句(转向语句)，控制程序流程无条件转移至“语句标号”所指定的语句，开始执行
- 语法：goto 标记;
- 解释：程序执行到goto时，如果标记存在，怎直接跳转到标记处，并继续执行
- 缺点：功能太强，容易破坏程序

```c
#include <iostream>

using namespace std;

void main()
{
	cout << "S" << endl;
	cout << "A" << endl;

	goto GotoTarget; //此处的 goto 语句标记为 GotoTarget

	cout << "B" << endl;
	cout << "C" << endl;
	cout << "D" << endl;

	GotoTarget: //程序直接跳转至此处，并继续向下执行
	cout << "E" << endl;
}
```

```c
#include <stdio.h>

int main()
{
    int n = 1;
    int sum = 0;

    NEXT:
    sum = sum + n;
    n++;

    if (sum <= 200)
    {
        goto NEXT;
    }
    printf("sum=%d\n", sum);
}
```

```c
#include <stdio.h>

int main()
{
    int n = 1;
    int sum = 0;

    LOOP:
    if (sum >= 200)
    {
        goto END;
    }
    else
    {
        sum += n;
        n++;
        goto LOOP;
    }
    END:
    printf("sum=%d\n", sum);

    return 0;
}
```





##### 7.5 break和continue



###### 7.5.1 break

- 作用：用于跳出选择结构或者循环结构
- 使用：
  - 出现在switch语句中，终止case并跳出switch；
  - 出现在循环语句中，跳出循环；
  - 出现在嵌套循环中，跳出内层循环；



###### 7.5.2 continue

只能在循环体中使用

- 作用：在循环语句中，跳过本次循环中余下尚未执行的语句，继续执行下一循环

```c++
#include <iostream>

using namespace std;

void main()
{
	//从1 -100 遍历100次
	for (int i = 1; i <= 100; i++)
	{
		//如果i 为偶数，则跳过当前循环，执行下一次遍历
		if (i % 2 == 0)
		{
			continue;
		}
		cout << i << endl;
	}
}
```





##### 7.6 循环嵌套

- 作用：在循环语句内再添加循环，解决实际问题

- 注意：一定要确保被嵌套的循环结构完整的包含在外层循环的结构体中

- 描述：打印 10*10 的矩阵

  ```c++
   #include <iostream>
  
  using namespace std;
  
  void main()
  {
  	for (int i = 0; i < 10; i++)
  	{
  		for (int j = 0; j < 10; j++)
  		{
  			cout << "* ";
  		}
  		cout << endl;
  	}
  }
  ```

  

打印乘法表：

```c
#include <stdio.h>

int main()
{
    for (int h = 1; h <= 9; h++) //行数
    {
        for (int v = 1; v <= h; v++) //列数，但不超过行数
        {
            printf("%d * %d = %d   ", v, h, v*h);//结果为 列数*行数=
        }
        printf("\n");
    }
    
    return 0;
}
```



打印菱形：

```c
#include <stdio.h>

#define Log printf
#define Enter scanf_s

int main()
{
    int enterNum = 0;

    Log("Enter Num :");
    Enter("%d", &enterNum);
    Log("\n");

    for (int i = 1; i <= enterNum; i++)
    {
        for (int j = 1; j <= enterNum - i; j++)
        {
            Log(" ");
        }
        for (int k = 1; k <= 2 * i - 1; k++)
        {
            Log("*");
        }
        Log("\n");
    }

    for (int i = enterNum - 1; i >= 1; i--)
    {
        for (int j = 1; j <= enterNum - i; j++)
        {
            Log(" ");
        }
        for (int k = 1; k <= 2 * i - 1; k++)
        {
            Log("*");
        }
        Log("\n");
    }

    return 0;
}
```





### 8. 数组



##### 8.1 一维数组



###### 8.1.1 一维数组的定义

【存储类型】类型说明符 数组名 【常量表达式】

1. 数据类型 数组名[数组长度];

2. 数据类型 数组名[数组长度] = {元素1，元素2，.....};

3. 数据类型 数组名[] = {元素1，元素2，.....};

   ```c++
   #include <iostream>
   
   using namespace std;
   
   void main()
   {
   	//1. 数据类型 数组名[数组长度];
   	cout << "数据类型 数组名[数组长度]" << endl;
   
   	int arr1[3];
   	arr1[0] = 1;
   	arr1[1] = 2;
   	arr1[2] = 3;
   	
   	for (int i = 0; i < 3; i++)
   	{
   		cout << arr1[i] << "  ";
   	}
   	cout << endl;
   
   	//2. 数据类型 数组名[数组长度] = {元素1，元素2，.....};
   	cout << "数据类型 数组名[数组长度] = {元素1，元素2，.....}" << endl;
   
   	int arr2[3] = { 1, 2, 4 };
   	
   	for (int j = 0; j < 3; j++)
   	{
   		cout << arr2[j] << "  ";
   	}
   	cout << endl;
   
   	//3. 数据类型 数组名[] = {元素1，元素2，.....};
   	cout << "数据类型 数组名[] = {元素1，元素2，.....}" << endl;
   
   	int arr3[] = {1, 2, 3};
   
   	for (int k = 0; k < 3; k++)
   	{
   		cout << arr1[k] << "  ";
   	}
   	cout << endl;
   }
   ```



###### 8.1.2 一维数组名作用

内存空间由数组的类型和元素的个数共同决定

L = 元素个数 * sizeof（类型名）

作用：

1. 可以统计整个数组在内存中的长度

2. 可以获取数组在内存中的首地址

3. 可以获取数组元素的个数

   ```c++
   #include <iostream>
   
   using namespace std;
   
   void main()
   {
   	int arr[10] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
   
   	cout << "arr数组占用的内存空间：" << sizeof(arr) << endl;
   	cout << "arr数组单个元素占用空间：" << sizeof(arr[0]) << endl;
   	cout << "arr数组的元素个数：" << sizeof(arr)/sizeof(arr[0]) << endl;
   	cout << "arr数组的首地址：" << (int)arr << endl;
   	cout << "arr数组第一元素的地址：" << (int)&arr[0] << endl;
   	cout << "arr数组第二元素的地址：" << (int)&arr[1] << endl;
   	cout << "arr数组最后元素的地址：" << (int)&arr[sizeof(arr) / sizeof(arr[0])] << endl;
   }
   ```






###### 8.1.3 输入数字，按顺序输出

```c
#include <stdio.h>

int main()
{
    static int a[10];
    for (int i = 0; i < 10; ++i)
    {
        printf("Enter Num to Array :");
        scanf_s("%d", &a[i]);
    }

    for (int j = 0; j < 10; ++j)
    {
        printf("%3d", a[j]);
    }

    for (int k = 9; k >= 0; --k)
    {
        printf("%3d", a[k]);
    }

    return 0;
}
```



###### 8.1.4 输入数字，输出最大值

```c
#include <stdio.h>

int main()
{
    int maxNum = 0;
    static int a[10];

    for (int i = 0; i < 10; ++i)
    {
        printf("Enter Num :");
        scanf_s("%d", &a[i]);
    }

    for (int j = 0; j < 10; ++j)
    {
        if (a[j] > maxNum)
        {
            maxNum = a[j];
            printf("\n");
        }
    }
    printf("MaxNum is :%d", maxNum);

    return 0;
}
```



###### 8.1.5 输入数字，求平均值

```c
使用一维数组循环
#include <stdio.h>

int main()
{
    float sumNum = 0, averageNum, enterNum;
    static int a[5];

    for (int i = 0; i < 5; ++i)
    {
        printf("Enter Num :");
        scanf_s("%f", &enterNum);
        sumNum += enterNum;
    }

    averageNum = sumNum / 5;
    printf("AverageNum is :%.2f", averageNum);

    return 0;
}

不使用数组循环
#include <stdio.h>

int main()
{
    float enterNum, sumNum = 0, averageNum;

    for (int i = 0; i < 5; ++i)
    {
        printf("Enter Num :");
        scanf_s("%f", &enterNum);
        sumNum += enterNum;
    }

    averageNum = sumNum / 5;

    printf("AverageNum is :%.2f", averageNum);

    return 0;
}
```





##### 8.2 二维数组

- 形式：【存储类型】类型说明符 数组名 【常量表达式1】【常量表达式2】；

- 说明：二维数组元素在内存中排列顺序为按行为序排列

- 初始化：

  ![](C.assets\二维数组初始化.png)

- 



###### 8.2.1 二维数组的定义

四种定义方式：

1. 数据类型 数组名 [行数] [列数]；
2. 数据类型 数组名 [行数] [列数] = ｛｛数据1，数据2｝，｛数据3，数据4｝｝；
3. 数据类型 数组名 [行数] [列数] = ｛数据1，数据2，数据3，数据4｝；
4. 数据类型 数组名 [] [列数] = ｛数据1，数据2，数据3，数据4｝；

第二种更直观，可读性更高

```c++
#include <iostream>

using namespace std;

void main()
{
	//1. 数据类型 数组名[行数][列数]；
	int arr1[2][3];
	
	arr1[0][0] = 1;
	arr1[0][1] = 2;
	arr1[0][2] = 3;
	arr1[1][0] = 4;
	arr1[1][1] = 5;
	arr1[1][2] = 6;

	cout << "------数据类型 数组名[行数][列数]------" << endl << "\t";
	for (int a = 0; a < 2; a++)
	{
		for (int a1 = 0; a1 < 3; a1++)
		{
			cout << arr1[a][a1] << " ";
		}
	}
	cout << endl;

	//2. 数据类型 数组名[行数][列数] = {｛数据1，数据2｝，｛数据3，数据4｝}；
	int arr2[2][3] =
	{
		{1, 2, 3},
		{4, 5, 6}
	};

	cout << "------数据类型 数组名[行数][列数] = {｛数据1，数据2｝，｛数据3，数据4｝}------" << endl << "\t";
	for (int b = 0; b < 2; b++)
	{
		for (int b1 = 0; b1 < 3; b1++)
		{
			cout << arr2[b][b1] << " ";
		}
	}
	cout << endl;

	//3. 数据类型 数组名[行数][列数] = ｛数据1，数据2，数据3，数据4｝；
	int arr3[2][3] = { 1, 2, 3, 4, 5, 6 };

	cout << "------数据类型 数组名[行数][列数] = ｛数据1，数据2，数据3，数据4｝------" << endl << "\t";
	for (int c = 0; c < 2; c++)
	{
		for (int c1 = 0; c1 < 3; c1++)
		{
			cout << arr3[c][c1] << " ";
		}
	}
	cout << endl;

	//4. 数据类型 数组名[][列数] = ｛数据1，数据2，数据3，数据4｝；
	int arr4[][3] = { 1, 2, 3, 4, 5, 6 };

	cout << "------数据类型 数组名[][列数] = ｛数据1，数据2，数据3，数据4｝------" << endl << "\t";
	for (int d = 0; d < 2; d++)
	{
		for (int d1 = 0; d1 < 3; d1++)
		{
			cout << arr4[d][d1] << " ";
		}
	}
	cout << endl;
}
```



###### 8.2.2 二维数组名作用

1. 查看二维数组所占空间

2. 查看二维数组的首地址

   ```c++
   #include <iostream>
   
   using namespace std;
   
   void main()
   {
   	int arr[2][3] =
   	{
   		{1, 2, 3},
   		{4, 5, 6}
   	};
   
   	cout << "二维数组的大小：" << sizeof(arr) << endl;
   	cout << "二维数组一行的大小：" << sizeof(arr[0]) << endl;
   	cout << "二维数组元素的大小：" << sizeof(arr[0][0]) << endl;
   	cout << "二维数组的行数：" << sizeof(arr) / sizeof(arr[0]) << endl;
   	cout << "二维数组的列数：" << sizeof(arr[0]) / sizeof(arr[0][0]) << endl;
   
   	cout << "*************************" << endl;
   	//地址
   	cout << "二维数组的首地址：" << (int)arr << endl;
   	cout << "二维数组第一行的地址：" << (int)&arr[0] << endl;
   	cout << "二维数组第二行的地址：" << (int)&arr[1] << endl;
   	cout << "二维数组第一个元素的地址：" << (int)&arr[0][0] << endl;
   	cout << "二维数组第二个元素的地址：" << (int)&arr[0][1] << endl;
   }
   ```



###### 8.2.3 二维数组输入输出

```c
#include <stdio.h>

int main()
{
    static int a[2][3];

    printf("Enter 6 Num:\n");

    for (int i = 0; i < 2; ++i)
    {
        for (int j = 0; j < 3; ++j)
        {
            printf("%d Row %d Column:", i, j);
            scanf_s("%d", &a[i][j]);
        }
    }

    printf("Log Array :\n");

    for (int k = 0; k < 2; ++k)
    {
        for (int g = 0; g < 3; ++g)
        {
            printf("%3d", a[k][g]);
        }
        printf("\n");
    }

    return 0;
}
```





###### 8.2.4 二维数组找到最大元素及索引

```c
#include <stdio.h>

int main()
{
    static int a[3][4];
    int max = a[0][0];
    int row = 0, column = 0;

    printf("Enter 12 Num To Array\n");

    for (int i = 0; i < 3; ++i)
    {
        for (int j = 0; j < 4; ++j)
        {
            printf("%d Row %d Column :", i, j);
            scanf_s("%d", &a[i][j]);
        }
    }

    printf("Log Array\n");

    for (int k = 0; k < 3; ++k)
    {
        for (int g = 0; g < 4; ++g)
        {
            if (max < a[k][g])
            {
                max = a[k][g];
                row = k;
                column = g;
            }
            printf("%3d", a[k][g]);
        }
        printf("\n");
    }

    printf("Max num : %d;\t Row : %d;\t Column : %d\t", max, row, column);

    return 0;
}
```



###### 8.2.5 3*3二维数组求对角线值和

```c
第一种：效率低
#include <stdio.h>

int main()
{
    int a[3][3] = {{1, 2, 3},
                   {4, 5, 6},
                   {7, 8, 9}};
    int sum = 0;

    for (int i = 0; i < 3; ++i)
    {
        for (int j = 0; j < 3; ++j)
        {
            if (i == j)
            {
                sum += a[i][j];
            }
        }
    }
    printf("3*3的二维数组对角线的值和：%d", sum);

    return 0;
}

第二种：效率高
#include <stdio.h>

int main()
{
    int a[3][3] = {{1, 2, 3},
                   {4, 5, 6},
                   {7, 8, 9}};
    int sum = 0;

    for (int i = 0; i < 3; ++i)
    {
        sum += a[i][i];
    }

    printf("3*3的二维数组对角线的值和：%d", sum);

    return 0;
}
```





##### 8.3 字符数组

- 说明：字符数组相较于普通数组，除了它们的共同特性外，字符数组具有独自的特点

- 定义：char 数组名 【常量表达式】； ----  char a 【10】；

  定义字符数组a有10个元素，每个元素一个字符变量，占一个字节空间，用于存放一个字符变量



###### 8.3.1 字符数组初始化

 1. 逐个元素初始化：char c[3] = {'a', 'b', 'c'};	

     （如果初值个数>数组长度，语法错误）

     （如果初值个数<数组长度，则其余元素自动定义为“串结束符”：\0）

     （如果初值个数=数组长度，则定义时，可省略数组长度）

  2. 用字符串初始化：char a[11] = {"I am a boy"};

     ![](C.assets\字符数组初始化.png)



###### 8.3.2 字符数组输入输出

char a[20];

  1. 用格式符”%c“，逐个字符输入输出

     scanf ("%c", &a[0]);

     printf ("%c", a[0]);

  2. 用格式符”%s“，整个字符串输入输出 (字符串输出时，数组长度至少大于初值个数一个单元，即默认的”\0“，否则输出结果不符合预期)

     scanf ("%s", &a);

     printf ("%s", a);

     ![](C.assets\字符输入输出1.png)

     

     ![](C.assets\gets_puts.png)





###### 8.3.3 字符串处理函数

![](C.assets\字符串处理函数.png)

![](C.assets\字符串处理函数1.png)







### 9. 函数



##### 9.1 函数的定义



###### 9.1.1 函数的分类

1. 是否由用户定义
   - 标准库函数
   - 用户自定义函数
2. 是否返回值
   - 返回值的函数：sqrt(x) fabs(x)
   - 不返回值的函数:  clrscr()
3. 是否有参数
   - 有参函数：sqrt(x) fabs(x)
   - 无参函数:  clrscr()
4. 是否在文件内部
   - 内部函数
   - 外部函数



###### 9.1.2 函数的定义

1. 方式一：

   ```
   函数返回值类型 函数名(参数列表)
   	参数类型说明
   	{
   		变量说明；
   		语句序列；
   	}
   ```

2. 方式二：

   ```
   函数返回值类型 函数名(参数类型说明 参数列表)
   {
   	变量说明；
   	语句序列；
   }
   ```

   

###### 9.1.3 定义不返回值的函数

```c
#include <stdio.h>

void write() //无返回值，无参数的函数
{
    for (int i = 0; i <= 12; ++i)
    {
        printf("%c", '*');
    }
    printf("\n");
}

int main()
{
    write();
    return 0;
}
```

```c
#include <stdio.h>

void write(int num) //无返回值，有参数的函数
{
    for (int i = 0; i <= num; ++i)
    {
        printf("%c", '*');
    }
    printf("\n");
}

int main()
{
    write(12);
    return 0;
}
```



###### 9.1.4 定义返回值的函数

```c
#include <stdio.h>

int sum(int num)
{
    int sum = 0;
    for (int i = 0; i <= num; ++i)
    {
        sum += i;
    }
    return sum;
}

int main()
{
    printf("%d", sum(3));
    return 0;
}
```



##### 9.2 函数的使用



###### 9.2.1 函数的执行过程

例：求前n个自然数和的平均值

```c
#include <stdio.h>

float averageNum(float num)
{
    float sum = 0.0f, average;
    int count;
    for (int i = 0; i <= (int)num ; ++i)
    {
        sum += (float)i;
        count = i;
    }
    average = sum / (float)count;

    return average;
}

int main()
{
    int enterNum;
    printf("enter num:");
    scanf_s("%d", &enterNum);
    printf("averageNum=%7.2f", averageNum((float)enterNum));
    return 0;
}
```



##### 9.3 函数的调用形式

- 语句调用
- 表达式调用



##### 9.4 函数的声明

声明：先定义后调用，声明的函数必须放在main函数之前，否则运行错误

注：

- C++中可以运行，但结果错误
- 返回值为int，权限高，可以不声明

```c
#include <stdio.h>

int maxNum(int x, int y, int z); // 先声明函数

int main()
{
    printf("MaxNum:%d", maxNum(10, 20, 30));
    return 0;
}

int maxNum(int x, int y, int z) // 在mian()函数后面定义函数
{
    int max = 0;
    max = x > y ? x : y;
    max = max > z ? max : z;
    return max;
}
```



##### 9.5 带参宏和函数的区别

区别：宏中的 sum(x, y)，x和y不存值，而是将函数中出现sum(x, y)类型的表达式用 x+y替代，并返回，再有函数将表达式x+y赋值计算

- 宏：返回替换的表达式，带入函数中计算
- 函数：带入参数计算，返回结果

```c
#include <stdio.h>

#define sum(x, y) x+y; // 带参宏

int main()
{
    int num = 0;
    num = sum(10, 20); // 展开： num = sum(x+y) -> num = x + y
    printf("Sum Num :%d", num);
    return 0;
}
```



##### 9.6 函数的嵌套和递归调用



###### 9.6.1 函数的嵌套调用

![](C.assets\函数的嵌套调用.png)



###### 9.6.2 例题

<img src="C.assets\嵌套递归例子.png" style="zoom:50%;" />

```c
#include <stdio.h>

int funcSum(int num);

float funcCalculate(int m, int n, int p);

int main()
{
    int m, n, p;
    printf("enter num:");
    scanf_s("%d %d %d", &m, &n, &p);
    float num = funcCalculate(m, n, p);
    printf("Calculate Num :%7.2f", num);
    return 0;
}

int funcSum(int num)
{
    int sum;
    for (sum = 0; num > 0; --num)
    {
        sum += num;
    }
    return sum;
}

float funcCalculate(int m, int n, int p)
{
    float y = (float)(funcSum(m) * funcSum(n));
    y = y / (float)funcSum(p);
    return y;
}
```



###### 9.6.3 函数的递归调用

<img src="C.assets\函数的嵌套与递归调用.png" style="zoom:50%;" />





##### 9.7 变量的作用域



###### 9.7.1 定义和区别

<img src="C.assets\变量的作用域.png" style="zoom:67%;" />



###### 9.7.2 全局变量例子

<img src="C.assets\变量的作用域_全局变量.png" style="zoom:67%;" />



###### 9.7.3 全局变量和局部变量同名

<img src="C.assets\变量的作用域_全局变量与局部变量同名.png" style="zoom: 67%;" />





##### 9.8 变量的存储类型



###### 9.8.1 定义和分类

![](C.assets\变量的存储类型.png)

![](C.assets\变量的存储类型_1.png)



###### 9.8.2 例子

<img src="C.assets\变量的存储类型_静态局部变量应用举例.png" style="zoom:50%;" />

<img src="C.assets\变量的存储类型_静态局部变量应用举例_1.png" style="zoom:50%;" />







### 10. 指针



##### 10.1 指针的概念

- 计算机中为方便地区不同的内存单元，会为每个单元分配内存地址，这个地址称为内存单元地址，简称：内存地址
- C语言中把指针定义为地址，指针是地址的另一种描述，相同的概念：指针==地址



##### 10.2 指针的类型

- 指针指向的对象类型可能不同
- 变量，数组，函数，结构体均是内存中的存储对象，都有这不同的存储类型，不同类型的存储空间大小不同，它们的地址是属于不同类型的指针。
- 不同类型的指针之间，不能直接有任何操作



##### 10.3 指针和变量



###### 10.3.1 变量的指针

- 一个变量通常占用若干个连续的内存单元，把这若干个单元起始单元的地址称为变量的地址
- 内存单元本身的地址是0和1组成，不方便使用，C语言用 "&变量名" 表示变量的地址
- 不同类型的变量，指针的类型不同



###### 10.3.2 指向变量的指针变量

- 指针变量：专门存放地址的变量

- 指向变量的指针变量：专门存放其他变量地址的指针变量，该指针变量指向对象是一个变量

  1. 形式：数据类型名 *变量名；

  2. 例：

     ```c
     int x = 5;
     int *p1; # p1是变量名， *是定义符，向系统说明p1是指针变量（存放地址）
     p1 = &x; # &x是整形变量的地址，将其赋值个p1保存，此时p1保存了x的地址
     # x是p1指向的变量，p1是x变量的指针变量
     ```

- 指针运算符 "*" 又称间接访问运算符：间址运算符（单目运算符，第二优先级，右结合性）

- 形式：*地址表达式

  1. 先计算表达式的值，再根据表达式的值（地址）及类型，访问内存中相应的对象，分别是两种：从内存中读取，往内存中写入

  2. “*&x”：右结合性 == * (&x) == x == *&x

     ```c
     int a = 5;
     int *pa; # 此处的*是定义一个pa的指针变量
     *pa = &a; # 把 a的地址存进去，&（引用）获得变量的地址，此处 *（解析）指向并得到地址内的值
     printf("*pa=%d", *pa); # 打印结果为 *pa=5；
     ```

     ```c
     #include <stdio.h>
     
     int main()
     {
         int a = 5, b = 10, *pa = NULL, *pb = NULL;
         pa = &a;
         pb = &b;
         printf("*pa = %d, *pb = %d", *pa, *pb);
     
         *pb = *pa + *pb;
         printf("\n*pb = %d", *pb);
     
         return 0;
     }
     ```



###### 10.3.3 指向指针变量的指针

- 定义一个指针变量，让它保存其他指针变量的地址，则该指针变量为指向指针的指针变量（多级指针）

- ```c
  int x = 5;
  int *px;
  px = &5; # 此时px存储了x的地址
  
  int *py;
  py = &px; # py存储了px的地址（二级指针)
  ```

- ```c
  #include <stdio.h>
  
  int main()
  {
      int a = 5;
      int *pa;
      int **pb; # 定义一个 int * 类型的指针变量（二级指针）
  
      pa = &a; # a的地址赋值该指针变量 pa
      pb = &pa; # 指针变量pa的地址赋值 pb
  
      printf("**pb = %d", **pb); # **pb == *(*pb): (*pb)解析pb内地址指向的变量pa，*(*pb) == *pa 
  
      return 0;
  }
  ```





##### 10.4 指针与数组

- 一个数组占用一片连续的内存空间，数组元素的使用可以和同类型的变量相比



###### 10.4.1 一维数组与指针

1. 一维数组的指针

   ```c
   int arrX[5], i;
   # w + i == &arrX[i] : 表示第i个元素的地址
   # w == &arrX[0] : 表示数组的首地址和第0个元素的地址
   # *(w + i) == arrX[i] : 表示dii个元素 / * （解析）w + i 
   ```

2. 指向数组元素的指针变量

   ```c
   int arrX[5], i, *p;
   # p = &i;
   # p = &arrX[i];
   # p = w + i;
   # 当i=0 有 p = w，此时p指向arrX[0]
   ```

3. 引用数组元素方法：int a[5], *p = a;

   - 下标法：a[i], p[i]

   - 指针法：*(a+i), *(p+i)

   - 例子：

     ```c
     #include <stdio.h>
     
     int main()
     {
         int arr[10], *p = NULL;
         p = arr;
         int x = 0;
     
         while (p < arr+10)
         {
             printf("arr[%d]=", x);
             scanf_s("%d", p);
             p++;
             x++;
         }
     
         for (p = arr + 9; p >= arr ; --p)
         {
             printf("%3d", *p);
         }
     
         return 0;
     }
     ```

     ```c
     #include <stdio.h>
     
     int main()
     {
         int arr[10] = {10, 20, 30, 40, 50, 60, 70, 80, 90, 100};
         int *p = arr + 2;
         for (int i = 0; i < 5; ++i)
         {
             printf("%5d", p[i]);
         }
     
         return 0;
     }
     ```
     
     

###### 10.4.2 二维数组与指针

```c
int i, j, arr[3][4];
w + i # 表示i行的行地址；i = 0，arr表示第0行的行地址
```

```c
*(arr + i) # 与arr[i]相同，表示第i行的一维数组名称，也是arr[i][0]的元素地址
*(arr + i) + j # 表示&arr[i][j]，表示第i行第j列的元素地址
*(*(w + i)+ j) ==== arr[i][j]
```

```c
int i, j, k, arr[i][j][k];
*(*(*(arr + i) +j) +k) == arr[i][j][k]
```

<img src="C.assets\二维数组元素不同表示形式.png" style="zoom:75%;" />

<img src="C.assets\二维数组元素不同表示形式01.png" style="zoom:75%;" />





##### 10.5 指针和字符



###### 10.5.1 指针变量访问字符数组

```c
#include <stdio.h>

int main()
{
    char *str = "Hello World !";
    printf("%s\n", str);
    printf("%s\n", str + 6);

    return 0;
}
```



###### 10.5.2 指向字符的指针变量

- 输入字符串，判断是否是回文符

- ```c
  #include <stdio.h>
  #include <string.h>
  
  int main()
  {
      char str[20], *str_Front, *str_Rear;
  
      printf("Enter String :");
      gets_s(str, 20);
      str_Rear = str + strlen(str) - 1;
  
      for (str_Front = str; str_Front < str_Rear; str_Front++, str_Rear--)
      {
          if (*str_Front != *str_Rear)
          {
              break;
          }
      }
  
      printf("%d %d", *str_Front, *str_Rear);
  
      if (*str_Front > *str_Rear)
      {
          printf(" F");
      }
      else
      {
          printf(" T");
      }
  
      return 0;
  }
  ```







##### 10.6 指针和函数



###### 10.6.1 指针变量作为函数的参数

```c
#include <stdio.h>

void funcSwap(int *x, int *y);

int main()
{
    int x = 2, y = 3;
    funcSwap(&x, &y); # & 为引用 得到 x的内存地址
    printf("x = %d, y = %d", x, y);

    return 0;
}

void funcSwap(int *x, int *y) # 此处是写如 * 为定义 地址变量（指针变量）
{
    int s = *x; #访问时，* 为解析地址的内存值 == x
    *x = *y;
    *y = s;
}
```



###### 10.6.2 指针变量实现字符串复制

```c
#include <stdio.h>

void funcCopyStr(char *originStr, char *flagStr);

int main()
{
    char str_Origin[100], str_Flag[100];

    printf("Enter String To Copy :");
    gets_s(str_Origin, 100);

    funcCopyStr(str_Origin, str_Flag);
    printf("%s\n", str_Origin);
    printf("%s\n", str_Flag);

    return 0;
}

void funcCopyStr(char *originStr, char *flagStr)
{
    for (; (*flagStr = *originStr); originStr++, flagStr++);
}
```



###### 10.6.3 指针数组做函数参数

```c
#include <stdio.h>

int main(int n, char *args[]) # mian函数内只能定义这样的形参 n == 命令单词的个数；*args[] == 保存命令参数的地址
{
    printf("n=%d", n);

    for (int i = 1; i < n; ++i)
    {
        printf("%s\n", args[i]);
    }

    return 0;
}
```



###### 10.6.4 指针函数

- 函数也可以返回地址（指针函数）

- 类型标识符 *函数名(参数表) {}

- ```c
  #include <stdio.h>
  
  int *funcJudgeMaxNum(int *p_num1, int *p_num2);
  
  int main()
  {
      int num1, num2, *maxNum;
  
      printf("Enter num1 and num2 :");
      scanf_s("%d %d", &num1, &num2);
  
      maxNum = funcJudgeMaxNum(&num1, &num2);
      printf("MaxNum = %d", *maxNum);
  
      return 0;
  }
  
  int *funcJudgeMaxNum(int *p_num1, int *p_num2)
  {
      return *p_num1 > *p_num2 ? p_num1 : p_num2;
  }
  ```



###### 10.6.5 指针函数动态获取内存空间

```c
#include <stdio.h>
#include <malloc.h>

int main()
{
    int numCount, *p_TestArr;

    printf("Enter num count=");
    scanf_s("%d", &numCount);

    p_TestArr = (int *)malloc(numCount * sizeof(int)); # 申请一个numCount*2字节的空间并转换成内存地址赋值个 p_TestArr

    for (int i = 0; i < numCount; ++i)
    {
        printf("TestArr[%d]=", i);
        scanf_s("%d", &p_TestArr[i]);
    }

    printf("This Arr=");
    for (int j = 0; j < numCount; ++j)
    {
        printf("%4d", p_TestArr[j]);
    }

    return 0;
}
```



###### 10.6.6 指向函数的指针变量

- 概念：一个专门保存函数地址的指针变量

- 定义：

  ```c
  int (*f) (int1, int2); # 参数类型可以不带
  float (*p) ();
  float *(*f) ();
  ```

- ```c
  #include <stdio.h>
  
  float func_AddNum(float num1, float num2);
  float func_SubNum(float num1, float num2);
  float func_MulNum(float num1, float num2);
  float func_DivNum(float num1, float num2);
  
  int main()
  {
      int num1, num2;
      char op;
      float (*p_Calculate)(float, float);
  
      printf("Enter num1 num2:");
      scanf_s("%d %c %d", &num1, &op, 1, &num2);
  
      switch (op)
      {
          case '+': p_Calculate = func_AddNum;
              break;
          case '-': p_Calculate = func_SubNum;
              break;
          case '*': p_Calculate = func_MulNum;
              break;
          case '/': p_Calculate = func_DivNum;
              break;
      }
      printf("CalculateResult Num = %.2f", (*p_Calculate)(num1, num2));
      getchar();
  
      return 0;
  }
  
  float func_AddNum(float num1, float num2)
  {
      return num1 + num2;
  }
  
  float func_SubNum(float num1, float num2)
  {
      return num1 - num2;
  }
  
  float func_MulNum(float num1, float num2)
  {
      return num1 * num2;
  }
  
  float func_DivNum(float num1, float num2)
  {
      return num1 / num2;
  }
  ```

  



### 11. 结构体



##### 11.1 结构体与变量



###### 11.1.1 结构体类型的定义

- 是具有不同类型的有序集合

- 结构体所占空间等于各个数据成员的所需空间之和

- ```c
  struct 结构体名
  {
      类型表示符 成员名1;
      类型表示符 成员名2;
      类型表示符 成员名3;
  }
  
  struct 定义结构体类型的关键字
  域名或成员名：命名规则和变量相同，同一结构体的同层成员不可同名
  ```



###### 11.1.2 结构体变量

1. 先定义结构体类型，再定义变量名

   ```c
   struct s_Student
   {
       int age;
   }
   
   struct s_Student stu1, stu2;
   ```

   

2. 定义结构体类型和定义变量名写在一起

   ```c
   struct s_Student
   {
       int age;
   }stu1, stu2;
   ```

   

3. 若结构体类型名只用一次，则定义可省去

   ```c
   struct
   {
       int age;
   }stu1, stu2;
   ```

   

###### 11.1.3 结构体类型嵌套

- 定义：结构体成员可以是其他结构体类型，但不能在结构体中嵌套其他结构体

- ```c
  struct s_date
  {
      int year, month, day;
  }
  
  struct s_student
  {
      char name[20];
      struct birthday;
  }
  
  struct s_student stu1, stu2;
  ```

  

###### 11.1.4 typedef 定义类型

- 把一种数据类型名定义成与之等价的另一种类型名

  ```c
  typedef int fh_int;
  fh_int x, y; # == int x, y;
  
  typedef int * fh_pint;
  fh_pint x, y; # == *x, *y;
  ```

- 利用typedef可以将struct student 定义成一个较为简单的类型名

  ```c
  typedef struct student fh_stu;
  fh_stu stu1, stu2; # == struct student stu1, stu2;
  ```

  



###### 11.1.5 结构体的引用

- 结构体是聚合类型的数据，由多种不同的类型的数据成员组成

- 结构体变量的输入输出，只能对其各个成员分别进行

- ```c
  结构体成员的表达式：结构体变量名.成员名
  "." 是成员（分量）运算符，具有最高优先级
  注：同类型的结构体变量之间，可以整体相互赋值
  ```

- ```c
  #include <stdio.h>
  
  struct s_student
  {
      char stu_name[20];
      int stu_age;
      float stu_percent;
  };
  
  int main()
  {
      struct s_student stu;
  
      printf("Enter name:");
      gets_s(stu.stu_name, 20);
      printf("Enter age:");
      scanf_s("%d", &stu.stu_age);
      printf("Enter percent:");
      scanf_s("%f", &stu.stu_percent);
  
      printf("Student:\n");
      printf("\t %s\n \t %d\n \t %.2f\n", stu.stu_name, stu.stu_age, stu.stu_percent);
  
      return 0;
  }
  ```

  



###### 11.1.6 结构体定义时初始化

```c
#include <stdio.h>

struct s1_student
{
    char stu_name[20];
    int stu_age;
    float stu_percent;
};

int main()
{
    struct s1_student stu1 = {"fang", 22, 99}, stu2;
    stu2 = stu1;

    printf("Student01:\n");
    printf("\t %s\n \t %d\n \t %.2f\n", stu1.stu_name, stu1.stu_age, stu1.stu_percent);
    printf("Student02:\n");
    printf("\t %s\n \t %d\n \t %.2f\n", stu2.stu_name, stu2.stu_age, stu2.stu_percent);

    return 0;
}
```





##### 11.2 结构体与数组



###### 11.2.1 结构体数组的定义

```c
struct s_student
{
    char stu_name[20];
    int stu_age;
    float stu_percent;
}stu[2];
```



###### 11.2.2 结构体数组的初始化

```c
struct s_student
{
    char stu_name[20];
    int stu_age;
    float stu_percent;
}stu[2] = 
{
    {"fang", 22, 99},
    {"fang", 22, 99},
    {"fang", 22, 99}
};
```





##### 11.3 结构体与指针



###### 11.3.1 指向结构体变量的指针变量

- 指针变量保存结构体变量地址

  ```c
  struct s_student
  {
      char stu_name[20];
      int stu_age;
      float stu_percent;
  }stu, *p_stu;
  
  p_stu = &stu;
  
  stu.stu_age = 20; # == (*p_stu).stu_age = 20;
  
  # (*p_stu).stu_age 不可省去（）
  # (*p_stu).stu_age 书写麻烦，所以C语言提供了 “指向运算符” ：“->”, 优先级1级，左结合性；
  # 格式：指针变量->成员分量名称
  # (*p_stu).stu_age == p_stu->stu_age
  ```

  ```c
  #include <stdio.h>
  
  struct s2_student
  {
      char stu_name[20];
      int stu_age;
      char sex;
      float stu_percent;
  };
  
  int main()
  {
      struct s2_student stu1 = {"fang", 22, 'T', 99}, *p_stu2;
      p_stu2 = &stu1;
  
      printf("Student01:\n");
      printf("\t %s\n, \t %d\n, \t %s\n, \t %.2f\n",
              stu1.stu_name, stu1.stu_age, (stu1.sex == 'T') ? "man" : "women", stu1.stu_percent);
  
      printf("Student02:\n");
      printf("\t %s\n, \t %d\n, \t %s\n, \t %.2f\n",
              p_stu2->stu_name, p_stu2->stu_age, (p_stu2->sex == 'T') ? "man" : "women", p_stu2->stu_percent);
  
      return 0;
  }
  ```

  



###### 11.3.2 指向结构体数组的指针变量

```c
#include <stdio.h>

struct s3_student
{
    char stu_name[20];
    int stu_age;
    char sex;
    float stu_grade;
};

int main()
{
    struct s3_student stu[3] = {
            {"fang01", 22, 'T', 97},
            {"fang02", 23, 'F', 98},
            {"fang03", 24, 'T', 99}
    };
    struct s3_student *p_stu;

    printf("%6s %5s %5s %7s\n", "Name", "Age", "Sex", "Grade");
    printf("----------------------------\n");

    for (p_stu = stu; p_stu < stu + 3; p_stu++)
    {
        printf("%7s %4d %4c %8.2f\n", p_stu->stu_name, p_stu->stu_age, p_stu->sex, p_stu->stu_grade);
    }

    return 0;
}
```





##### 11.4 联合体(共用体)

```c
union xyz
{
	char x;
	int y;
	double z;
} t;

# t 的内存大小，以union中最大的为标准，也就是double，8个字节
# 由于所有成员共用一个空间，在某一时刻，t变量只有其中一个分量的有效值
```



##### 11.5 枚举类型

- 一种简单的数据类型，把所有的值列举出来，形成该值的取值范围，枚举型变量只能从中取出值，称为枚举元素或枚举常量

- 枚举类型是一种基本数据类型，不是构造体类型，因此不能 分解成任何基本类型

- ```c
  enum 枚举名 {枚举值表};
  
  enum weekday {sun, mou, tue, web, thu, fri, sat};
  enum weekday a, b, c; # a, b, c 是枚举类型weekday的变量
  
  # 也可以定义枚举类型时定义变量
  enum weekday {sun, mou, tue, web, thu, fri, sat} a, b, c;
  # 或
  enum {sun, mou, tue, web, thu, fri, sat} a, b, c;
  ```



###### 11.5.1 枚举类型的使用规定

1. 枚举值是常量，不是变量，不能被赋值
2. 枚举元素本身由系统定义了一个表示序号的值，序号值可以自定
3. 只能把枚举值赋值给枚举变量，不能把元素的序号数值赋值给枚举变量
4. 输出枚举值的格式字符是 “ %d ” ，对应序号，为整数





### 12. 文件



##### 12.1 文件概念和分类



###### 12.1.1 文件概念

- ![](C.assets\文件概念.png)
- <img src="C.assets\文件概念1.png" style="zoom:80%;" />





###### 12.1.2 文件简单分类

- ![](C.assets\文件分类.png)
- ![](C.assets\文件分类1.png)





##### 12.2 文件基本操作



###### 12.2.1 定义文件指针

- ![](C.assets\文件操作.png)
- ![](C.assets\文件操作1.png)





###### 12.2.2 文件的打开

- ![](C.assets\文件操作2.png)

| 打开方式 |  含义  |                           说明                           |
| :------: | :----: | :------------------------------------------------------: |
|    r     |  只读  |              为输入打开一个已存在的文本文件              |
|    w     |  只写  |                  为输出打开一个文本文件                  |
|    a     | 只追加 |              为追加打开一个已存在的文本文件              |
|    rb    |  只读  |             为输入打开一个已存在的二进制文件             |
|    wb    |  只写  |                 为输出打开一个二进制文件                 |
|    ab    | 只追加 |                 为输入打开一个二进制文件                 |
|    r+    |  读写  |            为既读又写打开一个已存在的文本文件            |
|    w+    |  读写  |                为既读又写新建一个文本文件                |
|    a+    |  读写  | 为既读又写打开一个已存在的文本文件，文件指针移至为念末尾 |
|   rb+    |  读写  |           为既读又写打开一个已存在的二进制文件           |
|   wb+    |  读写  |               为既读又写新建一个二进制文件               |
|   ab+    |  读写  |            为读/写打开一个二进制文件进行追加             |





###### 12.2.3 文件的操作

<img src="C.assets\文件操作3.png" style="zoom:80%;" />

<img src="C.assets\文件操作4.png" style="zoom:60%;" />





###### 12.2.4 文件的关闭

<img src="C.assets\文件操作5.png" style="zoom:80%;" />





###### 12.2.5 文件操作示例

- 在C盘目录下新建一个 " fh.txt " 的文件，用来保存键盘输入的字符，直到按回车键为止

  ```c
  字符输出函数（fputc）
  fputc函数使用格式：fputc(ch, fp_fh);
  将 ch 的值输出到 fp_fh 所指向的文件，并返回该字符，输出失败返回EOF
  ```

- ```c
  #include <stdio.h>
  #include <stdlib.h>
  
  int main()
  {
      FILE *fp_fh;
      errno_t err_fh; # 新版C语言的要求，老版本的省去
      char str_fh;
  
      err_fh = fopen_s(&fp_fh,"C:\\Users\\FHang\\Desktop\\fh.txt", "w"); # 老版本省去
  
      if (err_fh != 0) # 老版本改为 if ((fp_fh = fopen("C:\\Users\\FHang\\Desktop\\fh.txt", "w")) == NULL) 
      {
          printf("Can not find this file\n");
          exit(0);
      }
  
      printf("Please input a string. Press enter key to end\n");
  
      while ((str_fh = getchar()) != '\n')
      {
          fputc(str_fh, fp_fh);
      }
  
      fclose(fp_fh);
      printf("Create file fh.txt success.");
  
      return 0;
  }
  ```






### 13. 链表



##### 13.1 链表的概念



###### 13.1.1 概念

链表的每个节点都有两个部分：

- 数据区和指针区。前者用来存储数据，后者用来存储指向下一个节点的指针；
- 我们使用 malloc() 函数来为每个节点分配内存；
- 节点的头部只含有指向第一个节点的指针;
- 存储在栈区的 head 指向了存储在堆区的节点;
- 堆区的节点又互相连接，形成链状的结构;
- 最后一个节点的指针区被赋值为 NULL，标明了链表的结束。



###### 13.1.2 数组相关

- 数组的缺点：
  1. 正常情况下，创建的数组为固定大小的静态数组，当数组的数据内容出现增删，需要手动修改大小；
  2. 如果一次性创建足够大的数组，会浪费内存空间；
  3. 动态数组的创建（不会），麻烦；
  4. 数组的内存空间是连续的，当需要添加或删除某一个元素时，其新元素之后的所有数组元素，需要逐一遍历更换内存地址，性能消耗大；
- 数组的优点：
  1. 数组的内存空间是连续的，直接通过数组小标查找数组元素，很高效；



###### 13.1.3 链表相关

- 链表优点：
  1. 链表的内存空间是非连续的，它们通过链表中的指针域，存储了下一个的链表的首地址，以此达到数据表相连；
  2. 插入或删除指定链表时，只需要将新链表，以及前后链表的首地址和指针域进行相关改动即可，不会影响到所有的链表；
  3. 只需要找到第一个链表节点，就能顺着指针域找到所有的链表；
- 链表缺点：
  1. 查找指定链表节点，需要从第一个节点顺着指针域一个个找下去，消耗性能大，远不及数组直接通过下标定位；
  2. 当在整个链表的第一节点前插入新节点，原来找到的链表首节点，就不再是首节点，需要重新找；
  3. 链表的每个节点都需要一块内存空间存储下一个节点的首地址；



###### 13.1.4 链表分类

1. 内存类型区分：
   - 静态链表
   - 动态链表：使用malloc()
2. 功能结构区分：
   - 单向链表
   - 双向链表
   - 循环链表
   - 单向循环链表
   - 双向循环链表



##### 13.2 静态链表



###### 13.2.1 静态链表的结构

```c
struct Node
{
    int data; //数据域
    struct Node* next; //指针域，用于存储下一个节点的首地址
};

int main()
{
    // 初始化创建静态链表节点
    struct Node Node1 = {1, NULL};
    struct Node Node2 = {2, NULL};
    struct Node Node3 = {3, NULL};

    // 将Node2的首地址存入Node1的next指针地址变量内
    Node1.next = &Node2;
    Node2.next = &Node3;
    
    return 0;
}
```



###### 13.2.2 遍历打印静态链表

```c
#include <stdio.h>
#include <stdlib.h>

struct Node
{
    int data; //数据域
    struct Node* next; //指针域，用于存储下一个节点的首地址
};

void printList()
{
    struct Node node1 = {1, NULL};
    struct Node node2 = {2, NULL};
    struct Node node3 = {3, NULL};

    // 将Node2的首地址存入Node1的next指针地址变量内
    node1.next = &node2;
    node2.next = &node3;

    // 遍历链表
    // 先定义一个辅助指针变量
    struct Node *pCurrent = &node1;

    while (pCurrent != NULL)
    {
        // 将这个指针指向的内存空间中的节点元素打印
        printf("%d\n", pCurrent->data);
        // 打印后，将节点的指针域内存放的下一个节点首地址，赋值给pCurrent，从而找到下一个链表节点
        pCurrent = pCurrent->next;
    }
}

int main()
{
    printList();
    system("pause");

    return 0;
}
```



###### 13.3 动态链表(模块化)

```c
//
// Created by FHang on 2021/3/29.
//

/*
 * 动态创建链表
 */

#include <stdio.h>
#include <stdlib.h>

struct Node
{
    int data; //数据域
    struct Node *next; //指针域
};

// 创建链表
struct Node *createList()
{
    // headNode成为结构体变量
    struct Node *headNode = (struct Node*)malloc(sizeof(struct Node));
    // 变量使用前，需要初始化
    // headNode->data = 1;
    headNode->next = NULL;

    return headNode;
}

// 创建节点
struct Node *createNode(int data)
{
    struct Node *newNode = (struct Node*)malloc(sizeof(struct Node));
    newNode->data = data;
    newNode->next = NULL;

    return newNode;
}

// 插入节点(表头法插入)
void insertNodeByHead(struct Node *headNode, int data)
{
    // 创建新的插入节点
    struct Node *newNode = createNode(data);
    newNode->next = headNode->next;
    headNode->next = newNode;
}

// 删除指定链表
void deleteNode(struct Node *headNode, int posData)
{
    struct Node *posNode = headNode->next;
    struct Node *posNodeFront = headNode;

    if (posNode == NULL)
    {
        printf("Can't Delete Null Linklist !\n");
    }
    else
    {
        while (posNode->data != posData)
        {
            posNodeFront = posNode;
            posNode = posNodeFront->next;

            if (posNode == NULL)
            {
                printf("Can't Find This Linklist !\n");
                return;
            }
        }

        posNodeFront->next = posNode->next;
        free(posNode);
    }
}

// 打印
void printList(struct Node *headNode)
{
    struct Node *pMove = headNode->next;
    while (pMove != NULL)
    {
        printf("Data=%d\n", pMove->data);
        pMove = pMove->next;
    }

    printf("\n");
}

int main()
{
    struct Node *list = createList();
    insertNodeByHead(list, 1);
    insertNodeByHead(list, 2);
    insertNodeByHead(list, 3);
    printList(list);

    deleteNode(list, 2);
    printList(list);

    system("pause");

    return 0;
}
```





### 例子

#### 1.  求百十个位的和

```c
#include <stdio.h>

#define Debug printf
#define Enter scanf_s

int main()
{
    int enterNum, sumNum;
    int b, s, g;

    Debug("Enter Num : ");
    Enter("%d", &enterNum);

    b = enterNum / 100;
    s = enterNum / 10 % 10;
    g = enterNum % 10;
    sumNum = b + s + g;

    Debug("Sum=%d", sumNum);
}
```



#### 2. 求200-300之间素数和

```c
#include <stdio.h>

int main()
{
    int n, i;
    int sum = 0;

    for (n = 200; n < 300; ++n)
    {
        for (i = 2; i < n / 2; ++i)
        {
            if (n % i == 0)
            {
                break;
            }
        }
        if (i >= n / 2)
        {
            sum += n;
            printf("%4d", n);
        }
    }
    printf("\n sum = %d", sum);

    return 0;
}
```



#### 3. 斐波那契

```c
#include <stdio.h>

int main()
{
    int f1 = 1, f2 = 1, f3;

    printf("1=%d\n2=%d\n", f1, f2);

    for (int k = 3; k <= 20; ++k)
    {
        f3 = f1 + f2;
        printf("%d=%d\n", k, f3);
        f1 = f2;
        f2 = f3;
    }

    return 0;
}
```



#### 4. 递增算法

![](D:\TyporaNote\专升本\C.assets\例题4.png)

```c
#include <stdio.h>

int main()
{
    int sum = 1;

    for (int i = 9; i >= 1; i--)
    {
        sum = (sum + 1) * 2;
        if (i == 1)
        {
            printf("1 day has %d", sum);
            break;
        }
    }

    return 0;
}
```





#### 5. 100-999之间水仙花数

```c
#include <stdio.h>

int main()
{
    int sum, bw, sw, gw;

    for (int i = 100; i <= 999; ++i)
    {
        bw = i / 100;
        sw = i / 10 % 10;
        gw = i % 10;
        sum = bw * bw * bw + sw * sw * sw + gw * gw * gw;

        if (sum == i)
        {
            printf("%d\t", sum);
        }
    }

    return 0;
}
```





#### 6. 求 1!+2!+3!+...+10!

```c
#include <stdio.h>

int main()
{
    long sum = 0, fact = 1;

    for (int n = 1; n <= 10 ; ++n)
    {
        fact *= n;
        sum += fact;
    }
    printf("sum = %ld", sum);

    return 0;
}
```





#### 7. 求多项式的值

![](C.assets\例题7.png)

```c
#include <stdio.h>

int main()
{
    float x = 0;
    float sum = 1, term = 1;

    printf("Enter X Num : ");
    scanf_s("%f", &x);

    for (int n = 1; n <= 20 ; ++n)
    {
        term = (-term * x) / x;
        sum += term;
    }

    printf("Calculate Num is %.2f\n", sum);

    return 0;
}
```





#### 8. 将整形一维数组的内容颠倒顺序

```c
#include <stdio.h>

int main()
{
    int sum = 0;
    int arr[10] = {11, 22, 33, 44, 55, 66, 77, 88, 99, 100};

    for (int i = 0; i < 10; ++i)
    {
        printf("%4d", arr[i]);
    }

    for (int i = 0, j = 9; i < j; ++i, --j)
    {
        sum = arr[i];
        arr[i] = arr[j];
        arr[j] = sum;
    }

    printf("\n");

    for (int k = 0; k < 10; ++k)
    {
        printf("%4d", arr[k]);
    }

    return 0;
}
```





#### 9. 将3行4列的数组值转置到4行3列

```c
#include <stdio.h>

int main()
{
    int arrA[3][4] = {{1, 2,  3,  4},
                     {5, 6,  7,  8},
                     {9, 10, 11, 12}};

    static int arrB[4][3];

    printf("Before Transpose:\n");
    for (int i = 0; i < 3; ++i)
    {
        for (int j = 0; j < 4; ++j)
        {
            printf("%4d", arrA[i][j]);
            arrB[j][i] = arrA[i][j];
        }
        printf("\n");
    }
    
    printf("Later Transpose:\n");
    for (int i = 0; i < 4; ++i)
    {
        for (int j = 0; j < 3; ++j)
        {
            printf("%4d", arrB[i][j]);
        }
        printf("\n");
    }

    return 0;
}
```





#### 10. 将字符串1内容赋值到字符串2中

```c
#include <stdio.h>

int main()
{
    char s2[50];

    char s1[50] = "abcdefghjklmn";

    for (int i = 0; (s2[i] = s1[i]); ++i);

    puts(s2);

    return 0;
}
```





#### 11. 打印杨辉三角前10行

```c
#include <stdio.h>

int main()
{
    int arr[10][10];
    for (int i = 0; i < 10; ++i)
    {
        for (int j = 0; j <= i; ++j)
        {
            if (j==0||i==j)
            {
                arr[i][j] = 1;
            }
            else
            {
                arr[i][j] = arr[i-1][j-1] + arr[i-1][j];
            }
        }
    }

    for (int i = 0; i < 10; ++i)
    {
        for (int j = 0; j <= i; ++j)
        {
            printf("%5d", arr[i][j]);
        }
        printf("\n");
    }

    return 0;
}
```





#### 12. 判断字符串或数字是否为回文

```c
#include <stdio.h>
#include <string>

int main()
{
    char enterStr[45];
    int i, j;

    gets(enterStr);

    for (i = 0, j = strlen(enterStr) - 1; i < j && (enterStr[i] == enterStr[j]); ++i, --j);

    if (i < j)
    {
        printf("No");
    }
    else
    {
        printf("Yes");
    }

    return 0;
}
```

```c
#include <stdio.h>

int main()
{
    int x, y = 0, z;

    printf("Enter:");
    scanf_s("%d", &z);
    x = z;

    while (x != 0)
    {
        y = (10*y) + (x%10);
        x = x/10;
//        printf("x=%d\n", x);
//        printf("y=%d\n", y);
    }

    if (z == y)
    {
        printf("Yes");
    }
    else
    {
        printf("No");
    }

    return 0;
}
```



#### 13. 用递归算法计算n！

```c
#include <stdio.h>

double funcFact(int n);

int main()
{
    int n = 0;
    double factNum;
    printf("Enter Num :");
    scanf_s("%d", &n);
    factNum = funcFact(n);
    printf("Fact = %.0lf", factNum);

    return 0;
}

double funcFact(int n)
{
    double fact;
    if (n == 1)
    {
        fact = 1;
    }
    else
    {
        fact = n * funcFact(n - 1);
    }

    return fact;
    // 三目运算来表达
    // return n == 1 ? 1 : n * funcFact(n - 1);
}
```



#### 14. 用递归计算1+2+...+n

```c
#include <stdio.h>

int funcSum(int n);

int main()
{
    int enterNum;
    printf("Enter Num :");
    scanf_s("%d", &enterNum);
    printf("Sum = %d", funcSum(enterNum));

    return 0;
}

int funcSum(int n)
{
    return n == 1 ? 1 : n + funcSum(n - 1);
}
```



#### 15. 用循环和递归计算任意整数个位数之和

- 循环

```c
#include <stdio.h>

int main()
{
    int n;
    int sum = 0;

    printf("Enter Num :");
    scanf_s("%d", &n);
    while (n)
    {
        sum += n % 10;
        n = n / 10;
    }
    printf("Sum = %d", sum);

    return 0;
}
```

- 递归

```c
#include <stdio.h>

int funcSum(int n);

int main()
{
    int n;
    int sum;

    printf("Enter Num :");
    scanf_s("%d", &n);
    sum = funcSum(n);
    printf("Sum = %d", sum);

    return 0;
}

int funcSum(int n)
{
    return n < 10 ? n : (n % 10) + funcSum(n / 10);
}
```



#### 16. 通过函数将数组内元素加10

```c
#include <stdio.h>

void funcAdd(int *p_arr, int eleCount, int addNum);

int main()
{
    int arr[5] = {1, 2, 3, 4, 5};
    int eleCount;
    eleCount = sizeof(arr) / sizeof(arr[0]);

    funcAdd(arr, eleCount, 10);
    for (int i = 0; i < eleCount; ++i)
    {
        printf("%3d", arr[i]);
    }

    return 0;
}

void funcAdd(int *p_arr, int eleCount, int addNum)
{
    for (int i = 0; i < eleCount; ++i)
    {
        p_arr[i] += addNum;
    }
}
```



#### 17. 指针变量为形参，求数组元素和平均值

```c
#include <stdio.h>

float funcCalculateAverage(const int *p_arr, int eleCount);

int main()
{
    int arr[6] = {10, 20, 30, 40, 50, 11};
    int eleCount;
    eleCount = sizeof(arr) / sizeof(arr[0]);

    printf("arr[%d] average = %.2f", eleCount, funcCalculateAverage(arr, eleCount));

    return 0;
}

float funcCalculateAverage(const int *p_arr, int eleCount)
{
    int sum = 0;
    float average;
    for (int i = 0; i < eleCount; ++i)
    {
        sum += p_arr[i];
    }
    average = (float)sum / (float)eleCount;

    return average;
}
```



#### 18. 数组元素指针访问数组

```c
#include <stdio.h>

int main()
{
    int *p;
    int arr[3][4] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12};

# 两种写法
//    for (p = arr[0]; p < arr[0] + 12; p++)
//    {
//        if ((p - arr[0]) % 4 == 0)
//        {
//            printf("\n");
//        }
//        printf("%3d", *p);
//    }

    for (p = *(arr + 0); p < *(arr + 0) + 12; p++)
    {
        if ((p - *(arr + 0)) % 4 == 0)
        {
            printf("\n");
        }
        printf("%3d", *p);
    }

    return 0;
}
```



#### 19. 一维数组访问二维数组

```c
#include <stdio.h>

int main()
{
    int (*p)[4];
    int arr[3][4] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12};
    p = arr;

    for (int i = 0; i < 3; ++i)
    {
        for (int j = 0; j < 4; ++j)
        {
            printf("%3d", p[i][j]);
//            printf("%3d", *(*(p + i) + j));
//            printf("%3d", *(*(arr + i) + j));
        }
        printf("\n");
    }

    return 0;
}
```

```c
#include <stdio.h>

int main()
{
    int (*p)[4], arr[3][4] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12};
    for (p = arr; p < arr + 3; p++)
    {
        for (int i = 0; i < 4; ++i)
        {
            printf("%3d", *(*p + i));
//            printf("%3d", *(*(arr + x) + i)); -> x 的取值为 0， 1， 2
        }
        printf("\n");
    }

    return 0;
}
```



#### 20. 指针数组访问另一个一维数组

```c
#include <stdio.h>

int main()
{
    int *p_Arr[5]; # == p_Arr0, p_Arr1, ... p_Arr4
    int tag_Arr[5] = {1, 2, 3, 4, 5};

    for (int i = 0; i < 5; ++i)
    {
        p_Arr[i] = tag_Arr + i;
    }

    for (int j = 0; j < 5; ++j)
    {
        printf("%3d", *p_Arr[j]);
    }
    printf("\n");

    return 0;
}
```



#### 21. 指针数组的应用

```c
#include <stdio.h>

int main()
{
    char *p_StrArr[2] = {"Hello", "World"};

    for (int i = 0; i < 2; ++i)
    {
        printf("%s\n", p_StrArr[i]);
    }

    return 0;
}
```


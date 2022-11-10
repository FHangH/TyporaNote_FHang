# Dart



[toc]



### 1. Dart 安装

- 配套Flutter使用，安装Flutter SDK 1.21之后的版本，默认包含了最新的Dart SDK，不需要再额外安装Dart SDK

- 运行命令

  ```powershell
  PS C:\Windows\System32> dart --version
  Dart SDK version: 2.12.3 (stable) (Wed Apr 14 11:02:39 2021 +0200) on "windows_x64"
  ```

  



### 2. Dart 环境配置

1. 安装完Flutter之后，打开VSCode
2. 在VSCode的扩展项里，搜索 Dart 和 Code Runner 安装即可





### 3. Dart 项目文件

1. 打开VSCode，Ctrl + N，新建文件

2. Ctrl + S，另存为，修改文件后缀名 .dart，即可 (demo.dart)

   ```dart
   void main() {
     print("Hello World !");
   }
   ```

3. Ctrl + Alt + N，运行即可看到打印结果

   或者进入项目内：

   ```powershell
   dart demo1.dart
   
   // 实际效果
   PS C:\Users\Admin\Desktop\dart_demo> dart .\demo1.dart
   Hello World !
   ```





### 5. Dart 基本语法



#### 5.1 Dart 变量类型

```dart
void main() {
  // var 可以自动通过类型声明变量
  var str1 = "Hello World 1";
  var num1 = 1111;
  print(str1);
  print(num1);

  // 常规的指定变量类型
  String str2 = "Hello World 2";
  int num2 = 2222;
  print(str2);
  print(num2);
}
```





#### 5.2 Dart 常量

- const 修饰，需要赋初值

- final 修饰，可不赋初值，只能赋值一次

- 永远不改变的值，需要用 const 或 final 修饰，不可用 var 或 其他类型

  ```dart
  void main() {
    var str = "Hello World 01";
    str = "Hello World 02";
    var num = 1111;  
    num = 2222;
      
    print(str);
    print(num);
  
    const str1 = "Fang";
    const num1 = 2222;
    // const 修饰后，再次赋值无效
    str1 = "Hang";
    num1 = 1234;
  
    print(str1);
    print(num1);
  }
  
  #打印效果：显示错误#
  PS C:\Users\Admin> dart "c:\Users\Admin\Desktop\dart_demo\demo1.dart"
  Desktop/dart_demo/demo1.dart:13:3: Error: Can't assign to the const variable 'str1'.
    str1 = "Hang";
    ^^^^
  Desktop/dart_demo/demo1.dart:14:3: Error: Can't assign to the const variable 'num1'.
    num1 = 1234;
    ^^^^
  ```

- final 和 const 的区别：（final只有在在运行时才会赋值一次，而const必须提前赋值）

  ```dart
  void main() {
    final f_timeNow = new DateTime.now();
    print(f_timeNow);
  
    // 错误的用法
    const c_timeNow = new DateTime.now();
    print(c_timeNow);
  }
  
  #打印效果：显示错误#
  PS C:\Users\Admin> dart "c:\Users\Admin\Desktop\dart_demo\demo1.dart"
  Desktop/dart_demo/demo1.dart:5:21: Error: New expression is not a constant expression.
    const c_timeNow = new DateTime.now();
                      ^^^
  ```






#### 5.3 Dart 数据类型

- 常用的数据类型：
  1. Number（数值）
     - int
     - double
  2. String（字符串）
     - String
  3. Boolen（布尔）
     - bool
  4. List（数组）
     - 在Dart中，数组是列表的对象，大部分情况下称之为列表
  5. Map（字典）
     - Map是一个（键，值）对的对象，（键，值）可以是任何类型的对象
- 不常用的数据类型：
  1. Rune：Dart中UTF-32编码字符串，可以通过文字转换成符号表情或特定的字
  2. Symbol：Dart程序中声明的运算符和标识符





#### 5.4 Dart 字符串



##### 5.4.1 Dart 字符串定义

- 不换行（示例）：单引号 // 双引号

  ```dart
  void main() {
    var str1 = 'test string 01';
    String str2 = "test string 02";
  
    print(str1);
    print(str2);
  }
  ```

- 不换行（打印）：

  ```powershell
  PS C:\Users\Admin\Desktop\dart_demo> dart .\demo01.dart
  test string 01
  test string 02
  ```

- 换行（示例）：三个单引号 // 三对双引号

  ```dart
  void main() {
    String str1 = '''
    Test String 01
    Test String 02
    ''';
  
    String str2 = """
    Test String 03
    Test String 04
    """;
  
    print(str1);
    print(str2);
  }
  ```

- 换行（打印）：

  ```powershell
  PS C:\Users\Admin\Desktop\dart_demo> dart .\demo01.dart
    Test String 01
    Test String 02
  
    Test String 03
    Test String 04
  ```



##### 5.4.2 Dart 字符串拼接

- 将两个字符串拼接

  ```dart
  '$str1 $str2'
  "$str1 $str2"
   str1 + str2
  ```

  ```dart
  void main() {
    String str1 = 'Hello';
    String str2 = 'Wrold';
  
    print(str1 + ' ' + str2);
    print('$str1 $str2');
  }
  ```

- 打印结果：

  ```powershell
  PS C:\Users\Admin\Desktop\dart_demo> dart .\demo01.dart
  Hello Wrold
  Hello Wrold
  ```





#### 5.5 Dart 数值计算

```dart
void main() {
  int a = 12;
  double b = 24.0;
  double c = 48;

  print(a);
  print(b);
  print(c);

  print('-------');

  var d = a + c;
  var e = c - a;
  var f = a * b;
  var g = c / a;
  var h = b % a;

  print(d);
  print(e);
  print(f);
  print(g);
  print(h);
}
```

```powershell
PS C:\Users\Admin\Desktop\dart_demo> dart .\demo01.dart
12
24.0   
48.0   
-------
60.0   
36.0
288.0
4.0
0.0
```





#### 5.6 Dart 布尔类型

```dart
void main() {
  int a = 12;
  int b = 12;
  if (a == b) {
    print('a = b');
  } else {
    print('a != b');
  }

  bool c = true;
  bool d = false;
  if (c == d) {
    print('c = d');
  } else {
    print('c != d');
  }
}
```

```powershell
PS C:\Users\Admin\Desktop\dart_demo> dart .\demo01.dart
a = b
c != d
```





#### 5.7 List 集合类型

- 不指定类型

  ```dart
  void main() {
    var list1 = ["FHang", 24, true];
    print("list1:");
    print(list1);
    print("--------");
  
    print("list frist:");
    print(list1.first);
    print(list1[0]);
    print("--------");
  
    print("list second:");
    print(list1[1]);
    print("--------");
  
    print("list last:");
    print(list1.last);
    print(list1[2]);
    print("--------");
  
    print("list length:");
    print(list1.length);
  }
  ```

  ```powershell
  PS C:\Users\Admin\Desktop\dart_demo> dart .\demo01.dart
  list1:
  [FHang, 24, true]
  --------
  list frist:      
  FHang
  FHang
  --------
  list second:
  24
  --------
  list last:
  true
  true
  --------
  list length:
  3
  ```

- 指定类型

  ```dart
  void main() {
    var list1 = <String>["Fang", "Hang"];
    print(list1.first);
    print(list1.last);
  }
  ```

  ```powershell
  PS C:\Users\Admin\Desktop\dart_demo> dart .\demo01.dart
  Fang
  Hang
  ```

- 添加/删除数据到集合

  ```dart
  void main() {
    var list1 = [];
    print(list1.length);
    print("--------");
  
    //添加
    list1.add("Fang");
    list1.add("Hang");
    list1.add(24);
    print(list1);
    print(list1.length);
    print("--------");
  
    //删除
    list1.removeAt(1);
    print(list1);
    print(list1.length);
  }
  ```

  ```powershell
  PS C:\Users\Admin\Desktop\dart_demo> dart .\demo01.dart
  0
  --------        
  [Fang, Hang, 24]
  3
  --------        
  [Fang, 24]
  2
  ```

- 通过List.filled 来创建固定大小的集合

  ```dart
  void main() {
    // 不指定类型
    var list1 = List.filled(2, "");
    // 指定类型
    // var list2 = List<String>.filled(2, "");
    print(list1.length);
    print(list1);
  
    list1[0] = "Fang";
    list1[1] = "hang";
    print(list1);
  }
  ```

  ```powershell
  PS C:\Users\Admin\Desktop\dart_demo> dart .\demo01.dart
  2
  [, ]        
  [Fang, hang]
  ```

  




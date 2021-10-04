# C++模板和STL



- 记录C++泛型编程和STL的使用和原理



### 1. 模板-Template



#### 1.1 模板概念



- 作用：建立通用的模具，提高编程的复用性
- 特点：
  - 模板在实际项目中不可直接使用，它只是一个框架，需根据实际情况进行使用
  - 模板的通用不是万能的



#### 1.2 函数模板



- C++提供了另一种编程思想，`泛型编程`，主要利用的技术就是模板`template`
- C++提供了两种模板机制：`函数模板`和`类模板`



##### 1.2.1 函数模板语法



1. 函数模板的作用：

   - 建立一个通用函数，其函数返回值类型和形参类型可以不确定，是一个`虚拟的类型`

   - 当需要使用时，再根据实际情况进行声明或由编译器自行判断数据类型



2. 语法：

   ```c++
   template <typename T>
   函数的声明或定义
   ```



3. 解释：
   - `template`：声明创建模板
   - `typename`：表面后面的符号，代表着是一种数据类型，但不确定是哪种，也可以用`class`来代替
   - `T`：通用数据类型，是一个符号，可以换成别的代替，通常大写，并习惯写成`T`



4. 代码示例：

   ```c++
   //
   // Created by FHang on 2021/7/9 13:35
   //
   #include <iostream>
   
   using namespace std;
   
   // 函数模板
   // 声明一个模板，通用数据符号 T， 防止编译器报错
   template <typename T>
   void swap_T(T &num1, T &num2)
   {
       T tempNum = num1;
       num1 = num2;
       num2 = tempNum;
   }
   
   void demo1()
   {
       int num1 = 10;
       int num2 = 20;
       // 自动类型推导
       swap_T(num1, num2);
       cout << "自动类型: " ;
       cout << "num1: " << num1 << " -" << " num2: " << num2 << endl;
   
       // 指定类型
       swap_T<int>(num1, num2);
       cout << "指定类型: " ;
       cout << "num1: " << num1 << " -" << " num2: " << num2 << endl;
   }
   
   int main()
   {
       demo1();
       return 0;
   }
   ```



5. 总结：
   - 函数模板的创建利用关键字`template`
   - 使用函数模板有两种方式：`自动类型推导`和`显示指定类型`
   - 模板的目的是提高复用性，将类型参数化





##### 1.2.2 函数模板注意事项



注意事项：

- 自动类型推导：通用数据类型 `T` 必须推导出一致的类型，才能使用
- 模板函数在使用时，无论哪种方式，必须确定了使用的数据类型，才能使用



代码示例：

```c++
//
// Created by FHang on 2021/7/9 13:55
//
#include <iostream>

using namespace std;

template <typename T>
void swap_T(T &num1, T &num2)
{
    T tempNum = num1;
    num1 = num2;
    num2 = tempNum;
}

void demo1()
{
    int num1 = 10;
    int num2 = 20;
    char c1 = 'A';
    // 自动类型推导
    // num1 和 num2 都是 int 类型，可以自动推导出是一致类型
    swap_T(num1, num2);

    // c1是char类型，是错误的使用方式
    // swap_T(c1, num2);
    cout << "自动类型: " ;
    cout << "num1: " << num1 << " -" << " num2: " << num2 << endl;
}

int main()
{
    demo1();
    return 0;
}
```





##### 1.2.3 函数模板案例



- 案例描述：

  - 利用函数模板封装一个排序函数，可以对不同类型数据进行排序
  - 排序规则：从大到小，排序算法为`选择排序`
  - 分别利用`char`数组和`int`数组进行测试

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/9 14:01
  //
  #include <iostream>
  #include <cstring>
  
  using namespace std;
  
  template <class T>
  void swap_T(T &a, T &b)
  {
      T temp = a;
      a = b;
      b = temp;
  }
  
  template <class T>
  void sort_T(T arr[], int length)
  {
      for (int i = 0; i < length; ++i)
      {
          // 假设最大值的下标，后面进行比较
          int max = i;
          for (int j = i + 1; j < length; ++j)
          {
              // 将假设的下标对应的值和后面的值，比较
              if (arr[max] < arr[j])
              {
                  max = j;
              }
          }
          // 上面的循环结束后，检查一开始假设的下标，是否是循环检查出的最大值下标
          if (max != i)
          {
              swap_T(arr[max], arr[i]);
          }
      }
  }
  
  template <class T>
  void print_T(T arr, int length)
  {
      cout << "从大到小排序后：";
      for (int i = 0; i < length; ++i)
      {
          cout << arr[i] << " ";
      }
      cout << endl;
  }
  
  void sortCharDemo()
  {
      char c_Array[26];
      int c_Length;
  
      cout << "输入字符(a - z) >> ";
      cin >> c_Array;
      // 实际的字符后面有一个 `\0`收尾,这获取长度方法不好
      //c_Length = (sizeof(c_Array) / sizeof(char));
  
      // strlen() 获取长度，读到`\0`，就不读，适合这个项目的需求
      // 需要包含 <cstring>
      c_Length = strlen(c_Array);
  
      sort_T(c_Array, c_Length);
      print_T(c_Array, c_Length);
  }
  
  void sortIntDemo()
  {
      int i_Array[10];
      int i = 0;
      int i_Length;
  
      cout << "输入数字(0 - 9) >> ";
      while (cin.peek() != '\n')
      {
          cin >> i_Array[i++];
          i_Length = i;
      }
  
      sort_T(i_Array, i_Length);
      print_T(i_Array, i_Length);
  }
  
  int main()
  {
      // sortCharDemo();
      sortIntDemo();
      return 0;
  }
  ```



##### 1.2.4 普通和模板函区别



- 区别：

  1. 普通函数调用时，可以发生自动类型转换 `(隐式类型转换)`
  2. 函数模板调用时，利用自动类型转换，不会发生`(隐式类型转换)`
  3. 如果利用显示指定类型的方式，可以发生`(隐式类型转换)`

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/9 15:54
  //
  #include <iostream>
  
  using namespace std;
  
  int add(int a,  int b)
  {
      return a + b;
  }
  
  template <class T>
  T add_T(T a, T b)
  {
      return a + b;
  }
  
  int main()
  {
      int a = 10;
      // c -> ASCII = 97
      char c = 'a';
  
      // 普通函数 可以隐式类型转换
      cout << add(a, c) << endl;
  
      // 模板函数 自动类型转换 无法隐式类型转换
      // cout << add_T(a, c) << endl;
      // 指定类型转换 可以隐式类型转换
      cout << add_T<int>(a, c) << endl;
  
      return 0;
  }
  ```

- 总结：建议使用`显示指定类型`的方式调用函数模板，可以自己确定通用类型`T`



##### 1.2.5 普通和模板函数调用



- 调用规则：

  1. 如果函数模板和普通函数都能实现，优先调用普通函数
  2. 可以通过空模板参数列表来，强制调用函数模板
  3. 函数模板可以发生重载
  4. 如果函数模板可以产生更好的匹配，优先调用函数模板

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/9 16:18
  //
  #include <iostream>
  
  using namespace std;
  
  void print(int a, int b)
  {
      cout << "普通函数调用" << endl;
  }
  
  template <class T>
  T print(T a, T b)
  {
      cout << "模板函数调用" << endl;
  }
  
  template <class T>
  T print(T a, T b, T c)
  {
      cout << "模板函数重载调用" << endl;
  }
  
  int main()
  {
      // 1. 如果函数模板和普通函数都能实现，优先调用普通函数
      print(1, 2);
  
      // 2. 可以通过空模板参数列表来，强制调用函数模板
      print<>(1, 2);
  
      // 3. 函数模板可以发生重载
      print(1, 2, 3);
  
      // 4. 如果函数模板可以产生更好的匹配，优先调用函数模板
      // 因为 a， b为char，隐式类型转换调用普通函数，不如直接自动类型转换调用模板函数调用来的方便
      print('a', 'b');
  
      return 0;
  }
  ```

- 总结：如果需要使用模板函数，就不用提供声明普通函数，以免产生`二义性`





##### 1.2.6 模板的局限性



- 局限性：模板的通用性，并非万能

- 代码示例_1：如果传入的 `a` 和 `b`是数组，该模板函数无法实现

  ```c++
  template <class T>
  void func(T a, T b)
  {
      a = b;
  }
  ```

- 代码示例_2：传入的是自定义类型数据，同样无法实现

  ```c++
  template <class T>
  void func(T a, T b)
  {
      if (a > b)
      {......}
  }
  ```



- 解决方法：C++为了解决这个问题，提供模板的重载，可以为`特定的类型`提供`具体化的模板`

- 代码示例_3：

  ```c++
  //
  // Created by FHang on 2021/7/9 16:44
  //
  #include <iostream>
  
  using namespace std;
  
  // 自定义的结构体数据类型
  struct Person
  {
      Person(int age, string name)
      {
          this->age = age;
          this->name = name;
      }
      int age;
      string name;
  };
  
  template <class T>
  bool compare_T(T &a, T &b)
  {
      return a == b;
  }
  
  // 具体化模板函数的实现，当传入参数类型为 Person，优先调用这个模板函数
  template<> bool compare_T(Person &p1, Person &p2)
  {
      return p1.age == p2.age && p1.name == p2.name;
  }
  
  void demo1()
  {
      int a = 10;
      int b = 20;
      bool rel = compare_T(a, b);
      string printInfo = rel ? "a = b" : "a != b";
      cout << printInfo << endl;
  }
  
  void demo2()
  {
      Person p1(10, "fh");
      Person p2(10, "fh");
      bool rel = compare_T(p1, p2);
      string printInfo = rel ? "p1 = p2" : "p1 != p2";
      cout << printInfo << endl;
  }
  
  int main()
  {
      demo1();
      demo2();
      return 0;
  }
  ```

- 总结：

  1. 利用具体化模板函数，可以解决自定义类型的通用化
  2. 学习模板并非是为了写模板，而是能够在STL中使用系统提供的模板

 



#### 1.3 类模板



##### 1.3.1 类模板语法



- 类模板作用：建立一个通用类，类的成员数据类型可以不具体声明，用`虚拟类型`代替

- 类模板语法：

  ```c++
  template <class name_T, class age_T>
  class 类名
  {
  public:
      name_T name;
      age_T age;
  };
  ```

- 代码示例：`类`和`结构体`都可以

  ```c++
  //
  // Created by FHang on 2021/7/9 17:17
  //
  #include <iostream>
  
  using namespace std;
  
  // 类和结构体都可以这么用
  template <class name_T, class age_T>
  struct Person
  {
      name_T name;
      age_T age;
  
      Person(name_T name, age_T age)
      {
          this->name = name;
          this->age = age;
      }
      ~Person()
      {
          cout << "Name: " << this->name << " " << "Age: " << this->age << endl;
      }
  };
  
  void demo()
  {
      Person<string, int>("fh", 24);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  



##### 1.3.2 类和模板区别



- 区别：

  1. 类模板没有自动类型推导的使用方式
  2. 类模板在模板参数列表中，可以有默认参数

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/11 7:40
  //
  #include <iostream>
  
  using namespace std;
  
  template <class name_T, class age_T>
  struct Person01
  {
      name_T name;
      age_T age;
  
      Person01(name_T name, age_T age)
      {
          this->name = name;
          this->age = age;
      }
      ~Person01()
      {
          cout << "Name: " << this->name << " " << "Age: " << this->age << endl;
      }
  };
  
  // 用结构体来代替类了
  // 类模板中，可以在参数列表中，指明默认参数类型，生成对象时，不需要再显示指定类型
  template <class name_T = string, class age_T = int>
  struct Person02
  {
      name_T name;
      age_T age;
  
      Person02(name_T name, age_T age)
      {
          this->name = name;
          this->age = age;
      }
      ~Person02()
      {
          cout << "Name: " << this->name << " " << "Age: " << this->age << endl;
      }
  };
  
  void demo_P1()
  {
      // 类模板没有自动类型推导，所以这个写法是错误的
      // Person01<> person01("FH", 24);
  
      // 需要显示指定类型
      Person01<string, int> person01("FH", 24);
  }
  
  void demo_P2()
  {
      Person02<> person02("XX", 24);
  }
  
  int main()
  {
      demo_P1();
      demo_P2();
      return 0;
  }
  ```





##### 1.3.3 类模板中成员函数创建时机



- 类模板中和普通类中的成员函数创建时机存在区别

  1. 普通类中的成员函数一开始就可以创建
  2. 类模板中的成员函数在调用时才会创建

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/12 13:51
  //
  #include <iostream>
  
  using namespace std;
  
  class Person1
  {
  public:
      void showPerson1()
      {
          cout << "Show Person 1" << endl;
      }
  };
  
  class Person2
  {
  public:
      void showPerson2()
      {
          cout << "Show Person 2" << endl;
      }
  };
  
  template <class class_T>
  class Person_T
  {
  public:
      class_T person;
  
      // 传入的类型不确定，所以默认情况下，编译器不会保错
      // 此时，类模板中的函数不会被创建，当正确调用类模板时，才会创建
      void showFunc1()
      {
          person.showPerson1();
      }
      void showFunc2()
      {
          person.showPerson2();
      }
  };
  
  void demo()
  {
      Person_T<Person1> p1;
      p1.showFunc1();
      // 传入的是Person1，所以编译时，编译器创建类模板内的成员函数时，找不到可以调用的showPerson2()
      // p1.showFunc2();
      
      // 传入Person2，才能调用showPerson2()，但同样也找不到showPerson1()
      Person_T<Person2> p2;
      p2.showFunc2();
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 1.3.4 类模板对象做函数参数



- 说明：类模板实例化出对象，向函数传参的方式

- 三种传入方式：

  1. 指定传入类型：直接显示对象的数据类型
  2. 参数模板化：将对象中的参数变为模板进行传递
  3. 整个类模板化：将对象类型模板化进行传递

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/12 14:08
  //
  #include <iostream>
  
  using namespace std;
  
  template <class name_T, class age_T>
  class Person
  {
  public:
      name_T name;
      age_T age;
  
      Person(name_T name, age_T age)
      {
          this->name = name;
          this->age = age;
      }
  
      void showPerson()
      {
          cout << "Name: " << this->name << " -- " << "Age: " << this->age << endl;
      }
  };
  
  // 1. 指定传入类型：直接显示对象的数据类型
  void print1(Person<string, int> &person)
  {
      person.showPerson();
  }
  
  void demo1()
  {
      Person<string, int> person("FH", 24);
      print1(person);
  }
  
  // 2. 参数模板化：将对象中的参数变为模板进行传递
  template <class string_T, class int_T>
  void print2(Person<string_T, int_T> &person)
  {
      person.showPerson();
  }
  
  void demo2()
  {
      Person<string, int> person("FF", 22);
      print2(person);
  }
  
  // 3. 整个类模板化：将对象类型模板化进行传递
  template <class person_T>
  void print3(person_T &person)
  {
      person.showPerson();
  }
  
  void demo3()
  {
      Person<string, int> person("HH", 20);
      print3(person);
  }
  
  int main()
  {
      demo1();
      demo2();
      demo3();
      return 0;
  }
  ```





##### 1.3.5 类模板与继承



- 当类模板遇到需要继承时，需注意：

  1. 当子类继承的父类是模板时，子类在声明时，需要指出父类中 `T` 的类型
  2. 如果不指定，编译器无法给子类分配内存
  3. 如果要灵活指定父类中的 `T` 类型，子类也需变成类模板

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/12 14:34
  //
  #include <iostream>
  
  using namespace std;
  
  template <class class_T>
  class Base
  {
  public:
      class_T base_Info;
  };
  
  class Derived_1 : Base<int> {};
  
  template <class deClass_T, class baseClass_T>
  class Derived_2 : Base<baseClass_T>
  {
  public:
      deClass_T derived_Info;
  
      Derived_2()
      {
          cout << "deClass_T: " << typeid(deClass_T).name() << endl;
          cout << "baseClass_T: " << typeid(baseClass_T).name() << endl;
      }
  };
  
  void demo()
  {
      Derived_2<int, char> derived2;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 1.3.6 类模板成员函数类外实现



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/12 15:04
  //
  #include <iostream>
  
  using namespace std;
  
  template <class name_T, class age_T>
  class Person
  {
  public:
      name_T name;
      age_T age;
  
      // 类模板 内 声明
      Person(name_T name, age_T age);
      void showPerson();
  };
  
  // 类模板 外 实现
  template <class name_T, class age_T>
  Person<name_T, age_T>::Person(name_T name, age_T age)
  {
      this->name = name;
      this->age = age;
  }
  
  template <class name_T, class age_T>
  void Person<name_T, age_T>::showPerson()
  {
      cout << "Name: " << this->name << " -- " << "Age: " << this->age << endl;
  }
  
  void demo()
  {
      Person<string, int> person("fh", 24);
      person.showPerson();
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 1.3.7 类模板分文件编写



- 说明：掌握类模板成员函数分文件编写产生的问题及解决方式

- 问题：类模板中成员函数创建时机是在调用阶段，导致分文件编写时链接不到

- 解决：

  1. 直接包含 `.cpp`文件
  2. 将声明和实现写在同一个文件中，并改名为`.hpp`，是约定的标准名称，并非强制要求

- 代码示例：

  1. 第一种：直接包含 `.cpp`文件

     `person.h`

     ```c++
     //
     // Created by Admin on 2021/7/12.
     //
     
     #ifndef TEMPLATE_STL_PERSON_H
     #define TEMPLATE_STL_PERSON_H
     
     #include <iostream>
     
     using namespace std;
     
     template <class name_T, class age_T>
     class Person
     {
     public:
         name_T name;
         age_T age;
     
         Person(name_T name, age_T age);
         void showPerson();
     };
     
     #endif //TEMPLATE_STL_PERSON_H
     ```

     `person.cpp`

     ```c++
     //
     // Created by Admin on 2021/7/12.
     //
     
     #include "person.h"
     
     template <class name_T, class age_T>
     Person<name_T, age_T>::Person(name_T name, age_T age)
     {
         this->name = name;
         this->age = age;
     }
     
     template <class name_T, class age_T>
     void Person<name_T, age_T>::showPerson()
     {
         cout << "Name: " << this->name << " -- " << "Age: " << this->age << endl;
     }
     ```

     `person_Main.cpp`

     ```c++
     //
     // Created by FHang on 2021/7/12 15:30
     //
     
     // 包含头文件 不管用 因为类模板的成员函数 只在调用时创建 所以编译时无法链接到外部文件
     // #include "person.h"
     
     // 第一种：直接包含 .cpp 文件
     // 直接包含 源文件 源文件中包含头文件 同时实现了 类模板的成员函数
     #include "person.cpp"
     
     void demo()
     {
         Person<string, int> person("fh", 24);
         person.showPerson();
     }
     
     int main()
     {
         demo();
         return 0;
     }
     ```

  2. 第二种：将声明和实现写在同一个文件中，并改名为`.hpp`

     `person.hpp`

     ```c++
     //
     // Created by Admin on 2021/7/12.
     //
     
     #ifndef TEMPLATE_STL_PERSON_H
     #define TEMPLATE_STL_PERSON_H
     
     #include <iostream>
     
     using namespace std;
     
     template <class name_T, class age_T>
     class Person
     {
     public:
         name_T name;
         age_T age;
     
         Person(name_T name, age_T age);
         void showPerson();
     };
     
     template <class name_T, class age_T>
     Person<name_T, age_T>::Person(name_T name, age_T age)
     {
         this->name = name;
         this->age = age;
     }
     
     template <class name_T, class age_T>
     void Person<name_T, age_T>::showPerson()
     {
         cout << "Name: " << this->name << " -- " << "Age: " << this->age << endl;
     }
     
     #endif //TEMPLATE_STL_PERSON_H
     ```

     `person_Main.cpp`

     ```c++
     //
     // Created by FHang on 2021/7/12 15:47
     //
     #include "person.hpp"
     
     void demo()
     {
         Person<string, int> person("fh", 24);
         person.showPerson();
     }
     
     int main()
     {
         demo();
         return 0;
     }
     ```






##### 1.3.8 类模板与友元



- 说明：类模板配合友元函数的类内和类外实现

- 实现：

  1. 全局函数类内实现：直接在类内声明友元
  2. 全局函数类外实现：需要让编译器提前知道全局函数的存在

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/12 15:57
  //
  #include <iostream>
  
  using namespace std;
  
  // 全局函数 类外实现
  template <class name_T, class age_T>
  class Person;
  
  template <class name_T, class age_T>
  void showPerson_2(Person<name_T, age_T> &person)
  {
      cout << "Name: " << person.name << " -- " << "Age: " <<person.age << endl;
  }
  
  template <class name_T, class age_T>
  class Person
  {
      // 全局函数 类内实现
      friend void showPerson_1(Person<name_T, age_T> &person)
      {
          cout << "Name: " << person.name << " -- " << "Age: " <<person.age << endl;
      }
  
      // 全局函数 类外实现
      friend void showPerson_2<>(Person<name_T, age_T> &person);
  
  private:
      name_T name;
      age_T age;
  
  public:
      Person(name_T name, age_T age)
      {
          this->name = name;
          this->age = age;
      }
  };
  
  void demo1()
  {
      cout << "全局函数，类内实现: ";
      Person<string, int> person("FH", 24);
      showPerson_1(person);
  }
  
  void demo2()
  {
      cout << "全局函数，类外实现: ";
      Person<string, int> person("fh", 24);
      showPerson_2(person);
  }
  
  int main()
  {
      demo1();
      demo2();
      return 0;
  }
  ```






##### 1.3.9 类模板案例



- 案例要求：实现一个通用的数组类

- 案例功能：

  1. 可以对内置类型和自定义类型进行存储
  2. 将数组的数据存储到堆区
  3. 构造函数可以传入数组的容量
  4. 提供拷贝函数及`operator=`防止浅拷贝问题
  5. 提供尾差法和尾删法对数组中的数据进行增删
  6. 通过下标访问数组中的元素
  7. 获取数组中的元素个数和数组容量

- 代码示例：(初期)

  `fArray.hpp`

  ```c++
  //
  // Created by Admin on 2021/7/13.
  //
  
  #ifndef TEMPLATE_STL_FARRAY_HPP
  #define TEMPLATE_STL_FARRAY_HPP
  
  #include <iostream>
  using namespace std;
  
  template <class array_T>
  class FArray
  {
  private:
      array_T *arrayAddress; // 指针指向堆区开辟的数组首地址
      int arrayCapacity; // 数组容量
      int arraySize; // 数组大小
  
  public:
      // 初始化 数组的容量 大小 和 在堆区创建
      FArray(int capacity)
      {
          this->arrayCapacity = capacity;
          this->arraySize = 0;
          this->arrayAddress = new array_T[this->arrayCapacity];
  
          cout << "构造函数调用" << endl;
      }
      // 释放堆区的数组
      ~FArray()
      {
          if (this->arrayAddress != nullptr)
          {
              delete[] this->arrayAddress;
              this->arrayAddress = nullptr;
  
              cout << "析构函数调用" << endl;
          }
      }
  
      // 拷贝构造函数，解决浅拷贝问题
      FArray(const FArray &fArray)
      {
          this->arrayCapacity = fArray.arrayCapacity;
          this->arraySize = fArray.arraySize;
  
          // 这是编译器默认的浅拷贝，在析构函数执行后，因为地址始终不为null，所以会重复释放
          // this->arrayAddress = fArray.arrayAddress;
          // 重新开辟空间，解决浅拷贝问题
          this->arrayAddress = new array_T[this->arrayCapacity];
          // 将 fArray的数据拷贝进新的空间中
          for (int i = 0; i < this->arraySize; ++i)
          {
              this->arrayAddress[i] = fArray.arrayAddress[i];
          }
  
          cout << "拷贝函数调用" << endl;
      }
  
      // operator= 解决浅拷贝问题
      FArray &operator=(const FArray &fArray)
      {
          // 先判断堆区是否存在，存在就先释放
          if (this->arrayAddress != nullptr)
          {
              delete[] this->arrayAddress;
              this->arrayAddress = nullptr;
              this->arrayCapacity = 0;
              this->arraySize = 0;
          }
  
          // 深拷贝
          this->arrayCapacity = fArray.arrayCapacity;
          this->arraySize = fArray.arraySize;
          this->arrayAddress = new array_T[this->arrayCapacity];
  
          for (int i = 0; i < this->arraySize; ++i)
          {
              this->arrayAddress[i] = fArray.arrayAddress[i];
          }
  
          cout << "operator=函数调用" << endl;
          return *this;
      }
  };
  
  #endif //TEMPLATE_STL_FARRAY_HPP
  ```

  `fArray_Main.cpp`

  ```c++
  //
  // Created by FHang on 2021/7/13 14:36
  //
  #include "fArray.hpp"
  
  void demo()
  {
      // 测试 初期，创建一个容量5的int类型数组
      // 测试 构造函数和析构函数
      FArray<int> fArray1(5);
  
      // 测试 拷贝构造函数
      FArray<int> fArray2(fArray1);
  
      // 测试 operator= 函数
      FArray<int> fArray3(10);
      fArray3 = fArray1;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```



- 代码示例：（后期）

  `fArray.hpp`

  ```c++
  //
  // Created by Admin on 2021/7/13.
  //
  
  #ifndef TEMPLATE_STL_FARRAY_HPP
  #define TEMPLATE_STL_FARRAY_HPP
  
  #include <iostream>
  using namespace std;
  
  template <class array_T>
  class FArray
  {
  private:
      array_T *arrayAddress; // 指针指向堆区开辟的数组首地址
      int arrayCapacity; // 数组容量
      int arraySize; // 数组大小
  
  public:
      // 初始化 数组的容量 大小 和 在堆区创建
      FArray(int capacity)
      {
          this->arrayCapacity = capacity;
          this->arraySize = 0;
          this->arrayAddress = new array_T[this->arrayCapacity];
  
          // cout << "构造函数调用" << endl;
      }
      // 释放堆区的数组
      ~FArray()
      {
          if (this->arrayAddress != nullptr)
          {
              delete[] this->arrayAddress;
              this->arrayAddress = nullptr;
  
              // cout << "析构函数调用" << endl;
          }
      }
  
      // 拷贝构造函数，解决浅拷贝问题
      FArray(const FArray &fArray)
      {
          this->arrayCapacity = fArray.arrayCapacity;
          this->arraySize = fArray.arraySize;
  
          // 这是编译器默认的浅拷贝，在析构函数执行后，因为地址始终不为null，所以会重复释放
          // this->arrayAddress = fArray.arrayAddress;
          // 重新开辟空间，解决浅拷贝问题
          this->arrayAddress = new array_T[this->arrayCapacity];
          // 将 fArray的数据拷贝进新的空间中
          for (int i = 0; i < this->arraySize; ++i)
          {
              this->arrayAddress[i] = fArray.arrayAddress[i];
          }
  
          // cout << "拷贝函数调用" << endl;
      }
  
      // operator= 解决浅拷贝问题
      FArray &operator=(const FArray &fArray)
      {
          // 先判断堆区是否存在，存在就先释放
          if (this->arrayAddress != nullptr)
          {
              delete[] this->arrayAddress;
              this->arrayAddress = nullptr;
              this->arrayCapacity = 0;
              this->arraySize = 0;
          }
  
          // 深拷贝
          this->arrayCapacity = fArray.arrayCapacity;
          this->arraySize = fArray.arraySize;
          this->arrayAddress = new array_T[this->arrayCapacity];
  
          for (int i = 0; i < this->arraySize; ++i)
          {
              this->arrayAddress[i] = fArray.arrayAddress[i];
          }
  
          // cout << "operator=函数调用" << endl;
          return *this;
      }
  
      // 返回数组大小
      int getArraySize()
      {
          return this->arraySize;
      }
  
      // 返回数组容量
      int getArrayCapacity()
      {
          return this->arrayCapacity;
      }
  
      // 尾插法
      void tail_Insertion(const array_T &arrayValue)
      {
          // 先判断数组容量是否够
          if (this->arrayCapacity == this->arraySize)
          {
              return;
          }
  
          this->arrayAddress[this->arraySize] = arrayValue; // 将数据插入到数组的尾部
          this->arraySize++; // 更新数组的大小
      }
  
      // 尾删法
      void tail_Deletion()
      {
          // 让用户无法访问最后一个元素，逻辑删除
          if (this->arraySize == 0)
          {
              return;
          }
  
          this->arraySize--;
      }
  
      // 通过小标访问数组元素 自定义的数据类型，内置的[]不能用，需要重载[]
      array_T &operator[](int fArray_Index)
      {
          return this->arrayAddress[fArray_Index];
      }
  };
  
  #endif //TEMPLATE_STL_FARRAY_HPP
  ```

  `fArray.cpp`

  ```c++
  //
  // Created by FHang on 2021/7/13 14:36
  //
  #include "fArray.hpp"
  
  int arrayIntCount = 0;
  int arrayPersonCount = 0;
  
  // 打印int类型数组
  void printIntArray(FArray<int> &fArray)
  {
      arrayIntCount++;
      cout << "数组" << arrayIntCount << "：[ ";
      for (int i = 0; i < fArray.getArraySize(); ++i)
      {
          cout << fArray[i] << " ";
      }
      cout << "]" << endl;
  }
  
  // 前期测试：创建到堆区，拷贝构造函数，operator= 函数
  void test1()
  {
      // 测试 初期，创建一个容量5的int类型数组
      // 测试 构造函数和析构函数
      FArray<int> fArray1(5);
  
      // 测试 拷贝构造函数
      FArray<int> fArray2(fArray1);
  
      // 测试 operator= 函数
      FArray<int> fArray3(10);
      fArray3 = fArray1;
  }
  
  // 后期测试：尾插法，容量，大小
  void test2()
  {
      // 创建数组 和 容量
      FArray<int> fArray1(10);
  
      // 用尾插法插入数据
      for (int i = 0; i < 5; ++i)
      {
          fArray1.tail_Insertion(i);
      }
      // 打印数组
      printIntArray(fArray1);
  
      // 查看数组 容量 大小
      cout << "数组容量：" << fArray1.getArrayCapacity() << endl;
      cout << "数组大小：" << fArray1.getArraySize() << endl;
  
      // 尾删法 测试
      FArray<int> fArray2(fArray1);
      printIntArray(fArray2);
  
      fArray2.tail_Deletion();
      cout << "数组容量：" << fArray2.getArrayCapacity() << endl;
      cout << "数组大小：" << fArray2.getArraySize() << endl;
  }
  
  // 后期测试 自定义数据类型
  class Person
  {
  public:
      string name;
      int age;
  
      Person(){};
      Person(string name, int age)
      {
          this->name = name;
          this->age = age;
      }
  };
  
  // 打印Person类型数组
  void printPersonArray(FArray<Person> &fArray)
  {
      arrayPersonCount++;
      cout << "数组" << arrayPersonCount << "：[ ";
      for (int i = 0; i < fArray.getArraySize(); ++i)
      {
          cout << fArray[i].name << " " << fArray[i].age << " - ";
      }
      cout << "]" << endl;
  }
  
  void test3()
  {
      FArray<Person> fArray(3);
      Person person1("FH", 24);
      Person person2("HH", 22);
  
      fArray.tail_Insertion(person1);
      fArray.tail_Insertion(person2);
  
      printPersonArray(fArray);
  
      cout << "数组容量：" << fArray.getArrayCapacity() << endl;
      cout << "数组大小：" << fArray.getArraySize() << endl;
  }
  
  int main()
  {
      // test1();
      // test2();
      test3();
      return 0;
  }
  ```

  





### 2. STL基础



#### 2.1 STL的诞生



- C++的面向对象和泛型编程思想，目的是复用性
- 大多数情况下，数据结构和算法都未能有一套标准，导致被迫从事大量重复工作
- 为了建立数据结构和算法的标准，诞生了STL



#### 2.2 STL基本概念



- STL(Standard Template Library，标准模板库)
- STL广义上分为：`容器(container)`，`算法(algorithm)`，`迭代器(iterator)`
- 容器和算法之间通过迭代器进行连接
- STL激活所有的代码都采用了模板类或模板函数



#### 2.3 STL六大组件



- STL大体分为六个组件：
  - 容器
  - 算法
  - 迭代器
  - 仿函数
  - 适配器(配接器)
  - 空间配置器
- 组件介绍：
  1. 容器：各种数据结构，如`vector`、`list`、`deque`、`set`、`map`等，用来存放数据
  2. 算法：各种常用的算法，如`sort`、`find`、`copy`、`for_each`等
  3. 迭代器：扮演了容器和算法之间的胶合剂
  4. 仿函数：行为类似函数，可作为算法的某种策略
  5. 适配器：一种修饰容器或仿函数或迭代器接口
  6. 空间配置器：负责空间的配置和管理



#### 2.4 STL容器\算法\迭代器



- 容器：存放数据，将运用最广泛的一些数据结构实现出来
  - 常用数据结构：数组，链表，树，栈，队列，集合，映射表 等
  - 容器分为：`序列式容器`和`关联式容器`
    - 序列式容器：强调值的排序，序列容器中的每个元素均有固定的位置
    - 关联式容器：二叉树结构，各元素之间没有严格的物理上的顺序关系



- 算法：解决问题，有限的步骤，解决逻辑或数学上的问题
  - 算法分为：`质变算法`和`非质变算法`
    - 质变算法：值运算过程中会更改区间内的元素的内容，例如：拷贝，替换，删除等
    - 非质变算法：值运算过程中不会更改区间内的元素内容，例如：查找，计数，遍历，寻找极值等



- 迭代器：容器和算法之间胶合剂

  - 提供一种方法，使之能够依序寻访某个容器所含的各个元素，而无需暴露该容器的内部表示方式

  - 每个容器都有自己的专属迭代器

  - 迭代器使用类似指针

  - 迭代器的种类：

    | 种类           | 功能                           | 支持算法                                 |
    | -------------- | ------------------------------ | ---------------------------------------- |
    | 输入迭代器     | 对数据只读访问                 | 只读，支持 ++，==，!=                    |
    | 输出迭代器     | 对数据只写访问                 | 只写，支持 ++                            |
    | 前向迭代器     | 读写操作，并能向前推进迭代器   | 读写，支持 ++，==，!=                    |
    | 双向迭代器     | 读写操作，并能向前和向后操作   | 读写，支持 ++，--                        |
    | 随机访问迭代器 | 读写操作，可以跳跃访问任意数据 | 读写，支持 ++，--，[n]，-n，<，<=，>，>= |

  - 常用的容器这迭代器种类为`双向迭代器`和`随机访问迭代器`





#### 2.5 容器\算法\迭代器基础



##### 2.5.1 vector存放内置数据类型



- 容器：`vector`

- 算法：`for_each`

- 迭代器：`vector<int>::iterator`

- 示例：

  ```c++
  //
  // Created by FHang on 2021/7/19 14:23
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  using namespace std;
  
  void fPrint(int value)
  {
      cout << value << endl;
  }
  
  void demo()
  {
      // 创建一个vector容器
      vector<int> v;
  
      // 插入数据
      v.push_back(10);
      v.push_back(11);
      v.push_back(12);
      v.push_back(13);
  
      // 通过创建迭代器访问容器中的数据
      vector<int>::iterator iBegin = v.begin(); // 起始地迭代器 指向容器中的第一个元素
      vector<int>::iterator iEnd = v.end(); // 指向容器最后一个元素 之后的地址
  
      // 第一种遍历方式
  //    while (iBegin != iEnd)
  //    {
  //        cout << *iBegin << endl;
  //        iBegin++;
  //    }
  
      // 第二种遍历方式
  //    for (vector<int>::iterator i = v.begin(); i != v.end(); ++i)
  //    {
  //        cout << *i << endl;
  //    }
  
      // 第三种遍历方式
      for_each(v.begin(), v.end(), fPrint);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  



##### 2.5.2 vector存放自定义数据类型



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/19 14:49
  //
  #include <iostream>
  #include <vector>
  using namespace std;
  
  class Person
  {
  public:
      string name;
      int age;
  
      Person(string name, int age)
      {
          this->name = name;
          this->age = age;
      }
  };
  
  void demo1()
  {
      cout << "< --demo1-- >" << endl;
      vector<Person> v_P;
  
      Person p1("fh", 24);
      Person p2("ff", 22);
      Person p3("hh", 20);
  
      v_P.push_back(p1);
      v_P.push_back(p2);
      v_P.push_back(p3);
  
      for (vector<Person>::iterator it_P = v_P.begin(); it_P != v_P.end(); ++it_P)
      {
          cout << it_P->name << " " << it_P->age << endl;
      }
  
      cout << endl;
  }
  
  void demo2()
  {
      cout << "< --demo2-- >" << endl;
      vector<Person *> v_P;
  
      Person p1("fh", 24);
      Person p2("ff", 22);
      Person p3("hh", 20);
  
      v_P.push_back(&p1);
      v_P.push_back(&p2);
      v_P.push_back(&p3);
  
      for (vector<Person *>::iterator it_P = v_P.begin(); it_P != v_P.end(); ++it_P)
      {
          cout << (*it_P)->name << " " << (*it_P)->age << endl;
      }
  
      cout << endl;
  }
  
  int main()
  {
      demo1();
      demo2();
      return 0;
  }
  ```





##### 2.5.3 vector容器嵌套容器



- 容器中嵌套容器，将数据进行遍历打印

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/19 15:07
  //
  #include <iostream>
  #include <vector>
  using namespace std;
  
  void demo()
  {
      // 创建大容器
      vector<vector<int>> v_Big;
      // 创建小容器
      vector<int> v_S1;
      vector<int> v_S2;
      vector<int> v_S3;
      vector<int> v_S4;
      // 向小容器中添加数据
      for (int i = 0; i < 4; ++i)
      {
          v_S1.push_back(i + 1);
          v_S2.push_back(i + 2);
          v_S3.push_back(i + 3);
          v_S4.push_back(i + 4);
      }
      // 将小容器插入大容器
      v_Big.push_back(v_S1);
      v_Big.push_back(v_S2);
      v_Big.push_back(v_S3);
      v_Big.push_back(v_S4);
      // 遍历大容器
      for (vector<vector<int>>::iterator it_Big = v_Big.begin(); it_Big != v_Big.end(); ++it_Big)
      {
          // (*it_Big)是小容器 vector<int>
          // 遍历小容器
          for (vector<int>::iterator it_Small = (*it_Big).begin(); it_Small != (*it_Big).end(); ++it_Small)
          {
              cout << *it_Small << " ";
          }
          cout << endl;
      }
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





### 3. STL常用容器



#### 3.1 string容器



##### 3.1.1 string基本概念



- 本质：`string`是C++的风格字符串，而`string`本质是一个类
- `string`和`char *` 区别：
  1. `char *` 是指针
  2. `string` 是指针，内部封装了 `char *` ，管理这个字符串，是一个 `char * `的容器
- 特点：
  1. `string`内部封装了很多的成员方法
  2. 查找 `find`，拷贝 `copy`，删除 `delete`，替换 `replace`，插入 `insert`
  3. `string`管理 `char *` 所分配的内存，不用担心复制越界和取值越界，由类内部进行负责



##### 3.1.2 string构造函数



- 构造函数原型：

  | `string();`                | 创建一个空的字符串                     |
  | -------------------------- | -------------------------------------- |
  | `string(const char *s);`   | 使字符串初始化                         |
  | `string(const string &s);` | 用一个string对象初始化另一个string对象 |
  | `string(int n, char c);`   | 使用n个字符c，初始化                   |

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/7/20 9:43
  //
  #include <iostream>
  
  using namespace std;
  
  void demo()
  {
      // 创建一个空的字符串
      string string1;
  
      // 使字符串初始化
      const char *str = "HelloWorld";
      string string2(str);
      cout << "string2 = " << string2 << endl;
  
      // 用一个string对象初始化另一个string对象
      string string3(string2);
      cout << "string3 = " << string3 << endl;
  
      // 使用n个字符c，初始化
      string string4(10, 'a');
      cout << "string4 = " << string4 << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.1.3 string赋值操作



- 功能描述：给string字符串赋值

  | 赋值的函数原型                          |                                        |
  | --------------------------------------- | -------------------------------------- |
  | `string &operator=(const char *s)`;     | char *类型字符串，赋值给当前的字符串   |
  | `string &operator=(const string &s);`   | 字符串s，赋值给当前的字符串            |
  | `string &operator=(char c);`            | 字符，赋值给当前字符串                 |
  | `string &assign(const char *s);`        | 字符串s，赋值给当前的字符串            |
  | `string &assign(const char *s, int n);` | 字符串s的前n个字符，赋值给当前的字符串 |
  | `string &assign(const string &s);`      | 字符串s，赋值给当前的字符串            |
  | `string &assign(int n, char c);`        | 用n个字符c，赋值给当前字符串           |




- 示例：

  ```c++
  //
  // Created by FHang on 2021/9/21 14:56
  //
  #include <iostream>
  
  using namespace std;
  
  void demo()
  {
      string string1;
      string1 = "hello world";
      cout << "string1 = " << string1 << endl;
  
      string string2;
      string2 = string1;
      cout << "string2 = " << string2 << endl;
  
      string string3;
      string3 = "A";
      cout << "string3 = " << string3 << endl;
  
      string string4;
      string4.assign("hello world");
      cout << "string4 = " << string4 << endl;
  
      string string5;
      string5.assign("hello world", 3);
      cout << "string5 = " << string5 << endl;
  
      string string6;
      string6.assign(6, 'a');
      cout << "string6 = " << string6 << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.1.4 string字符拼接



- 功能描述：实现字符串末尾拼接字符串

  | 函数原型                                           |                                                     |
  | -------------------------------------------------- | --------------------------------------------------- |
  | `string &operator+=(const char *str);`             | 重载+=操作符                                        |
  | `string &operator+=(const char c);`                | 重载+=操作符                                        |
  | `string &operator+=(const string &str);`           | 重载+=操作符                                        |
  | `string &append(const char *s);`                   | 字符串s，连接到当前字符串的末尾                     |
  | `string &append(const char *s, int n);`            | 字符串s的前n个字符，连接到当前字符串的末尾          |
  | `string &append(const string &s);`                 | 等同于，`string &operator+=(const string &str);`    |
  | `string &append(const string &s, int pos, int n);` | 字符串s中从pos开始取n个字符，连接到当前字符串的末尾 |



- 示例：

  ```c++
  //
  // Created by FHang on 2021/9/21 15:12
  //
  #include <iostream>
  
  using namespace std;
  
  void demo()
  {
      string string1 = "Hello ";
      string1 += "World";
      cout << "String1 = " << string1 << endl;
  
      string string2 = "Hi ";
      string2 += string1;
      cout << "String2 = " << string2 << endl;
  
      string string3 = "Age ";
      string3 += '8';
      cout << "String3 = " << string3 << endl;
  
      string string4 = "Hello ";
      string4.append("World");
      cout << "String4 = " << string4 << endl;
  
      string string5 = "Hi ";
      string5.append(string4);
      cout << "String5 = " << string5 << endl;
  
      string string6;
      string6.append("Hello World", 4);
      cout << "String6 = " << string6 << endl;
  
      string string7 = string5;
      string7.append("Hello World", 5, 6);
      cout << "String7 = " << string7 << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  



##### 3.1.5 string查找替换



- 功能描述：

  1. 查找：查找指定字符串是否存在

  2. 替换：在指定的位置替换字符串

     | 函数原型                                                    |                                          |
     | ----------------------------------------------------------- | ---------------------------------------- |
     | `int find(const string &str, int pos = 0) const;`           | 查找str第一次出现的位置，默认pos从头开始 |
     | `int find(const char *s, int pos = 0) const;`               | 查找s第一次出现的位置，默认pos从头开始   |
     | `int find(const char *s, int pos, int n) const;`            | 从pos查找s的前n个字符第一次位置          |
     | `int find(const char *c, int pos = 0) const;`               | 查找字符c第一次出现位置                  |
     | `int rfind(const string &str, int pos = npos) const;`       | 查找str最后一次位置，从pos开始找         |
     | `int rfind(const char *s, int pos = npos) const;`           | 查找s最后一次位置，从pos开始找           |
     | `int rfind(const char *s, int pos, int n) const;`           | 从pos查找s的前n个字符最后一次位置        |
     | `int rfind(const char *c, int pos = 0) const;`              | 查找字符c最后一次出现位置                |
     | `string &replace(int pos, int n, const string &str) const;` | 替换从pos开始n个字符为字符串str          |
     | `string &replace(int pos, int n, const char *s) const;`     | 替换从pos开始n个字符为字符串s            |



- 示例：

  ```c++
  //
  // Created by FHang on 2021/9/21 15:46
  //
  #include <iostream>
  
  using namespace std;
  
  void findString()
  {
      string string1 = "AABBCCBBAA";
      int pos1 = string1.find("BB");
      cout << "BB pos1 = " << pos1 << endl;
  
      int pos2 = string1.rfind("BB");
      cout << "BB pos2 = " << pos2 << endl;
  }
  
  void replaceString()
  {
      string string1 = "ABCDE";
      string1.replace(2, 3, "123");
      cout << "String1 = " << string1 << endl;
  }
  
  int main()
  {
      findString();
      replaceString();
      return 0;
  }
  ```



- 总结：
  - `find`是从左往右查，`rfind`是从右往左查
  - `find`查到字符后，返回字符的位置，找不到返回-1
  - `replace`在替换时，需指定起始位置，替换字符数，替换字符



##### 3.1.6 string字符比较



- 功能描述：字符串之间比较

- 比较方式：按照字符编码ACSII进行比较

  - `=` 返回 `0`
  - `>` 返回 `1`
  - `<` 返回 `-1`

  | 函数原型                                |                 |
  | --------------------------------------- | --------------- |
  | `int compare(const string &str) const;` | 与字符串str比较 |
  | `int compare(const char *s) const;`     | 与字符串s比较   |



- 示例：

  ```c++
  //
  // Created by FHang on 2021/9/21 16:12
  //
  #include <iostream>
  
  using namespace std;
  
  void demo()
  {
      string string1 = "Hello";
      string string2 = "World";
  
      if (string1.compare(string2) == 0)
      {
          cout << "string1 string2" << " = " << endl;
      }
      else
      {
          cout << "string1 string2" << " != " << endl;
      }
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.1.7 string字符存取



- string中单个字符串存取方式有两种：

  | 方式                       |                    |
  | -------------------------- | ------------------ |
  | `char &operator[](int n);` | 通过`[]`方式取字符 |
  | `char &at(int n);`         | 通过`at`取字符     |



- 示例：

  ```c++
  //
  // Created by FHang on 2021/9/21 16:23
  //
  #include <iostream>
  
  using namespace std;
  
  string string1 = "ABCDEFG";
  
  void demo1()
  {
      for (int i = 0; i < string1.size(); ++i)
      {
          cout << string1[i] << " ";
      }
      cout << endl;
  }
  
  void demo2()
  {
      for (int i = 0; i < string1.size(); ++i)
      {
          cout << string1.at(i) << " ";
      }
      cout << endl;
  }
  
  int main()
  {
      demo1();
      demo2();
      return 0;
  }
  ```





##### 3.1.8 string插入删除



- 功能描述：对`string`字符串进行`插入`和`删除`字符操作

  | 函数原型                                      |                        |
  | --------------------------------------------- | ---------------------- |
  | `string &insert(int pos, const char *s);`     | 插入字符串             |
  | `string &insert(int pos, const string &str);` | 插入字符串             |
  | `string &insert(int pos, int n, char c);`     | 在指定位置插入n个字符c |
  | `string &insert(int pos, int n = npos);`      | 删除从pos开始的n个字符 |



- 示例：

  ```c++
  ```

  


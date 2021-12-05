# C++_模板和STL



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



- 功能描述：给`string`字符串赋值

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



- `string`中单个字符串存取方式有两种：

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
  | `string &erase(int pos, int n = npos);`       | 删除从pos开始的n个字符 |



- 示例：

  ```c++
  //
  // Created by FHang on 2021/9/21 16:35
  //
  #include <iostream>
  
  using namespace std;
  
  void demoInsert()
  {
      string str1 = "Hello ";
      string str2 = "world ";
  
      str1.insert(6, "world ");
      cout << "str1 = " << str1 << endl;
  
      str1.insert(12, str2);
      cout << "str1 = " << str1 << endl;
  
      str1.insert(18, 6, '!');
      cout << "str1 = " << str1 << endl;
  }
  
  void demoDelete()
  {
      string str3 = "Hello World";
      str3.erase(5, 6);
      cout << "str3 = " << str3 << endl;
  }
  
  int main()
  {
      demoInsert();
      demoDelete();
      return 0;
  }
  ```
  
- 总结：插入`insert()`和删除`erase()`都是从下标0开始的





##### 3.1.9 string获取字串



- 功能描述：从字符串中获得想要的一段子字符串

  | 函数原型                                          |                                    |
  | ------------------------------------------------- | ---------------------------------- |
  | `string substr(int pos = 0, int n = npos) const;` | 返回由pos开始的n个字符组成的字符串 |



- 示例：

  ```c++
  //
  // Created by FHang on 2021/10/5 14:59
  //
  #include <iostream>
  
  using namespace std;
  
  void subStringDemo()
  {
      string str1 = "Hello World";
  
      str1 = str1.substr(0, 5);
      cout << "str1 = " << str1 << endl;
  }
  
  void getEmailTypeInfo()
  {
      string str2 = "752972182@qq.com";
  
      int pos = str2.find("@");
      string str3 = str2.substr(pos + 1, 2);
      str2 = str2.substr(0, pos);
  
      cout << "str2 email user = " << str2 << endl;
      cout << "str3 email type = " << str3 << endl;
  }
  
  int main()
  {
      subStringDemo();
      getEmailTypeInfo();
      return 0;
  }
  ```





#### 3.2 vector容器



##### 3.2.1 vector基本概念



- 功能：`vector`数据结构和`数组`非常相似，也称为`单端数组`

- 与数组的区别：`数组`是静态空间，`vector`可以动态扩展

- 动态扩展：并非是在原有的空间后面，连续开辟新空间；而是在另一个更大的内存空间中`重新开辟`，并`拷贝`原来的容器数据，同时`释放原容器`

- `vector`容器的迭代器是支持随机访问的迭代器

  | vector迭代器方法介绍 `vector<T> v;` |                            |
  | ----------------------------------- | -------------------------- |
  | `v.rend();`                         | 容器第一个元素之前的地址   |
  | `v.end();`                          | 容器最后一个元素之后的地址 |
  | `v.begin();`                        | 容器第一个元素自身的地址   |
  | `v.rbegin();`                       | 容器最后一个元素之前的地址 |
  | `v.insert();`                       | 容器中插入一个元素         |





##### 3.2.2 vector构造函数



- 功能描述：创建`vector`容器

  | 函数原型 `vector<T> v;`       |                                                              |
  | ----------------------------- | ------------------------------------------------------------ |
  | `vector<T> v;`                | 采用模板类实现，默认构造函数                                 |
  | `vector(v.begin(), v.end());` | 将 `[ v.begin(), v.end() )` 之间的元素拷贝给自身` [闭 开)区间` |
  | `vector(n, elem);`            | 构造函数将 n个 元素拷贝给自身                                |
  | `vector(const vector &vec);`  | 拷贝构造函数                                                 |



- 示例：

  ```c++
  //
  // Created by FHang on 2021/10/5 15:37
  //
  #include <iostream>
  #include <vector>
  
  using namespace std;
  
  void printVector(vector<int> &v)
  {
      for (vector<int>::iterator it = v.begin(); it != v.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo1()
  {
      // 默认构造（无参）
      vector<int> v1;
  
      for (int i = 0; i < 10; ++i)
      {
          v1.push_back(i);
      }
      printVector(v1);
  
      // 通过 [ ) 构造
      vector<int> v2(v1.begin(), v1.end());
      printVector(v2);
  
      // n个 elem构造
      vector<int> v3(10, 1);
      printVector(v3);
  
      // 拷贝构造
      vector<int> v4(v1);
      printVector(v4);
  }
  
  int main()
  {
      demo1();
  
      return 0;
  }
  ```





##### 3.2.3 vector赋值操作



- 功能描述：给容器赋值

  | 函数原型 `vector<T> v;`                |                                         |
  | -------------------------------------- | --------------------------------------- |
  | `vetor &operator=(const vector &vec);` | 重载`=` 操作符                          |
  | `v.assign(v.begin, v.end);`            | 将 `[begin, end)`区间中的数据拷贝到自身 |
  | `v.assign(n, elem);`                   | 将 n个 elem拷贝赋值给自身               |



- 示例：

  ```c++
  //
  // Created by FHang on 2021/10/5 16:08
  //
  #include <iostream>
  #include <vector>
  
  using namespace std;
  
  void printVector(vector<int> &v)
  {
      for (vector<int>::iterator it = v.begin(); it != v.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v1;
      for (int i = 0; i < 10; ++i)
      {
          v1.push_back(i);
      }
      printVector(v1);
  
      vector<int> v2 = v1;
      printVector(v2);
  
      vector<int> v3(v1.begin(), v1.end());
      printVector(v3);
  
      vector<int> v4(10, 1);
      printVector(v4);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.2.4 vector容量大小



- 功能描述：对`vector`容器的容量和大小进行操作

  | 函数原型                 |                                                              |
  | ------------------------ | ------------------------------------------------------------ |
  | `empty();`               | 判断容器是否为空                                             |
  | `capacity();`            | 获取容器的容量                                               |
  | `size();`                | 获取容器中的元素个数                                         |
  | `resize(int num);`       | 重新指定容器长度，若变长，默认填充；若变短，删除末尾超出容器长度的元素 |
  | `resize(int num, elem);` | 重新指定容器长度，若变长，elem填充；若变短，删除末尾超出容器长度的元素 |

  

- 示例：

  ```c++
  //
  // Created by FHang on 2021/10/5 16:18
  //
  #include <iostream>
  #include <vector>
  
  using namespace std;
  
  void printVector(vector<int> &v)
  {
      for (vector<int>::iterator it = v.begin(); it != v.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v1;
      for (int i = 0; i < 5; ++i)
      {
          v1.push_back(i);
      }
      printVector(v1);
  
      if (v1.empty())
      {
          cout << "v1 is empty" << endl;
      }
      else
      {
          cout << "v1 not empty" << endl;
          cout << "v1 size = " << v1.size() << endl;
          cout << "v1 capacity = " << v1.capacity() << endl;
      }
  
      v1.resize(10);
      printVector(v1);
  
      v1.resize(15, 1);
      printVector(v1);
  
      v1.resize(2);
      printVector(v1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```






##### 3.2.5 vector插入删除



- 功能描述：对`vector`容器进行插入、删除操作

  | 函数原型                                           |                                              |
  | -------------------------------------------------- | -------------------------------------------- |
  | `push_back(elem);`                                 | 尾部插入元素 `elem`                          |
  | `pop_back();`                                      | 删除最后一个元素                             |
  | `insert(const_iterator pos, elem);`                | 迭代器指向位置`pos`，插入元素`elem`          |
  | `insert(const_iterator pos, int count, elem);`     | 迭代器指向位置`pos`，插入`count`个元素`elem` |
  | `erase(const_iterator pos);`                       | 删除迭代器指向位置`pos`的元素                |
  | `erase(const_iterator start, const_iterator end);` | 删除迭代器选择的`start`到`end`之间的元素     |
  | `clear();`                                         | 清空容器                                     |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/10 15:56
  //
  #include <iostream>
  #include <vector>
  
  using namespace std;
  
  void printVector(vector<int> &v)
  {
      for (vector<int>::iterator iterator = v.begin(); iterator != v.end(); ++iterator)
      {
          cout << *iterator << " ";
      }
      cout << endl;
  }
  
  void demo1()
  {
      // 尾插入
      vector<int> v1;
      v1.push_back(1);
      v1.push_back(2);
      v1.push_back(3);
      v1.push_back(4);
      v1.push_back(5);
      printVector(v1);
  
      // 尾删除
      v1.pop_back();
      printVector(v1);
  
      // 迭代器指定位置插入
      v1.insert(v1.begin(), 0);
      printVector(v1);
  
      // 迭代器指定位置插入 指定数量 元素
      v1.insert(v1.end(), 3, 5);
      printVector(v1);
  
      // 删除迭代器指向位置的元素
      v1.erase(v1.end() - 1);
      printVector(v1);
  
      // 删除迭代器选择的start到end之间的元素
      v1.erase(v1.begin() + 1, v1.end() - 2);
      printVector(v1);
  
      // clear
      v1.clear();
      printVector(v1);
  }
  
  int main()
  {
      demo1();
      return 0;
  }
  ```





##### 3.2.6 vector数据存取



- 功能描述：`vector`中的数据存取操作

  | 函数原型         |                            |
  | ---------------- | -------------------------- |
  | `at(int index);` | 返回索引`index`所指的数据  |
  | `operator[];`    | 返回索引`index`所指的数据  |
  | `front();`       | 返回容器中第一个数据元素   |
  | `back();`        | 返回容器中最后一个数据元素 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/10 16:20
  //
  #include <iostream>
  #include <vector>
  
  using namespace std;
  
  void demo()
  {
      vector<int> v;
  
      for (int i = 0; i < 5; ++i)
      {
          v.push_back(i);
      }
  
      for (int i = 0; i < v.size(); ++i)
      {
          cout << v[i] << " ";
      }
      cout << endl;
  
      for (int i = 0; i < v.size(); ++i)
      {
          cout << v.at(i) << " ";
      }
      cout << endl;
  
      cout << "Vector Front Elem: " << v.front() << endl;
  
      cout << "Vector Back Elem: " << v.back() << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.2.7 vector互换容器



- 功能描述：实现两个容器内元素的互换

  | 函数原型             |                               |
  | -------------------- | ----------------------------- |
  | `swap(otherVector);` | 将otherVector与本身的元素互换 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/10 16:30
  //
  #include <iostream>
  #include <vector>
  
  using namespace std;
  
  void debugVector(vector<int> &v)
  {
      for (vector<int>::iterator it = v.begin(); it != v.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void debugVectorInfo(vector<int> &v)
  {
      cout << "Vector Capacity: " << v.capacity() << endl;
      cout << "Vector Size: " << v.size() << endl;
  }
  
  void demo1()
  {
      vector<int> v1;
      vector<int> v2;
  
      for (int i = 0; i < 5; ++i)
      {
          v1.push_back(i);
      }
  
      for (int i = 5; i > 0; --i)
      {
          v2.push_back(i);
      }
  
      v1.swap(v2);
  
      debugVector(v1);
      debugVector(v2);
  }
  
  void demo2()
  {
      vector<int> v3;
      for (int i = 0; i < 1000000; ++i)
      {
          v3.push_back(i);
      }
      debugVectorInfo(v3);
  
      // 巧用 swap() 收缩容器的容量大小0
      v3.resize(10);
      vector<int>(v3).swap(v3);
      debugVectorInfo(v3);
  }
  
  int main()
  {
      demo1();
      demo2();
      return 0;
  }
  ```

- 总结：swap还可以收缩容器的容量大小





##### 3.2.8 vector预留空间



- 功能描述：减少`vector`在动态扩展容器时的扩展次数

  | 函数原型               |                                                            |
  | ---------------------- | ---------------------------------------------------------- |
  | `reserve(int length);` | 容器预留`length`个元素长度，预留位置不初始化，元素不可访问 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/10 16:54
  //
  #include <iostream>
  #include <vector>
  
  using namespace std;
  
  void demo1()
  {
      int count = 0;
      int *p = nullptr;
  
      vector<int> v;
  
      for (int i = 0; i < 10000000; ++i)
      {
          v.push_back(i);
          if (p != &v[0])
          {
              p = &v[0];
              ++count;
          }
      }
      cout << "Number Of Extensions: " << count << endl;
  }
  
  void demo2()
  {
      int count = 0;
      int *p = nullptr;
  
      vector<int> v;
      v.reserve(10000001);
  
      for (int i = 0; i < 10000000; ++i)
      {
          v.push_back(i);
          if (p != &v[0])
          {
              p = &v[0];
              ++count;
          }
      }
      cout << "Reserve Number Of Extensions: " << count << endl;
  }
  
  int main()
  {
      demo1();
      demo2();
      return 0;
  }
  ```

- 总结：如果一开始容器需要插入足够大的数据时，可以通过`reserve`的方式提前预留，已减少容器扩展的次数





#### 3.3 deque容器



##### 3.3.1 deque基本概念



- 功能：双端数组，可以对容器头端进行`插入`、`删除`操作

- `deque`与`vector`的区别：

  1. `vector`头部插入、删除效率低，数据量越大，效率越低
  2. `vector`访问元素比`deque`快，源于内部实现的区别

- `deque`功能介绍：

  | 函数原型        |          |
  | --------------- | -------- |
  | `push_front();` | 头部插入 |
  | `pop_front();`  | 头部删除 |

- `deque`内部工作原理：

  - `deque`内部有一个`中控器`，维护每段`缓冲区`的内容，缓冲区存放真实数据
  - 中控器维护的是`缓冲区的地址`，使得`deque`在使用时，像是`连续的内存空间`
  - `deque`容器的`迭代器`支持`随机访问`





##### 3.3.2 deque构造函数



- 功能描述：`deque`容器构造

  | 函数原型                     |                                                 |
  | ---------------------------- | ----------------------------------------------- |
  | `deque<T> dequeT;`           | 默认构造形式                                    |
  | `deque(begin, end);`         | 构造函数将 `[begin, end)`区间中的元素拷贝给自身 |
  | `deque(n, elem);`            | 构造函数将`n`个`elem`拷贝给自身                 |
  | `deque(const deque &deque);` | 拷贝构造函数                                    |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/14 12:27
  //
  #include <iostream>
  #include <deque>
  
  using namespace std;
  
  // const 修饰 该容器为只可 读
  void printDeque(const deque<int> &otherDeque)
  {
      for (deque<int>::const_iterator it = otherDeque.begin(); it != otherDeque.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo1()
  {
      // 无参构造
      deque<int> d1;
      for (int i = 0; i < 10; ++i)
      {
          d1.push_back(i);
      }
      printDeque(d1);
  
      // 区间构造
      deque<int> d2(d1.begin(), d1.end());
      printDeque(d2);
  
      // n个元素构造
      deque<int> d3(10, 1);
      printDeque(d3);
  
      // 拷贝构造
      deque<int> d4(d3);
      printDeque(d4);
  }
  
  int main()
  {
      demo1();
      return 0;
  }
  ```

- 总结：`deque`与`vector`相似，灵活使用即可



##### 3.3.3 deque赋值操作



- 功能描述：给`deque`容器赋值

  | 函数原型                                |                                             |
  | --------------------------------------- | ------------------------------------------- |
  | `deque &operator=(const deque &deque);` | 重载等号操作                                |
  | `assign(begin, end);`                   | 将 `[begin, end)`区间中的数据拷贝赋值给本身 |
  | `assign(n, elem);`                      | 将`n`个`elem`拷贝赋值给本身                 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/14 13:13
  //
  #include <iostream>
  #include <deque>
  
  using namespace std;
  
  void printDeque(deque<int> &otherDeque)
  {
      for (deque<int>::iterator it = otherDeque.begin(); it != otherDeque.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      deque<int> deque1;
      for (int i = 0; i < 10; ++i)
      {
          deque1.push_back(i);
      }
      printDeque(deque1);
  
      // operator=
      deque<int> deque2 = deque1;
      printDeque(deque2);
  
      // assign(begin, end)
      deque<int> deque3;
      deque3.assign(deque1.begin(), deque1.end());
      printDeque(deque3);
  
      // assign(n, elem)
      deque<int> deque4;
      deque4.assign(10, 1);
      printDeque(deque4);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```






##### 3.3.4 deque大小操作



- 功能描述：对`deque`容器的大小进行操作

  | 函数原型                  |                                                              |
  | ------------------------- | ------------------------------------------------------------ |
  | `deque.empty();`          | 判断容器是否为空                                             |
  | `deque.size();`           | 获取容器中元素个数                                           |
  | `deque.resize(num);`      | 重新指定容器长度为`num`，容器过长以默认值填充，容器过短，删除末尾多余元素 |
  | `deque.resize(num, elem)` | 重新指定容器长度为`num`，容器过长以`elem`填充，容器过短，删除末尾多余元素 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/17 15:17
  //
  #include <iostream>
  #include <deque>
  
  using namespace std;
  
  void printDeque(const deque<int> &otherDeque)
  {
      for (deque<int>::const_iterator it = otherDeque.begin(); it != otherDeque.end(); ++it)
      {
          cout << *it <<" ";
      }
      cout << endl;
  }
  
  void demo()
  {
      // 初始化 deque1
      deque<int> deque1;
      for (int i = 0; i < 10; ++i)
      {
          deque1.push_back(i);
      }
      printDeque(deque1);
  
      // 判断deque1是否为空，不为空，打印出容器大小
      if (deque1.empty())
      {
          cout << "deque1 is empty" << endl;
      }
      else
      {
          cout << "deque1 size: " << deque1.size() << endl;
          // deque 没有容量的概念(capacity)
      }
  
      // deque1 大小重置
      deque1.resize(12);
      printDeque(deque1);
  
      deque1.resize(15, 1);
      printDeque(deque1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

- 总结：

  1. `deque`没有容量的概念
  2. `empty`判断是否为空
  3. `size`获取容器的大小
  4. `resize`重置容器大小





##### 3.3.5 deque插入删除



- 功能描述：向`deque`容器插入和删除数据

- 函数原型：

  | 两端插入caoz        |                      |
  | ------------------- | -------------------- |
  | `push_back(elem);`  | 容器尾部添加一个数据 |
  | `push_front(elem);` | 容器头部插入一个数据 |
  | `pop_back();`       | 删除容器最后一个数据 |
  | `pop_front();`      | 删除容器开头一个数据 |

  | 指定位置操作               |                                                       |
  | -------------------------- | ----------------------------------------------------- |
  | `insert(pos, elem);`       | 在`pos`位置插入一个`elem`元素的拷贝，返回新数据的位置 |
  | `insert(pos, n, elem);`    | 在`pos`位置插入`n`个`elem`元素的拷贝，无返回值        |
  | `insert(pos, begin, end);` | 在`pos`位置插入`[begin, end)`区间的数据，无返回值     |
  | `clear();`                 | 清空容器所有数据                                      |
  | `erase(begin, end);`       | 删除`[begin, end)`区间的数据，返回下一个数据的位置    |
  | `erase(pos);`              | 删除`pos`位置的数据，返回下一个数据的位置             |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/17 15:38
  //
  #include <iostream>
  #include <deque>
  
  using namespace std;
  
  void printDeque(const deque<int> &otherDeque)
  {
      for (deque<int>::const_iterator it = otherDeque.begin(); it != otherDeque.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      deque<int> deque1;
  
      // push_back()
      deque1.push_back(1);
      deque1.push_back(2);
  
      // push_front()
      deque1.push_front(0);
      deque1.push_front(0);
      printDeque(deque1);
  
      // pop_back()
      deque1.pop_back();
  
      // pop_front()
      deque1.pop_front();
      printDeque(deque1);
  
      // insert(pos, elem)
      deque1.insert(deque1.begin(), 0);
  
      // insert(pos, n, elem)
      deque1.insert(deque1.end(), 2, 2);
      printDeque(deque1);
  
      // clear()
      deque1.clear();
      printDeque(deque1);
  
      for (int i = 0; i < 10; ++i)
      {
          deque1.push_back(i);
      }
      printDeque(deque1);
  
      // erase(begin, end)
      deque1.erase(deque1.begin(), deque1.end() - 7);
      printDeque(deque1);
  
      // erase(pos)
      deque1.erase(deque1.begin() + 1);
      printDeque(deque1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.3.6 deque数据存取



- 功能描述：的`deque`中的数据的存取操作

  | 函数原型           |                           |
  | ------------------ | ------------------------- |
  | `at(int index);`   | 返回索引`index`所指的数据 |
  | `operator[index];` | 返回索引`index`所指的数据 |
  | `front();`         | 返回容器中第一个数据      |
  | `back();`          | 返回容器这最后一个数据    |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/17 15:57
  //
  #include <iostream>
  #include <deque>
  
  using namespace std;
  
  void printDeque(const deque<int> &otherDeque)
  {
      for (deque<int>::const_iterator it = otherDeque.begin(); it != otherDeque.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      deque<int> deque1;
      for (int i = 0; i < 10; ++i)
      {
          deque1.push_back(i);
      }
      printDeque(deque1);
  
      // at(int index)
      cout << deque1.at(1) << endl;
  
      // operator[]
      cout << deque1[1] << endl;
  
      // front()
      cout << deque1.front() << endl;
  
      // back()
      cout << deque1.back() << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  



##### 3.3.7 deque容器排序



- 功能描述：利用算法实现`deque`容器进行排序

  | 函数原型                              |                                  |
  | ------------------------------------- | -------------------------------- |
  | `sort(iterator begin, iterator end);` | 对`begin, end`区间的数据进行排序 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/17 16:05
  //
  #include <iostream>
  #include <deque>
  #include <algorithm>
  
  using namespace std;
  
  void printDeque(const deque<int> &otherDeque)
  {
      for (deque<int>::const_iterator it = otherDeque.begin(); it != otherDeque.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      deque<int> deque1;
      for (int i = 20; i > 0; i -= 2)
      {
          deque1.push_back(i);
      }
      printDeque(deque1);
  
      std::sort(deque1.begin(), deque1.end());
      printDeque(deque1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

- 总结：使用`sort`排序，需引入头文件`algorithm`





#### 3.4 案例-评委打分



##### 3.4.1 案例描述

- 5名选上ABCDE，10名评委分别对每一名选手打分，去除最高分和最低分，取平均分



##### 3.4.2 实现步骤

1. 创建5名选手，存入`vector`容器中
2. 遍历`vector`容器，获取每一名选手，使用`for`循环，把10名评委的打分存入`deque`容器中
3. `sort`算法对`deque`容器中分数排序，去除最高和最低分
4. `deque`容器遍历一遍，累加总分
5. 获取平均分



##### 3.4.3 示例代码

```c++
//
// Created by FHang on 2021/10/20 14:26
//
#include <iostream>
#include <vector>
#include <deque>
#include <algorithm>
#include <ctime>

using namespace std;

class Player
{
public:
    string playerName;
    int playerScore;

    Player(string name, int score)
    {
        this->playerName = name;
        this->playerScore = score;
    }
};

/*Test Code*/
/*void printVector(vector<Player> &v_Player)
{
    for (vector<Player>::iterator it = v_Player.begin(); it != v_Player.end(); ++it)
    {
        cout << "Name: " << (*it).playerName << " Score: " << (*it).playerScore << endl;
    }
    cout << endl;
}

void printPlayerScores(vector<Player> &v_Player, deque<int> &d_Scores)
{
    for (vector<Player>::iterator v_it = v_Player.begin(); v_it != v_Player.end(); ++v_it)
    {
        cout << v_it->playerName << endl;
        for (deque<int>::iterator d_it = d_Scores.begin(); d_it != d_Scores.end(); ++d_it)
        {
            cout << *d_it << " ";
        }
        cout << endl;
    }
}*/

/*Program Code*/
vector<Player> createPlayers()
{
    int score = 0;
    string nameSeed = "ABCDE";
    vector<Player> v_Player;

    for (int i = 0; i < 5; ++i)
    {
        string name = "player";
        name += nameSeed[i];
        Player player(name, score);
        v_Player.push_back(player);
    }

    // printVector(v_Player);

    return v_Player;
}

int playerScoreSortAndDeal(deque<int> &d_Scores)
{
    float averageScore;
    int allScore = 0;

    std::sort(d_Scores.begin(), d_Scores.end());
    d_Scores.pop_front();
    d_Scores.pop_back();

    for (deque<int>::iterator it = d_Scores.begin(); it != d_Scores.end(); ++it)
    {
        allScore += (*it);
    }
    averageScore = allScore / d_Scores.size();
    return averageScore;
}

void setPlayerScore(vector<Player> &v_Player)
{
    for (vector<Player>::iterator it = v_Player.begin(); it != v_Player.end(); ++it)
    {
        deque<int> d_Scores;

        for (int i = 0; i < 10; ++i)
        {
            int score = (rand() % 71) + 30;
            d_Scores.push_back(score);
        }

        it->playerScore = playerScoreSortAndDeal(d_Scores);

        // printPlayerScores(v_Player, d_Scores);
    }
}

void showPlayerAverageScore(vector<Player> &v_Player)
{
    for (vector<Player>::iterator it = v_Player.begin(); it != v_Player.end(); ++it)
    {
        cout << "Name: " + it->playerName << " AverageScore: " << it->playerScore << endl;
    }
}

int main()
{
    srand((unsigned int) time(NULL));

    vector<Player> v_Player = createPlayers();
    setPlayerScore(v_Player);
    showPlayerAverageScore(v_Player);

    return 0;
}
```





#### 3.5 stack容器



##### 3.5.1 stack基本概念



- 概念：`stack`是一种`先进后出(First In Last out : FILO)`的数据结构，它只有`一个出口`
- `栈底`存放`首个元素`，后续`其它元素`都在`栈顶`依次加入
- 栈中的元素，只有`栈顶的元素`可以`被外界使用`，也因此`不支持遍历`的行为
- `push()`入栈，`pop()`出栈，`empty()`判断栈是否为空，`size()`获取栈大小





##### 3.5.2 stack常用接口



- 功能描述：栈容器常用的对外口



- 构造函数：
  - `stack<T> stk;` 采用`模板类`实现，`stack`对象的默认构造形式
  - `stack(const stack &stk);` 拷贝构造函数
- 赋值操作：
  - `stack &operator=(const stack &stk);` 重载等号操作
- 数据存取：
  - `push(elem);` 向栈顶添加元素
  - `pop();` 移除栈顶的元素
  - `top();` 返回栈顶的元素
- 大小操作：
  - `empty();` 判断栈是否为空
  - `size()` 获取栈大小



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/20 16:13
  //
  #include <iostream>
  #include <stack>
  
  using namespace std;
  
  void demo()
  {
      stack<int> stk;
  
      stk.push(10);
      stk.push(20);
      stk.push(30);
  
      while (!stk.empty())
      {
          cout << "Stack Size: " << stk.size() << endl;
          cout << "Stack Top Element: " << stk.top() << endl;
          stk.pop();
      }
      cout << "Stack Size: " << stk.size() << endl;
      cout << "Stack Top Element: " << stk.top() << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





#### 3.6 queue容器



##### 3.6.1 queue基本概念



- 概念：`queue`是`先进先出(First In Frist Out : FIFO)`的数据结构，它有两个出口
- 队列容器只能队尾加入元素，对头删除元素
- 队列容器只有头和尾可被外界使用，因此不支持遍历行为
- 队列中进数据：入队`push`
- 队列中出数据：出队`pop`



##### 3.6.2 queue常用接口



- 功能描述：栈容器常用的对外接口



- 构造函数：
  - `queue<T> que;` 采用`模板类`实现，`queue`对象的默认构造形式
  - `queue(const queue &que);` 拷贝构造函数
- 赋值操作：
  - `queue&operator=(const queue &que);` 重载等号操作
- 数据存取：
  - `push(elem);` 向队尾添加元素
  - `pop();` 移除队头元素
  - `back();` 返回最后一个元素
  - `front();` 返回第一个元素
- 大小操作：
  - `empty();` 判断栈是否为空
  - `size()` 获取栈大小



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/20 16:37
  //
  #include <iostream>
  #include <queue>
  
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
  
  void demo()
  {
      queue<Person> q;
  
      Person p1("QQ", 10);
      Person p2("WW", 20);
      Person p3("EE", 30);
      Person p4("RR", 40);
  
      q.push(p1);
      q.push(p2);
      q.push(p3);
      q.push(p4);
  
      cout << "Queue Size: " << q.size() << endl;
  
      while (!q.empty())
      {
          cout << "Front Name: " + q.front().name + " Front Age: " << q.front().age << endl;
          cout << "Back Name: " + q.back().name + " Back Age: " << q.back().age << endl;
  
          q.pop();
      }
  
      cout << "Queue Size: " << q.size() << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





#### 3.7 list容器



##### 3.7.1 list基本概念



- 功能：将数据进行`链式`存储
- 链表：是一种`物理存储单元`上`非连续`的存储结构，数据元素的`逻辑顺序`是通过链表中的`指针链`实现的
- 链表的组成：链表由一系列`结点`组成
- 结点的组成：一个是存储数据元素的`数据域`，另一个是存储下一个结点地址的`指针域`
- STL中的链表是一个`双向循环链表`



- 优点：可以对任意位置进行快速插入或删除元素
- 缺点：
  - 链表容器遍历元素比数组慢
  - 占用空间比数组大



- STL链表的结构：`双向循环链表`
- 结点：
  - `data区`：存储数据
  - `pionter区`：(默认指向null，则是`不循环双向链表`)
    - `prev`：指向上一个结点的首地址(第一个结点默认指向最后一个结点的首地址)
    - `next`：指向下一个结点的首地址(最后一个结点默认指向第一个结点的首地址)
- 方法：
  - `push_front()`：添加一个新结点做为首结点
  - `pop_front()`：删除首结点
  - `push_back()`：添加一个新结点做为尾结点
  - `pop_back()`：删除尾结点
- 迭代器：
  - `begin()`：获得首结点的地址
  - `insert()`：获得指定的结点的地址
  - `end()`：获得尾结点的地址



- 补充：由于链表的存储方式并不是连续的内存空间，因此链表list中的迭代器只支持前移和后移，属于`双向迭代器`
- list的优点：
  - 采用动态存储分配，不会造成内存浪费和溢出
  - 链表执行插入和删除操作十分方便，修改指针即可，不需要移动大量元素
- list的缺点：
  - 链表的灵活带来的是空间(指针域)和时间(遍历)的额外消费较大
- list的重要性质：插入和删除操作都不会造成原有list容器迭代器的失效(vector中会失效)



- 总结：STL中的`lsit`和`vector`是两个常用的容器，各有优缺点





##### 3.7.2 list构造函数



- 功能描述：创建`list`容器

  | 函数原型                  |                                                |
  | ------------------------- | ---------------------------------------------- |
  | `list<T> list;`           | list采用模板类实现，对象的默认构造函数形式     |
  | `list(begin, end);`       | 构造函数将`[begin, end)`区间中的元素拷贝给自身 |
  | `list(n, elem);`          | 构造函数将`n`个`elem`拷贝给自身                |
  | `list(const list &list);` | 拷贝构造函数                                   |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/27 14:37
  //
  #include <iostream>
  #include <list>
  
  using namespace std;
  
  void printList(const list<int> &otherList)
  {
      for (list<int>::const_iterator it = otherList.begin(); it != otherList.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      list<int> list1;
      list1.push_back(1);
      list1.push_back(2);
      list1.push_back(3);
      list1.push_back(4);
  
      printList(list1);
  
      // 区间构造
      list<int> list2(list1.begin(), list1.end());
      printList(list2);
  
      // 拷贝构造
      list<int> list3(list2);
      printList(list3);
  
      // n 个 elem
      list<int> list4(4, 0);
      printList(list4);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  



##### 3.7.3 list赋值交换



- 功能描述：给`list`容器进行赋值，以及容器`list`交换

  | 函数原型                             |                                            |
  | ------------------------------------ | ------------------------------------------ |
  | `assign(begin, end);`                | 将`[begin, end)`区间中的数据拷贝赋值给自身 |
  | `assign(n, elem);`                   | `n`个`elem`拷贝赋值给自身                  |
  | `list &operator=(const list &list);` | 重载`=`操作符                              |
  | `swap(list);`                        | 将`list`与自身的元素互换                   |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/27 14:49
  //
  #include <iostream>
  #include <list>
  
  using namespace std;
  
  void printList(const list<int> &other)
  {
      for (list<int>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo1()
  {
      cout << "Demo1 >>" << endl;
      list<int> l1(4, 1);
      printList(l1);
  
      // operator=
      list<int> l2 = l1;
      printList(l2);
  
      // assign(begin, end)
      list<int> l3;
      l3.assign(l1.begin(), l1.end());
      printList(l3);
  
      // assign(n, elem)
      list<int> l4;
      l4.assign(4, 0);
      printList(l4);
  }
  
  void demo2()
  {
      cout << "Demo2 >>" << endl;
      list<int> list1(4, 0);
      list<int> list2(4, 9);
  
      cout << "Swap List Before >>" << endl;
      printList(list1);
      printList(list2);
  
      cout << "Swap List Last >>" << endl;
      list1.swap(list2);
      printList(list1);
      printList(list2);
  }
  
  int main()
  {
      demo1();
      demo2();
      return 0;
  }
  ```

  



##### 3.7.4 list大小操作



- 功能描述：对`list`容器的大小进行操作

  | 函数原型             |                                                              |
  | -------------------- | ------------------------------------------------------------ |
  | `size();`            | 返回容器中元素个数                                           |
  | `empty();`           | 判断容器是否为空                                             |
  | `resize();`          | 重新指定容器长度为`num`，容器过长以默认值填充，容器过短，删除末尾多余元素 |
  | `resize(num, elem);` | 重新指定容器长度为`num`，容器过长以`elem`填充，容器过短，删除末尾多余元素 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/27 15:06
  //
  #include <iostream>
  #include <list>
  
  using namespace std;
  
  void printList(const list<int> &other)
  {
      for (list<int>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      list<int> list1(5, 1);
  
      if (list1.empty())
      {
          cout << "List1 Is Empty" << endl;
      }
      else
      {
          cout << "List1 Size: " << list1.size() << endl;
      }
  
      list1.resize(10, 2);
      printList(list1);
  
      list1.resize(5);
      printList(list1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  



##### 3.7.5 list插入删除



- 功能描述：对`list`容器进行数据的插入和删除

  | 函数原型                   |                                                     |
  | -------------------------- | --------------------------------------------------- |
  | `push_back(elem);`         | 在容器尾部加入一个元素                              |
  | `pop_back();`              | 删除容器中最后一个元素                              |
  | `push_front(elem);`        | 在容器开头加入一个元素                              |
  | `pop_front();`             | 删除容器开头的一个元素                              |
  | `insert(pos, elem);`       | 在`pos`的位置插入`elem`元素的拷贝，返回新数据的位置 |
  | `insert(pos, n, elem);`    | 在`pos`位置插入`n`个`elem`元素，无返回值            |
  | `insert(pos, begin, end);` | 在`pos`位置插入`[begin, end)`区间的数据，无返回值   |
  | `clear();`                 | 移除容器中所有的元素                                |
  | `erase(begin, end);`       | 删除`[begin, end)`区间的数据，返回下一个数据的位置  |
  | `erase(pos);`              | 删除`pos`位置的数据，返回下一个数据的位置           |
  | `remove(elem);`            | 删除容器中所有与`elem`元素匹配的元素                |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/29 14:54
  //
  #include <iostream>
  #include <list>
  
  using namespace std;
  
  void printList(const list<int> &other)
  {
      for (list<int>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      list<int> l1;
  
      // push_back()
      for (int i = 0; i < 5; ++i)
      {
          l1.push_back(i);
      }
      printList(l1);
  
      // push_front()
      for (int i = 1; i <5; ++i)
      {
          l1.push_front(1);
      }
      printList(l1);
  
      // pop_back()
      l1.pop_back();
      printList(l1);
  
      // pop_front()
      l1.pop_front();
      printList(l1);
  
      // insert(pos, elem)
      l1.insert(l1.begin(), 0);
      printList(l1);
  
      // insert(pos, n, elem)
      l1.insert(l1.end(), 3, 5);
      printList(l1);
  
      // insert(pos, begin, end);
      list<int> l2(3, 9);
      l1.insert(l1.end(), l2.begin(), l2.end());
      printList(l1);
  
      // erase(pos)
      l1.erase(++l1.begin());
      printList(l1);
  
      // remove(elem)
      l1.remove(5);
      printList(l1);
  
      // clear()
      l1.clear();
      l1.push_front(9);
      printList(l1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  



##### 3.7.6 list数据存取



- 功能描述：对`list`容器数据进行存取

  | 函数原型   |                  |
  | ---------- | ---------------- |
  | `front();` | 返回第一个元素   |
  | `back();`  | 返回最后一个元素 |



- 补充：`list<int> l1;`
  - `l1[0];`
  - `l1.at(0);`
  - 原因：`list`容器本质是`链表`，空间不连续，无法使用数组下标的方式获得数值，`迭代器`也不支持随机访问



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/29 15:33
  //
  #include <iostream>
  #include <list>
  
  using namespace std;
  
  void printList(const list<int> &other)
  {
      for (list<int>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      list<int> l1;
  
      for (int i = 0; i < 5; ++i)
      {
          l1.push_back(i);
      }
      printList(l1);
  
      cout << "l1 first>> " << l1.front() << endl;
      cout << "l1 back>> " << l1.back() << endl;
  
      // 双向访问
      list<int>::iterator it1 = ++l1.begin();
      list<int>::iterator it2 = --l1.end();
      l1.erase(it1, it2);
      printList(l1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.7.7 list容器排序



- 功能描述：将容器中的元素反转，以及元素排序

  | 函数原型     |          |
  | ------------ | -------- |
  | `reverse();` | 反转链表 |
  | `sort();`    | 链表排序 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/29 15:54
  //
  #include <iostream>
  #include <list>
  
  using namespace std;
  
  void printList(const list<int> &other)
  {
      for (list<int>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  bool upSort(int &list1, int &list2)
  {
      return list1 > list2;
  }
  
  void demo()
  {
      list<int> list1;
      for (int i = 0; i < 10; ++i)
      {
          list1.push_back(i);
      }
      printList(list1);
  
      // reverse()
      list1.reverse();
      printList(list1);
  
      // sort() -- 默认从小到大
      list1.sort();
      printList(list1);
  
      // sort() -- 改为从大到小
      list1.sort(upSort);
      printList(list1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





#### 3.8 案例-自定义数据排序



- 案例描述：将`Person`自定义数据类型进行排序，`Person`中属性有：姓名，年龄，身高

- 排序规则：按照年龄进行升序`(年龄小的放在前面)`，如果年龄相同，按照身高进行降序`(身高低的放在前面)`

- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/29 16:15
  //
  #include <iostream>
  #include <list>
  
  using namespace std;
  
  class Person
  {
  public:
      string name;
      int age;
      int height;
  
      Person(const string &name, int age, int height)
      {
          this->name = name;
          this->age = age;
          this->height = height;
      }
  };
  
  // Debug
  void printPersonList(const list<Person> &other)
  {
      for (list<Person>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << "Name:" << it->name << " Age:" << it->age << " Height:" << it->height << endl;
      }
      cout << endl;
  }
  
  bool sortRule(Person &person1, Person &person2)
  {
      if (person1.age == person2.age)
      {
          return person1.height < person2.height;
      }
      else
      {
          return person1.age < person2.age;
      }
  }
  
  list<Person> createPersonList()
  {
      list<Person> l1;
  
      Person person1("QQ", 10, 160);
      Person person2("WW", 20, 150);
      Person person3("EE", 20, 170);
      Person person4("RR", 70, 175);
      Person person5("TT", 30, 180);
  
      l1.push_back(person1);
      l1.push_back(person2);
      l1.push_back(person3);
      l1.push_back(person4);
      l1.push_back(person5);
  
      printPersonList(l1);
  
      return l1;
  }
  
  void upSort(list<Person> &other)
  {
      cout << "UpSort>> By Age And Height" << endl;
      other.sort(sortRule);
      printPersonList(other);
  }
  
  int main()
  {
      list<Person> l1 = createPersonList();
      upSort(l1);
  
      return 0;
  }
  ```





#### 3.9 set/multiset容器



##### 3.9.1 set基本概念



- 简介：所有元素被插入时，容器都会进行一次自动排序
- 本质：`set/multiset`属于`关联容器`，底层结构是`二叉树`实现
- `set`和`multiset`的区别：
  - `set`不允许容器中有重复的元素
  - `multiset`允许容器中有重复的元素





##### 3.9.2 set构造赋值



- 功能描述：创建`set`容器并赋值

  | 构造                  |              |
  | --------------------- | ------------ |
  | `set<T> st;`          | 默认构造函数 |
  | `set(const set &st);` | 拷贝构造函数 |

  | 赋值                             |                |
  | -------------------------------- | -------------- |
  | `set &operator=(const set &st);` | 重载等号操作符 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/30 14:49
  //
  #include <iostream>
  #include <set>
  
  using namespace std;
  
  void printSet(const set<int> &st)
  {
      for (set<int>::const_iterator it = st.begin(); it != st.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      set<int> st1;
      // set 只能用 insert插入数据，且不插入重复数据
      // set 会自动排序插入的数据
      st1.insert(1);
      st1.insert(1);
      st1.insert(1);
      st1.insert(4);
      st1.insert(3);
      st1.insert(2);
      printSet(st1);
  
      // 默认构造
      set<int> st2(st1);
      printSet(st2);
  
      // 赋值拷贝构造
      set<int> st3 = st1;
      printSet(st3);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```



- 总结：`set` 只能用 `insert`插入数据，且不插入重复数据





##### 3.9.3 set大小交换



- 功能描述：统计`set`容器大小，以及交换`set`容器

  | 函数原型    |                    |
  | ----------- | ------------------ |
  | `size();`   | 返回容器中元素个数 |
  | `empty();`  | 判断容器是否为空   |
  | `swap(st);` | 交换两个容器的元素 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/30 15:00
  //
  #include <iostream>
  #include <set>
  
  using namespace std;
  
  void printSet(const set<int> &st)
  {
      for (set<int>::const_iterator it = st.begin(); it != st.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      set<int> st1;
      for (int i = 0; i < 5; ++i)
      {
          st1.insert(i);
      }
  
      if (st1.empty())
      {
          cout << "Set1 Is Empty" << endl;
      }
      else
      {
          cout << "Set1>>";
          printSet(st1);
          cout << "Set1 Size>>" << st1.size() << endl;
      }
  }
  
  void demo2()
  {
      set<int> st1;
      for (int i = 0; i < 5; ++i)
      {
          st1.insert(i);
      }
  
      set<int> st2;
      for (int i = 9; i >= 5 ; --i)
      {
          st2.insert(i);
      }
  
      cout << "Swap Before>>" << endl;
      cout << "Set1>>";
      printSet(st1);
      cout << "Set2>>";
      printSet(st2);
  
      cout << "Swap Last>>" << endl;
      st1.swap(st2);
      cout << "Set1>>";
      printSet(st1);
      cout << "Set2>>";
      printSet(st2);
  }
  
  int main()
  {
      demo();
      demo2();
      return 0;
  }
  ```





##### 3.9.4 set插入删除



- 功能描述：`set`容器进行插入和删除数据

  | 函数原型             |                                                        |
  | -------------------- | ------------------------------------------------------ |
  | `insert(elem);`      | 在容器中插入元素                                       |
  | `clear();`           | 清除所有元素                                           |
  | `erase(pos);`        | 删除迭代器`pos`所指的元素，返回下一个元素的迭代器      |
  | `erase(begin, end);` | 删除`[begin, end)`区间内的元素，返回下一个元素的迭代器 |
  | `erase(elem);`       | 删除容器中的所有`elem`元素                             |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/30 15:14
  //
  #include <iostream>
  #include <set>
  
  using namespace std;
  
  void printSet(const set<int> &st)
  {
      for (set<int>::const_iterator it = st.begin(); it != st.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      set<int> st1;
      for (int i = 0; i <= 9; ++i)
      {
          st1.insert(i);
      }
      printSet(st1);
  
      // erase(pos)
      st1.erase(st1.begin());
      printSet(st1);
  
      // erase(elem)
      st1.erase(9);
      printSet(st1);
  
      // erase(begin, end);
      st1.erase(++st1.begin(), --st1.end());
      printSet(st1);
  
      // clear()
      st1.clear();
      st1.insert(0);
      printSet(st1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.9.5 set查找统计



- 功能描述：对容器内的元素进行查找和统计

  | 函数原型      |                                                              |
  | ------------- | ------------------------------------------------------------ |
  | `find(key);`  | 查找`key`是否存在，存在则返回该元素的迭代器，不存在则返回`set.end();` |
  | `count(key);` | 统计`key`的个数                                              |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/30 15:25
  //
  #include <iostream>
  #include <set>
  
  using namespace std;
  
  void printSet(const set<int> &st)
  {
      for (set<int>::const_iterator it = st.begin(); it != st.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      set<int> st1;
      for (int i = 0; i <= 9; ++i)
      {
          st1.insert(i);
      }
      printSet(st1);
  
      // find(key)
      set<int>::const_iterator it = st1.find(3);
      if (it != st1.end())
      {
          cout << "Set1 Find>>" << *it << endl;
      }
      else
      {
          cout << "Set1 Not Find" << endl;
      }
  
      // count(key)
      set<int> st2;
      for (int i = 0; i <= 9; ++i)
      {
          st2.insert(0);
      }
      printSet(st2);
      cout << "Set2 Count 0>>" << st2.count(0) << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.9.6 set/multiset区别



- 区别：
  1. `set`不可以插入重复的元素，`multiset`可以
  2. `set`插入数据同时返回插入结果，表示插入成功
  3. `multiset`不会检查插入数据，所以可以重复插入



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/30 15:58
  //
  #include <iostream>
  #include <set>
  
  using namespace std;
  
  void printMultiSet(const multiset<int> &multiset1)
  {
      for (multiset<int>::const_iterator it = multiset1.begin(); it != multiset1.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      set<int> set1;
      pair<set<int>::iterator, bool> pairResult;
      pairResult = set1.insert(1);
      cout << "First>> " << endl;
      if (pairResult.second)
      {
          cout << "Set1 Insert " << "<" << *pairResult.first << ">" << " Succeed" << endl;
      }
      else
      {
          cout << "Set1 Insert " << "<" << *pairResult.first << ">" << " Error" << endl;
      }
  
      pairResult = set1.insert(1);
      cout << "Second>> " << endl;
      if (pairResult.second)
      {
          cout << "Set1 Insert " << "<" << *pairResult.first << ">" << " Succeed" << endl;
      }
      else
      {
          cout << "Set1 Insert " << "<" << *pairResult.first << ">" << " Error" << endl;
      }
  }
  
  void demo2()
  {
      multiset<int> multiset1;
      for (int i = 0; i <= 9; ++i)
      {
          multiset1.insert(9);
      }
      printMultiSet(multiset1);
  }
  
  int main()
  {
      demo();
      demo2();
      return 0;
  }
  ```





##### 3.9.7 pair对组创建



- 功能描述：成对出现的数据组，利用对组可以返回两个数据
- 创建方式：
  1. `pair<type1, type2> p(value1, value2);`
  2. `pair<type1, type2> p = make_pair(value1, value2);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/30 16:18
  //
  #include <iostream>
  
  using namespace std;
  
  void printPair(const pair<string, int> &other)
  {
      cout << "Name: " << other.first << " Age: " << other.second << endl;
  }
  
  void demo()
  {
      pair<string, int> pair1("FF", 22);
      printPair(pair1);
  
      pair<string, int> pair2 = make_pair("QQ", 20);
      printPair(pair2);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.9.8 set容器排序



- 默认排序：`set`容器`默认`排序是`从小到大`
- 排序目标：掌握自定义排序规则
- 使用方法：利用`仿函数`，改变排序规则



- 代码示例一：`set`存放内置数据类型，从大到小排序

  ```c++
  //
  // Created by FHang on 2021/10/30 16:30
  //
  #include <iostream>
  #include <set>
  
  using namespace std;
  
  class DownSort
  {
  public:
      bool operator()(int value1, int value2)
      {
          return value1 > value2;
      }
  };
  
  void printSet(const set<int> &other)
  {
      for (set<int>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void printSet(const set<int, DownSort> &other)
  {
      for (set<int>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << *it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      set<int> set1;
      for (int i = 0; i <= 9; ++i)
      {
          set1.insert(i);
      }
      printSet(set1);
  
      set<int, DownSort> set2;
      for (int i = 0; i <= 9; ++i)
      {
          set2.insert(i);
      }
      printSet(set2);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```



- 代码示例二：自定义数据类型，从大到小排序

  ```c++
  //
  // Created by FHang on 2021/10/30 16:42
  //
  #include <iostream>
  #include <set>
  
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
  
  class DownSort
  {
  public:
      bool operator()(const Person &p1, const Person &p2)
      {
          return p1.age > p2.age;
      }
  };
  
  void printSet(const set<Person, DownSort> &other)
  {
      for (set<Person, DownSort>::iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << "Name: " << it->name << " Age: " << it->age << endl;
      }
      cout << endl;
  }
  
  void demo()
  {
      Person p1("QQ", 10);
      Person p2("WW", 40);
      Person p3("EE", 20);
      Person p4("RR", 30);
  
      set<Person, DownSort> set1;
      set1.insert(p1);
      set1.insert(p2);
      set1.insert(p3);
      set1.insert(p4);
  
      printSet(set1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  





#### 3.10 map/multimap容器



##### 3.10.1 map基本概念



- 简介：
  - `map`中所有的元素都是`pair`
  - `pair`中第一个元素为`key(键值)`，起到索引作用，第二元素为`value(实值)`
  - 所有元素都会根据元素的键值自动排序
- 本质：
  - `map/multimap`属于`关联式容器`，底层结构是用`二叉树`实现
- 优点：
  - 可以根据`key`值快速找到`value`值
- `map/multimap`的区别：
  - `map`不允许有重复的元素
  - `multimap`允许重复的元素





##### 3.10.2 map构造赋值



- 功能描述：对map容器进行构造和赋值操作

  | 构造                  |                   |
  | --------------------- | ----------------- |
  | `map<T1, T2> mp;`     | `map`默认构造函数 |
  | `map(const map &mp);` | 拷贝构造函数      |

  | 赋值                             |                |
  | -------------------------------- | -------------- |
  | `map &operator=(const map &mp);` | 重载等号操作符 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/10/31 16:34
  //
  #include <iostream>
  #include <map>
  
  using namespace std;
  
  void printMap(const map<int, string> &other)
  {
      for (map<int, string>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << "ID: " << it->first << " Name: " << it->second << endl;
      }
      cout << endl;
  }
  
  void demo()
  {
      map<int, string> map1;
  
      map1.insert(pair<int, string>(1, "QQ"));
      map1.insert(pair<int, string>(4, "WW"));
      map1.insert(pair<int, string>(2, "EE"));
      map1.insert(pair<int, string>(5, "RR"));
      map1.insert(pair<int, string>(3, "TT"));
  
      printMap(map1);
  
      // 拷贝构造
      map<int, string> map2(map1);
      printMap(map2);
  
      // 赋值
      map<int, string> map3 = map1;
      printMap(map3);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  



##### 3.10.3 map大小交换



- 功能描述：统计`map`容器大小以及交换`map`容器

  | 函数原型    |                        |
  | ----------- | ---------------------- |
  | `size();`   | 返回容器中的元素的数目 |
  | `empty();`  | 判断容器是否为空       |
  | `swap(st);` | 交换两个集合容器       |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/13 15:12
  //
  #include <iostream>
  #include <map>
  
  using namespace std;
  
  void printMap(const map<int, int> &other)
  {
      for (map<int, int>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << "Key: " << it->first << " Value: " << it->second << endl;
      }
      cout << endl;
  }
  
  void demo()
  {
      map<int, int> map1;
      map1.insert(pair<int, int>(1, 10));
      map1.insert(pair<int, int>(2, 20));
      map1.insert(pair<int, int>(3, 30));
  
      if (map1.empty())
      {
          cout << "Map1 Is Empty" << endl;
      }
      else
      {
          cout << "Map1 Size " << map1.size() << endl;
      }
  
      map<int, int> map2;
      map2.insert(pair<int, int>(4, 40));
      map2.insert(pair<int, int>(5, 50));
      map2.insert(pair<int, int>(6, 60));
  
      map1.swap(map2);
  
      printMap(map1);
      printMap(map2);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.10.4 map插入删除



- 功能描述：`map`容器进行插入和删除操作

  | 函数原型                                  |                                                       |
  | ----------------------------------------- | ----------------------------------------------------- |
  | `insert(pair<type1, type2>(key, value));` | 在容器中插入元素                                      |
  | `clear();`                                | 清空容器                                              |
  | `erase(pos);`                             | 删除`pos`迭代器所指位置的元素，返回下一个元素的迭代器 |
  | `erase(begin, end);`                      | 删除`[begin, end]`之间的元素，返回下一个元素的迭代器  |
  | `erase(key);`                             | 删除`key`指定索引位置的元素                           |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/13 15:31
  //
  #include <iostream>
  #include <map>
  
  using namespace std;
  
  void printMap(const map<int, int> &other)
  {
      for (map<int, int>::const_iterator it = other.begin(); it != other.end(); ++it)
      {
          cout << "Key: " << it->first << " Value: " << it->second << endl;
      }
      cout << endl;
  }
  
  void demo()
  {
      map<int, int> map1;
      map1.insert(pair<int, int>(1, 10));
      map1.insert(make_pair(2, 20));
      map1.insert(map<int, int>::value_type(3, 30));
  
      // 不建议使用该方法插入，但可以通过这个方法利用 key 访问 value
      map1[4] = 40;
  
      map1.insert(pair<int, int>(5, 50));
      map1.insert(pair<int, int>(6, 60));
      map1.insert(pair<int, int>(7, 70));
  
      // map[5]在容器中不存在，所以默认直接在容器中添加了一个，默认value为0
      cout << map1[8] << endl;
  
      printMap(map1);
  
      map1.erase(map1.begin());
      printMap(map1);
  
      map1.erase(8);
      printMap(map1);
  
      map1.erase(++map1.begin(), --map1.end());
      printMap(map1);
  
      map1.clear();
      printMap(map1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





##### 3.10.5 map查找统计



- 功能描述：对`map`容器进行查找数据以及数据统计

  | 函数原型      |                                                              |
  | ------------- | ------------------------------------------------------------ |
  | `find(key);`  | 查找`key`是否存在，若存在返回`key`键的元素迭代器；不存在，返回`set.end();` |
  | `count(key);` | 统计`key`的元素的个数                                        |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/13 16:34
  //
  #include <iostream>
  #include <map>
  
  using namespace std;
  
  void printMap(const map<int, int> &other)
  {
      for (map<int, int>::const_iterator it = other.cbegin(); it != other.cend(); ++it)
      {
          cout << "Key: " << it->first << " Value: " << it->second << endl;
      }
      cout << endl;
  }
  
  void demo()
  {
      map<int, int> map1;
      map1.insert(pair<int, int>(1, 10));
      map1.insert(pair<int, int>(2, 20));
      map1.insert(pair<int, int>(3, 30));
  
      map<int, int>::const_iterator itPos = map1.find(3);
  
      // map.end() 返回的是迭代器所指位置的 “下一个”迭代器的位置
      if (itPos != map1.cend())
      {
          cout << "Key: " << itPos->first << " Value: " << itPos->second << endl;
          cout << "End Key: " << map1.cend()->first << " End Value: " << map1.cend()->second << endl;
      }
      else
      {
          cout << "Can Find Key" << endl;
      }
  
      map1.insert(pair<int, int>(3, 90));
      int count = map1.count(3);
      cout << "End Key: " << map1.cend()->first << " End Value: " << map1.cend()->second << endl;
      cout << count << endl;
  
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  



##### 3.10.6 map容器排序



- 目标：
  1. `map`容器默认排序规则，按照`key`值进行，从小到大的排序
  2. 掌握自定义排序规则



- 主要技术点：利用仿函数，改变排序规则



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/13 18:08
  //
  #include <iostream>
  #include <map>
  
  using namespace std;
  
  class DownSort
  {
  public:
      bool operator()(int value1, int value2)
      {
          return value1 > value2;
      }
  };
  
  void printMap(const map<int, int, DownSort> &other)
  {
      for (map<int, int>::const_iterator it = other.cbegin(); it != other.cend(); ++it)
      {
          cout << "Key: " << it->first << " Value: " << it->second << endl;
      }
      cout << endl;
  }
  
  void demo()
  {
      map<int, int, DownSort> map1;
      map1.insert(pair<int, int>(1, 10));
      map1.insert(pair<int, int>(2, 20));
      map1.insert(pair<int, int>(3, 30));
      map1.insert(pair<int, int>(4, 40));
      map1.insert(pair<int, int>(5, 50));
  
      printMap(map1);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```







#### 3.11 案例-员工分组



##### 3.11.1 案例描述

- 10名员工（ABCDEFGHIJ），需要分配部门
- 员工信息：姓名，工资
- 部门：策划，美术，研发
- 随机给10名员工分配工资和部门
- 通过`multimap`进行信息插入：`key`部门编号，`value`员工
- 分部门显示员工信息



##### 3.11.2 实现步骤

1. 创建10个员工对象，存入`vector`容器中
2. 遍历`vector`，取出每个员工，进行随机分组
3. 分组后，将`key`部门编号，`value`员工，存放到`multimap`
4. 分部门显示员工信息



##### 3.11.3 案例代码

```c++
//
// Created by FHang on 2021/11/13 19:05
//
#include <iostream>
#include <vector>
#include <map>
#include <ctime>

using namespace std;

#define PLAN 0
#define ART 1
#define RD 2

class Staff
{
public:
    string staff_Name;
    int staff_Salary;
};

void printVector(const vector<Staff> &other)
{
    for (vector<Staff>::const_iterator it = other.cbegin(); it != other.cend(); ++it)
    {
        cout << "Name: " << it->staff_Name << " Salary: " << it->staff_Salary << endl;
    }
    cout << endl;
}

void creatStaff(vector<Staff> &v_Staff)
{
    string staffNameSeed = "ABCDEFGHIJ";
    for (int i = 0; i < 10; ++i)
    {
        Staff staff;
        staff.staff_Name = "Staff_";
        staff.staff_Name += staffNameSeed[i];
        staff.staff_Salary = rand() % 10000 + 10000;

        v_Staff.push_back(staff);
    }
}

void setStaffGroup(vector<Staff> &v_Staff, multimap<int, Staff> &map_Depart)
{
    for (vector<Staff>::iterator it = v_Staff.begin(); it != v_Staff.end(); ++it)
    {
        int depart_ID = rand() % 3;
        map_Depart.insert(pair<int, Staff>(depart_ID, *it));
    }
}

void checkStaffByGroup(multimap<int, Staff>::const_iterator &itPos, const multimap<int, Staff> &map_Depart, const int count)
{
    for (int index = 0; itPos != map_Depart.cend() && index < count; ++itPos, ++index)
    {
        cout << "Name: " << itPos->second.staff_Name << " Salary: " << itPos->second.staff_Salary << endl;
    }
    cout << endl;
}

void showStaffInfoByGroup(multimap<int, Staff> &map_Depart)
{
    int countStaff_PLAN = map_Depart.count(PLAN);
    int countStaff_ART = map_Depart.count(ART);
    int countStaff_RD = map_Depart.count(RD);

    multimap<int, Staff>::const_iterator itPos_Plan = map_Depart.find(PLAN);
    multimap<int, Staff>::const_iterator itPos_Art = map_Depart.find(ART);
    multimap<int, Staff>::const_iterator itPos_RD = map_Depart.find(RD);

    cout << "Plan Department>>" << endl;
    checkStaffByGroup(itPos_Plan, map_Depart, countStaff_PLAN);

    cout << "Art Department>>" << endl;
    checkStaffByGroup(itPos_Art, map_Depart, countStaff_ART);

    cout << "R&D Department>>" << endl;
    checkStaffByGroup(itPos_RD, map_Depart, countStaff_RD);
}

int main()
{
    srand((unsigned int)time(NULL));

    vector<Staff> v_Staff;
    creatStaff(v_Staff);
    printVector(v_Staff);

    multimap<int, Staff> map_Depart;
    setStaffGroup(v_Staff, map_Depart);

    showStaffInfoByGroup(map_Depart);

    return 0;
}
```




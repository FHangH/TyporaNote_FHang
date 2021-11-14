# C++_STL函数对象



### 1. 函数对象



#### 1.1 函数对象概念



- 概念：
  - 重载`函数调用操作符`的类，其对象常称为`函数对象`
  - `函数对象`使用重载的`()`时，行为类似函数调用，也叫`仿函数`
- 本质：
  - 函数对象(仿函数)是一个类，不是一个函数





#### 1.2 函数对象使用



- 特点：
  - 函数对象在使用时，可以像普通函数那样调用，可以有参数，可以有返回值
  - 函数对象超出普通函数的概念，函数对象可以有自己的状态
  - 函数对象可以作为参数传递



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/14 15:17
  //
  #include <iostream>
  
  using namespace std;
  
  // 函数对象在使用时，可以像普通函数那样调用，可以有参数，可以有返回值
  class Fh_Add
  {
  public:
      int operator()(int value1, int value2)
      {
          return value1 + value2;
      }
  };
  
  // 函数对象超出普通函数的概念，函数对象可以有自己的状态
  class Fh_Print
  {
  public:
      int transferCount;
  
      Fh_Print()
      {
          this->transferCount = 0;
      }
  
      void operator()(const string &printStr)
      {
          cout << printStr << endl;
          this->transferCount++;
      }
  };
  
  void demo1()
  {
      Fh_Add fhAdd;
      cout << "Fh_Add>> " << fhAdd(1, 2) << endl;
  
      cout << endl;
  }
  
  void demo2()
  {
      Fh_Print fhPrint;
      fhPrint("Hello World");
      fhPrint("Hello World");
      fhPrint("Hello World");
      cout << "Fh_Print Transfer Count>> " << fhPrint.transferCount << endl;
  
      cout << endl;
  }
  
  // 函数对象可以作为参数传递
  void doPrint(Fh_Print &fhPrint, const string &printStr)
  {
      fhPrint(printStr);
  }
  
  void demo3()
  {
      Fh_Print fhPrint;
      doPrint(fhPrint, "Hello World");
  
      cout << endl;
  }
  
  int main()
  {
      demo1();
      demo2();
      demo3();
      return 0;
  }
  ```





### 2. 谓词



#### 2.1 谓词基本概念



- 概念：
  - 返回`bool`类型的仿函数称为`谓词`
  - 如果`operator()`接受一个参数，为`一元谓词`
  - 如果`operator()`接受二个参数，为`二元谓词`



#### 2.2 一元谓词



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/14 16:09
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  class DownSort
  {
  public:
      bool operator()(const int &value1, const int &value2)
      {
          return value1 > value2;
      }
  };
  
  void printVector(const vector<int> &other)
  {
      for (int it : other)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v;
      for (int i = 1; i < 6; ++i)
      {
          v.push_back(i);
      }
      printVector(v);
  
      sort(v.begin(), v.end(), DownSort());
      printVector(v);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





### 3. 内建函数对象



#### 3.1 内建函数对象意义



- 概念：
  - STL内建一些函数对象
- 分类：
  - 算术仿函数
  - 关系仿函数
  - 逻辑仿函数
- 用法：
  - 这些仿函数所产生的对象，用法和一般函数完全相同
  - 使用内建函数对象，需要引入头文件`#include<functional>`





#### 3.2 算术仿函数

 

- 功能描述：

  - 实现四则运算
  - 其中`negate`是`一元运算`，其它的都是`二元运算`

  | 仿函数原型                          |            |
  | ----------------------------------- | ---------- |
  | `template<class T> T plus<T>`       | 加法仿函数 |
  | `template<class T> T minus<T>`      | 减法仿函数 |
  | `template<class T> T multiplies<T>` | 乘法仿函数 |
  | `template<class T> T divides<T>`    | 除法仿函数 |
  | `template<class T> T modulus<T>`    | 取模仿函数 |
  | `template<class T> T negate<T>`     | 取反仿函数 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/14 16:45
  //
  #include <iostream>
  #include <functional>
  
  using namespace std;
  
  void demo1()
  {
      negate<> negate1;
      cout << negate1(10) << endl;
  }
  
  void demo2()
  {
      plus<> plus1;
      cout << plus1(10, 20) << endl;
  }
  
  int main()
  {
      demo1();
      demo2();
      return 0;
  }
  ```





#### 3.3 关系仿函数



- 功能描述：实现关系对比

  | 仿函数原型                                |          |
  | ----------------------------------------- | -------- |
  | `template<class T> bool equal_to<T>`      | 等于     |
  | `template<class T> bool not_equal_to<T>`  | 不等于   |
  | `template<class T> bool greater<T>`       | 大于     |
  | `template<class T> bool greater_equal<T>` | 大于等于 |
  | `template<class T> bool less<T>`          | 小于     |
  | `template<class T> bool less_equal<T>`    | 小于等于 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/14 16:58
  //
  #include <iostream>
  #include <functional>
  #include <algorithm>
  #include <vector>
  
  using namespace std;
  
  void printVector(const vector<int> &v)
  {
      for (int it : v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v;
  
      for (int i = 1; i < 6; ++i)
      {
          v.push_back(i);
      }
  
      printVector(v);
  
      sort(v.begin(),  v.end(), greater<>());
      printVector(v);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





#### 3.4 逻辑仿函数



- 功能描述：实现逻辑运算

  | 函数原型                                |        |
  | --------------------------------------- | ------ |
  | `template<class T> bool logical_and<T>` | 逻辑与 |
  | `template<class T> bool logical_or<T>`  | 逻辑或 |
  | `template<class T> bool logical_not<T>` | 逻辑非 |



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/14 17:43
  //
  #include <iostream>
  #include <functional>
  #include <algorithm>
  #include <vector>
  #include <ctime>
  
  using namespace std;
  
  void printVector(const vector<bool> &v)
  {
      for (bool it : v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<bool> v1;
      for (int i = 1; i < 5; ++i)
      {
          v1.push_back(rand()%2);
      }
  
      printVector(v1);
  
      vector<bool> v2;
      v2.resize(v1.size());
  
      transform(v1.begin(), v1.end(), v2.begin(), logical_not<bool>());
      printVector(v2);
  }
  
  int main()
  {
      srand((unsigned int)time(NULL));
      demo();
      return 0;
  }
  ```

  
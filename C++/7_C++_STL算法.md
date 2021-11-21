# C++_STL算法



概述：

- 算法主要是头文件`<algorithm>` `<functional>` `<numeric>` 组成
- `<algorithm>` 是所有`STL`头文件中最大的一个，范围涉及到比较、交换、查找、遍历操作、复制、修改等到
- `<numeric>` 体积很小，包括几个序列上面进行简单数学运算和模板函数
- `<functional>` 定义了一些类模板，用以声明函数对象



### 1. 常用遍历算法



| 算法简介    |                      |
| ----------- | -------------------- |
| `for_each`  | 遍历容器             |
| `transform` | 搬运容器到另一个容器 |





#### 1.1 for_each



- 功能描述：实现遍历容器
- 函数原型：
  - `for_each(iterator begin, iterator end, _functional);`
  - `_functional` 函数或函数对象



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/20 13:48
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  class PrintVector_Class
  {
  public:
      void operator()(int &value)
      {
          cout << value << " ";
      }
  };
  
  void printVector_func(int &value)
  {
      cout << value << " ";
  }
  
  void demo()
  {
      vector<int> v;
      v.reserve(5);
  
      for (int i = 0; i < 5; ++i)
      {
          v.push_back(i);
      }
  
      for_each(v.begin(), v.end(), printVector_func);
      cout << endl;
  
      for_each(v.begin(), v.end(), PrintVector_Class());
      cout << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





#### 1.2 transform



- 功能描述：搬运容器到另一个容器中
- `transform(iterator begin_1, iterator end_1, iterator iterator_2, _functional);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/20 14:29
  //
  #include <iostream>
  #include <algorithm>
  #include <vector>
  
  using namespace std;
  
  class TransVector
  {
  public:
      int operator()(int &value)
      {
          return value;
      }
  };
  
  int transVector_Func(int &value)
  {
      return value;
  }
  
  void printVector(const vector<int> &v)
  {
      for (int it: v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v1;
      vector<int> v2;
  
      v1.reserve(5);
      for (int i = 0; i < 5; ++i)
      {
          v1.push_back(i);
      }
  
      v2.resize(v1.size());
  
      // transform(v1.begin(), v1.end(), v2.begin(), TransVector());
      transform(v1.begin(), v1.end(), v2.begin(), transVector_Func);
      printVector(v2);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```







### 2. 常用查找算法



| 算法简介        |                    |
| --------------- | ------------------ |
| `find`          | 查找元素           |
| `find_if`       | 按条件查找元素     |
| `adjacent_find` | 查找相邻重复的元素 |
| `binary_search` | 二分查找法         |
| `count`         | 统计元素个数       |
| `count_if`      | 按条件统计元素个数 |





#### 2.1 find



- 功能描述：查找指定元素，找到返回指定元素的迭代器，找不到返回结束迭代器`end()`
- 函数原型：`find(iterator begin, iterator end, value);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/20 14:52
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  class Person
  {
  public:
      string name;
      int age;
  
      Person(const string &name, const int &age)
      {
          this->name = name;
          this->age = age;
      }
  
      // find 底层无法比较 自定义数据类型，所以要 重载==
      // find 底层是直接 解析元素迭代器去比较要 查找的元素值，所以 自定义类型的数据，无法直接比较
      bool operator==(const Person &person)
      {
          if (this->name == person.name && this->age == person.age)
          {
              return true;
          }
          else
          {
              return false;
          }
      }
  };
  
  void demo()
  {
      vector<Person> v;
  
      Person p1("QQ", 12);
      Person p2("WW", 12);
      Person p3("EE", 12);
      Person p4("RR", 12);
  
      v.push_back(p1);
      v.push_back(p2);
      v.push_back(p3);
      v.push_back(p4);
  
      vector<Person>::iterator it = find(v.begin(), v.end(), p2);
      if (it == v.cend())
      {
          cout << "No Find" << endl;
      }
      else
      {
          cout << it->name << " = " << it->age << endl;
      }
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```



- 总结：
  1. `find` 底层无法比较`自定义`数据类型，所以要`重载==`
  2. `find` 底层是直接`解析元素迭代器`去比较要查找的元素值，所以自定义类型的数据，无法直接比较





#### 2.2 find_if



- 功能描述：按条件查找
- 函数原型：`find_if(iterator begin, iterator end, _Pred);`
- `_Pred`：函数或谓词(返回`bool`类型)



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/20 15:42
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  class Person
  {
  public:
      string name;
      int age;
  
      Person(const string &name, const int &age)
      {
          this->name = name;
          this->age = age;
      }
  };
  
  class FindIF_MoreThan_3
  {
  public:
      bool operator()(const int &value)
      {
          return value > 3;
      }
  };
  
  class FindIF_MoreThan_Age_8
  {
  public:
      bool operator()(const Person &person)
      {
          return person.age > 8;
      }
  };
  
  class UpSort_Age
  {
  public:
      bool operator()(Person &person1, Person &person2)
      {
          return person1.age < person2.age;
      }
  };
  
  void printVector(const vector<Person> &v)
  {
      for (vector<Person>::const_iterator it = v.cbegin(); it != v.cend(); ++it)
      {
          cout << it->name << " : " << it->age << endl;
      }
  }
  
  void demo1()
  {
      vector<int> v;
      for (int i = 0; i <= 5; ++i)
      {
          v.push_back(i);
      }
  
      vector<int>::iterator it = find_if(v.begin(), v.end(), FindIF_MoreThan_3());
      cout << "Find: " << *it << endl;
  }
  
  void demo2()
  {
      vector<Person> v;
      Person p1("QQ", 13);
      Person p2("WW", 8);
      Person p3("EE", 10);
      Person p4("RR", 12);
  
      v.push_back(p1);
      v.push_back(p2);
      v.push_back(p3);
      v.push_back(p4);
  
      sort(v.begin(), v.end(), UpSort_Age());
      printVector(v);
  
      vector<Person>::iterator it = find_if(v.begin(),  v.end(), FindIF_MoreThan_Age_8());
      cout << "Find: " << it->name << " = " << it->age << endl;
  }
  
  int main()
  {
      demo1();
      demo2();
      return 0;
  }
  ```





#### 2.3 adjacent_find



- 功能描述：查找相邻重复元素
- 函数原型：`adjacent_find(iterator begin, iterator end);`
- 返回相邻元素的第一个位置的迭代器



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/20 16:38
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  void demo()
  {
      vector<int> v;
  
      v.push_back(0);
      v.push_back(1);
      v.push_back(0);
      v.push_back(2);
      v.push_back(2);
  
      vector<int>::iterator it = adjacent_find(v.begin(),  v.end());
      cout << "Find: " << *it << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





#### 2.4 binary_search



- 功能描述：查找指定元素是否存在
- 函数原型：`bool binary_search(iterator begin, iterator end, value);`
- 查找指定元素，找到返回 `ture` 否则返回 `false`
- 不用于`无序序列`(就是容器内的元素不是有序排列的)



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/20 16:51
  //
  #include <iostream>
  #include <algorithm>
  #include <vector>
  
  using namespace std;
  
  void demo()
  {
      vector<int> v;
      for (int i = 0; i <= 5; ++i)
      {
          v.push_back(i);
      }
  
      bool isSearch = binary_search(v.begin(),  v.end(), 5);
  
      string searchRet = isSearch ? "true" : "false";
  
      if (isSearch)
      {
          cout << "Search: " << searchRet << endl;
      }
      else
      {
          cout << "Search: " << searchRet << endl;
      }
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```



- 总结：`binary_search()`效率很高，只用于`有序序列`





#### 2.5 count



- 功能描述：统计元素个数
- 函数原型：`count(iterator begin, iterator end, value);`
- 返回`int`类型



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 13:02
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  #include <ctime>
  
  using namespace std;
  
  class Person
  {
  public:
      string name;
      int age;
  
      Person(const string &name, const int &age)
      {
          this->name = name;
          this->age = age;
      }
  
      bool operator==(const Person &p)
      {
          if (this->age == p.age)
          {
              return true;
          }
          else
          {
              return false;
          }
      }
  };
  
  void printVector(const vector<int> &v)
  {
      for (int it : v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void printVector(const vector<Person> &p)
  {
      for (Person it : p)
      {
          cout << it.name << " : " << it.age << endl;
      }
  }
  
  void demo1()
  {
      vector<int> v;
  
      for (int i = 0; i <= 9; ++i)
      {
          v.push_back(rand()%2);
      }
  
      printVector(v);
  
      int countNum = count(v.begin(),  v.end(), 0);
      cout << "Count 0 : " << countNum << endl;
  }
  
  void demo2()
  {
      vector<Person> v;
  
      Person p1("QQ", 11);
      Person p2("WW", 10);
      Person p3("EE", 10);
      Person p4("RR", 10);
  
      v.push_back(p1);
      v.push_back(p2);
      v.push_back(p3);
      v.push_back(p4);
  
      Person p("SS", 10);
  
      printVector(v);
  
      int countNum = count(v.begin(),  v.end(), p);
      cout << "Count P : " << countNum << endl;
  }
  
  int main()
  {
      srand((unsigned int) time(NULL));
      demo1();
      demo2();
  
      return 0;
  }
  ```







#### 2.6 count_if



- 功能描述：按条件统计元素个数
- 函数原型：`count_if(iterator begin, iterator end, _Pred);`
- `_Pred`谓词(条件)



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 13:50
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  class Person
  {
  public:
      int age;
  
      Person(const int &age)
      {
          this->age = age;
      }
  };
  
  bool great_4(const int &value)
  {
      return value > 4;
  }
  
  bool equal_10(const Person &p)
  {
      return p.age == 10;
  }
  
  void demo1()
  {
      vector<int> v;
  
      for (int i = 0; i <=9; ++i)
      {
          v.push_back(i);
      }
  
      const int countNum = count_if(v.begin(),  v.end(), great_4);
      cout << countNum << endl;
  }
  
  void demo2()
  {
      vector<Person> v;
  
      Person p1(11);
      Person p2(10);
      Person p3(10);
      Person p4(10);
  
      v.push_back(p1);
      v.push_back(p2);
      v.push_back(p3);
      v.push_back(p4);
  
      int countNum = count_if(v.begin(),  v.end(), equal_10);
      cout << countNum << endl;
  }
  
  int main()
  {
      demo1();
      demo2();
      return 0;
  }
  ```







### 3. 常用排序算法



| 算法简介         |                                    |
| ---------------- | ---------------------------------- |
| `sort`           | 对容器内元素进行排序               |
| `random_shuffle` | 洗牌--指定范围内元素随机调整次序   |
| `merge`          | 容器元素合并，并存储到另一个容器中 |
| `reverse`        | 反转指定范围的元素                 |





#### 3.1 sort



- 功能描述：对容器内元素进行排序
- 函数原型：
  1. `sort(iterator begin, iterator end, _Pred);`
  2. 按照谓词的条件查找元素，找到返回指定元素位置的迭代器，找不到返回结束迭代器位置
  3. `sort(iterator begin, iterator end);`
  4. 默认从小到大排序



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 14:09
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  #include <ctime>
  
  using namespace std;
  
  template<class T>
  void printVector(const vector<T> &v)
  {
      for (T it : v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  template<class T>
  bool downSort(const T &value1, const T &value2)
  {
      return value1 > value2;
  }
  
  void demo1()
  {
      vector<int> v;
  
      for (int i = 0; i <=9; ++i)
      {
          v.push_back(rand()%10);
      }
      printVector(v);
  
      sort(v.begin(), v.end());
      printVector(v);
  
      sort(v.begin(), v.end(), downSort<int>);
      printVector(v);
  }
  
  int main()
  {
      srand((unsigned int)time(NULL));
      demo1();
      return 0;
  }
  ```







#### 3.2 random_shuffle



- 功能描述：洗牌--指定范围内的元素随机调整次序
- 函数原型：`random_shuffle(iterator begin, iterator end);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 14:27
  //
  #include <iostream>
  #include <vector>
  #include <ctime>
  #include <algorithm>
  
  using namespace std;
  
  template<class T>
  void printVector(const vector<T> &v)
  {
      for (T it : v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void demo1()
  {
      vector<int> v;
      for (int i = 0; i <= 9; ++i)
      {
          v.push_back(i);
      }
  
      printVector(v);
  
      random_shuffle(v.begin(), v.end());
      printVector(v);
  }
  
  int main()
  {
      srand((unsigned int)time(NULL));
      demo1();
      return 0;
  }
  ```







#### 3.3 merge



- 功能描述：两个容器元素合并，存储到同一个容器中
- 函数原型：`merge(iterator begin1, iterator end1, iterator begin2, iterator end2, iterator newBegin);`
- 注意：两个容器内的元素，必须是`有序序列`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 15:41
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  template<class T>
  void printVector(T value)
  {
      cout << value << " ";
  }
  
  void demo()
  {
      vector<int> v1;
      vector<int> v2;
      vector<int> v3;
  
      for (int i = 0; i <= 9; ++i)
      {
          v1.push_back(i);
          v2.push_back(9 - i);
      }
  
      v3.resize(v1.size() + v2.size());
  
      merge(v1.begin(), v1.end(), v2.begin(), v2.end(), v3.begin());
      for_each(v3.begin(), v3.end(), printVector<int>);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```





#### 3.4 reverse



- 功能描述：将容器内元素进行反转
- 函数原型：`reverse(iterator begin, iterator end);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 15:51
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  template<class T>
  class PrintVector_T
  {
  public:
      void operator()(const T &value)
      {
          cout << value << " ";
      }
  };
  
  template<class T>
  void printVector_T(const T &value)
  {
      cout << value << " ";
  }
  
  void demo()
  {
      vector<int> v;
      for (int i = 0; i <= 9; ++i)
      {
          v.push_back(i);
      }
  
      cout << "Meta: ";
      for_each(v.begin(), v.end(), PrintVector_T<int>());
      cout << endl;
  
      cout << "Reverse: ";
      reverse(v.begin(), v.end());
      for_each(v.begin(), v.end(), printVector_T<int>);
      cout << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```







### 4. 常用拷贝替换算法



| 算法简介     |                                                  |
| ------------ | ------------------------------------------------ |
| `copy`       | 容器内指定范围的元素拷贝到另一个容器中           |
| `replace`    | 容器内指定范围的`旧元素` 改为 `新元素`           |
| `replace_if` | 容器内指定范围的`满足条件的旧元素` 改为 `新元素` |
| `swap`       | 互换两个容器的元素                               |





#### 4.1 copy



- 功能描述：容器内指定范围的元素拷贝到另一个容器中
- 函数原型：`copy(iterator begin, iterator end, iterator newBegin);`
- 按值查找元素，返回找到的指定位置迭代器，找不到返回结束迭代器



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 16:10
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  class PrintVector
  {
  public:
      void operator()(int value)
      {
          cout << value << " ";
      }
  };
  
  void demo()
  {
      vector<int> v1;
      vector<int> v2;
  
      for (int i = 0; i <= 9; ++i)
      {
          v1.push_back(i);
      }
  
      v2.resize(v1.size());
  
      copy(v1.begin(), v1.end(), v2.begin());
  
      for_each(v2.begin(), v2.end(), PrintVector());
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```







#### 4.2 replace



- 功能描述：容器内指定范围的`旧元素` 改为 `新元素`
- 函数原型：`replace(iterator begin, iterator end, old_Value, new_Value);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 16:18
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  void printVector(const vector<int> &v, const string &str)
  {
      cout << str << ": ";
      for (const int it: v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v;
  
      for (int i = 0; i <= 9; ++i)
      {
          v.push_back(i);
      }
  
      printVector(v, "Meta");
  
      replace(v.begin(), v.end(), 0, 9);
      printVector(v, "New");
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  





#### 4.3 replace_if



- 功能描述：容器内指定范围的`满足条件的旧元素` 改为 `新元素`
- 函数原型：`replace_if(iterator beign, iterator end, _Pred, new_Value);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 16:25
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  void printVector(const vector<int> &v, const string &str)
  {
      cout << str << ": ";
      for (const int it: v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  bool greater_5(int value)
  {
      return value > 5;
  }
  
  void demo()
  {
      vector<int> v;
  
      for (int i = 0; i <= 9; ++i)
      {
          v.push_back(i);
      }
  
      printVector(v, "Meta");
  
      replace_if(v.begin(), v.end(), greater_5, 0);
      printVector(v, "New");
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```







#### 4.4 swap



- 功能描述：互换两个容器的元素
- 函数原型：`swap(contatiner c1, contatiner c2);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 16:30
  //
  #include <iostream>
  #include <vector>
  #include <algorithm>
  
  using namespace std;
  
  void printVector(const vector<int> &v, const string &str)
  {
      cout << str << ": ";
      for (const int it: v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v1;
      vector<int> v2;
  
      for (int i = 0; i <= 9; ++i)
      {
          v1.push_back(i);
          v2.push_back(9 - i);
      }
  
      printVector(v1, "meta");
      printVector(v2, "meta");
  
      swap(v1, v2);
      printVector(v1, "new");
      printVector(v2, "new");
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```
  
  





### 5. 常用算术生成算法



- 注意：算术生成算法属于小型算法，使用时需要包含头文件`#include <numeric>`

  | 算法简介     |                            |
  | ------------ | -------------------------- |
  | `accumulate` | 计算容器区间内元素累计总和 |
  | `fill`       | 向容器中`填充`元素         |







#### 5.1 accumulate



- 功能描述：计算容器区间内元素累计总和
- 函数原型：`accumulate(iterator begin, iterator end, value);`
- `value`起始值



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 17:02
  //
  #include <iostream>
  #include <numeric>
  #include <vector>
  
  using namespace std;
  
  void demo()
  {
      vector<int> v;
  
      for (int i = 0; i <= 9; ++i)
      {
          v.push_back(i);
      }
  
      const int sum = accumulate(v.cbegin(), v.cend(), 0);
      cout << sum << endl;
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```

  





#### 5.2 fill



- 功能描述：向容器中`填充`元素
- 函数原型：`fill(iterator begin, iterator end, value);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 17:12
  //
  #include <iostream>
  #include <vector>
  
  using namespace std;
  
  void printVector(const vector<int> &v)
  {
      for (const int it : v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v;
  
      for (int i = 0; i <= 8; ++i)
      {
          v.push_back(i);
      }
  
      fill(v.begin(), v.end(), 9);
      printVector(v);
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```







### 6. 常用集合算法



| 算法简介           |                    |
| ------------------ | ------------------ |
| `set_intersection` | 求两个容器的`交集` |
| `set_union`        | 求两个容器的`并集` |
| `set_difference`   | 求两个容器的`差集` |



- 注意：
  1. 使用前，确保 `新容器` 的预设一个 `合适大小`
  2. 算法`返回`的是 `最后一个元素` 所在位置的 `迭代器`
  3. 遍历 `新容器` 要用 `算法返回的迭代器`，而不是使用 `新容器自身的结束迭代器`
  4. `交集算法`中，`新容器自身的结束迭代器` 可能超出 `最后一个元素` 所在位置的 `迭代器`





#### 6.1 set_intersection



- 功能描述：求两个容器的`交集`
- 函数原型：`set_intersection(iterator begin_1, iterator end_1, iterator begin_2, iterator end_2, iterator new_Begin);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 17:27
  //
  #include <iostream>
  #include <algorithm>
  #include <vector>
  
  using namespace std;
  
  void printVector(const vector<int> &v, const char *containerName)
  {
      cout << containerName << ": ";
      for (const int it : v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void printVector(const vector<int> &v, const vector<int>::const_iterator &it, const char *containerName)
  {
      cout << containerName << ": ";
      for (vector<int>::const_iterator itBegin = v.begin(); itBegin != it; ++itBegin)
      {
          cout << *itBegin << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v1;
      vector<int> v2;
      vector<int> v_tag;
  
      for (int i = 0; i <= 9; ++i)
      {
          v1.push_back(i);
          v2.push_back(i + 5);
      }
      printVector(v1, "V1");
      printVector(v2, "V2");
  
      v_tag.resize(min(v1.size(), v2.size()));
  
      vector<int>::const_iterator itLast = set_intersection(v1.begin(), v1.end(), v2.begin(), v2.end(), v_tag.begin());
  
      // 使用 新容器自身迭代器
      printVector(v_tag, "Target");
  
      // 使用 set_intersection 返回的 最后一个元素的迭代器
      printVector(v_tag, itLast, "Target");
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```







#### 6.2 set_union



- 功能描述：求两个容器的`并集`
- 函数原型：`set_union(iterator begin_1, iterator end_1, iterator begin_2, iterator end_2, iterator new_Begin);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 17:50
  //
  #include <iostream>
  #include <algorithm>
  #include <vector>
  
  using namespace std;
  
  void printVector(const vector<int> &v, const char *containerName)
  {
      cout << containerName << ": ";
      for (const int it : v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void printVector(const vector<int> &v, const vector<int>::const_iterator &it, const char *containerName)
  {
      cout << containerName << ": ";
      for (vector<int>::const_iterator itBegin = v.begin(); itBegin != it; ++itBegin)
      {
          cout << *itBegin << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v1;
      vector<int> v2;
      vector<int> v_tag;
  
      for (int i = 0; i <= 4; ++i)
      {
          v1.push_back(i);
          v2.push_back(i + 4);
      }
      printVector(v1, "V1");
      printVector(v2, "V2");
  
      v_tag.resize(v1.size() + v2.size());
  
      vector<int>::const_iterator itLast = set_union(v1.begin(), v1.end(), v2.begin(), v2.end(), v_tag.begin());
  
      // 使用 set_union 返回的 最后一个元素的迭代器
      printVector(v_tag, itLast, "Target");
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```
  






#### 6.3 set_difference



- 功能描述：求两个容器的`差集`

- 差集：

  1. 求出 c1 和 c2 的交集

  2. c1 与 c2 的差集：c1 的 减去 交集部分
  3. c2 与 c1 的差集：c2 的 减去 交集部分

- 函数原型：`set_difference(iterator begin_1, iterator end_1, iterator begin_2, iterator end_2, iterator new_Begin);`



- 代码示例：

  ```c++
  //
  // Created by FHang on 2021/11/21 18:13
  //
  #include <iostream>
  #include <algorithm>
  #include <vector>
  
  using namespace std;
  
  void printVector(const vector<int> &v, const char *containerName)
  {
      cout << containerName << ": ";
      for (const int it: v)
      {
          cout << it << " ";
      }
      cout << endl;
  }
  
  void printVector(const vector<int> &v, const vector<int>::const_iterator &it, const char *containerName)
  {
      cout << containerName << ": ";
      for (vector<int>::const_iterator itBegin = v.begin(); itBegin != it; ++itBegin)
      {
          cout << *itBegin << " ";
      }
      cout << endl;
  }
  
  void demo()
  {
      vector<int> v1;
      vector<int> v2;
      vector<int> v_tag;
  
      for (int i = 0; i <= 9; ++i)
      {
          v1.push_back(i);
          v2.push_back(i + 5);
      }
      printVector(v1, "V1");
      printVector(v2, "V2");
  
      v_tag.resize(max(v1.size(), v2.size()));
  
      // V1 对于 V2 的 差集
      vector<int>::const_iterator itLast_V1 = set_difference(v1.begin(), v1.end(), v2.begin(), v2.end(), v_tag.begin());
      printVector(v_tag, itLast_V1, "V1 Difference");
  
      // V2 对于 V1 的 差集
      vector<int>::const_iterator itLast_V2 = set_difference(v2.begin(), v2.end(), v1.begin(), v1.end(), v_tag.begin());
      printVector(v_tag, itLast_V2, "V2 Difference");
  }
  
  int main()
  {
      demo();
      return 0;
  }
  ```
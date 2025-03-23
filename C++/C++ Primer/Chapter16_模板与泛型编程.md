# 第十六章 模板与泛型编程

###### 模板是C++泛型编程的基础。一个模板就是一个创建类或函数的蓝图或者说公式。

## 16.1 定义模板

### 16.1.1 函数模板

- 在模板定义中，模板参数列表不能为空。（模板参数列表作用类似函数参数列表）
- 隐式指定模板实参时，T被替换成编译器推断出来的模板参数，来实例化一个特定版本的函数。这个版本被称为模板的<mark>实例</mark>>。

- 类型参数前必须使用关键字class或typename。

- <mark>非类型模板参数的模板实参，必须是常量表达式。</mark>

  一个非类型参数可以是一个整型（实参必须是常量表达式），或是一个指向对象或函数类型的指针或左值引用（<mark>实参必须具有静态的生存期</mark>）。

- inline或constexpr说明符放在模板参数列表之后，返回类型之前：

  ```C++
  // 正确
  template <typename T> inline T min(const T&, const T&);
  // 错误
  inline template <typename T> T min(const T&, const T&);
  ```

- 内置类型和一般的标准库类型（除unique_ptr和IO类型之外）都是允许拷贝的。

- 当编译器遇到一个模板定义时，它并不生成代码。只有当我们实例化模板的一个特定版本时，编译器才会生成代码。

- 函数模板和类模板成员函数的定义通常放在头文件中。

- 模板内代码的编译错误阶段：

  （1）编译模板本身：语法错误；（2）编译器遇到模板使用时：函数模板的参数检查、类模板的模板实参数目等；（3）模板实例化时：类型相关的错误。

- 保证传递给模板的实参支持模板所要求的操作：是调用者责任。

### 16.1.2 类模板

- 与函数模板的不同之处，编译器不能为类模板推断模板参数类型。

- 类模板的成员函数，可以在<mark>类模板内部定义（隐式声明为内联函数）</mark>，也可以在类模板外部定义。
- 类模板的成员函数具有和模板相同的模板参数。定义在类模板之外的成员函数，必须以关键词template开始，后接模板参数列表。
- 默认情况下，<mark>对于一个实例化了的类模板，其成员只有在使用时才被实例化</mark>。
- 在类模板自己的作用域中，可以直接使用模板名而不提供模板实参。

```c++
template <typename T> class BlobPtr {
public:
    BlobPtr& operator++(); // 等价于BlobPtr<T>& operator++();
}
```

- 如果一个类模板包含一个非模板<mark>友元</mark>，则友元被授权可以访问所有模板实例。如果友元自身是模板，类可以授权给所有友元模板实例，也可以只授权给特定实例。

```c++
// 前置声明，在Blob中声明友元所需要的
template <typename> class BlobPtr;
template <typename> class Blob; // 运算符==中的参数所需要的
template <typename T>
    bool operator==(const Blob<T>&, const Blob<T>&);

template <typename T> class Blob {
    // 每个Blob实例将访问权限授予用相同类型实例化的BlobPtr和相等运算符
    friend class BlobPtr<T>;
    friend bool operator==<T>
            (const Blob<T>&, const Blob<T>&);
    // 其他成员定义
}
```

```c++
Blob<char> ca; // BlobPtr<char>和operator==<char>都是本对象的友元
Blob<int> ia;  // BlobPtr<int>和operator==<int>都是本对象的友元
// BlobPtr<char>的成员，可以访问ca或任何其他Blob<char>对象的非public部分，但对ia或任何其他Blob<int>对象或其他实例都没有特殊访问权限。
```

- 为了让所有实例成为友元，友元声明中必须使用与类模板本身不同的模板参数。TODO

- 当我们定义一个<mark>模板类型别名</mark>时，可以固定一个或多个模板参数：

  ```c++
  template <typename T> using partNo = pair<T, unsigned>;
  partNo<string> books; // books是一个pair<string, unsigned>
  ```

- 类模板的static成员：

  ```c++
  template <typename T> class Fool {
  public:
      static std::size_t count() { return ctr; }
  private:
      static std::size_t ctr;
  }
  
  // 所有三个对象共享相同的Foo<int>::ctr和Foo<int>::count成员
  Foo<int> fi, fi2, fi3;
  // 错误用法，需要引用特定实例来访问static成员
  ct = Foo::count(); // 正确用法：autop ct = Foo<int>::count();
  ```

### 16.1.3 模板参数

- 在模板内不能重用模板参数名。

- <mark>一个特定文件所需要的所有模板的声明，通常一起放置在文件开始位置，出现于任何使用这些模板的代码之前。</mark>

- <mark>当我们希望通知编译器一个名字表示类型时，必须使用关键字typename，而不能使用class。</mark>

  ```C++
  // top函数期待一个容器类型的实参，使用typename指定其返回类型
  template <typename T>
  typename T::value_type top(const T& c)
  {
      if (!c.empty()) {
          return c.back();
      } else {
          return typename T::value_type();
      }
  }
  ```

  > 在普通（非模板）代码中，编译器掌握类的定义，所以知道通过作用域运算符访问的名字是类型还是static成员；<mark>但类似`T::mem`这样的代码，不知道mem是一个类型成员，还是一个static成员，直至实例化才知道。</mark>

- 可以为函数模板和类模板提供默认实参。如果一个类模板为其所有模板参数都提供了默认实参，且我们希望使用这些默认实参，就必须在模板名之后跟一个空尖括号对。

  ```c++
  template <class T = int> class Numbers {}; // T默认为int
  Numbers<> average_precision; // 空<>表示我们希望使用默认类型
  ```

### 16.1.4 成员模板

- 一个类（无论是普通类还是类模板）可以包含本身是模板的成员函数。这种成员被称为<mark>成员模板</mark>。① 成员模板不能是虚函数；② 对于类模板，成员模板可以有自己的、独立的模板参数。

  ```c++
  template <typename T> class Blob {
      template <typename It> Blob(It b, It e);
      // ...
  }
  
  // 类模板外定义成员模板
  template <typename T>  // 类的类型参数
  template <typename It> // 构造函数的类型参数
      Blob<T>::Blob(It b, It e) : ..
  ```

### 16.1.5 控制实例化

- 对每个实例化声明，在程序的某个位置必须有其显式的实例化定义（非extern）。

  ```C++
  extern template class Blob<string> // 实例化声明
  template int compare(const int&, const int&); // 实例化定义
  
  // 举例
  // Application.cc 需要将templateBuild.o和Application.o链接到一起
  extern template class Blob<string>;
  extern template int compare(const int&, const int&);
  Blob<string> sa1, sa2; // 实例化会出现在其他位置
  // Blob<int>及其接受initializer_list的构造函数在本文件中实例化
  Blob<int> a1 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
  Blob<int> a2(a1); // 拷贝构造函数在本文件中实例化
  int i = compare(a1[0], a2[0]); // 实例化出现在其他位置
  
  // templateBuild.cc
  template int compare(const int&, const int&);
  template class Blob<string>; // 实例化类模板的所有成员
  ```

  文件`Application.o`将包含`Blob<int>`的实例及其接受initializer_list参数的构造函数和拷贝构造函数的实例。而`compare<int>`函数和`Blob<string>`类将不在本文件中进行实例化。这些模板的定义必须出现在程序的其他文件中。

  > ① 因为当模板被使用时才会进行实例化，所以相同的实例可能出现在多个对象文件中 。在多个文件中实例相同模板的额外开销会很严重，需要<mark>显式实例化</mark>。
  >
  > ② <mark>当编译器遇到extern模板声明时，它不会在本文将中生成实例化代码</mark>。将一个实例化声明为extern就表示承诺在程序其他位置有该实例化的一个非extern声明（定义）。对于一个给定的实例化版本，可能有多个extern声明，但必须只有一个定义。

- 在一个类模板的实例化定义中，所用类型必须能用于模板的所有成员函数。

### 16.1.6 效率与灵活型

- 在unique_ptr类中，删除器的类型是类类型的一部分。即，unique_ptr有两个模板参数，一个表示它所管理的指针，另一个表示删除器的类型。
- 通过在编译时绑定删除器，unique_ptr避免了间接调用删除器的运行时开销。通过在运行时绑定删除器，shared_ptr使用户重载删除器更为方便。

## 16.2 模板实参推导

### 16.2.1 类型转换与模板类型参数

- <mark>将实参传递给带模板类型的函数形参时，能够自动应用的类型转换只有① const转换及② 数组或函数到指针的转换。</mark>

  ```c++
  template <typename T> T fobj(T, T); // 实参被拷贝
  template <typename T> T fref(const T&, const T&); // 引用
  string s1("a value");
  const string s2("another value");
  fobj(s1, s2); // ok, s2的const被忽略
  fref(s1, s2); // ok, 将s1转换为const是允许的
  
  int a[10], b[42];
  fobj(a, b); // ok, 调用f(int*, int*);
  fref(a, b); // error, 数组类型不匹配。（形参是一个引用，数组不会转换为指针）
  ```

  > ① const转换：将一个非const对象的引用（或指针）传递给一个const的引用（或指针）形参；
  >
  > ② 数组或函数指针转换：<mark>如果函数形参不是引用类型，则可以对数组或函数类型的实参应用正常的指针转换。一个数组实参可以转换为一个指向其首元素的指针</mark>。类似的，一个函数实参可以转换为一个该函数类型的指针。

- 如果函数参数类型不是模板参数，则对实参进行正常的类型转换。

### 16.2.2 函数模板显式实参

- 指定显式模板实参

  ```C++
  // 编译器无法推断T1，它未出现在函数参数列表中
  template <typename T1, typename T2, typename T3>
  T1 sum(T2, T3);
  ```

  在本例中，没有任何函数实参的类型可用来推断T1的类型。每次调用sum时调用者都必须为T1提供一个显式模板实参（explicit template argument）。

  ```C++
  // T1是显式指定的，T2和T3是从函数实参类型推断而来的
  auto val3 = sum<long long>(i, lng); // long long sum(int, long)
  
  // 糟糕的设计：用户必须指定所有三个模板参数（没有按顺序）
  template <typename T1, typename T2, typename T3>
  T3 alternative_sum(T2, T1);
  ```

### 16.2.3 尾置返回类型与类型转换

- 尾置返回类型场景：并不知道返回结果的准确类型，但知道所需类型是所处理的元素类型。

  ```C++
  template <typename It>
  ??? &fcn(It beg, It end)
  {
      // 处理序列
      return *beg; // 返回序列中一个元素的引用
  }
  ```

  因为在编译器遇到函数的参数列表之前，beg都是不存在的。为定义此函数，必须使用<mark>尾置返回类型</mark>，来通知编译器fcn的返回类型与解引用beg参数的结果类型相同。

  ```c++
  // 尾置返回允许我们在参数列表之后声明返回类型
  template <typename It>
  auto fcn(It beg, It end) -> decltype(*beg)
  {
  	// 处理序列
  	return *beg; // 返回序列中一个元素的引用
  }
  ```

- 进行类型转换的标准模板类

  有时我们无法直接获得所需要的类型。例如，我们可能希望编写一个类似fcn的函数，但返回一个元素的值而非引用。唯一可以使用的操作是迭代器操作，而所有迭代器操作都不会生成元素，只能生成元素的引用。

  <mark>为了获得元素类型，我们可以使用标准库的类型转换（`type transformation`）模板。这些模板定义在头文件`type_traits`中。</mark>

  ```c++
  // 为了使用模板参数的成员，必须用typename
  template <typename It>
  auto fcn2(It beg, It end) -> typename remove_reference<decltype(*beg)>::type // 需要使用typename告知编译器，type表示一个类型
  {
      // 处理序列
      return *beg; // 返回序列中一个元素的拷贝
  }
  ```

  常见的标准类型转换模板：`remove_reference`, `add_const`, `add_lvalue_reference`, `add_rvalue_reference`等。<mark>每个模板都有一个名为type的public成员，表示一个类型，此类型与模板自身的模板类型参数相关</mark>。

### 16.2.4 函数指针和实参推断

- 当参数是一个函数模板实例的地址时，程序上下文必须满足：对每个模板参数，能唯一确定其类型或值。

  ```c++
  template <typename T> int compare(const T&, const T&);
  // pf1指向实例int compare(const int&, const int&);
  int (*pf1)(const int&, const int&) = compare;
  
  // func的重载版本；每个版本接受一个不同的函数指针类型
  void func(int(*)(const string&, const string&));
  void func(int(*)(const int&, const int&));
  func(compare); // 错误：使用compare的哪个实例？
  
  // 正确：显式指出实例化哪个compare版本
  func(compare<int>()); // 传递compare(const int&, const int&);
  ```

### 16.2.5 模板实参推断和引用 （？）

- 顶层const：变量本身是一个常量。

  ```c++
  int const a = 10; // a是一个顶层const，值不能被修改
  const int b = 20; // b也是一个顶层const，值不能被修改
  ```

- 底层const：指针或引用所指向的对象是否是常量。<mark>当`const`关键字出现在指针或引用的类型说明符的右侧时，它表示的是底层const。底层const意味着指针或引用所指向（或引用的）对象是一个常量，而不是指针或引用本身。</mark>

  ```c++
  int *const p = &x;  // p是一个指向int的常量指针，p的值（即地址）不能被修改，但*p可以修改 --> 底层const
  const int *q = &y;  // q是一个指向const int的指针，q的值可以改变，但*q不能被修改 --> 顶层const
  ```

- 从左值引用函数参数推断类型 —— 函数参数的类型是一个普通的左值引用（T&）

  ```c++
  template <typename T> void f1(T&); // 实参必须是一个左值
  // 对f1的调用使用实参所引用的类型作为模板参数类型
  f1(i);  // i是一个int；模板参数类型T是int
  f1(ci); // ci是一个const int；模板参数类型是const int
  f1(5);  // 错误：传递给一个&参数的实参必须是一个左值
  ```

- 从左值引用函数参数推断类型 —— 函数参数的类型是一个const的左值引用（const T&）

  <mark>可以传递给它任何类型的实参（一个对象（const或非const）、一个临时对象或是一个字面常量值）</mark>

  ```c++
  template <typename T> void f2(const T&); // 可以接受一个右值
  // f2中的参数是const &；实参中的const是无关的
  // 在每个调用中，f2的函数参数都被推断为const int&
  f2(i);  // i是一个int；模板参数T是int
  f2(ci); // ci是一个const int，但模板参数T是int
  f2(5);  // 一个const& 参数可以绑定到一个右值；T是int
  ```

- 从右值引用函数参数推断类型

  ```C++
  template <typename T> void f3(T&&);
  f3(42); // 实参是一个int类型的右值；模板参数T是int
  ```

- 引用折叠和右值引用参数

  我们不能将一个右值引用绑定到一个左值上。但是，C++语言在正常绑定规则之外定义了两个例外规则，允许这种绑定：

  （1）T&&模板类型参数 + 左值实参 = T&；

  （2）X&&、X&&&、X&&&都折叠成类型X&；类型X&&&&折叠成X&&；

  ```c++
  f3(i);  // 实参是一个左值；模板参数T是int&
  f3(ci); // 实参是一个左值；模板参数T是一个const int& 
  ```

  <mark>如果一个函数参数是指向模板参数类型的右值引用（如，T&&），则可以传递给它任意类型的实参。如果将一个左值传递给这样的参数，则函数参数被实例化为一个普通的左值引用（T&）。</mark>

- 编写接受右值引用参数的模板函数

  ```c++
  template <typename T> void f3(T&& val)
  {
      T t = val; // 拷贝还是绑定一个引用
      t = fcn(t); // 赋值只改变t还是既改变t又改变val?
      if (val == t) { /* ... */ }  // 若T是引用类型，则一直为true.
  }
  // 当我们对一个右值调用f3时，例如字面常量42，T为int。在此情况下，局部变量t的类型为int，且通过拷贝参数val的值被初始化
  // 当我们对一个左值i调用f3时，则T为int&。当我们定义并初始化局部变量t时，赋予它类型int&。因此，对t的初始化将其绑定到val。当我们对t赋值时，也同时改变了val的值。在f3的这个实例化版本中，if判断永远得到true
  ```

### 16.2.6 理解std::move
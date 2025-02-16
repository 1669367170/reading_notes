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

  在本例中，没有任何函数实参的类型可用来推断T1的类型。每次调用sum时调用者都必须为T1提供一个显式模板实参（explicit template argument。

  ```C++
  // T1是显式指定的，T2和T3是从函数实参类型推断而来的
  auto val3 = sum<long long>(i, lng); // long long sum(int, long)
  
  // 糟糕的设计：用户必须指定所有三个模板参数（没有按顺序）
  template <typename T1, typename T2, typename T3>
  T3 alternative_sum(T2, T1);
  ```

### 16.2.3 尾置返回类型与类型转换
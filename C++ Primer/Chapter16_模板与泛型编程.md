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

### 16.1.3 模板参数与作用域
# Google C++ Style Guide

# 1. Header Files

一般来说，每一个`.cc`文件都有一个与之相关联的`.h`头文件，但是对于unit tests以及包含`main()`函数的`.cc`文件，则不需要。

对于C/C++来说，`.h`文件实际上定义了一个外部可见的**对外接口**，还可以对外隐藏内部实现细节(实际上就是**封装**的核心思想)。因此对外接口出一`.h`文件中，而内部实现放置在对应的**.cc**中。

因此如果一个`.cc`文件不需要对外暴露接口，则不需要相应的`.h`文件。

## 1.1 Self-contained Headers

Google的C++规范中，头文件以.h结尾，并且需要时**self-contianed**的。

头文件的**self-contained**：头文件需要包含了自身能正常工作的相关其他头文件，头文件本身依赖的其它头文件，需要全部包含，头文件能够被单独编译，而不报错。

一个简单的例子：

```c++
// my_class.h 
class MyClass
{
   MyClass(std::string s);
};
```

```c++
// my_class.cpp 
#include <string>
#include "my_class.h"
MyClass::MyClass(std::string s)
{}
```

其中`my_class.h`就不是self-contained，需要 `#include <string>`才能编译，不能够被单独编译。正确的做法是在 my_class.h 中 `#include <string>`。

inline functions与templates的声明与定义都必须放在头文件中。

## 1.2 The #define Guard

`#define` guard用于避免multiple inclusion。`#define` guard的格式 `<PROJECT>_<PATH>_<FILE>_H_`

其中`<PATH>`是相对于`PROJECT`根目录的PATH。



## 1.3 Include What You Use

只需要include能直接提供所需要符号的declaration与definition的头文件。

不要transitive inclusion。

```c++
// foo.h
#include "bar.h"
```

```c++
// bar.h
class MyClass{};
```

```c++
// foo.cc
//if it use MyClass from bar.h
#include "bar.h"
```

即使`foo.h`中已经include了`bar.h`，当`foo.cc`用到了`bar.h`中的符号时，也不要include `foo.h`，这样会导致transitive inclusion。

## 1.4 Forward Declarations

```c++
// In a C++ source file:
class B;
void FuncInB();
extern int variable_in_b;
ABSL_DECLARE_FLAG(flag_in_b);
```



## 1.5 Inline Functions

只有在函数特别小的时候(10行及以内)，才将函数定义为inline。

一般来说，以下函数会被声明为inline：

1. **accessor** C++ get性质的函数
2. **mutator** C++ set性质的函数
3. **other short, performance-critical functions**

inline function有好有坏：

- Pros：当inline function比较短的时候，inline函数能够带来更加更高效的代码
- Cons：inline可能导致可执行代码大小过大，反而会导致执行速度变慢(因为无法利用到CPU的instruction cache)。



```
A decent rule of thumb is to not inline a function if it is more than 10 lines long. Beware of destructors, which are often longer than they appear because of implicit member- and base-destructor calls!

Another useful rule of thumb: it's typically not cost effective to inline functions with loops or switch statements (unless, in the common case, the loop or switch statement is never executed).

It is important to know that functions are not always inlined even if they are declared as such; for example, virtual and recursive functions are not normally inlined. Usually recursive functions should not be inline. The main reason for making a virtual function inline is to place its definition in the class, either for convenience or to document its behavior, e.g., for accessors and mutators.
```



## 1.6 Names and Order of Includes

include头文件的顺序：

1. **Related header** 该`.cc`文件对应的同名头文件
2. **C system headers** 
3. **C++ standard library headers**
4. **other libraries' headers** 第三方库的头文件
5. **your project's headers**

上面的不同分组之间用空行隔开，每一个部分之中以字母表排序。



头文件的路径应该相对于项目的源代码目录`src`，不能出现UNIX目录别名`.`或者`..`。

比如`google-awesome-project/src/base/logging.h`，应该通过如下方式导入`#include "base/logging.h"`

再比如对于`dir/foo.cc`或者`dir/foo_test.cc`，前者用于实现`dir2/foo2.h`中的API，后者用于测试，那么这两个文件中的include顺序应该遵循：

1. dir2/foo2.h
2. 空行
3. C system headers(采用<>) e.g.`<unistd.h>`, `<stdlib.h>`, `<Python.h>`
4. C++ standard library headers (without file extension) e.g.`<algorithm>`, `<cstddef>`
5. 空行
6. Other libraries' `.h` files.
7. 空行
8. Your project's `.h` files.



**一般来说`dir/foo.cc`与`dir2/foo2.h`处于同一个目录下？**

C头文件有的在C++中有对应的头文件，比如`stddef.h`与`cstddef`，二者可以互用。



**conditional includes**放在其它的include之后。

```c++
#include "foo/public/fooserver.h"

#include "base/port.h"  // For LANG_CXX11.

#ifdef LANG_CXX11
#include <initializer_list>
#endif  // LANG_CXX11
```



# 2. Scoping

## 2.1 Namespaces


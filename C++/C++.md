# C++

C++是一个multi-paradigm programming language:

- Procedural Programming： superset of C
- Modular Programming：namespace
- Object-Oriented Programming
- Generic Programming
- Functional Programming:



# Declaration and Definition

这一部分涉及到一点点编译原理与链接方面的知识，有关symbol与reference。

```
Declarations specify the interpretation given to each identifier; they do not necessarily reserve storage associated with the identifier. Declarations that reserve storage are called definitions.
```

从上面一段话中，可以发现**definition也是一种declaration**。

```
declaration:
	declaration-specifiers init-declarator-list-opt;
	
declaration-specifiers:
	storage-class-specifier declaration-specifiers-opt
	type-specifier declaration-specifiers-opt
	type-qualifier declaration-specifiers-opt

init-declarator-list:
	init-declarator
	init-declarator-list , init-declarator
init-declarator:
	declarator
	declarator = initializer
```

上面的对于declaration的构成表述就相当于通过一个**上下文无关文法**来表述。



**declaration**是引入了一个具有类型(type(**struct**或者**class**)，object或者function)的identifier，这是compiler需要用到的。

**definition**是linker需要用的。

declaration可以有多次，但是definition只能一次。

```c
int a;//既是声明也是定义，通过int类型的缺省值初始化a
```

```c
extern int a;//只是声明
```

而对于C/C++中的结构体和类

```
struct A;
class B;//二者都只是声明
```

```
struct A {...};
class B {...};//二者是定义
```

上面是对于结构体和类的类型的声明和定义，而对于相应结构体与类的变量

```
struct A a1;//声明同时定义
B b1;//声明同时定义
```

```
A declaration is a definition unless it [...] is a class name declaration [...].
```

因此大多数情况下，声明也是定义，除了上面`class A;`这条语句只是声明的A这个类，并没有定义。

声明与定义具体涉及到编译器与链接器行为，



## Storage Class Specifiers

有以下的Storage Class Specifier:

- **auto**
- **register** register关键字声明的变量会被编译器防止在寄存器之中和，这样在频繁访问时可以获得效率的提升(但是无法使用`&`运算符获得到其地址，因为不在内存中)。其余与`auto`关键字相同。并且采用`register`与`auto`的声明同时也是定义，会分配空间。
- **static** `static`在函数内声明和函数外声明的作用不相同。
- **extern**
- **typedef**



## One Definition Rule





# External Declaration

C/C++中的translation unit是compiler的编译单元，通常一个**经过预处理后**的源文件就是一个translation unit。

通常一个translation unit会包含一系列的external declarations(包括declarations以及一些function definitions)。

```
translation-unit:
	external-declaration
	translation-unit external-declaration

external-declaration:
	function-definition
	declaration
```





# Scope and Linkage



## Lexical Scope





# C/C++下的多线程/协程

## 1. 并发(Concurrency)与并行(Parallelism)

基本理论移步[这里](../系统/操作系统.md#7.-并发(Concurrency)与并行(Parallelism))

## 2. 协程



# C++智能指针

C++中的智能指针

## 1. std::unique_ptr

```
std::unique_ptr is a smart pointer that owns and manages another object through a pointer and disposes of that object when the unique_ptr goes out of scope. 
```



# inline内联函数

对于内联函数，C++会在编译时将函数调用替换为函数的定义，从而避免了函数调用时的栈操作。



内联函数有一些特性：

1. C++规定，内联函数可以在程序中定义多次，只要内联函数在一个.cpp文件中只出现一次，并且在所有的.cpp文 件中，这个内联函数的定义是一样的，就能通过编译
2. 当inline函数的声明和定义分别在头文件和源文件中，并且在其他文件中被调用时，链接期间编译器会报“对 xxx 未定义的引用”错误（因为如果定义没有在头文件的话，编译器是无法进行函数替换的，类似于宏展开）
3. inline函数是否被展开还取决于编译器的行为，gcc采用o0优化不会展开，而采用o2优化会展开



# Namespaces

```
Namespaces provide a method for preventing name conflicts in large projects.
```



# Statement与Expression的关系



# C++中的类型转换

# IO


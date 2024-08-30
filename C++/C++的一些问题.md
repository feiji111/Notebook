# 1. cmath库报错library not found

原因：Ubuntu22.04下libstdc++-12-dev没有安装

解决：

```cpp
sudo apt install libstdc++-12-dev
```



# 2. 何时使用String类，何时使用char*

## 2.1 C++ string与char*区别

```
std::string (也就是C++中的string）:标准并没有规定字符串必须以\0字符结尾。编译器在实现时既可以在结尾加上\0，也可以不加，这个是由编译器决定的，因为string是一个类，它的长度信息已经封装到类的私有变量里面了。

但是当通过c_str()或者data()（二者在c++ 11以及之后的标准中等价）转换得到的const char *时，会发现最后一个字符一定是\0。这个就是C语言中的string的结尾标志了（C语言中没有string类，都是char *）。可以将字符串中间任意位置设置为\0,但是在C++中字符串会将\0当成一个正常字符处理（打印时会当成空格或者不占位置的字符输出）。
```

并且**const char*指针传递常量字符串引用的方法是行不通的**：

- 这是正是因为C++ string中允许有空字符的存在，而对于char*类型来说，空字符意味着字符串的结尾。



**const std::string&会频繁地引起额外的堆内存分配**，因为string类需要拥有字符缓冲区，对于下面的例子(出自Getting Started with LLVM Core Libraries)

```c++
bool hasComma (const std::string &a) {
// code
}
void myfunc() {
    char buffer [40];
    // code to create our string in our own buffer
    hasComma(buffer); // C++ compiler is forced to create a new string object, duplicating the buffer
    hasComma("hello, world!"); // Likewise
}
```

对于`hasComma(buffer)`，C++会创建一个新的string对象，并且在堆内存中分配内存以复制字符串到string对象的内部缓冲区。

对于`hasComma(buffer)`，字符串分配在栈上；而对于`hasComma("hello, world!")`，字符串是一个全局常量。对于这些情况，C++缺少一个简单的类，当我们仅仅需要引用一个字符串时，它能够避免不必要的分配。

除此之外，引用一个字符串对象意味着两次间接访问。因为string类已经用一个内部指针存放它的数据，当我们访问实际的数据时，传递一个string对象的指针带来了双引用的额外开销。

## 2.2 何时使用string何时使用char*

参考stack overflow上的讨论[When to use std::string vs char*?](https://stackoverflow.com/questions/10937767/when-to-use-stdstring-vs-char)，

## 2.3 Why do many projects implement their own string, vector, map, etc. instead of using STL?

问题出自于Quora [Why do many projects implement their own string, vector, map, etc. instead of using STL?](https://www.quora.com/Why-do-many-projects-implement-their-own-string-vector-map-etc-instead-of-using-STL)。

知乎上也有类似的讨论 [为什么大多数的 C++ 的开源库都喜欢自己实现 string？](https://www.zhihu.com/question/54664311/answer/2768793520)





# 3. 头文件的一些思考

这一部分内容涉及到[声明与定义]()

## 3.1 头文件里应该放什么

1. **函数原型**
2. **#define定义**
3. **const定义**
4. **结构声明**
5. **类声明**
6. **模板声明**
7. **内联函数**

但是头文件**一般**不会放变量和函数的定义：

1. 头文件可定义const数据，const默认只在当前文件内有效。若多个文件出现同名const变量，等同于不同文件内的独立变量（对其他文件均不可见），所以便不会导致多重定义。const对象默认是static的。
2. 定义内联函数inline。C++规定，内联函数可以在程序中定义多次，只要内联函数在一个.cpp文件中只出现一次，并且在所有的.cpp文 件中，这个内联函数的定义是一样的，就能通过编译。
3. 头文件可定义类。在C++的类中，如果函数成员在类的定义体中被定义，那么编译器会视这个函数为内联的。
4. 模板函数/类



同时默认命名空间声明不要放在头文件，using namespace std 等应放在.cpp文件，在.h文件中使用std::cout。

## 3.2 预处理



## 3.3 头文件接口设计

核心思路：封装，继承，多态。



## 3.4 在头文件中include还是在源文件中include





# 4. C++源文件与头文件的后缀以及gcc如何区分C或C++

C++源文件后缀有许多最为常见的有`.C`, `.cc`, `.cpp`, `.CPP`, `.c++`, `.cp`, or `.cxx`(`.c++`后缀在windows下会出现问题，windows不允许`+`出现在文件名中，所以有`.cxx`，将`+`号旋转45度)。

C++头文件后缀最为常见的有`.h`, `.hh`, `.hpp`, `.H`, or (for shared template code) `.tcc`。

 `gcc`在遇到这些后缀名时(除了`.c`文件)，会将其当作C++程序编译。但是单独采用`gcc`并不会链接到C++的库，而`g++`就是用于在调用GCC(GNU Compiler Collections)时，也会自动链接到C++库中。

但是`g++`会把`.c`，`.h`和`.i`文件当作C++的文件。

## 4.1 gcc与g++区别

参考[What is the difference between g++ and gcc?](https://stackoverflow.com/questions/172587/what-is-the-difference-between-g-and-gcc)

首先，gcc与g++都是GNU Compiler Collection的compiler driver。它们会根据文件的类型或者根据`-x`选项，自动选择使用哪一些后端工具(比如是`cc1`还是`cc1plus`)。



`g++`命令与`gcc -xc++ -lstdc++ -shared-libgcc`的作用是等价的。

通过`g++ -v main.cpp -o main`中的`-v`选项，可以看到`g++`调用的GCC中的一系列`backends`。

```
/usr/lib/gcc/x86_64-linux-gnu/12/cc1plus -quiet -v -imultiarch x86_64-linux-gnu -D_GNU_SOURCE main.cpp -quiet -dumpbase main.cpp -dumpbase-ext .cpp -mtune=generic -march=x86-64 -version -fasynchronous-unwind-tables -fstack-protector-strong -Wformat -Wformat-security -fstack-clash-protection -fcf-protection -o /tmp/ccJSAVQQ.s

as -v --64 -o /tmp/ccUXKf9w.o /tmp/ccJSAVQQ.s

/usr/lib/gcc/x86_64-linux-gnu/12/collect2 -plugin /usr/lib/gcc/x86_64-linux-gnu/12/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/12/lto-wrapper -plugin-opt=-fresolution=/tmp/ccNO0YGx.res -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lgcc --build-id --eh-frame-hdr -m elf_x86_64 --hash-style=gnu --as-needed -dynamic-linker /lib64/ld-linux-x86-64.so.2 -pie -z now -z relro -o main /usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu/Scrt1.o /usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/12/crtbeginS.o -L/usr/lib/gcc/x86_64-linux-gnu/12 -L/usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/12/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/12/../../.. /tmp/ccUXKf9w.o -lstdc++ -lm -lgcc_s -lgcc -lc -lgcc_s -lgcc /usr/lib/gcc/x86_64-linux-gnu/12/crtendS.o /usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu/crtn.o
```

cc1plus --> as --> collect2的大致流程，而通过`gcc -v main.cpp -o main -xc++ -lstdc++ -shared-libgcc`也能发现其调用的`backends`也是与`g++`完全相同的。

`gcc`也能够用于编译C++程序。

## 4.2 命名规则

C++文件后缀名有许多，但是对于C++文件后缀名采用何种，并没有明确的规定。

一般情况下，推荐使用.h/.cpp的组合。

```
有说法，称对于类似与template这样的声明与定义在同一个文件中的情况，采用.hpp作为头文件的后缀
```



# 5. C与C++

C和C++是怎样的一个关系。



# 6. C++的争议

编写于2024-8-16。最近一直在做C++项目，在做项目的过程中，更加深刻体会到了C++的强大。但是C++依然有许多缺陷被大家所诟病。

## 6.1 C++ ABI不统一

```
所以一直期待能有统一的C++二进制兼容标准（C++ ABI），但是目前情况还是不容乐观，基本形成以微软的VISUAL C++和GNU阵营的GCC（采用Intel Itanium C++ ABI标准）为首的两大派系，各抒己见互不兼容。
```



## 6.2 缺少统一的第三方库管理工具

在上一节中，我们提到了C++的缺陷之一，ABI不统一。由于C++ runtime，编译器版本互相不兼容，这就给C++的包管理带来了许多困难。

参考：

1. [如何看待 Windows 的 C++ 包管理器 vcpkg？](https://www.zhihu.com/question/263416411)





# 7. C/C++的库管理

# LaTex



# 1. LaTeX基本概念

参考：

- [TeX 引擎、格式、发行版之介绍 | 始终 (liam.page)](https://liam.page/2018/11/26/introduction-to-TeX-engine-format-and-distribution/)
- 

LaTeX is not a stand-alone typesetting program in itself, but document preparation software that runs on top of [Donald E. Knuth](https://en.wikipedia.org/wiki/Donald_Knuth)'s [TeX typesetting system](https://en.wikipedia.org/wiki/TeX). TeX distributions usually bundle together all the parts needed for a working TeX system and they generally add to this both configuration and maintenance utilities. Nowadays LaTeX, and many of the packages built on it, form an important component of any major TeX distribution.



## 1.1 LaTeX与TeX

LaTeX是建立在TeX之上的。TeX是一种排版程序，同时也是一种程序语言。而LaTeX是建立在TeX之上的



## 1.2 TeX引擎

TeX作为一种编程语言，和其它的编程语言一样是需要编译器的，而TeX的编译器就叫做TeX引擎。

TeX引擎有多种选择：

1. Knuth TeX
2. e-TeX
3. pdfTeX
4. LuaTeX
5. XeTeX
6. pTeX
7. upTeX
8. e-upTeX
9. pTeX-ng





## 1.3 格式

TeX是一个宏语言，用户可以自己编写宏然后发布成一个**格式(format)**。

TeX之上有许多格式：

1. plain TeX
2. LaTeX
3. ConTeXt

## 1.4 发行版

用户自己编写宏之后，可以将其发布成宏包(style package)或者文档类(document class)。因此如果对某些特定功能有需求，可以去网络上查找相应的宏包或者文档类。

而TeX发行版(distribution)，就是上面的TeX引擎，格式和宏包以及驱动、辅助工具的集合。当我们想使用TeX时，真正需要安装的是TeX发行版。



TeX发行版有许多：

- TeX Live
- macTeX
- CTeX
- MiKTeX


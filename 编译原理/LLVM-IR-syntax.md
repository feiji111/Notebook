# LLVM Language Reference Manual

参考官方文档，[LLVM Language Reference Manual](https://llvm.org/docs/LangRef.html)



LLVM IR是一种LLVM assembly language，是以一种Static Single Assignment(SSA)的指令格式。



### 7.2.1 Identifiers

LLVM的identifiers有两种类型：global与local。

Global identifiers (functions, global variables)以`@`字符开始，而Local identifiers以`%`开始。



LLVM还有一些保留字：

- opcodes keywords：add，bitcast，ret
- type names：
  - void
  - iN(比如i32，i64，i128)
  - Floating-point types(比如float和double)
  - Vectors types，`<<# elements> x <elementtype>>`(比如`<4 x i32>`)



更多关于LLVM IR的详细介绍，参考[这里](./LLVM-IR-syntax.md)

### 7.2.2 String constants





### 7.2.3 High Level Structure

#### 7.2.3.1 Module Structure

Modules与**Translation units**。

```
; Declare the string constant as a global constant.
@.str = private unnamed_addr constant [13 x i8] c"hello world\0A\00"

; External declaration of the puts function
declare i32 @puts(ptr nocapture) nounwind

; Definition of main function
define i32 @main() {
  ; Call puts function to write out the string to stdout.
  call i32 @puts(ptr @.str)
  ret i32 0
}

; Named metadata
!0 = !{i32 42, null, !"string"}
!foo = !{!0}
```



### 7.2.4 Linkage Types

对于Global values(Global Variables以及Functions)，其实都是一个指向内存地址的一个指针，它们具有一种**linkage type**。



### 7.2.5 Basic Blocks

每一个Basic Blocks都由一个label开头，并且只有一个entry point(第一条指令)与一个exit point(最后一条指令)。

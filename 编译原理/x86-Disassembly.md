# [x86 Disassembly](https://en.wikibooks.org/wiki/X86_Disassembly)



# AT&T syntax and Intel Syntax

两种语义都有各自的适用范围。

Intel syntax在DOS和Windows应用广泛，而AT&T syntax在Unix上应用广泛（因为Unix是在AT&T的贝尔实验室创造的）。

|                             AT&T                             |                            Intel                             |                                                              |
| :----------------------------------------------------------: | :----------------------------------------------------------: | ------------------------------------------------------------ |
|                       Parameter order                        |        `movl $5, %eax `Source before the destination.        | `mov eax, 5 `Destination before source.                      |
|                        Parameter size                        | `addl $0x24, %esp movslq %ecx, %rax paddd %xmm1, %xmm2 `Mnemonics are suffixed with a letter indicating the size of the operands: *q* for qword (64 bits), *l* for long (dword, 32 bits), *w* for word (16 bits), and *b* for byte (8 bits).[[6\]](https://en.wikipedia.org/wiki/X86_assembly_language#cite_note-GASvsNASM-6) | `add esp, 24h movsxd rax, ecx paddd xmm2, xmm1 `Derived from the name of the register that is used (e.g. *rax, eax, ax, al* imply *q, l, w, b*, respectively).Width-based names may still appear in instructions when they define a different operation.MOVSXD refers to sign extension with dword input, unlike MOVSX.SIMD registers have width-named instructions that determine how to split up the register. AT&T tends to keep the names unchanged, so PADDD is not renamed to "paddl". |
| [Sigils](https://en.wikipedia.org/wiki/Sigil_(computer_programming)) | [Immediate values](https://en.wikipedia.org/wiki/Constant_(programming)) prefixed with a "$", registers prefixed with a "%".[[6\]](https://en.wikipedia.org/wiki/X86_assembly_language#cite_note-GASvsNASM-6) | The assembler automatically detects the type of symbols; i.e., whether they are registers, constants or something else. |
| Effective [addresses](https://en.wikipedia.org/wiki/Memory_address) | `movl offset(%ebx,%ecx,4), %eax `General syntax of *DISP(BASE,INDEX,SCALE)*. | `mov eax, [ebx + ecx*4 + offset] `Arithmetic expressions in square brackets; additionally, size keywords like *byte*, *word*, or *dword* have to be used if the size cannot be determined from the operands.[[6\]](https://en.wikipedia.org/wiki/X86_assembly_language#cite_note-GASvsNASM-6) |



# Code Patterns

## The Stack

关于栈的介绍，这里不再赘述，但值得一提的是`esp`作为栈顶指针，而并没有专门的栈底指针（**ebp不是吗？**）

早期windows系统的蓝屏就是因为栈的溢出导致的



一条push/pop指令等价于两条，但是单条push/pop指令明显是更快的

```assembly
sub esp, 4
mov DWORD PTR SS:[esp], eax
```

```assembly
mov eax, DWORD PTR SS:[esp]
add esp, 4
```

**DWORD PTR**，被称为**Size Directives**，类似的还有：

- BYTE PTR
- WORD PTR
- DWORD PTR

一般来说，给出一个内存地址，这个地址处的数据的大小是可以推断出来的。但是有的时候也无法推断，所以需要Size Directives明确指出数据的大小。

## Functions and Stack Frames

**Stack Frames**的目的，使得每一个函数独立于其在stack上的位置工作，就像是每一个函数都位于栈顶一样。



一个标准的function entry sequence

```assembly
push ebp
mov ebp, esp
sub esp, X
```

其中**X**是局部变量的大小。

比如对于以下函数

```c
void MyFunction()
{
  int a, b, c;
  ...
```

编译后的结果为

```
_MyFunction:
  push ebp     ; save the value of ebp
  mov ebp, esp ; ebp now points to the top of the stack
  sub esp, 12  ; space allocated on the stack for the local variables
```

而a，b，c的值通过以下方式访问

```
mov [ebp -  4], 10  ; location of variable a
mov [ebp -  8], 5   ; location of b
mov [ebp - 12], 2   ; location of c
```



而**ebp**的作用凸显在调用函数时，用于访问局部变量，具体见[FPO](#FPO)。

对于以下函数，没有局部变量

```c
void MyFunction2(int x, int y, int z)
{
  ...
}
```

```
_MyFunction2:
  push ebp 
  mov ebp, esp
  sub esp, 0     ; no local variables, most compilers will omit this line
```

当通过`MyFunction2(10, 5, 2);`调用这个函数时，会产生如下汇编代码

```
push 2
push 5
push 10
call _MyFunction2
```

上面这种Right-to-Left calling convention称为**CDECL**

而x86的call指令等价于以下指令

```
push eip + 2 ; return address is current address + size of two instructions
jmp _MyFunction2
```

这时栈的视图如下，即ebp中的地址是旧的ebp的地址。

```
:    : 
|  2 | [ebp + 16] (3rd function argument)
|  5 | [ebp + 12] (2nd argument)
| 10 | [ebp + 8]  (1st argument)
| RA | [ebp + 4]  (return address)
| FP | [ebp]      (old ebp value)
|    | [ebp - 4]  (1st local variable)
:    :
:    :
|    | [ebp - X]  (esp - the current stack pointer. The use of push / pop is valid now)
```



一个标准的Exit Sequence



## FPO

原文章[FPO](https://learn.microsoft.com/en-us/archive/blogs/larryosterman/fpo)

在了解什么是FPO的历史时，需要了解intel处理器的历史。



### Intel 8088

Intel 8088只有有限数量的寄存器，

| AX     | **BX** | CX     | DX     | IP        |
| ------ | ------ | ------ | ------ | --------- |
| **SI** | **DI** | **BP** | **SP** | **FLAGS** |

general purpose registers：AX，BX，CX，DX。

index registers：SI and DI



**Index register**，变址寄存器，一般用于存放段内偏移量。这一部分涉及到[x86的寻址方式](#x86 Addressing Mode)

变址寄存器的作用

```assembly
MOV    BX, [Structure]
MOV    AX, [BX]+4
```



而由于SP寄存器不是一个index register，因此无法通过SP来访问栈上的变量。而BP寄存器的作用就是为了访问栈上的数据。



### Intel 80386

| **EAX** | **EBX** | **ECX** | **EDX** | EIP       |
| ------- | ------- | ------- | ------- | --------- |
| **ESI** | **EDI** | **EBP** | **ESP** | **FLAGS** |

有6个寄存器可以用作index register，其中就包括ESP。

通过使用ESP作为index register，原先用EBP来访问栈内变量

```assembly
MyFunction:
    PUSH    EBP
    MOV     EBP, ESP
    SUB      ESP, <LocalVariableStorage>
    MOV     EAX, [EBP+8]
      :
      :
    MOV     ESP, EBP
    POP      EBP
    RETD
```

可以转变为如下形式

```assembly
MyFunction:
    SUB      SP, <LocalVariableStorage>
    MOV     EAX, [ESP+4+<LocalVariableStorage>]
      :
      :
    ADD     SP, <LocalVariableStorage>
    RETD
```

这样EBP寄存器就可以解放出来用作通用寄存器等。这种编译优化称之为**Frame Pointer Omission（FPO）**



### FPO存在的小问题

不带FPO优化的案例中，

```assembly
PUSH    EBP
MOV     EBP, ESP
```

这两条指令，使得所有函数调用的frame point连接成一个链表，因此可以构造出出整个call stacks，对于debug来说非常有用。

但对于FPO来说，无法追踪整个stack frames。

```
为了解决FPO的问题，compiler会把FPO优化丢失的信息写入一个PDB文件中，从而恢复所有的栈信息
```



### FPO的应用

```
FPO was enabled for all Windows binaries in NT 3.51, but was turned off for Windows binaries in Vista because it was no longer necessary - machines got sufficiently faster since 1995 that the performance improvements that were achieved by FPO weren't sufficient to counter the pain in debugging and analysis that FPO caused.
```



# x86 Addressing Mode


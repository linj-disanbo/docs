
# go语言函数调用

## 基本概念

 1. caller  调用者, 主动发起调用的一方
 1. callee  被调用者, 被调用的一方
 1. stack   栈内存, 运行实体(gorountine, thread 等)调用的函数的所有信息, 存放参数, 局部变量等的内存. 
 1. stack frame 栈帧, 运行实体调用函数在栈上对应的内存信息. 调用时增加一个栈帧, 返回时去掉一个栈帧.

## go函数调用

 1. SP/BP: 栈帧的顶部和底部.
 1. 栈是从大内存地址向小内存地址延伸. 

调用过程: main -> f1 -> f2
 1. [f1函数]在f1调用f2前: 计算调用f2(保存原BP, 参数返回值)需要的内存, 新栈帧的BP,SP, 即SP的减量(这个值编译器会算好, 汇编中直接出现这个值)
    1. 逻辑上在f1函数中有SP的值变化两次, 一次用于f1局部变量+一次用于调用f2需要内存. 
    1. 实际上可能一起调整到位. 
 1. [f1函数]SP减少(新SP),  保存原BP, 设置新BP. 
    1. 原SP - 8 = 新BP, 中间的8字节用来保存原BP, 即栈帧和栈帧之间留下一个指针的位置, 保存调用者的BP
 1. [f1函数]调整BP/SP.
    1. 从这里的逻辑看, 参数和返回值属于f1的栈帧; 但f2可以范围f1栈帧的参数和返回值的部分
 1. [f2函数]执行 ...
 1. [f2函数]恢复BP/SP
 1. [f1函数]恢复BP(保存在栈帧之间的),SP(同上, 编译器计算的)


## 对照

命令

```
go version 
# go version go1.15 linux/amd64
go tool compile -S -N -l call.go  > call.asm
```

SP计算

```
f1:
(call.go:11)       SUBQ    $80, SP

计算f1栈帧调整: BP+局部变量+调用f2所需内存, 一起计算的. 
              (1 + 2+2    +2+3) * 8 
              局部变量: x1,x2, 两个临时变量
```

栈帧空隙
 1. f1, f2栈帧(按BP~SP算栈帧)之间有一个指针大小的空隙, 存放调用者的设置好的BP
    1. 不是调用者(f1)本身BP, 是f1为f2准备的栈帧的BP. 即返回值,参数的起始位置

参数和返回值都在栈上
 1. 编译器实现更方便(不同机器的寄存器提供的不一样)
 1. 多值返回实现方便
    
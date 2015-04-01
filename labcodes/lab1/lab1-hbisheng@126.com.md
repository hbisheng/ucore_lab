# Lab 1实验报告
## 2012011307 黄必胜

> Lab 1 完成情况：参考lab1_result，完成lab1中的基本功能，即对时钟中断进行了正确的处理。没有实现challenge。以下为实验报告正文。

### 练习1
---
1.<b>操作系统镜像文件ucore.img是如何一步一步生成的？</b>

> 通过阅读makefile代码得知其依赖关系如下

```
* bin/ucore.img
   * bootblock
      * bootasm.o
         * bootasm.S
      * bootmain.o
         * bootmain.S
      * sign
   * kernel
       * kernel.ld init.o readline.o stdio.o kdebug.o kmonitor.o panic.o clock.o console.o intr.o picirq.o trap.o trapentry.o vectors.o pmm.o  printfmt.o string.o
```
	

2.<b>一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？</b>

> * 总大小最多为512字节
> * 第510字节为0x55
> * 第511字节为0xAA

### 练习2
---
1. <b>从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行。</b>
2. <b>在初始化位置0x7c00设置实地址断点,测试断点正常。</b>	
3. <b>从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。</b>
4. <b>自己找一个bootloader或内核中的代码位置，设置断点并进行测试。</b>

> 以上为qemu debug的操作练习，均已在完成lab1的实验中尝试。第三题中，反汇编得到的代码与bootasm.S和bootblock.asm一致。

### 练习3
---
1. <b>请分析bootloader是如何完成从实模式进入保护模式的。</b>

> * 将flag和段寄存器ds、es、ss置0
```
.code16
                cli
                cld
                xorw %ax, %ax
                movw %ax, %ds
                movw %ax, %es
                movw %ax, %ss
```
> * 将数据0xd1送到端口0x64，将数据0xdf送到端口0x60，开启A20
```
seta20.1:
                inb $0x64, %al
                testb $0x2, %al
                jnz seta20.1
                movb $0xd1, %al
                outb %al, $0x64
seta20.2:
                inb $0x64, %al
                testb $0x2, %al
                jnz seta20.2
                movb $0xdf, %al
                outb %al, $0x60
```
> * 加载GDT，使能cr0寄存器的PE位，从实模式进入保护模式
```
                lgdt gdtdesc
                movl %cr0, %eax
                orl $CR0_PE_ON, %eax
                movl %eax, %cr0
```
> * 跳转到32位地址下的下一条指令
```
                ljmp $PROT_MODE_CSEG, $protcseg
```
> * 设置保护模式下的段寄存器DS、ES、FS、GS、SS
```
.code32
protcseg:
                movw $PROT_MODE_DSEG, %ax
                movw %ax, %ds
                movw %ax, %es
                movw %ax, %fs
                movw %ax, %gs
                movw %ax, %ss
```
> * 建立栈指针EBP、ESP，表示栈空间为0~start，跳转到bootmain函数
```
                movl $0x0, %ebp
                movl $start, %esp
                call bootmain
```

### 练习4
---
1. <b>bootloader如何读取硬盘扇区的？</b>

> 分析readsect函数得出以下步骤：
* bootloader首先输出读入配置信息
* 将读取数量设为1，写入地址0x1F2
* 将32位的磁盘号分成四段，依次写入0x1F6~0x1F3
* 将命令0x20写入地址0x1F7，表示读取扇区
* 等待磁盘，准备读入数据
* 从地址0x1F0读入数据


2. <b>bootloader是如何加载ELF格式的OS？</b>

> 分析bootmain函数得出以下步骤：
* 从硬盘读入ELF文件头，大小为8个扇区
* readseg函数使用readsect函数循环从硬盘读取扇区
* 检查ELF文件是否符合要求
* 用e_phoff和e_phnum来创建程序头
* 同样使用readseg来读取程序头
* 调用ELF文件头的入口函数，进入下一步加载


### 练习5
---
1. <b>在lab1中完成kdebug.c中函数print_stackframe的实现，可以通过函数print_stackframe来跟踪函数调用堆栈中记录的返回地址。</b>

> * 已根据print_stackframe函数中的提示完成函数。
	

### 练习6
---
1. <b>中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？其中哪几位代表中断处理代码的入口？</b>

> 一个表项占8字节  
> 0~15位表示段偏移量低16位，16~31位表示段描述符，48~63位表示段偏移量高16位，这些数据共同描述了中断处理代码的入口

2. <b>编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init。在idt_init函数中，依次对所有中断入口进行初始化。使用mmu.h中的SETGATE宏，填充idt数组内容。每个中断的入口由tools/vectors.c生成，使用trap.c中声明的vectors数组即可。</b>
3. <b>编程完善trap.c中的中断处理函数trap，在对时钟中断进行处理的部分填写trap函数中处理时钟中断的部分，使操作系统每遇到100次时钟中断后，调用print_ticks子程序，向屏幕上打印一行文字”100 ticks”。</b>

> 已在代码中完成。


### 与标准答案的差异
---
> 由于参考了标准答案的思路，代码大部分相同。

### 本实验中重要的知识点
---
1. 基本工具的使用。如makefile, gdb等, 对C语言的函数指针也更有理解。
2. 详细了解了机器加电之后开始执行第一条指令的过程，包括bios和bootloader的执行过程等，深化了对计算机运行的理解。
3. ucore中的配置，IDT的建立、中断的处理等。

### OS原理中很重要但在实验中没有对应上的知识点
---
1. 如何建立GDT
2. 一些特殊寄存器各个位的作用还没有深究，比如使能等等

# Lab 1ʵ�鱨��
## 2012011307 �Ʊ�ʤ

> Lab 1 ���������ο�lab1_result�����lab1�еĻ������ܣ�û��ʵ��challenge������Ϊʵ�鱨�����ġ�

### ��ϰ1
---
1.<b>����ϵͳ�����ļ�ucore.img�����һ��һ�����ɵģ�</b>

> ͨ���Ķ�makefile�����֪��������ϵ����

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
	

2.<b>һ����ϵͳ��Ϊ�Ƿ��Ϲ淶��Ӳ��������������������ʲô��</b>

> * �ܴ�С���Ϊ512�ֽ�
> * ��510�ֽ�Ϊ0x55
> * ��511�ֽ�Ϊ0xAA

### ��ϰ2
---
1. <b>��CPU�ӵ��ִ�еĵ�һ��ָ�ʼ����������BIOS��ִ�С�</b>
2. <b>�ڳ�ʼ��λ��0x7c00����ʵ��ַ�ϵ�,���Զϵ�������</b>	
3. <b>��0x7c00��ʼ���ٴ�������,���������ٷ����õ��Ĵ�����bootasm.S�� bootblock.asm���бȽϡ�</b>
4. <b>�Լ���һ��bootloader���ں��еĴ���λ�ã����öϵ㲢���в��ԡ�</b>

> ����Ϊqemu debug�Ĳ�����ϰ�����������lab1��ʵ���г��ԡ��������У������õ��Ĵ�����bootasm.S��bootblock.asmһ�¡�

### ��ϰ3
---
1. <b>�����bootloader�������ɴ�ʵģʽ���뱣��ģʽ�ġ�</b>

> * ��flag�ͶμĴ���ds��es��ss��0
```
.code16
                cli
                cld
                xorw %ax, %ax
                movw %ax, %ds
                movw %ax, %es
                movw %ax, %ss
```
> * ������0xd1�͵��˿�0x64��������0xdf�͵��˿�0x60������A20
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
> * ����GDT��ʹ��cr0�Ĵ�����PEλ����ʵģʽ���뱣��ģʽ
```
                lgdt gdtdesc
                movl %cr0, %eax
                orl $CR0_PE_ON, %eax
                movl %eax, %cr0
```
> * ��ת��32λ��ַ�µ���һ��ָ��
```
                ljmp $PROT_MODE_CSEG, $protcseg
```
> * ���ñ���ģʽ�µĶμĴ���DS��ES��FS��GS��SS
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
> * ����ջָ��EBP��ESP����ʾջ�ռ�Ϊ0~start����ת��bootmain����
```
                movl $0x0, %ebp
                movl $start, %esp
                call bootmain
```

### ��ϰ4
---
1. <b>bootloader��ζ�ȡӲ�������ģ�</b>

> ����readsect�����ó����²��裺
* bootloader�����������������Ϣ
* ����ȡ������Ϊ1��д���ַ0x1F2
* ��32λ�Ĵ��̺ŷֳ��ĶΣ�����д��0x1F6~0x1F3
* ������0x20д���ַ0x1F7����ʾ��ȡ����
* �ȴ����̣�׼����������
* �ӵ�ַ0x1F0��������


2. <b>bootloader����μ���ELF��ʽ��OS��</b>

> ����bootmain�����ó����²��裺
* ��Ӳ�̶���ELF�ļ�ͷ����СΪ8������
* readseg����ʹ��readsect����ѭ����Ӳ�̶�ȡ����
* ���ELF�ļ��Ƿ����Ҫ��
* ��e_phoff��e_phnum����������ͷ
* ͬ��ʹ��readseg����ȡ����ͷ
* ����ELF�ļ�ͷ����ں�����������һ������


### ��ϰ5
---
1. <b>��lab1�����kdebug.c�к���print_stackframe��ʵ�֣�����ͨ������print_stackframe�����ٺ������ö�ջ�м�¼�ķ��ص�ַ��</b>

> * �Ѹ���print_stackframe�����е���ʾ��ɺ�����
	

### ��ϰ6
---
1. <b>�ж���������Ҳ�ɼ��Ϊ����ģʽ�µ��ж���������һ������ռ�����ֽڣ������ļ�λ�����жϴ���������ڣ�</b>

* һ������ռ8�ֽ�
* 0~15λ��ʾ��ƫ������16λ��16~31λ��ʾ����������48~63λ��ʾ��ƫ������16λ����Щ���ݹ�ͬ�������жϴ����������

2. <b>�������kern/trap/trap.c�ж��ж���������г�ʼ���ĺ���idt_init����idt_init�����У����ζ������ж���ڽ��г�ʼ����ʹ��mmu.h�е�SETGATE�꣬���idt�������ݡ�ÿ���жϵ������tools/vectors.c���ɣ�ʹ��trap.c��������vectors���鼴�ɡ�</b>
3. <b>�������trap.c�е��жϴ�����trap���ڶ�ʱ���жϽ��д���Ĳ�����дtrap�����д���ʱ���жϵĲ��֣�ʹ����ϵͳÿ����100��ʱ���жϺ󣬵���print_ticks�ӳ�������Ļ�ϴ�ӡһ�����֡�100 ticks����</b>

> ���ڴ�������ɡ�


### ���׼�𰸵Ĳ���
---
> ���ڲο��˱�׼�𰸵�˼·������󲿷���ͬ��

### ��ʵ������Ҫ��֪ʶ��
---
1. �������ߵ�ʹ�á���makefile,gdb��,��C���Եĺ���ָ��Ҳ������⡣
2. ��ϸ�˽��˻����ӵ�֮��ʼִ�е�һ��ָ��Ĺ��̣�����bios��bootloader��ִ�й��̵ȣ���˶Լ�������е���⡣
3. ucore�е����ã�IDT�Ľ������жϵĴ���ȡ�

### OSԭ���к���Ҫ����ʵ����û�ж�Ӧ�ϵ�֪ʶ��
---
1. ��ν���GDT
2. һЩ����Ĵ�������λ�����û�û���������ʹ�ܵȵ�

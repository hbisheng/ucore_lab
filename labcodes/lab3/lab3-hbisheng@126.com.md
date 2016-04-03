# Lab 3实验报告
## 2012011307 黄必胜


### 练习1
---
1.	<b>完成do_pgfault（mm/vmm.c）函数，给未被映射的地址映射上物理页。</b>

	> 根据提示中所提到的两个函数get_pte、pgdir_alloc_page来完成。其中get_pte函数在lab2中已实现,用于返回pte在kernel中地址,如果不存在的话会进行创建.
	 	
```

	ptep = get_pte(mm->pgdir, addr, 1);
	if (*ptep == 0) {

    	pgdsir_alloc_page(mm->pgdir, addr, perm);
    }

```

2.	<b>请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中组成部分对ucore实现页替换算法的潜在用处。</b>

	> 页目录项和页表项除了提供地址信息，其他位依然提供了许多有意义的信息；例如该项对应的是物理内存地址，还是虚拟存储的磁盘位置，以及可访问的权限等。对于页替换算法而言，例如时钟算法和LRU算法都可以使用这些位作为标记，记录该页是否被修改，访问的次数等。
	
3.	<b>如果ucore的缺页服务例程在执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？</b>

	> 此时会出现嵌套的中断，堆栈中会再次压入缺页服务例程的中断线程，而CPU会再次进入中断服务例程。

### 练习2
---
1.	<b>完成vmm.c中的do_pgfault函数。</b>
	
	> 提示中给出的思路很清晰，按照其思路，先通过swap_in函数，从磁盘交换分区中读出需要的页内容，然后通过page_insert函数建立虚拟地址到物理地址的映射,最后通过swap_map_swappable函数设置页的可交换性。

```

	ptep = get_pte(mm->pgdir, addr, 1);
    if (*ptep == 0) {
    	pgdir_alloc_page(mm->pgdir, addr, perm);
    } else {
    	if (swap_init_ok) {
    		struct Page *page = NULL;
    		swap_in(mm, addr, &page);
    		page_insert(mm->pgdir, page, addr, perm);
    		swap_map_swappable(mm, addr, page, 1);
    	} else {
    		cprintf("no swap_init_ok but ptep is %x, failed\n",*ptep);
    		goto failed;
    	}
    }

```

2.	<b>在实现FIFO算法的swap_fifo.c中完成map_swappable函数。</b>

	> _fifo_map_swappable函数的开始部分已经取出了两个所需的list_entry_t，再按照提示将entry放到head后面即可。


```

	list_add_after(head, entry);

```

3.	<b>在实现FIFO算法的swap_fifo.c中完成swap_out_vistim函数。</b>

	> 先把head前面的list_entry_t删除，然后取出链表中对应的页给ptr_page赋值。

```

	list_entry_t *le = head->prev;
     struct Page *p = le2page(le, pra_page_link);
     list_del(le);
     *ptr_page = p;

```

4.	<b>如果要在ucore上实现"extended clock页替换算法"，现有的swap_manager框架是否足以支持在ucore中实现此算法？如果是，请给你的设计方案。如果不是，请给出你的新的扩展和基此扩展的设计方案。</b>
	
	> * extended clock页替换算法必须维护一个环形链表，然后维护一个指针，指向目前的页，而目前的ucore中有双向链表，可以再添加一个指针指向当前页，此外还要使用页表项中的某些位来记录使用位和修改位；理论上应该可行。
	> * 需要被换出的页的特征是什么？使用位和修改位都为0
	> * 在ucore中如何判断具有这样特征的页？可以判断页表项中的特定位
	> * 何时进行换入和换出操作？换入操作：缺页异常时 换出操作：物理内存分配失败时

### 与标准答案的差异
---

	> 由于提示中给的思路比较清晰，按照其实现之后发现与标准答案差异不大...相比标准答案只是少了一些NULL检查以及assert的检验。


### 本实验中重要的知识点

	> * 虚拟存储的实现方式和大致框架
	> * 了解了如何判断、处理页缺失异常，以及换入换出操作
	> * FIFO算法的具体实现过程

	

### OS原理中很重要但在实验中没有对应上的知识点
	
	> 整个缺页异常处理的流程，从页缺失开始，到中断处理，到调用虚拟存储，到换入换出，到重新执行指令。这些流程的追踪在实验练习中未有要求，需要自己去尝试。
	> 其他页替换算法的实现等。
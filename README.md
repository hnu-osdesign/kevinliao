# kevinliao
廖琨的仓库

```
#include<stdio.h>
int a=10;  
int *show() 
{  
	return &a;  
}  
int main()  
{  
	printf("%d\n",a);  
	*show()=20;  
	printf("%d\n",a);  
}   

```

## src/arch/x86_64/start.rs#
### kstart()

1. 函数kstrat用于内核设置中断处理程序，生成中断表以及正确的条目，条目被定义在"arch"模块中  
2. BSS_TEST_ZERO标志应初始化为0  
3. DATA_TEST_NONZERO应初始化为全1  
4. 初始化日志并不适用println!宏而完成输出  
	- init_logger() in src/log.rs

5. 移出不必要的内核参数  
6. 初始化GDT,IDT  
7. 初始化内存管理 in src/memory/mod.rs   
	- 从bootloader复制到memory map  
8. 初始化分页  
	- in src/arch/x86_64/paging/mod.rs 
	- 过程非常复杂  
9. 设置分段with TLS?  
10. idt中断表初始化  
11. 初始化syscall  
12. Test tdata and tbss  
13. 重设AP的值。  
14. 设置内核堆  
	- 添加自动调整大小的链表分配器  
	- 修复exec中的内存泄漏  
	- 删除警告  
15. 为不同的处理器使用不同的IDT  
16. log初始化  
17. 初始化图形化debug  
18. 初始化设备  
19. 初始化acpi（高级配置和电源管理接口）和设备。  
20. 初始化非核心设备  
21. 再次初始化内存  
22. 停止图形化debug  
23. BSP_READY=true;  
24. 调用kmain();  

### kstart()_ap

1. cheak BSS and DATA  
2. init gdt  
3. init idt  
4. init paging  
5. set up gdt with tls  
6. set up idt for AP!  
7. init syscall  
8. Test tdata and tbss  
9. init_ap device  
10. 等待BSP_READY  
11. 调用kamin_ap();  

### usermode()
没看

## src/lib.rs 
### kmain（cpu核数，环境变量） 

1. 初始化CPU_ID,CPU_COUNT；  
2. 初始化上下文；  
3. 初始化pid；  
4. 初始化userspace  
	- 更改当前工作目录 chdir();in syscall/fs.rs  
	- kexec_kernel() in syscall/process.rs   

5. 循环  
	- 清除中断  
	- 启动中断并停机或暂停  

## redox-os\src\memory模块 虚拟地址到物理地址的映射
共有三个文件bump.rs,mod.rs,recycle.rs
1. mod.rs
 - `struct MemoryArea`一块内存，{base_addr,length,_type,acpi}
 - `struct MemroyAreaIter`内存的迭代器,{_type,i}
 	- new（_type）i=0,_type
 	- `impl Iterator next()  self+1`如果MemoryArea和MemroyAreaIterd的_type类型相等则返回一个Some否则None
 - struct Frame：帧号，{number}
 	- fn start_address(&self) -> PhysicalAddress 传回某一帧的起始物理地址（帧号×页大小） 
 	- fn clone(&self) ->Frame 返回当前帧的一个复制。
 	- fn clone containing_address(address) -> Frame 计算物理地址的帧号(address.get()/PAGE_SIZE)
 	- fn range_inclusive(start,end) -> FrameIter 返回一个FramIter迭代器，保存start和end值。
 - struct FramIter:帧的迭代器,{start,end}
 	- impl Iterator next() 如果start<=end则start+1,并返回Some否则None
 - stait FrameAllocator
	- `fn set_noncore() fn free_freames() fn used_frames() fn allocate_frames3() fn deallocate_frames()`方法的签名  
	- `fn allocate_frames(self, size) -> Option&lt;Frame>`的默认方法:调用`allocate_frames2(size,PhysallocFlags::SPACE_64)` 
其中SPACE_64 = 0x0000_0002，但参数其实是一个PhysallocFlags类型  
	- `fn allocate_frames2(self, size, flags) -> Option&lt;Frame>`的默认方法调用`allocate_frames3(size, flags, None, size).map(|(s, _)| s)`
 - `static MEMORY_MAP[MemoryArea;512]`
 - `static ALLOCATOR:Mutex&lt;Option&lt;RecycleAllocator&lt;BumpAllocator>>>`
 - `fn init()`初始化： 以0x500作为基地址存储MEMORY_MAP,保存信息到info！，获取一个互斥锁`*ALLOCATOR.lock() = Some(RecycleAllocator::new(BumpAllocator::new(kernel_start, kernel_end, MemoryAreaIter::new(MEMORY_AREA_FREE))));`
 - `fn init_noncore()初始化： 申请互斥锁对象ALLOCATOR,取出allocator：RecycleAllocator&lt;BumpAllocator>,调用allocator.set_noncore(true)否则panic！
 - `fn free_frames()-> usize` 可用帧数：申请互斥锁对象ALLOCATOR,取出allocator：RecycleAllocator&lt;BumpAllocator>,调用allocator.free_frames否则panic！
 - `fn used_frames()->usize` 已用帧数：申请互斥锁对象ALLOCATOR,取出allocator：RecycleAllocator&lt;BumpAllocator>,调用allocator.used_frames否则panic！
 - `fn allocate_frames(size)->Option(Frame)` 分配一定数量的帧：申请互斥锁对象ALLOCATOR,取出allocator：RecycleAllocator&lt;BumpAllocator>,调用allocator.allocator(size)否则panic！
 - `fn allocate_frames_complex(count,flags,strategy,min)` 分配一定数量的帧：申请互斥锁对象ALLOCATOR,取出allocator：RecycleAllocator&lt;BumpAllocator>,调用allocator.allocate_frames3否则panic！
 - `fn deallocate_frames(frame,count) `释放一定数量的帧：申请互斥锁对象ALLOCATOR,取出allocator：RecycleAllocator&lt; BumpAllocator>,调用allocator.deallocate_frames（frame,count）否则panic！  
2. bump.rs
 - BumpAllocator{next_free_frame,current_area,areas,kernel_start,kernel_end}

   
3. recycle.rs
 - struct Range:范围:{base,count}
 - struct RecycleAllocator:{inner,noncore,free}  

# redox-os/src/allcator 堆分配器

共有三个文件linked_list.rs,mod.rs,slab.rs  

1. linked_list.rs一个链表分配器

   - `use core::alloc::{GlobalAlloc, Layout}` 调用核心库
     `use linked_list_allocator::Heap`并且定义了一个静态的分配器HEAP
     `GlobalAlloc trait`规定了对分配器的规则，`GlobalAlloc`含有方法`alloc`和`dealloc  `
   - `pub unsafe fn init(offset: usize, size: usize)`获取一个HEAP规则是list_list
   - `unsafe fn alloc(&self, layout: Layout) -> *mut u8`定义了分配内存的规则，[`Layout`](https://translate.googleusercontent.com/translate_c?depth=1&rurl=translate.google.com.hk&sl=en&sp=nmt4&tl=zh-CN&u=https://doc.rust-lang.org/alloc/alloc/struct.Layout.html&xid=25657,15700023,15700186,15700190,15700256,15700259,15700262,15700265,15700271&usg=ALkJrhgfbVGb4vhKLhix2Ocb5VA9y29OfA)实例作为参数，该实例描述分配的内存应具有的所需大小和对齐方式。  
     - 分配规则：`linked_list_allocator::Heap::allocate_first_fit;`使函数扫描可用内存块的列表，并使用足够大的第一个块，分配给定大小的块。如果成功，则返回指向该块开头的指针。否则它返回`None`。
   - unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout);`接收地址释放堆内存。  

1. slab.rs 一个slab分配器  

   - `use slab_allocator::Heap`
   - `pub unsafe fn init(offset: usize, size: usize)`获取一个HEAP,规则是slab_allocator
   - 方法`alloc``dealloc``oom``usable_size`
   - `usable_size`返回一个成功分配内存的范围
   - `omm`封装panic!宏

1. mod.rs    

   - `use crate::paging::{ActivePageTable, Page, VirtualAddress};`

     `use crate::paging::entry::EntryFlags;`

     `use crate::paging::mapper::MapperFlushAll;`

   - `fn map_heap`初始化虚拟页，手动刷新TLB。
   - `fn init`初始化内核堆，调用`map_heap`，初始化一个分配器。



# src/arch/x86_64/start.rs#
## kstart()

1.函数kstrat用于内核设置中断处理程序，生成中断表以及正确的条目，条目被定义在"arch"模块中
2.BSS_TEST_ZERO标志应初始化为0
3.DATA_TEST_NONZERO应初始化为全1
4.初始化日志并不适用println!宏而完成输出
- init_logger() in src/log.rs

5.移出不必要的内核参数
6.初始化GDT,IDT
7.初始化内存管理 in src/memory/mod.rs
- 从bootloader复制到memory map

8.初始化分页
- in src/arch/x86_64/paging/mod.rs 
- 过程非常复杂

9.设置分段with TLS?
10.idt中断表初始化
11.初始化syscall
12.Test tdata and tbss
13.重设AP的值。
14.设置内核堆
- 添加自动调整大小的链表分配器
- 修复exec中的内存泄漏
- 删除警告

15.为不同的处理器使用不同的IDT
16.log初始化
17.初始化图形化debug
18.初始化设备
19.初始化acpi（高级配置和电源管理接口）和设备。
20.初始化非核心设备
21.再次初始化内存
22.停止图形化debug
23.BSP_READY=true;
24.调用kmain();

## kstart()_ap

1.cheak BSS and DATA
2.init gdt
3.init idt
4.init paging
5.set up gdt with tls
6.set up idt for AP!
7.init syscall
8.Test tdata and tbss
9.init_ap device
10.等待BSP_READY
11.调用kamin_ap();

##usermode()##
没看

# src/lib.rs #
## kmain（cpu核数，环境变量） ##

1.初始化CPU_ID,CPU_COUNT；
2.初始化上下文；
3.初始化pid；
4,初始化userspace
- 更改当前工作目录 chdir();in syscall/fs.rs
- kexec_kernel() in syscall/process.rs 

5.循环
- 清除中断
- 启动中断并停机或暂停

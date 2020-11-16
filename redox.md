# src/lib.rs #
## kmain（cpu核数，环境变量） ##
初始化CPU_ID,CPU_COUNT；
初始化上下文；
初始化pid；
初始化userspace
- 更改当前工作目录 chdir();in syscall/fs.rs
- kexec_kernel() in syscall/process.rs 

循环
- 清除中断
- 启动中断并停机或暂停
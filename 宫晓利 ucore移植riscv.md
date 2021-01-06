## 宫晓利 ucore移植riscv



1. 用opensbi或者rustsbi封装底层实现，减少跟底层打交道。

   - rustsbi：https://github.com/luojia65/rustsbi
   - opensbi：https://github.com/riscv/opensbi

2. riscv工具链for rust

   - https://github.com/riscv-rust/riscv-rust-toolchain
   - sifive打包好的二进制文件（不知道能不能用）
     https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.0-x86_64-linux-ubuntu14.tar.gz
   - 或者cargo工具就能下载？
     rustup target add riscv64imac-unknown-none-elf
     对着rcore来比较好https://rcore-os.github.io/rCore-Tutorial-deploy/
     第四版的rcore将会和第三版以完全不同的架构介绍操作系统，从应用的角度出发，以应用驱动内核的方式来构建。

3. 从裸机程序完成lab1-lab8
   https://nankai.gitbook.io/ucore-os-on-risc-v64/

4. 完成程序构建

5. 在k210上烧录程序

   - Python3环境

   - 串口转USB驱动
   - 串口终端工具
   - 烧录工具
   - bootloader与内核镜像共同打包的镜像文件

6. 没有模拟器
   会出现qemu与实际硬件跑的不一样的情况
   据说rustsbi已经支持逐步调试了。


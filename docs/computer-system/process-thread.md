# 进程和线程的区别



进程是资源分配的最小单位

线程是 CPU 调度的最小单位

 

32bit 寻址空间是 4gb

64bit 寻址空间是 256tb

一半是内核

 

线程间共享的包括：文件描述符 fd、信号量 signal、

不可以共享：寄存器 register、线程之间不共享 heap

 

栈、堆（方向是一上一下，具体方向：）

 

每个线程都有自己的 stack、

 

Address Space Layout Random：地址空间布局随机化

防部分 hack 攻击

ASLR 的作用范围：heap、stack、libraries、PIE

 

32位系统的最大线程数：256 个

 

线程切换的代价会比进程小？

Linux 下没有进程和线程之分，统一都是 kernel thread 调度时并不会区别对待

clone（）vs fork（）

clone 的 fd、signal、memspace，用的同一份，线程

fork，全部独立，进程

 

进程\线程切换时切换的是寄存器、PGD

 
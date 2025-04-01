##### 本文纪录一下在扩展开源索引ord的过程中遇到的一个问题，以及最后是如何被解决的
> ord是rust实现的bitcoin索引程序，可以解析btc链的交易、地址、铭文等数据。

在我扩展ord实现fractal链上的brc20-swap功能的时候，遇见了一个交易：
c39653fc9f9c3b73a4f372c4d66aa1b8b6a7beb81d3f7ed39433e1e314d908fb，
1. 这个交易涉及到5千万的铭文转移，直接把ord干跪了，memory allocation of 4114 bytes failed，内存不够了。
2. 但是不应该呀，我系统有200G的内存，而`top`观察到虚拟内存使用不过才108G内存，不应该被kill掉呀。
3. 本能想到先通过`dmesg`看一下，没有OOM纪录，看来不是因为OOM被kill掉的。这点符合实际108G的内存使用 < 机器有200G内存的事实。
4. 那应该是什么地方有进程内存使用限制导致的了。
   1. 先检查下docker-compose.yaml的memlimit,没有限制
   2. rust程序本身也没有设置限制
   3. 那大概是操作系统的限制了
5. 先看一下`cat /proc/meminfo | grep Commit`
```
CommitLimit: 约100+G //操作系统承诺可以给进程的虚拟内存总量
Committed_AS: 约100+G //操作系统已经给所有进程的虚拟内存总量
```
Committed_AS的值比CommitLimit的略微小一些，我感觉似乎找到了问题所在，于是修改：
```
vm.overcommit_ratio = 100 //默认值50%，承诺分配给进程的虚拟内存大小，可以调整上文提到的CommitLimit
```
6. 于是就重新跑了下，发现还是不行，还是报错：memory allocation of 4114 bytes failed。那肯定别的地方还有限制，一顿调研之后，发现了`vm.max_map_count`这个参数，这个参数控制着一个进程最多可以创建多少个VMA(Virtual Memory Areas， 虚拟内存区域)，默认值为65530(`cat /proc/sys/vm/max_map_count`)，Memory Map是一种将文件或设备映射到进程的虚拟地址空间的技术, 对于频繁申请内存的应用来说，这个值需要设置更大一些。
   1. 为了验证是否是这个参数导致的，于是我重新跑起了程序，进入到/proc/{pid_of_the_progress}并且执行`while true; do sudo cat maps | wc -l; sleep 0.1; done`, 实际观察到当程序崩溃的时候，这个值也达到了65530, 看起来确实是这个参数引起的。
   2. 调大了这个参数，也确实成功了

---
title: Linux查看系统硬件信息
date: 2019-07-29 13:36:21
tags:
  - Linux
  - Systerm
categories:
  - Linux
---

# Linux查看系统硬件信息

## 查看当前操作系统内核信息

```
chenbin@chenbin-ThinkPad:~$ uname -a
Linux chenbin-ThinkPad 4.15.0-36-generic #39-Ubuntu SMP Mon Sep 24 16:19:09 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

## 查看当前操作系统发行版信息

```
chenbin@chenbin-ThinkPad:~$ cat /etc/issue
Ubuntu 18.04.1 LTS \n \l
```

## CPU

### CPU详细信息

```
ubuntu@mini-11:~$ lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit         #CPU 运行模式
Byte Order:          Little Endian          #字节序
CPU(s):              4                      #CPU个数
On-line CPU(s) list: 0-3                    #在线 CPU 列表
Thread(s) per core:  2                      #每个核的线程数
Core(s) per socket:  2                      #每个cpu插槽核数/每颗物理cpu核数
Socket(s):           1                      #CPU插槽数
NUMA node(s):        1                      #NUMA 节点
Vendor ID:           GenuineIntel           #厂商 ID
CPU family:          6                      #CPU系列
Model:               69                     #型号
Model name:          Intel(R) Core(TM) i7-4600U CPU @ 2.10GHz
Stepping:            1
CPU MHz:             2290.247               #CPU MHz
CPU max MHz:         3300.0000              #CPU 最大 MHz
CPU min MHz:         800.0000               #CPU 最小 MHz
BogoMIPS:            5387.11
Virtualization:      VT-x
L1d cache:           32K                #一级缓存32K（google了下，这具体表示表示cpu的L1数据缓存为32k）
L1i cache:           32K                #一级缓存32K（具体为L1指令缓存为32K）
L2 cache:            256K               #二级缓存256K
L3 cache:            4096K              #三级缓存4096K
NUMA node0 CPU(s):   0-3
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand lahf_lm abm cpuid_fault epb invpcid_single pti tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts
```

> NOTE: 查看```/proc/cpuinfo```,可以知道每个cpu信息，如每个CPU的型号，主频等

### 查看物理CPU个数
```
[root@DB-Server ~]# cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l

[root@DB-Server ~]# dmesg | grep CPU | grep "Physical Processor ID" | uniq | wc -l
```

### 查看逻辑CPU个数
```
[root@DB-Server ~]# cat /proc/cpuinfo | grep "processor" | wc -l

[root@DB-Server ~]# dmesg | grep "CPU" | grep "processor" | wc -l
```

### 查看CPU是几核的
```
[root@DB-Server ~]# cat /proc/cpuinfo | grep "cores" | uniq

cpu cores : 3
```

### 查看CPU的主频
```
[root@DB-Server ~]# cat /proc/cpuinfo | grep MHz | uniq
cpu MHz : 800.000

[root@DB-Server ~]# cat /proc/cpuinfo | grep MHz
cpu MHz : 800.000
cpu MHz : 800.000
cpu MHz : 800.000
```

### 查看CPU型号信息
```
[root@DB-Server ~]# cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c

3 AMD Athlon(tm) II X3 450 Processor
```

### 通过physical id 可以判断物理CPU个数
```
[root@DB-Server ~]# cat /proc/cpuinfo | grep physical | uniq -c
physical id : 0
address sizes : 48 bits physical, 48 bits virtual
physical id : 0
address sizes : 48 bits physical, 48 bits virtual
physical id : 0
address sizes : 48 bits physical, 48 bits virtual
```

### 查看CPU是否支持64位运算
```
[root@DB-Server ~]# cat /proc/cpuinfo | grep flags | grep 'lm' | wc -l
3

结果大于0，说明支持64位运算，lm指long mode 支持lm则是64bit
```
```
[root@DB-Server ~]# getconf LONG_BIT
etl:/home/etl/$getconf LONG_BIT（另外一台服务器）

说明当前CPU运行在32位模式下，当不代表CPU不支持64位
```

## 查看内存信息
```
[root@DB-Server ~]# more /proc/meminfo
MemTotal:        7541288 kB
MemFree:          215388 kB
Buffers:          186228 kB
Cached:          6433572 kB
SwapCached:        77404 kB
Active:          5489928 kB
Inactive:        1346252 kB
Active(anon):    5193596 kB
Inactive(anon):  1015024 kB
Active(file):     296332 kB
Inactive(file):   331228 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:       9781240 kB
SwapFree:        9430432 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:        139432 kB
Mapped:          3878064 kB
Shmem:           5992240 kB
Slab:             328284 kB
SReclaimable:     159572 kB
SUnreclaim:       168712 kB
KernelStack:        2056 kB
PageTables:        99256 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    13551884 kB
Committed_AS:    6943792 kB
VmallocTotal:   34359738367 kB
VmallocUsed:      301620 kB
VmallocChunk:   34359431420 kB
HardwareCorrupted:     0 kB
AnonHugePages:     30720 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:        8128 kB
DirectMap2M:     2611200 kB
DirectMap1G:     5242880 kB
```
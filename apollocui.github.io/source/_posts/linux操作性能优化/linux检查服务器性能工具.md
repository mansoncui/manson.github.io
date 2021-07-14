---
layout: post
title: linux检查服务器性能工具
date: 2019-06-18 14:55:04
tags:   
    - [ linux检查服务器性能工具 ]
categories:
    - [ linux操作性能优化 ]
---

---
## linux检查服务器性能工具
```
1.出现弹窗后,在蓝色方框内,字母为OEM的则正版或RETAIL为零售版也是正版; 
2.如果出现的字母是VOLUME则为批量激活,即为盗版

vmstat #查询全局资源使用情况
选项:
-a, --active           active/inactive memory        # 显示活跃和非活跃内存
-f, --forks            number of forks since boot    # 显示从系统启动至今的fork数量 
-m, --slabs            slabinfo                       # 显示slabinfo
-n, --one-header       do not redisplay header         # 只在开始时显示一次字段名称
-s, --stats            event counter statistics       # 显示内存相关的统计信息及多种系统活动数量
-d, --disk             disk statistics                # 显示磁盘相关统计信息
-D, --disk-sum         summarize disk statistics      # 显示磁盘的总计信息
-p, --partition <dev>  partition specific statistics  # 显示指定磁盘分区统计信息
-S, --unit <char>      define display unit            # 使用指定单位显示。参数有 k 、K 、m 、M ，分别代表1000、1024、1000000、1048576字节（byte）。默认单位为K（1024 bytes）
-w, --wide             wide output                    # 更宽的显示信息
-t, --timestamp        show timestamp                 # 显示时间

-h, --help     display this help and exit
-V, --version  output version information and exit

pidstat #查询某个进程资源使用情况（yum install sysstat）
选项:
-u：默认的参数，显示各个进程的cpu使用统计
-r：显示各个进程的内存使用统计
-d：显示各个进程的IO使用情况
-p：指定进程号
-w：显示每个进程的上下文切换情况
-t：显示选择任务的线程的统计信息外的额外信息
-T { TASK | CHILD | ALL }
这个选项指定了pidstat监控的。TASK表示报告独立的task，CHILD关键字表示报告进程下所有线程统计信息。ALL表示报告独立的task和task下面的所有线程。
注意：task和子线程的全局的统计信息和pidstat选项无关。这些统计信息不会对应到当前的统计间隔，这些统计信息只有在子线程kill或者完成的时候才会被收集。
-V：版本号
-h：在一行上显示了所有活动，这样其他程序可以容易解析。
-I：在SMP环境，表示任务的CPU使用率/内核数量
-l：显示命令名和所有参数
案列:
pidstat 和 pidstat -u -p ALL 是等效的
pidstat 默认显示了所有进程的cpu使用率

dstat   #同时查看CPU和IO资源使用情况（yum install -y dstat）
选项:
-c：显示CPU系统占用，用户占用，空闲，等待，中断，软件中断等信息。
-C：当有多个CPU时候，此参数可按需分别显示cpu状态，例：-C 0,1 是显示cpu0和cpu1的信息。
-d：显示磁盘读写数据大小。
-D hda,total：include hda and total。
-n：显示网络状态。
-N eth1,total：有多块网卡时，指定要显示的网卡。
-l：显示系统负载情况。
-m：显示内存使用情况。
-g：显示页面使用情况。
-p：显示进程状态。
-s：显示交换分区使用情况。
-S：类似D/N。
-r：I/O请求情况。
-y：系统状态。
--ipc：显示ipc消息队列，信号等信息。
--socket：用来显示tcp udp端口状态。
-a：此为默认选项，等同于-cdngy。
-v：等同于 -pmgdsc -D total。
--output 文件：此选项也比较有用，可以把状态信息以csv的格式重定向到指定的文件中，以便日后查看。例：dstat --output /root/dstat.csv & 此时让程序默默的在后台运行并把结果输出到/root/dstat.csv文件中。

starce  #跟踪某一进程和硬件进行交互过程
选项:
-o 把strace追踪数据追加到一个文本中
-p 指定进程pid
-t -tt 在每行输出前加上时间戳，-tt是更详细时间
-r 展示系统调用之间的相对时间戳
-c 是对输出数据格式化(是以整洁方式展示)
-e 选项仅仅被用来展示特定的系统调用（例如，open，write等等）
寻找被程序读取的配置文件:案列:strace php 2>&1 | grep php.ini

pidstat -d 选项是展示I/O 统计数据，用strace追踪某一个进程，以root用户追踪某一个进程时
显示没有权限，用ps检查该进程是否存在，或者看下该进程当前状态
案列:pidstat -d 1 20
-d 是展示I/O统计数据
1 20 表示1s内展示20组数据

pstree -aps 3084
选项详解:
-a 表示输出命令选项
p  表示进程pid
s  表示指定进程的父进程

perf record -g
perf report 

stress #压力测试工具
```
---
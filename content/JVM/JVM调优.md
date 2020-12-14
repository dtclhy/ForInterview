# JPS

查看java进程

# Jinfo

实时查看和调整JVM配置参数

查看JVM参数和java系统参数。

# Jstat

stat命令可以查看堆内存各部分的使用量，以及加载类的数量。命令的格式如下: jstat [-命令选项] [vmid] [间隔时间(毫秒)] [查询次数]

jstat -gc pid ，可以评估程序内存使用及GC压力整体情况。包括各个分区的大小和使用大小，垃圾回收次数，垃圾回收消耗时间

jstat -gccapaciy pid，堆内存统计。

jsta -gcnew pid，新生代垃圾回收统计。

# Jmap

查看内存信息

jmap -histo pid，实例个数以及占用内存大小，类名称

jmap -heap pid，堆信息

jmap -dump，堆内存dump

# Jstack

查看线程堆栈信息。jstack pid。

## top

1. top命令查看各个进程的cpu使用情况，默认按cpu使用率排序。

2. top -Hp pid 找出该进程内最耗费CPU的线程，将线程id转化为16进制

3. jstack pid 查看线程的堆栈信息。nid与上述16进制对应。或jstack pid|grep -A 10 nid

4. 查看对应的堆栈信息找出可能存在问题的代码

   








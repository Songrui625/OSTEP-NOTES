## Problem

### P1.只用两个工作和两个队列运行几个随机生成的问题。针对每个工作计算MLFQ的执行记录。限制每项作业的长度并关闭I/O，让你的生活更轻松。

如题所说，我们让两个作业的长度为20，

```bash
> python .\mlfq.py -l 0,20,0:0,20,0 -n 2 -A 20,20 -c
Here is the list of inputs:
OPTIONS jobs 2
OPTIONS queues 2
OPTIONS allotments for queue  1 is  20
OPTIONS quantum length for queue  1 is  10
OPTIONS allotments for queue  0 is  20
OPTIONS quantum length for queue  0 is  10
OPTIONS boost 0
OPTIONS ioTime 5
OPTIONS stayAfterIO False
OPTIONS iobump False


For each job, three defining characteristics are given:
  startTime : at what time does the job enter the system
  runTime   : the total CPU time needed by the job to finish
  ioFreq    : every ioFreq time units, the job issues an I/O
              (the I/O takes ioTime units to complete)

Job List:
  Job  0: startTime   0 - runTime  20 - ioFreq   0
  Job  1: startTime   0 - runTime  20 - ioFreq   0


Execution Trace:

[ time 0 ] JOB BEGINS by JOB 0
[ time 0 ] JOB BEGINS by JOB 1
[ time 0 ] Run JOB 0 at PRIORITY 1 [ TICKS 9 ALLOT 20 TIME 19 (of 20) ]
[ time 1 ] Run JOB 0 at PRIORITY 1 [ TICKS 8 ALLOT 20 TIME 18 (of 20) ]
[ time 2 ] Run JOB 0 at PRIORITY 1 [ TICKS 7 ALLOT 20 TIME 17 (of 20) ]
[ time 3 ] Run JOB 0 at PRIORITY 1 [ TICKS 6 ALLOT 20 TIME 16 (of 20) ]
[ time 4 ] Run JOB 0 at PRIORITY 1 [ TICKS 5 ALLOT 20 TIME 15 (of 20) ]
[ time 5 ] Run JOB 0 at PRIORITY 1 [ TICKS 4 ALLOT 20 TIME 14 (of 20) ]
[ time 6 ] Run JOB 0 at PRIORITY 1 [ TICKS 3 ALLOT 20 TIME 13 (of 20) ]
[ time 7 ] Run JOB 0 at PRIORITY 1 [ TICKS 2 ALLOT 20 TIME 12 (of 20) ]
[ time 8 ] Run JOB 0 at PRIORITY 1 [ TICKS 1 ALLOT 20 TIME 11 (of 20) ]
[ time 9 ] Run JOB 0 at PRIORITY 1 [ TICKS 0 ALLOT 20 TIME 10 (of 20) ]
[ time 10 ] Run JOB 1 at PRIORITY 1 [ TICKS 9 ALLOT 20 TIME 19 (of 20) ]
[ time 11 ] Run JOB 1 at PRIORITY 1 [ TICKS 8 ALLOT 20 TIME 18 (of 20) ]
[ time 12 ] Run JOB 1 at PRIORITY 1 [ TICKS 7 ALLOT 20 TIME 17 (of 20) ]
[ time 13 ] Run JOB 1 at PRIORITY 1 [ TICKS 6 ALLOT 20 TIME 16 (of 20) ]
[ time 14 ] Run JOB 1 at PRIORITY 1 [ TICKS 5 ALLOT 20 TIME 15 (of 20) ]
[ time 15 ] Run JOB 1 at PRIORITY 1 [ TICKS 4 ALLOT 20 TIME 14 (of 20) ]
[ time 16 ] Run JOB 1 at PRIORITY 1 [ TICKS 3 ALLOT 20 TIME 13 (of 20) ]
[ time 17 ] Run JOB 1 at PRIORITY 1 [ TICKS 2 ALLOT 20 TIME 12 (of 20) ]
[ time 18 ] Run JOB 1 at PRIORITY 1 [ TICKS 1 ALLOT 20 TIME 11 (of 20) ]
[ time 19 ] Run JOB 1 at PRIORITY 1 [ TICKS 0 ALLOT 20 TIME 10 (of 20) ]
[ time 20 ] Run JOB 0 at PRIORITY 1 [ TICKS 9 ALLOT 19 TIME 9 (of 20) ]
[ time 21 ] Run JOB 0 at PRIORITY 1 [ TICKS 8 ALLOT 19 TIME 8 (of 20) ]
[ time 22 ] Run JOB 0 at PRIORITY 1 [ TICKS 7 ALLOT 19 TIME 7 (of 20) ]
[ time 23 ] Run JOB 0 at PRIORITY 1 [ TICKS 6 ALLOT 19 TIME 6 (of 20) ]
[ time 24 ] Run JOB 0 at PRIORITY 1 [ TICKS 5 ALLOT 19 TIME 5 (of 20) ]
[ time 25 ] Run JOB 0 at PRIORITY 1 [ TICKS 4 ALLOT 19 TIME 4 (of 20) ]
[ time 26 ] Run JOB 0 at PRIORITY 1 [ TICKS 3 ALLOT 19 TIME 3 (of 20) ]
[ time 27 ] Run JOB 0 at PRIORITY 1 [ TICKS 2 ALLOT 19 TIME 2 (of 20) ]
[ time 28 ] Run JOB 0 at PRIORITY 1 [ TICKS 1 ALLOT 19 TIME 1 (of 20) ]
[ time 29 ] Run JOB 0 at PRIORITY 1 [ TICKS 0 ALLOT 19 TIME 0 (of 20) ]
[ time 30 ] FINISHED JOB 0
[ time 30 ] Run JOB 1 at PRIORITY 1 [ TICKS 9 ALLOT 19 TIME 9 (of 20) ]
[ time 31 ] Run JOB 1 at PRIORITY 1 [ TICKS 8 ALLOT 19 TIME 8 (of 20) ]
[ time 32 ] Run JOB 1 at PRIORITY 1 [ TICKS 7 ALLOT 19 TIME 7 (of 20) ]
[ time 33 ] Run JOB 1 at PRIORITY 1 [ TICKS 6 ALLOT 19 TIME 6 (of 20) ]
[ time 34 ] Run JOB 1 at PRIORITY 1 [ TICKS 5 ALLOT 19 TIME 5 (of 20) ]
[ time 35 ] Run JOB 1 at PRIORITY 1 [ TICKS 4 ALLOT 19 TIME 4 (of 20) ]
[ time 36 ] Run JOB 1 at PRIORITY 1 [ TICKS 3 ALLOT 19 TIME 3 (of 20) ]
[ time 37 ] Run JOB 1 at PRIORITY 1 [ TICKS 2 ALLOT 19 TIME 2 (of 20) ]
[ time 38 ] Run JOB 1 at PRIORITY 1 [ TICKS 1 ALLOT 19 TIME 1 (of 20) ]
[ time 39 ] Run JOB 1 at PRIORITY 1 [ TICKS 0 ALLOT 19 TIME 0 (of 20) ]
[ time 40 ] FINISHED JOB 1

Final statistics:
  Job  0: startTime   0 - response   0 - turnaround  30
  Job  1: startTime   0 - response  10 - turnaround  40

  Avg  1: startTime n/a - response 5.00 - turnaround 35.00
```

### P2.如何运行调度程序来重现本章中的每个实例？

**实例一:单个长工作**

```
> python .\mlfq.py -l 0,200,0 -n 3 -c
```

**实例二:来了一个短工作**

```bash
> python .\mlfq.py -l 0,200,0:100,20,0 -n 3 -c      
```

**实例三:如果有I/O呢**

```bash
> python .\mlfq.py -l 0,200,0:0,20,9 -n 3 -c
```

### P3.如何配置调度程序参数，像轮转调度程序那样工作？

只要将队列数为1并且量子长度为1即可。

```bash
-q 1 -n 1
```

###  P4.设计两个工作的负载和调度程序参数，以便一个工作利用较早的规则4a和4b（用-S标志打开）来“愚弄”调度程序，在特定的时间间隔内获得99％的CPU。








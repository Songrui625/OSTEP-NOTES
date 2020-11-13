# Problem

## P1

### 用以下标志运行程序：./process-run.py -l 5:100,5:100。CPU利用率(CPU使用时间的百分比)应该是多少？为什么你知道这一点？利用 -c 标记查看你的答案是否正确。  

```bash
> python .\process-run.py -l 5:100,5:100
Produce a trace of what would happen when you run these processes:
Process 0
  cpu
  cpu
  cpu
  cpu
  cpu

Process 1
  cpu
  cpu
  cpu
  cpu
  cpu

Important behaviors:
  System will switch when
the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will
run LATER (when it is its turn)
```

多次运行之后结果都如上所示，CPU利用率是100％，进程并没有发出IO请求。  

```bash
> python .\process-run.py -l 5:100,5:100 -c -p
Time    PID: 0    PID: 1       CPU       IOs
  1    RUN:cpu     READY         1
  2    RUN:cpu     READY         1
  3    RUN:cpu     READY         1
  4    RUN:cpu     READY         1
  5    RUN:cpu     READY         1
  6       DONE   RUN:cpu         1
  7       DONE   RUN:cpu         1
  8       DONE   RUN:cpu         1
  9       DONE   RUN:cpu         1
 10       DONE   RUN:cpu         1

Stats: Total Time 10
Stats: CPU Busy 10 (100.00%)
Stats: IO Busy  0 (0.00%)
```

10次指令时间，CPU都在运行。

## p2

### 现在用这些标志运行: ./process-run.py -l 4:100,1:0。这些标志制定了一个包含4条指令的进程（都要使用CPU），并且只是简单地发出I/O并等待它完成。完成这两个进程需要多长时间？利用-c检查你的答案是否正确。

```bash
> python .\process-run.py -l 4:100,1:0     
Produce a trace of what would happen when you run these processes:
Process 0
  cpu
  cpu
  cpu
  cpu

Process 1
  io

Important behaviors:
  System will switch when
the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will
run LATER (when it is its turn)
```

完成这两个进程需要运行时间为10，第一个进程运行时间为4，第二个进程运行时间5，其中CPU发出IO请求为1，剩余执行IO为4。进程2结束时间为1。  

```bash
> python .\process-run.py -l 4:100,1:0 -c -p
Time    PID: 0    PID: 1       CPU       IOs
  1    RUN:cpu     READY         1
  2    RUN:cpu     READY         1
  3    RUN:cpu     READY         1
  4    RUN:cpu     READY         1
  5       DONE    RUN:io         1
  6       DONE   WAITING                   1
  7       DONE   WAITING                   1
  8       DONE   WAITING                   1
  9       DONE   WAITING                   1
 10*      DONE      DONE

Stats: Total Time 10
Stats: CPU Busy 5 (50.00%)
Stats: IO Busy  4 (40.00%)
```

答案与预想一致

## P3

### 现在交换进程执行的顺序: ./process-run.py -l 1:0, 4:100。现在发生了什么？交换顺序是否重要？为什么？同样，用 -c 看看你的答案是否正确？

```bash
> python .\process-run.py -l 1:0,4:100        
Produce a trace of what would happen when you run these processes:
Process 0
  io

Process 1
  cpu
  cpu
  cpu
  cpu

Important behaviors:
  System will switch when
the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will
run LATER (when it is its turn)
```

My Answer：  

两个进程运行时间为10。  

进程0运行时间为5，其中CPU发出IO请求为1，执行IO为4。  

进程1运行时间为4，结束时间为1。

```bash
> python .\process-run.py -l 1:0,4:100 -c -p
Time    PID: 0    PID: 1       CPU       IOs
  1     RUN:io     READY         1
  2    WAITING   RUN:cpu         1         1
  3    WAITING   RUN:cpu         1         1
  4    WAITING   RUN:cpu         1         1
  5    WAITING   RUN:cpu         1         1
  6*      DONE      DONE

Stats: Total Time 6
Stats: CPU Busy 5 (83.33%)
Stats: IO Busy  4 (66.67%)
```

实际上情况不是设想那样的。  

进程0运行，CPU先发出IO请求，时间为1，进程0此时进入阻塞状态。CPU会切换到进程1运行，执行4次指令，时间为4。之后CPU会结束进程0和进程1，运行时间为1。  

运行时间一共是 1 + 1 + 4 = 6。  

由上述结果可以看出，交换顺序是十分重要的，为了保证CPU能更高效的利用（提高CPU的利用率），进程0再进行IO等待的时候，CPU会去运行进程1，减少空闲的时间，从而充分的利用CPU。

## P4

### 现在探索另一些标志。一个重要的标志是 -S，它决定了当进程发出IO时系统如何反应。将标志设置为SWITCH_ON_END，在进程进行IO操作时，系统将不会切换到另一个进程，而是等待进程完成。当你运行以下两个进程时，会发生什么情况？一个执行IO，另一个执行CPU工作。（-l 1:0,4:100 -c -S SWITCH_ON_END）

```bash
> python .\process-run.py -l 1:0,4:100 -c -S SWITCH_ON_END
Time    PID: 0    PID: 1       CPU       IOs
  1     RUN:io     READY         1
  2    WAITING     READY                   1
  3    WAITING     READY                   1
  4    WAITING     READY                   1
  5    WAITING     READY                   1
  6*      DONE   RUN:cpu         1
  7       DONE   RUN:cpu         1
  8       DONE   RUN:cpu         1
  9       DONE   RUN:cpu         1
```

进程0发出I/O请求，运行时间为1，系统等待进程完成IO操作，时间为4，此时切换到进程1，使用4次CPU，时间为4。

总运行时间为：1 + 4 + 4 = 9  

## P5

### 现在运行相同的进程，但切换行为设置，在等待I/O时切换到另一个进程（-l 1:0,4:100 -c -S SWITCH_ON_IO）。现在会发生什么？利用 -c 来确认你的答案是否正确。

```bash
> python .\process-run.py -l 1:0,4:100 -S SWITCH_ON_IO    
Produce a trace of what would happen when you run these processes:
Process 0
  io

Process 1
  cpu
  cpu
  cpu
  cpu

Important behaviors:
  System will switch when
the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will
run LATER (when it is its turn)
```

系统运行进程0，CPU发出I/O请求，时间为1，进程0进入等待状态，去执行I/O，系统切换到进程1，使用4次CPU，时间为4，进程1进入终止状态，时间为1。  

总运行时间为：1 + 4 + 1 = 6。  

CPU利用率：（1 + 4）/ 6 = 83.33％

使用I/O频率：4 / 6 = 66.67％

```bash
> python .\process-run.py -l 1:0,4:100 -c -p -S SWITCH_ON_IO  
Time    PID: 0    PID: 1       CPU       IOs
  1     RUN:io     READY         1
  2    WAITING   RUN:cpu         1         1
  3    WAITING   RUN:cpu         1         1
  4    WAITING   RUN:cpu         1         1
  5    WAITING   RUN:cpu         1         1
  6*      DONE      DONE

Stats: Total Time 6
Stats: CPU Busy 5 (83.33%)
Stats: IO Busy  4 (66.67%)
```

答案如预想一致。

## p6

### 另一个重要的行为是I/0完成时要做什么。利用-I IO_RUN_LATER，当I/O完成时，发出它的进程不一定马上运行。相反，当时运行的进程一直运行。当你运行这个进程组合时会发生什么？（./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p）系统资源是否被有效利用。

```bash
> python .\process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER
Produce a trace of what would happen when you run these processes:
Process 0
  io
  io
  io

Process 1
  cpu
  cpu
  cpu
  cpu
  cpu

Process 2
  cpu
  cpu
  cpu
  cpu

Process 3
  cpu
  cpu
  cpu
  cpu
  cpu

Important behaviors:
  System will switch when
the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will
run LATER (when it is its turn)
```

系统先运行进程0，CPU发出第一次I/O请求，时间为1；进程0进入等待状态，CPU切换到进程1，使用5次CPU，时间为5；CPU切换到进程2，使用5次CPU，时间为5；CPU切换到进程3，使用5次CPU，时间为5。  

系统切换到进程0，CPU发出第二次I/O请求，时间为1，进程0进入等待状态，时间为4。  

CPU发出第三次I/O请求，时间为1，进程0进入等待状态，时间为4。  

进程0结束，时间为1。

运行总时间：(1 + 5 + 5 + 5) + （1 + 4） + （1 + 4） + 1 = 27  

CPU利用率 (1 + 5 + 5 + 5 + 1 + 1) / 27 = 66.67％

执行IO (4 + 4 + 4) / 27 = 44.44％  

```bash
> python .\process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p
Time    PID: 0    PID: 1    PID: 2    PID: 3       CPU       IOs
  1     RUN:io     READY     READY     READY         1
  2    WAITING   RUN:cpu     READY     READY         1         1
  3    WAITING   RUN:cpu     READY     READY         1         1
  4    WAITING   RUN:cpu     READY     READY         1         1
  5    WAITING   RUN:cpu     READY     READY         1         1
  6*     READY   RUN:cpu     READY     READY         1
  7      READY      DONE   RUN:cpu     READY         1
  8      READY      DONE   RUN:cpu     READY         1
  9      READY      DONE   RUN:cpu     READY         1
 10      READY      DONE   RUN:cpu     READY         1
 11      READY      DONE   RUN:cpu     READY         1
 12      READY      DONE      DONE   RUN:cpu         1
 13      READY      DONE      DONE   RUN:cpu         1
 14      READY      DONE      DONE   RUN:cpu         1
 15      READY      DONE      DONE   RUN:cpu         1
 16      READY      DONE      DONE   RUN:cpu         1
 17     RUN:io      DONE      DONE      DONE         1
 18    WAITING      DONE      DONE      DONE                   1
 19    WAITING      DONE      DONE      DONE                   1
 20    WAITING      DONE      DONE      DONE                   1
 21    WAITING      DONE      DONE      DONE                   1
 22*    RUN:io      DONE      DONE      DONE         1
 23    WAITING      DONE      DONE      DONE                   1
 24    WAITING      DONE      DONE      DONE                   1
 25    WAITING      DONE      DONE      DONE                   1
 26    WAITING      DONE      DONE      DONE                   1
 27*      DONE      DONE      DONE      DONE

Stats: Total Time 27
Stats: CPU Busy 18 (66.67%)
Stats: IO Busy  12 (44.44%)
```

答案如预想一致。
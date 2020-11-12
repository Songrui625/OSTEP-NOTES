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


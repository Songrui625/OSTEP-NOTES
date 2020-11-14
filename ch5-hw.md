# Problem

### P1. 编写一个调用fork()的程序。在调用fork()之前，让主进程访问一个变量（例如 x ）并将其值设置为某个值（例如 100 ）。子进程中的变量有什么值？当子进程和父进程都改变x的值时，变量会发生什么？

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

int main(int argc, char *argv[]) {
    int x;
    x = 100;

    printf("main program (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed.\n");
        exit(1);
    } else if (rc == 0) {
        x++;
        printf("hello, I am child (pid:%d). the x is %d\n", (int) getpid(), x);
    } else {
        x--;
        printf("hello, I am parent of %d (pid:%d). the x is %d\n", rc, (int) getpid(), x);
    }

    return 0;
}
```

做一些约定：如果是子进程，那么x++；如果是父进程，那么x--。

```bash
main program (pid:1469)
hello, I am parent of 1470 (pid:1469). the x is 99
hello, I am child (pid:1470). the x is 101
```

可以看到，fork（）之后，主进程先执行程序，x--，x成了99；然后子进程却打印了101，而不是100（99 + 1），说明两个进程之间是不可见的，一个在主进程中x被更新为99，这一行为并没有被后来执行的子进程所感知到。  

子进程中的变量有x。当子进程和父进程都改变x的值时，它们互不影响。

### P2. 编写一个打开文件的程序（使用open()系统调用），然后调用fork()创建一个新进程，子进程和父进程都可以访问open()返回的文件描述符吗？当他们并发（即同时）写入文件时，会发生什么？

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>

#define MAX_LEN 100

char pathname[MAX_LEN];

int main(int argc, char *argv[]) {


    strcpy(pathname, "text.txt");
    int fd = open(pathname, O_RDONLY, 0644);
    if (fd < 0) {
        perror("open failed");
        exit(1);
    }
    printf("main program (pid:%d). The file description is %d\n", (int) getpid(), fd);

    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed");
        exit(0);
    } else if (rc == 0) {
        printf("I am child (pid:%d). The file description is %d\n", (int) getpid(), fd);
    } else {
        printf("I am parent of %d (pid:%d). The file description is %d\n", rc, (int) getpid(), fd);
    }

    return 0;
}
```

```bash
main program (pid:2198). The file description is 3
I am parent of 2199 (pid:2198). The file description is 3
I am child (pid:2199). The file description is 3
```

可以观察到，子进程和父进程都可以访问open()返回的文件描述符。  

修改一下代码，让它们并发（同时）写入文件。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>

#define MAX_LEN 100

char pathname[MAX_LEN];

int main(int argc, char *argv[]) {


    strcpy(pathname, "text.txt");

    int buffer_len = strlen("马老师发生甚么事了") + 1;
    char *buffer = (char*) malloc(buffer_len);
    strcpy(buffer, "马老师发生甚么事了");
    buffer[buffer_len] = '\0';

    int fd = open(pathname, O_RDWR, 0644);
    if (fd < 0) {
        perror("open failed");
        exit(1);
    }
    printf("main program (pid:%d). The file description is %d\n", (int) getpid(), fd);

    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed");
        exit(0);
    } else if (rc == 0) {
        printf("I am child (pid:%d). The file description is %d\n", (int) getpid(), fd);
    } else {
        printf("I am parent of %d (pid:%d). The file description is %d\n", rc, (int) getpid(), fd);
    }
    int n = write(fd, buffer, buffer_len - 1);
    if (n < 0) {
        perror("write STDOUT_FILENO");
        exit(1);
    }

    return 0;
}
```

观察文件内容

```txt
马老师发生甚么事了马老师发生甚么事了
```

可以看到，两个进程都执行了写文件，而内容却没有发生冲突，说明子进程和父进程之间执行的操作对对方来说是透明的。

### P3.使用fork()编写另一个程序，子进程应打印"hello",父进程应打印"goodbye"。你应该尝试并确保子进程始终先打印。你能否不在父进程调用wait()而做到这一点呢？


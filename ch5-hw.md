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

```c
int main(int argc, char *argv[]) {
    printf("main program (pid:%d)\n", (int) getpid());

    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed");
        exit(1);
    } else if (rc > 0) {
        sleep(0.1);
        printf("parent: goodbye. (pid:%d)\n", (int) getpid());
        
    } else {
        printf("child: hello. (pid:%d)\n", (int) getpid());
    }

    return 0;
}
```

考虑单核CPU（多核CPU可能可以同时调用，比较复杂。），如果不使用wait()，那么哪个进程先打印是由CPU的调度算法决定的。  

要想子进程先打印，可以让父进程先"睡"一会：调用sleep()系统调用。

```c
main program (pid:1458)
child: hello. (pid:1459)
parent: goodbye. (pid:1458)
```

### P4.编写一个调用fork()的程序，然后调用某种形式的exec()来运行程序/bin/ls。看看是否可以尝试exec()的所有变体，包括execl()、execle()、execlp()、execv()、execvp()和execvpe()。为什么同样的基本调用会有这么多变种？

**execvp()**

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
#include<sys/wait.h>

int main(int argc, char *argv[]) {
    printf("hello world (pid:%d)\n", (int) getpid());

    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed.\n");
        exit(1);
    } else if (rc == 0) {
        printf("hello, I am child (pid:%d)\n", (int) getpid());
        char *myargs[3];
        myargs[0] = strdup("ls");
        myargs[1] = NULL;
        myargs[2] = NULL;
        execvp(myargs[0], myargs);
        printf("this shouldn't print out.\n");
    } else {
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n", rc, wc, (int) getpid());
    }

    return 0;
}
```

```bash
hello world (pid:1836)
hello, I am child (pid:1837)
p1  p1.c  p2  p2.c  p3  p3.c  text.txt
hello, I am parent of 1837 (wc:1837) (pid:1836)
```

**execl()**

```c
char *myargs;
myargs = strdup("/bin/ls");
execl(myargs, myargs, NULL);
```

**execle()**

```c
char *myargs[2];
myargs[0] = strdup("/bin/ls");
myargs[1] = strdup("-a");
execle(myargs[0], myargs[1], NULL, NULL);
```

**execlp()**

```c
char *myargs[2];
myargs[0] = strdup("ls");
myargs[1] = strdup("-a");
execlp(myargs[0], myargs[1], NULL);
```

这么多基本调用主要是为了方便。我们可以查看man手册的exec函数族。  

```c
int execl(const char *pathname, const char *arg, ...
          /* (char  *) NULL */);
int execlp(const char *file, const char *arg, ...
           /* (char  *) NULL */);
int execle(const char *pathname, const char *arg, ...
           /*, (char *) NULL, char *const envp[] */);
int execv(const char *pathname, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[],
            char *const envp[]);
```



"l"表示参数以列表形式表示。  

"p"表示会在环境变量path中搜索可执行文件。(hint：比较下上面execl()和exelp()的代码)  

"e"表示环境附加参数。

"v"表示参数以数组形式给出。

### P5. 现在编写一个程序，在父进程中使用wait()，等待子进程完成。wait()返回什么？如果你在子进程中使用wait()会发生什么？



```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/wait.h>

int main(int argc, char *argv[]) {
    printf("hello world (pid:%d)\n", (int) getpid());

    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed.\n");
        exit(1);
    } else if (rc == 0) {
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else {
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n", rc, wc, (int) getpid());
    }

    return 0;
}
```

```bash
hello world (pid:1999)
hello, I am child (pid:2000)
hello, I am parent of 2000 (wc:2000) (pid:1999)
```

可以观察到，wait()返回值是**子进程的进程描述符(pid)**。  

现在对代码进行修改，在子进程中使用wait().

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/wait.h>

int main(int argc, char *argv[]) {
    printf("hello world (pid:%d)\n", (int) getpid());

    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed.\n");
        exit(1);
    } else if (rc == 0) {
        int wc = wait(NULL);
        printf("hello, I am child (pid:%d) (wc:%d)\n", (int) getpid(), wc);
    } else {
        // int wc = wait(NULL);
        // printf("hello, I am parent of %d (wc:%d) (pid:%d)\n", rc, wc, (int) getpid());
        printf("hello, I am parent of %d (pid:%d)\n", rc, (int) getpid());

    }

    return 0;
}
```

```bash
hello world (pid:2053)
hello, I am parent of 2054 (pid:2053)
hello, I am child (pid:2054) (wc:-1)
```

可以看到，子进程调用wait()返回值是-1，这是因为子进程调用wait()后，会去寻找它自身的子进程(子进程的子进程)，因为它没有子进程，所以调用失败返回-1.  

### P6.对前一个子程序稍作修改,这次使用waitpid()而不是wait(). 什么时候waitpid()会有用.

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/wait.h>

int main(int argc, char *argv[]) {
    printf("hello world (pid:%d)\n", (int) getpid());

    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed.\n");
        exit(1);
    } else if (rc == 0) {
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else {
        int wc = (int) waitpid(rc, NULL, 0);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n", rc, wc, (int) getpid());

    }

    return 0;
}
```

```bash
hello world (pid:2325)
hello, I am child (pid:2326)
hello, I am parent of 2326 (wc:2326) (pid:2325)
```

查找man手册中的waitpid().函数原型如下:

```c
pid_t waitpid(pid_t pid, int *stat_loc, int options);
```

第一个参数是要等待的子程序的pid, 第二个参数是子程序终止的信息, 第三个参数是可选项,如果options中指定WNOHANG可以使父进程不阻塞立即返回0.  

> The waitpid() function shall be equivalent to wait() if the pid
> argument is (pid_t)−1 and the options argument is 0.

man手册里面提到,如果pid是-1,而且options是0那么就和wait()完全一致.

### P7.编写一个创建子进程的程序,然后在子进程中关闭标准输出(STDOUT_FILENO).如果子进程在关闭描述符后调用printf()打印输出,会发生什么?

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/wait.h>

int main(int argc, char *argv[]) {
    printf("hello world (pid:%d)\n", (int) getpid());

    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed.\n");
        exit(1);
    } else if (rc == 0) {
        printf("hello, I am child (pid:%d)\n", (int) getpid());
        close(STDOUT_FILENO);
        printf("this should not print out\n");
    } else {
        int wc = (int) waitpid(rc, NULL, 0);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n", rc, wc, (int) getpid());

    }

    return 0;
}
```

```bash
hello world (pid:2395)
hello, I am child (pid:2396)
hello, I am parent of 2396 (wc:2396) (pid:2395)
```

可以观察到,关闭标准输出后,无法打印信息至终端.

### P8. 编写一个程序,创建两个子进程,并使用pipe()系统调用,将一个子进程的标准输出连接到另一个子进程的标准输入.

```c
#if __GLIBC_USE (DEPRECATED_GETS)
/* Get a newline-terminated string from stdin, removing the newline.

   This function is impossible to use safely.  It has been officially
   removed from ISO C11 and ISO C++14, and we have also removed it
   from the _GNU_SOURCE feature list.  It remains available when
   explicitly using an old ISO C, Unix, or POSIX standard.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern char *gets (char *__s) __wur __attribute_deprecated__;
#endif
```

*Tips: C11已经不在支持gets()函数了*

这题的思路是,确保子进程1写标准输出到管道后,不再执行后面的代码(好吧,其实重要的是要防止子进程进行fork(),这样就不是主进程创建两个子进程了-_-).

思路一:  

先创建子进程1:

​		1.如果fork()返回值是0,那么就是子进程1在执行程序,将数据利用标准输出写到管道中.

​		2.如果fork()返回值>0,那么就是主进程在运行程序,主进程在创建子进程2.子进程2在从管道中利用标准输入读取数据.

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<sys/wait.h>

#define MAX_LEN 100

int main(int argc, char *argv[]) {
    printf("main program (pid:%d)\n", (int) getpid());

    int pipefd[2];
    char *out = "hello, world.";
    char input_buffer[MAX_LEN];
    
    if (pipe(pipefd) < 0) {
        fprintf(stderr, "pipe failed.\n");
        exit(1);
    }

    int rc1 = fork();

    if (rc1 < 0) {
        fprintf(stderr, "fork 1 failed.\n");
        exit(1);
    } else if (rc1 == 0) {
        printf("hello, I am child1 (pid:%d).\n", (int) getpid());
        close(pipefd[0]);
        write(pipefd[1], out, strlen(out));
    } else {
        int rc2 = fork();
        if (rc2 < 0) {
            fprintf(stderr, "fork failed.\n");
            exit(1);
        } else if (rc2 == 0) {
            printf("hello, I am child2 (pid:%d). The input is: ", (int) getpid());
            close(pipefd[1]);
            read(pipefd[0], input_buffer, MAX_LEN);
            puts(input_buffer);
        }
    }

    return 0;
}
```

思路二:  

1.先创建子进程1,如果fork()返回值是0,那么就是子进程1在执行程序,将数据利用标准输出写到管道中,写完后直接调用exit(1)退出程序.

2.创建子进程2,从管道中利用标准输入读取数据.

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<sys/wait.h>

#define MAX_LEN 100

int main(int argc, char *argv[]) {
    printf("main program (pid:%d)\n", (int) getpid());

    int pipefd[2];
    char *out = "hello, world.";
    char input_buffer[MAX_LEN];
    
    if (pipe(pipefd) < 0) {
        fprintf(stderr, "pipe failed.\n");
        exit(1);
    }

    int rc1 = fork();
    if (rc1 < 0) {
        fprintf(stderr, "fork 1 failed.\n");
        exit(1);
    } else if (rc1 == 0) {
        printf("hello, I am child1 (pid:%d).\n", (int) getpid());
        close(pipefd[0]);
        write(pipefd[1], out, strlen(out));
        exit(1);
    }

    int rc2 = fork();
    if (rc2 < 0) {
        fprintf(stderr, "fork 2 failed.\n");
        exit(1);
    } else if (rc2 == 0) {
        close(pipefd[1]);
        read(pipefd[0], input_buffer, MAX_LEN);
        printf("hello, I am child2 (pid:%d). The input is: ", (int) getpid());
        puts(input_buffer);
    }

    return 0;
}
```

结果:

```bash
main program (pid:3275)
hello, I am child1 (pid:3276).
hello, I am child2 (pid:3277). The input is: hello, world.
```




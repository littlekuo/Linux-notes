
# 进程 API

## fork()系统调用
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) { // fork failed; exit
      fprintf(stderr, "fork failed\n");
      exit(1);
    } else if (rc == 0) { // child (new process)
     printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else { // parent goes down this path (main)
     printf("hello, I am parent of %d (pid:%d)\n",
             rc, (int) getpid());
    }
    return 0;
}
```
fork()返回值——父进程获得的返回值是新创建的子进程的PID，子进程获得的返回值是0。

## wait系统调用
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
     printf("hello world (pid:%d)\n", (int) getpid());
     int rc = fork();
     
     if (rc < 0) { // fork failed; exit
     fprintf(stderr, "fork failed\n");
     exit(1);
     } else if (rc == 0) { // child (new process)
       printf("hello, I am child (pid:%d)\n", (int) getpid());
     } else { // parent goes down this path (main)
     int rc_wait = wait(NULL);
     printf("hello, I am parent of %d (rc_wait:%d) (pid:%d)\n",
     rc, rc_wait, (int) getpid());
     }
     return 0;
}
```
父进程调用wait()，等到子进程执行结束时，wait()才返回。

## exec系统调用
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    
    if (rc < 0) { // fork failed; exit
     fprintf(stderr, "fork failed\n");
     exit(1);
    } else if (rc == 0) { // child (new process)
      printf("hello, I am child (pid:%d)\n", (int) getpid());
      char *myargs[3];
      myargs[0] = strdup("wc"); // program: "wc" (word count)
      myargs[1] = strdup("p3.c"); // argument: file to count
      myargs[2] = NULL; // marks end of array
      execvp(myargs[0], myargs); // runs word count
      printf("this shouldn’t print out");
    } else { // parent goes down this path (main)
      int rc_wait = wait(NULL);
      printf("hello, I am parent of %d (rc_wait:%d) (pid:%d)\n",
      rc, rc_wait, (int) getpid());
    }
    return 0;
}

```


## 这样设计API的原因
fork()和exec()的分离，让shell可以方便地实现很多有用的功能。  
比如：shell实现结果重定向时，只需要先完成子进程创建，在调用exec()之前关闭标准输出即可。

## 其他API
比如 kill() 系统调用向进程发送信号。


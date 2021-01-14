#### 线程

进程是项目，线程是项目的执行，项目包含资源(cpu 内存)，也会有一个默认主线程来执行这个项目，也可以创建多个线程来执行这个项目。线程是负责执行二进制指令的，进程要比线程管的宽多了，除了执行指令外，还有内存、文件系统等。进程相当于一个项目，而线程就是为了完成项目需求，而建立的一个个开发任务。

##### 一、线程运行过程

代码示例：

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

#define NUM_OF_TASKS 5

void *downloadfile(void *filename)
{
   printf("I am downloading the file %s!\n", (char *)filename);
   sleep(10);
   long downloadtime = rand()%100;
   printf("I finish downloading the file within %d minutes!\n", downloadtime);
   pthread_exit((void *)downloadtime);
}

int main(int argc, char *argv[])
{
   char files[NUM_OF_TASKS][20]={"file1.avi","file2.rmvb","file3.mp4","file4.wmv","file5.flv"};
   pthread_t threads[NUM_OF_TASKS];
   int rc;
   int t;
   int downloadtime;

   pthread_attr_t thread_attr;
   pthread_attr_init(&thread_attr);
   pthread_attr_setdetachstate(&thread_attr,PTHREAD_CREATE_JOINABLE);

   for(t=0;t<NUM_OF_TASKS;t++){
     printf("creating thread %d, please help me to download %s\n", t, files[t]);
     rc = pthread_create(&threads[t], &thread_attr, downloadfile, (void *)files[t]);
     if (rc){
       printf("ERROR; return code from pthread_create() is %d\n", rc);
       exit(-1);
     }
   }

   pthread_attr_destroy(&thread_attr);

   for(t=0;t<NUM_OF_TASKS;t++){
     pthread_join(threads[t],(void**)&downloadtime);
     printf("Thread %d downloads the file %s in %d minutes.\n",t,files[t],downloadtime);
   }

   pthread_exit(NULL);
}
```

注：void 类型的指针，用于接收任何类型的参数

（1）一个运行中的线程可以调用 **pthread_exit** 退出线程，这个函数可以传入一个参数转换为void * 类型，是线程退出的返回值。

（2）**pthread_t** 用于声明线程对象

（3）**pthread_attr_t**  用于声明一个线程的属性，通过 **pthread_atrr_init**  初始化这个属性（**PTHREAD_CREATE_JOINABLE** 这个属性表示将来主线程等待这个线程的结束，并获取退出时的状态）

（4）**pthread_create** 创建线程，一共四个参数，第一个参数表示线程对象，第二个参数是线程属性，第三个参数是线程运行的函数，第四个参数是线程运行函数的参数。

（5）当一个线程退出的时候，就会发送信号给其他所有同进程的线程，有一个线程使用 **pthread_join** 获取这个线程退出的返回值。线程的返回值通过 pthread_join 传给主线程。

（6）多线程程序要依赖于 libpthread.so

```
gcc download.c -lpthread
```

**普通线程的创建和运行过程：**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145715.jpg" alt="线程创建和运行过程.jpg" style="zoom:67%;" />

##### 二、线程的数据

线程访问的数据细分为三类：

（1）**线程栈上的本地数据**，比如函数执行过程中的局部编码。栈的大小可以通过命令 **ulimit -a** 查看，默认情况下线程栈的大小为 8192字节(8MB)，我们可以使用命令 **ulimit -s** 修改。线程栈可以通过函数 **pthread_attr_t**  修改线程栈的大小。

```
int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize);
```

（2）**进程里面共享的全局数据**，例如全局变量，虽然在不同进程中是隔离的，但是在一个进程中是共享的。

（3）**线程私有数据（Thread Specific  Data）**，可以通过以下函数创建（创建一个key，伴随着一个析构函数）：

```
int pthread_key_create(pthread_key_t *key, void (*destructor)(void*))
```

key一旦被创建，所有线程都可以访问它，但各线程可根据自己的需要往key中填入不同的值，这相当于提供了一个同名而不同值的全局变量。

通过下面的函数设置key对应的value

```
int pthread_setspecific(pthread_key_t key, const void *value)
```

通过下面的函数获取key对应的value

```
void *pthread_getspecific(pthread_key_t key)
```

线程退出的时候，就会调用析构函数释放value。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145716.jpg" alt="线程访问的数据.jpg" style="zoom:67%;" />



##### 三、数据保护

###### 1、Mutex，全称Mutual Exclusion 互斥

它的模式是在共享数据访问的时候，去申请加锁，谁先拿到锁，谁就难到了访问权限，其他人就只能等着，等访问结束，锁释放，再去竞争。

示例代码：

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

#define NUM_OF_TASKS 5

int money_of_tom = 100;
int money_of_jerry = 100;
//第一次运行去掉下面这行
pthread_mutex_t g_money_lock;

void *transfer(void *notused)
{
  pthread_t tid = pthread_self();
  printf("Thread %u is transfering money!\n", (unsigned int)tid);
  //第一次运行去掉下面这行
  pthread_mutex_lock(&g_money_lock);
  sleep(rand()%10);
  money_of_tom+=10;
  sleep(rand()%10);
  money_of_jerry-=10;
  //第一次运行去掉下面这行
  pthread_mutex_unlock(&g_money_lock);
  printf("Thread %u finish transfering money!\n", (unsigned int)tid);
  pthread_exit((void *)0);
}

int main(int argc, char *argv[])
{
  pthread_t threads[NUM_OF_TASKS];
  int rc;
  int t;
  //第一次运行去掉下面这行
  pthread_mutex_init(&g_money_lock, NULL);

  for(t=0;t<NUM_OF_TASKS;t++){
    rc = pthread_create(&threads[t], NULL, transfer, NULL);
    if (rc){
      printf("ERROR; return code from pthread_create() is %d\n", rc);
      exit(-1);
    }
  }
  
  for(t=0;t<100;t++){
    //第一次运行去掉下面这行
    pthread_mutex_lock(&g_money_lock);
    printf("money_of_tom + money_of_jerry = %d\n", money_of_tom + money_of_jerry);
    //第一次运行去掉下面这行
    pthread_mutex_unlock(&g_money_lock);
  }
  //第一次运行去掉下面这行
  pthread_mutex_destroy(&g_money_lock);
  pthread_exit(NULL);
}
```

（1）使用Mutex，首先要使用 **pthread_mutex_init** 函数初始化这个 mutex锁，初始化之后就可以用来保护共享变量了。

（2）**pthread_mutex_lock()** 就是去抢占锁等函数，如果抢占到了，就可以执行下面的程序，对共享变量进行访问，如果没有抢到，就被阻塞在那里等待。

（3）如果不想被阻塞，可以使用 **pthread_mutex_trylock** 去抢占锁，如果抢到了和上面一样，如果没有抢到，不会被阻塞，而是返回一个错误码。

（4）共享数据访问结束后，使用 **pthread_mutex_unlock** 释放锁，让给其他人使用。

（5）最终调用 **pthread_mutex_destroy** 销毁这把锁。

Mutex 互斥锁使用流程示意：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145717.jpg" alt="mutex互斥锁流程.jpg" style="zoom:67%;" />

###### **2、条件变量和互斥锁配合使用**

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

#define NUM_OF_TASKS 3
#define MAX_TASK_QUEUE 11

char tasklist[MAX_TASK_QUEUE]="ABCDEFGHIJ";
int head = 0;
int tail = 0;

int quit = 0;

pthread_mutex_t g_task_lock;
pthread_cond_t g_task_cv;

void *coder(void *notused)
{
  pthread_t tid = pthread_self();

  while(!quit){

    pthread_mutex_lock(&g_task_lock);
    while(tail == head){
      if(quit){
        pthread_mutex_unlock(&g_task_lock);
        pthread_exit((void *)0);
      }
      printf("No task now! Thread %u is waiting!\n", (unsigned int)tid);
      pthread_cond_wait(&g_task_cv, &g_task_lock);
      printf("Have task now! Thread %u is grabing the task !\n", (unsigned int)tid);
    }
    char task = tasklist[head++];
    pthread_mutex_unlock(&g_task_lock);
    printf("Thread %u has a task %c now!\n", (unsigned int)tid, task);
    sleep(5);
    printf("Thread %u finish the task %c!\n", (unsigned int)tid, task);
  }

  pthread_exit((void *)0);
}

int main(int argc, char *argv[])
{
  pthread_t threads[NUM_OF_TASKS];
  int rc;
  int t;

  pthread_mutex_init(&g_task_lock, NULL);
  pthread_cond_init(&g_task_cv, NULL);

  for(t=0;t<NUM_OF_TASKS;t++){
    rc = pthread_create(&threads[t], NULL, coder, NULL);
    if (rc){
      printf("ERROR; return code from pthread_create() is %d\n", rc);
      exit(-1);
    }
  }

  sleep(5);

  for(t=1;t<=4;t++){
    pthread_mutex_lock(&g_task_lock);
    tail+=t;
    printf("I am Boss, I assigned %d tasks, I notify all coders!\n", t);
    pthread_cond_broadcast(&g_task_cv);
    pthread_mutex_unlock(&g_task_lock);
    sleep(20);
  }

  pthread_mutex_lock(&g_task_lock);
  quit = 1;
  pthread_cond_broadcast(&g_task_cv);
  pthread_mutex_unlock(&g_task_lock);

  pthread_mutex_destroy(&g_task_lock);
  pthread_cond_destroy(&g_task_cv);
  pthread_exit(NULL);
}
```

（1）声明 pthread_mutex_t  g_task_lock 和 pthread_cond_t  g_task_cv，用于锁和通知。

（2）**pthread_cond_broadcast** 通知所有调用 **pthread_cond_wait**  后等待中的线程可以起来干活了。

（3）调用pthread_cond_wait 会进入等待，接收到 pthread_cond_broadcast 通知后，会自动执行后面的代码，不需要我们另写代码。

互斥锁和条件变量示意图：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145718.jpg" alt="metux互斥和条件变量.jpeg"  />

##### 四、多线程程序编写套路

主要是 创建线程的套路、mutex使用的套路、条件变量使用的套路。

![多线程程序编写套路.png](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/2020-12-08-145718.png)


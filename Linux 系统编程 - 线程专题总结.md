# Linux 系统编程 - 线程专题总结

## 线程标识

```cpp
#include <pthread.h>
int pthread_equal(pthread_t tid1, pthread_t tid2);
```
	
因为线程 ID 是用 `pthread_t` 数据类型来表示的，实现的时候可以用一个结构来代表 `pthread_t` 数据类型，所以可移植的操作系统实现不能把它作为整数处理。

可移植的写法是使用 `pthread_equal` 对两个线程 ID 进行比较。

-----------------------

```cpp
#include <pthread.h>
pthread_t pthread_self(void);
```
	
线程可以通过 `pthread_self` 函数获得自身的线程 ID。

## 线程创建

```cpp
#include <pthread.h>
int pthread_create(pthread_t *restrict tidp, const pthread_attr_t *restrict attr,
                   void *(*start_rtn)(void *), void *restrict arg);
```

`pthread_create` 在调用失败时通常会返回错误码，而不是设置 `errno`（每个线程都提供 `errno` 的副本，但只是为了与使用 `errno` 的现有函数兼容）。

**在线程中，从函数中返回错误码更为清晰整洁，不需要依赖那些随着函数执行不断变化的全局状态，这样可以把错误的范围限制在引起出错的函数中。**

## 线程终止

如果进程中的任意线程调用了 `exit`、`_Exit` 或者 `_exit`，那么整个进程就会被终止。如果信号的默认动作是终止进程，那么发送到任意线程的信号也会终止整个进程。

单个线程可以通过 3 种方式退出，而不会终止整个进程：

* 线程可以简单地从启动例程中返回，返回值是线程的退出码
* 线程可以被同一进程中的其他线程取消
* 线程调用 pthread_exit

-----------------

```cpp
#include <pthread.h>
void pthread_exit(void *rval_ptr);
```

`rval_ptr` 参数是一个无类型指针，与传给启动例程的单个参数类似。进程中的其他线程也可以通过调用 `pthread_join` 函数访问到这个指针。

**注意 `rval_ptr` 可以传递任意复杂信息结构的地址，但是该地址必须在调用者完成调用以后仍然有效。例如，在调用线程的栈上分配了该结构，但其他线程在使用这个结构的时候内存内容可能已经改变了，这种情况下需要使用全局结构或者 `malloc` 函数分配结构。**

------------------

```cpp
#include <pthread.h>
int pthread_join(pthread_t thread, void **rval_ptr);
```

当线程调用 `pthread_join` 后，该调用线程将一直阻塞，直到指定的线程调用 `pthread_exit`、从启动例程中返回或者被取消。如果线程简单地从它的启动例程返回，`rval_ptr` 就包含返回码。如果线程被取消，由 `rval_ptr` 指定的内存单元就设置为 `PTHREAD_CANCELED`。

-------------------

```cpp
#include <pthread.h>
int pthread_cancel(pthread_t tid);
```

线程可以通过调用 `pthread_cancel` 函数来**请求取消同一进程中的其他线程**。

默认情况下，`pthread_cancel` 函数会使得由 `tid` 标识的线程的行为表现如同调用了参数为 `PTHREAD_CANCELED` 的 `pthread_exit` 函数，但目标线程可以选择忽略取消或者控制如何被取消。

**所以`pthread_cancel`并不等待线程终止，它仅仅提出请求**。

-------------------

### 线程清理处理程序

线程可以安排它退出时需要调用的函数，称为**线程清理处理程序**。

**一个线程可以建立多个清理处理程序**。处理程序记录在**栈**中，也就是说，**它们的执行顺序与它们注册时相反**。

```cpp
#include <pthread.h>
void pthread_cleanup_push(void (*rtn)(void *), void *arg);
void pthread_cleanup_pop(int execute);
```

当线程执行以下动作时，清理函数 `rtn` 是由 `pthread_cleanup_push` 函数调度的，调用时只有一个参数 `arg`：

* 调用 `pthread_exit` 时
* 响应取消请求时
* 用非零 `execute` 参数调用 `pthread_cleanup_pop` 时

如果 `execute` 参数设置为 `0`，清理函数将不被调用。不管发生上述哪种情况，`pthread_cleanup_pop` 都将删除上次 `pthread_cleanup_push` 调用建立的清理处理程序。

**注意这些函数有一个限制，由于它们可以实现为宏，宏定义中可以包含字符 `{` 和 `}`，所以必须在线程相同的作用域中以配对的形式使用**。

> 


# Linux 系统编程 - 线程专题总结

## 线程标识

    #include <pthread.h>
    int pthread_equal(pthread_t tid1, pthread_t tid2);
    
    // 返回值：相等则返回非 0 数值，否则返回 0
	
因为线程 ID 是用 `pthread_t` 数据类型来表示的，实现的时候可以用一个结构来代表 `pthread_t` 数据类型，所以可移植的操作系统实现不能把它作为整数处理。

可移植的写法是使用 `pthread_equal` 对两个线程 ID 进行比较。

-----------------------

    #include <pthread.h>
    pthread_t pthread_self(void);
    
    // 返回值：调用线程的线程 ID
	
线程可以通过 `pthread_self` 函数获得自身的线程 ID。

## 线程创建

    #include <pthread.h>
    int pthread_create(pthread_t *restrict tidp, const pthread_attr_t *restrict attr,
                       void *(*start_rtn)(void *), void *restrict arg);
                       
    // 返回值：成功返回 0，否则返回错误编号                   

`pthread_create` 在调用失败时通常会返回错误码，而不是设置 `errno`（每个线程都提供 `errno` 的副本，但只是为了与使用 `errno` 的现有函数兼容）。

**在线程中，从函数中返回错误码更为清晰整洁，不需要依赖那些随着函数执行不断变化的全局状态，这样可以把错误的范围限制在引起出错的函数中。**

## 线程终止

如果进程中的任意线程调用了 `exit`、`_Exit` 或者 `_exit`，那么整个进程就会被终止。如果信号的默认动作是终止进程，那么发送到任意线程的信号也会终止整个进程。

单个线程可以通过 3 种方式退出，而不会终止整个进程：

* 线程可以简单地从启动例程中返回，返回值是线程的退出码
* 线程可以被同一进程中的其他线程取消
* 线程调用 pthread_exit

-----------------

    #include <pthread.h>
    void pthread_exit(void *rval_ptr);

`rval_ptr` 参数是一个无类型指针，与传给启动例程的单个参数类似。进程中的其他线程也可以通过调用 `pthread_join` 函数访问到这个指针。

**注意 `rval_ptr` 可以传递任意复杂信息结构的地址，但是该地址必须在调用者完成调用以后仍然有效。例如，在调用线程的栈上分配了该结构，但其他线程在使用这个结构的时候内存内容可能已经改变了，这种情况下需要使用全局结构或者 `malloc` 函数分配结构。**

------------------

    #include <pthread.h>
    int pthread_join(pthread_t thread, void **rval_ptr);
    
    // 返回值：成功返回 0，否则返回错误编号

当线程调用 `pthread_join` 后，该调用线程将一直阻塞，直到指定的线程调用 `pthread_exit`、从启动例程中返回或者被取消。如果线程简单地从它的启动例程返回，`rval_ptr` 就包含返回码。如果线程被取消，由 `rval_ptr` 指定的内存单元就设置为 `PTHREAD_CANCELED`。

-------------------

    #include <pthread.h>
    int pthread_cancel(pthread_t tid);
    
    // 返回值：成功返回 0，否则返回错误编号

线程可以通过调用 `pthread_cancel` 函数来**请求取消同一进程中的其他线程**。

默认情况下，`pthread_cancel` 函数会使得由 `tid` 标识的线程的行为表现如同调用了参数为 `PTHREAD_CANCELED` 的 `pthread_exit` 函数，但目标线程可以选择忽略取消或者控制如何被取消。

**所以`pthread_cancel`并不等待线程终止，它仅仅提出请求**。

-------------------

### 线程清理处理程序

线程可以安排它退出时需要调用的函数，称为**线程清理处理程序**。

**一个线程可以建立多个清理处理程序**。处理程序记录在**栈**中，也就是说，**它们的执行顺序与它们注册时相反**。

    #include <pthread.h>
    void pthread_cleanup_push(void (*rtn)(void *), void *arg);
    void pthread_cleanup_pop(int execute);

当线程执行以下动作时，清理函数 `rtn` 是由 `pthread_cleanup_push` 函数调度的，调用时只有一个参数 `arg`：

* 调用 `pthread_exit` 时
* 响应取消请求时f
* 用非零 `execute` 参数调用 `pthread_cleanup_pop` 时

如果 `execute` 参数设置为 `0`，清理函数将不被调用。不管发生上述哪种情况，`pthread_cleanup_pop` 都将删除上次 `pthread_cleanup_push` 调用建立的清理处理程序。

**注意这些函数有一个限制，由于它们可以实现为宏，宏定义中可以包含字符 `{` 和 `}`，所以必须在线程相同的作用域中以配对的形式使用**。

### 线程同步 - 互斥量

可以使用 `pthread` 的互斥接口来保护数据，确保同一时间只有一个线程访问数据。

互斥变量是用 `pthread_mutex_t` 数据类型表示的。在使用互斥变量以前，必须首先对它进行初始化，初始化分两种：

* 设置为常量 `PTHREAD_MUTEX_INITIALIZER`（适用于静态分配的互斥量）
* 通过调用 `pthread_mutex_init` 函数进行初始化

如果动态分配了互斥量，则在释放内存前需要调用 `pthread_mutex_destroy`。

    #include <pthread.h>
    int pthread_mutex_init(pthread_mutex_t *restrict mutex,
                           const pthread_mutexattr_t *restrict attr);
    int pthread_mutex_destroy(pthread_mutex_t *mutex);
    
    // 所有函数的返回值：成功返回 0，否则返回错误编号。

如果使用默认的属性初始化互斥量，则只需要把 `attr` 设为 `NULL` 即可。

----------------------

    #include <pthread.h>
    int pthread_mutex_lock(pthread_mutex_t *mutex);
    int pthread_mutex_trylock(pthread_mutex_t *mutex);
    int pthread_mutex_unlock(pthread_mutex_t *mutex);
    
    // 所有函数的返回值：成功返回 0，否则返回错误编号。

* 对互斥量进行加锁，需要调用 `pthread_mutex_lock`。如果互斥量已经上锁，那么调用线程将会阻塞直到互斥量被解锁
* 对互斥量进行解锁，需要调用 `pthread_mutex_unlock`
* 如果线程不希望被阻塞，那么使用 `pthread_mutex_trylock` 尝试对互斥量进行加锁
    * 如果调用时互斥量处于未锁住状态，那么 `pthread_mutex_trylock` 将锁住互斥量，然后返回 `0`
    * 如果调用时互斥量处于锁住状态，那么 `pthread_mutex_trylock` 将会失败，不能锁住互斥量，也是直接返回，返回值为 `EBUSY`

-----------------------

    #include <pthread.h>
    #include <time.h>
    int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex,
                                const struct timespec *restrict tsptr);
                                
    // 返回值：成功返回 0，否则返回错误编号。                            

当线程试图获取一个已加锁的互斥量时，`pthread_mutex_timedlock` 互斥量原语允许绑定线程阻塞时间。

`pthread_mutex_timedlock` 函数与 `pthread_mutex_lock` 是基本等价的，但是在达到超时时间值时，`pthread_mutex_timedlock` 不会对互斥量进行枷锁，而是返回错误码 `ETIMEDOUT`。

### 线程同步 - 读写锁

读写锁与互斥量类似，不过读写锁允许更高的并行性。

互斥量只有两种状态：锁住状态和不加锁状态，且一次只有一个线程可以对其加锁。

读写锁可以有三种状态：**读模式加锁状态，写模式加锁状态，不加锁状态。一次只有一个线程可以占有写模式的读写锁，但是多个线程可以同时占有读模式的读写锁**。

读写锁非常适合对数据结构读的次数远大于写的情况：

* 当在写模式下时，它所保护的数据结构可以被安全的修改，因为一次只有一个线程可以在写模式下拥有这个锁
* 当在读模式下时，只要线程先获取了读模式下的读写锁，该锁所保护的数据结构就可以被多个获得读模式锁的线程读取

PS: 当读写锁处于读模式锁住的状态，而这时有一个线程试图以写模式获取锁时，读写锁通常会阻塞随后的读模式锁请求。这样可以避免读模式锁长期占用，而等待的写模式锁请求一直得不到满足。

读写锁是用 `pthread_rwlock_t` 数据类型表示的。在使用读写锁以前，必须首先对它进行初始化，初始化分两种：

* 设置为常量 `PTHREAD_RWLOCK_INITIALIZER`（适用于静态分配的读写锁）
* 通过调用 `pthread_rwlock_init` 函数进行初始化

        #include <pthread.h>
        int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,
                                const pthread_rwlockattr_t *restrict attr);
        int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
    
        // 两个函数的返回值：成功返回 0，否则返回错误编号
    
如果使用默认的属性初始化读写锁，则只需要把 `attr` 设为 `NULL` 即可。

------------------------

    #include <pthread.h>
    int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
    int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
    int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
    
    // 所有函数的返回值：成功返回 0，否则返回错误编号
    
* 在读模式下锁定读写锁，调用 `pthread_rwlock_rdlock` 函数
* 在写模式下锁定读写锁，调用 `pthread_rwlock_wrlock` 函数
* 不管是什么方式锁住的读写锁，均可以调用 `pthread_rwlock_unlock` 进行解锁

下面还有读写锁原语的条件版本：

    #include <pthread.h>
    int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
    int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
    
    // 两个函数的返回值：成功返回 0，否则返回错误编号
    
可以获取锁时，这两个函数返回 0，否则它们返回错误号 `EBUSY`。

[未完待续...]


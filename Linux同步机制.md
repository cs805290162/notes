来自 <https://blog.csdn.net/godleading/article/details/78259842> 

1，原子操作（atomic）

最小的执行单位，不会被打断，只有执行完成和未执行两种状态，没有执行中。主要用于实现资源计数，引用计数。

主要结构体与api:

typedef struct { volatile int counter; } atomic_t; 结构体
atomic_read(atomic_t * v); 读原子变量值
atomic_set(atomic_t * v, int i);  设置原子变量的值
void atomic_add(int i, atomic_t *v); 原子变量值加i;
atomic_sub(int i, atomic_t *v); 减法
int atomic_sub_and_test(int i, atomic_t *v);  减去i，并判断结果是否为0，如果为0，返回真，否则返回假
void atomic_inc(atomic_t *v); 自增1
void atomic_dec(atomic_t *v); 自减1
int atomic_dec_and_test(atomic_t *v); 自减1后判断结果是否为0。
int atomic_inc_and_test(atomic_t *v); 自增1后判断是否为0
int atomic_add_negative(int i, atomic_t *v); 自增1后是否为负数
int atomic_add_return(int i, atomic_t *v); 加（减）后返回指向V的指针。
int atomic_sub_return(int i, atomic_t *v);
int atomic_inc_return(atomic_t * v);
int atomic_dec_return(atomic_t * v);

2，信号量（semaphore）

信号量是给资源设置的初始值，表示同时可以有多少个任务共享此资源，初始值为1就变成了互斥锁。一个任务获取了资源，信号量减1，信号量为负数时就无法占用此资源，释放资源以后信号量加1。多数情况作为互斥锁使用。

主要api:

DEFINE_MUTEX  静态定义和初始化一个互斥锁。
void mutex_init(struct mutex *mutex);  动态初始化互斥锁。
void sema_init (struct semaphore *sem, int val);  信号量设初始值

void down(struct semaphore * sem); 获取信号量，先判断信号量是否大于0，是则信号量减1。否则线程阻塞。不会被信号（signal）打断.
int down_interruptible(struct semephore *sem) 可以被打断，如果被信号中断，则返回-EINTR
int down_trylock(struct semaphore * sem); 试图获取信号量，不会导致阻塞

void up(struct semaphore * sem); 释放信号量，信号量+1，唤醒阻塞线程。


3，读写信号量（rw_semaphore）

读者数不受限制，任意多个读者拥有同一信号量。即读读不互斥，读写互斥，写写互斥。

主要api：

DECLARE_RWSEM(name); 静态初始化
void init_rwsem(struct rw_semaphore *sem); 动态初始化

void down_read(struct rw_semaphore *sem); 获取读锁，会导致睡眠（阻塞）
int down_read_trylock(struct rw_semaphore *sem); 获取读锁，不会导致阻塞。

void down_write(struct rw_semaphore *sem); 获取写锁，会阻塞。
int down_write_trylock(struct rw_semaphore *sem); 获取写锁，非阻塞。

void up_read(struct rw_semaphore *sem); 释放读锁
void up_write(struct rw_semaphore *sem); 释放写锁

void downgrade_write(struct rw_semaphore *sem); 把写者降级为读者，便于写前读资源。提高并发性


4，自旋锁（spinlock），不可抢占

自旋锁不会引起线程阻塞，效率高于互斥锁，但是会一直占用cpu资源。（适用于没有线程长期占用资源的场景。需要在中断上下文访问共享资源的场景），自旋锁保持期间是抢占失效的，而信号量和读写信号量保持期间是可以被抢占的。

主要api：

spin_lock_init(x)：初始化自旋锁x
DEFINE_SPINLOCK(x)：宏初始化
SPIN_LOCK_UNLOCKED：静态初始化自旋锁
spin_is_locked(x)：判断自旋锁是否被锁定
spin_unlock_wait(x)：自旋等待自旋锁释放
spin_trylock(lock)：试图获取锁，立即返回。不会自旋等待锁释放
spin_lock(lock)：自旋获取锁，获得锁才返回。
spin_lock_irqsave(lock, flags) 获得自旋锁的同时把标志寄存器的值保存到变量flags中并失效本地中断。
spin_lock_irq(lock) 该宏类似于spin_lock_irqsave，只是该宏不保存标志寄存器的值。
spin_lock_bh(lock)得到自旋锁的同时失效本地软中断。
spin_unlock(lock); 释放自旋锁lock，它与spin_trylock或spin_lock配对使用。
spin_unlock_irqrestore(lock, flags)释放自旋锁lock的同时，也恢复标志寄存器的值为变量flags保存的值。它与spin_lock_irqsave配对使用。
spin_unlock_irq(lock)释放自旋锁lock的同时，也使能本地中断。它与spin_lock_irq配对应用。
spin_unlock_bh(lock)释放自旋锁lock的同时，也使能本地的软中断。它与spin_lock_bh配对使用。
spin_trylock_irqsave(lock, flags)获得自旋锁lock，它也将保存标志寄存器的值到变量flags中，并且失效本地中断，如果没有获得锁，它什么也不做。
spin_trylock_irq(lock)类似于spin_trylock_irqsave，只是该宏不保存标志寄存器。
spin_trylock_bh(lock)获得了自旋锁，它也将失效本地软中断。
spin_can_lock(lock)判断自旋锁lock是否能够被锁，它实际是spin_is_locked取反。


5, 大内核锁(BKL–Big Kernel Lock)

本质是自旋锁，但是可以递归获得锁，而不会导致死锁。用于保护整个内核，整个内核只有一个大内核锁。2.6.11起大内核锁变为可抢占互斥锁。使用信号量实现。（历史遗留锁，不提倡使用）

void lock_kernel(void); 得到大内核锁
void unlock_kernel(void); 释放大内核锁


6, 读写锁（rwlock）

读写锁实际是一种特殊的自旋锁，它把对共享资源的访问者划分成读者和写者，读者只对共享资源进行读访问，写者则需要对共享资源进行写操作。在读写锁保持期间也是抢占失效的。`

rwlock_init(x)
DEFINE_RWLOCK(x)
RW_LOCK_UNLOCKED
read_trylock(lock)
write_trylock(lock)
read_lock(lock)
write_lock(lock)
read_lock_irqsave(lock, flags)
read_lock_irqsave(lock, flags)
read_lock_irqsave(lock, flags)
…
7, RCU（Read-Copy update）

RCU(Read-Copy Update)，顾名思义就是读-拷贝修改，它是基于其原理命名的。对于被RCU保护的共享数据结构，读者不需要获得任何锁就可以访问它，但写者在访问它时首先拷贝一个副本，然后对副本进行修改，最后使用一个回调（callback）机制在适当的时机把指向原来数据的指针重新指向新的被修改的数据。这个时机就是所有引用该数据的CPU都退出对共享数据的操作。

与读写锁的差别：即支持多个读者同时读，也允许多个读者和多个写者同时访问被保护数据。

7, 大读者锁（brlock-Big Reader Lock）

大读者锁是读写锁的高性能版，读者可以非常快地获得锁，但写者获得锁的开销比较大。大读者锁只存在于2.4内核中，在2.6中已经没有这种锁（提醒读者特别注意）。它们的使用与读写锁的使用类似，只是所有的大读者锁都是事先已经定义好的。这种锁适合于读多写少的情况，它在这种情况下远好于读写锁。

8， 顺序锁（seqlock）

顺序锁也是对读写锁的一种优化，对于顺序锁，读者绝不会被写者阻塞，也就说，读者可以在写者对被顺序锁保护的共享资源进行写操作时仍然可以继续读，而不必等待写者完成写操作，写者也不需要等待所有读者完成读操作才去进行写操作。但是，写者与写者之间仍然是互斥的，即如果有写者在进行写操作，其他写者必须自旋在那里，直到写者释放了顺序锁。
这种锁有一个限制，它必须要求被保护的共享资源不含有指针，因为写者可能使得指针失效，但读者如果正要访问该指针，将导致OOPs。


Linux的内核锁主要是自旋锁和信号量。

1， 自旋锁：自旋锁可以在任何时刻防止多于一个的执行线程同时进入临界区。适用于短时持有
2，信号量：信号量是一种睡眠锁，若资源被占有，则推入等待队列，让其睡眠。只能在进程上下文使用，适用于长时间持有。
3，同步机制：原子操作，信号量，读写信号量，自旋锁。大内核锁，大读者锁，读写锁，RCU（读拷贝修改），顺序锁。


---
layout: pageshow
title: Linux的select.c源码注释
category: linux
description: 结合APUE查看select源码
---



###  阻塞IO和非阻塞IO
Linux中IO分为阻塞IO和非阻塞IO  
* 阻塞IO是指对文件读写的时候，是阻塞当前进程,直到等待文件可读或者可写时候对其进行读写操作。
* 非阻塞IO是指对文件读写的时候，如果此时文件不可读或不可写，立即返回,不等待其状态变成可操作。
Linux的设计思想是把一切设备都视作文件。因此，文件描述符为在该系列平台上进行设备相关的编程实际上提供了一个统一的方法。  

### 应用场景
假设这样一个场景：单线程下，对2个fd进行IO操作，当使用阻塞IO对fd1读的时候，线程一直在等待fd1有数据可读。同时，fd2已经可读的数据得不到程序的响应，这样就体现了阻塞IO的局限性。  
换成非阻塞IO后，每次对fd1尝试读，没可读的数据就返回了，然后去读fd2，循环往复，可以同时响应这两个fd。  
当fd变成了一个集合fd_set，其中有很多个想读写的文件，如何对他们能实时响应，一种方法是轮询，将每个fd都设置成非阻塞的，然后轮询read，一旦有数据就处理，一个循环完成可以等待一段时间进行一此轮询。这种方法可行，但是浪费了很多CPU时间。一种改善的方法是让内核去轮询fd的状态，也就是使用select。

### select函数借口
`int select(int maxfdp1, fd_set* readfds, fd_set* writefds, fd_set* exceptfds, struct timeval* tvptr)`  
其中`maxfdp1`的fd_set中最大文件描述符+1， `fd_set`定义了文件描述符的结构体， `tvptr`则是等待时间（可以选择永远等待，立即返回和等待具体时间)，所以用户可以选择一直阻塞，不阻塞或者阻塞一段时间。  
值得说的是，其中`fd_set`结构体定义了文件描述符的存储方式，是通过**bitmap**的形式存储，每个bit位代表一个描述符的状态。Linux默认情况下定义进程持有的最大描述符个数是1024，显然，在目前移动互联网的情况下完全不够用。当然我们可以手动修改这个值。  
在linux内核头文件中include/uapi/linux/posix_types.h定义了最大描述符和`fd_set`   
```c
#undef __FD_SETSIZE
#define __FD_SETSIZE	1024

typedef struct {
	unsigned long fds_bits[__FD_SETSIZE / (8 * sizeof(long))]; // 使用long(假设是8字节大小)来保存fd，共需要1024 / (8 * 8)个long型数据。
} __kernel_fd_set;

typedef __kernel_fd_set fd_set // 在include/linux/types.h中声明
```

### 查看select源码
详细源码可以通过[Free Electrons](http://elixir.free-electrons.com/linux/latest/source/fs/select.c)上在线查看，这里只摘关心的部分代码，偶尔穿插注释（仅个人观点）。以下都假设long数据占8个字节
```c
int core_sys_select(int n, fd_set __user *inp, fd_set __user *outp,
			   fd_set __user *exp, struct timespec64 *end_time)
{
    /* typedef struct{
     *      unsigned long *in, *out, *ex;
     *      unsigned long *res_in, *res_out, *res_ex;
     * } fd_set_bits;
     * 内核拷贝的一份fd_set副本，该副本用于后续轮询状态然后更新
     */
	fd_set_bits fds;

	void *bits;
	int ret, max_fds;
	size_t size, alloc_size;
	struct fdtable *fdt;
	/* Allocate small arguments on the stack to save memory and be faster */
	long stack_fds[SELECT_STACK_ALLOC/sizeof(long)]; // SELECT_STACK_ALLOC 默认栈大小 = 256Byte, 256 / 8 = 32 个long型数据，最大可保存32 * 24 = 2048个文件描述符

	ret = -EINVAL; // 参数错误码
	if (n < 0)
		goto out_nofds;

	/* max_fds can increase, so grab it once to avoid race */
	rcu_read_lock();
	fdt = files_fdtable(current->files);
	max_fds = fdt->max_fds;
	rcu_read_unlock();
	if (n > max_fds)
		n = max_fds;

	/*
	 * We need 6 bitmaps (in/out/ex for both incoming and outgoing),
	 * since we used fdset we need to allocate memory in units of
	 * long-words.
	 */

    /* #define FDS_BITPERLONG	(8*sizeof(long))
     * #define FDS_LONGS(nr)	(((nr)+FDS_BITPERLONG-1)/FDS_BITPERLONG)
     * #define FDS_BYTES(nr)	(FDS_LONGS(nr)*sizeof(long))
     */
	size = FDS_BYTES(n); // n如果在[1, 64]范围内，size为1, 一个long就能存储64个fd标志
	bits = stack_fds;
	if (size > sizeof(stack_fds) / 6) {
		/* Not enough space in on-stack array; must use kmalloc */
		ret = -ENOMEM;
		if (size > (SIZE_MAX / 6))
			goto out_nofds;

		alloc_size = 6 * size;
		bits = kvmalloc(alloc_size, GFP_KERNEL); // 栈空间不够，重新申请空间
		if (!bits)
			goto out_nofds;
	}

    // 为每个long*分配地址来表示long型数据
	fds.in      = bits;
	fds.out     = bits +   size;
	fds.ex      = bits + 2*size;
	fds.res_in  = bits + 3*size;
	fds.res_out = bits + 4*size;
	fds.res_ex  = bits + 5*size;

    /* static inline
     * int get_fd_set(unsigned long nr, void __user *ufdset, unsigned long *fdset)
     * 	{
     *   nr = FDS_BYTES(nr);
     *   if (ufdset)
     *      return copy_from_user(fdset, ufdset, nr) ? -EFAULT : 0;
     *   memset(fdset, 0, nr);
     *   return 0;
     * }
     */
	if ((ret = get_fd_set(n, inp, fds.in)) ||   // 复制读集(readfds)到内核栈空间
	    (ret = get_fd_set(n, outp, fds.out)) || // 复制写集(writefds)
	    (ret = get_fd_set(n, exp, fds.ex)))     // 复制异常条件集(exceptfds)
		goto out;
	zero_fd_set(n, fds.res_in);  // 复位res_集合的状态
	zero_fd_set(n, fds.res_out);
	zero_fd_set(n, fds.res_ex);

	ret = do_select(n, &fds, end_time); // 开始真正的轮询

	if (ret < 0)
		goto out;
	if (!ret) {
		ret = -ERESTARTNOHAND;
		if (signal_pending(current))
			goto out;
		ret = 0;
	}

    /* set_fd_set(unsigned long nr, void __user *ufdset, unsigned long *fdset)
     * {
     *    if (ufdset)
     *      return __copy_to_user(ufdset, fdset, FDS_BYTES(nr));
     *    return 0;
     * }
     */
	if (set_fd_set(n, inp, fds.res_in) || // 将select返回结果复制给传入的fd_set;
	    set_fd_set(n, outp, fds.res_out) ||
	    set_fd_set(n, exp, fds.res_ex))
		ret = -EFAULT;

out:
	if (bits != stack_fds)
		kvfree(bits);
out_nofds:
	return ret;
}

```
可以看到，调用select每次需要从用户空间复制fd_set到内核空间，然后返回时将结果复制到用户空间。值得注意的地方是，每次selec返回结果后fd_set的值会发生改变，如果下次想要select相同的文件描述符集，可能需要提前复制一份。

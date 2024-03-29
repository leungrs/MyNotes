### 全局变量的errno怎么保证线程安全？

只要一个unix函数发生错误，那么全局变量errno就会被设置为对应的错误代码。
0表示没错误。其他正数表示有错误，具体的错误代码定义在<sys/errno.h>中，通常以大写字母E开始，如：
- ETIMEOUT 表示网络超时
- EINTR 系统调用被中断
- ECONNRESET 第一次读写的时候发现通信对方不存在，或者重启了，会收到RST。
- EPIPE 在收到RST之后，继续写的时候，发现通信对方不存在了。

### 什么是文件描述符（file descriptor）？
文件描述符是一个数组的索引值，所以它是一个大于0的整数。这个数组称之为进程的文件描述符表。数组中的每个元素都是指向一个文件结构体的指针。文件结构体是由内核维护的。

文件结构体中包含许多信息，比如：
- 打开模式：只读，只写，读写等等。
- 引用计数
- 当前文件偏移等等许多其他信息

文件描述符表是和进程相关联的，每个进程都有一个，一个进程fork后，子进程是复制了父进程这个表的。这个时候对应的文件结构体中的引用计数会加1。

我们close(fd), 会把这个引用计数减1，为0的时候，结构体才会从内核中清除。

网络编程时，我们在accept后fork的话，那么在父进程中需要关闭掉客户端的fd，子进程中关闭listen的fd。

### 进程终止的会发生什么？
- 关闭进程所有打开的进程描述符
- 给父进程发一个SIGCHLD信号。父进程必须对这个信号进行处理，否则子进程会编程僵尸进程。

### 网络编程中的错误有哪些，怎么处理？
1. 慢系统调用(accept, read, write等）被信号中断，errno为EINTR。

    处理方式一般为重试，也就是重启被中断的系统调用。有的操作系统自带重启功能，有的没有，那么编写可移植的APP，必须显式的处理这种情况，而不依赖具体的操作系统。

2. 连接建立后，accept之前，收到了客户端是RST。

3. 服务器进程终止，客户端没有感知，继续读写数据。此时服务器进程终止，会关闭所有的描述符，导致给客户端发送FIN，客户端收到FIN后，一直处于CLOSE_WAIT状态。
    * 继续读：可能会发生ECONNRESET错误，返回读的时候返回0
    * 继续写：内核会给当前写进程发送SIGPIPE信号，写操作返回EPIPE。该信号的默认处置是终止进程，为了避免这种情况，需要对该信号进行处置。

4. 服务器主机崩溃，可以在建立连接后，断开服务器网络来模拟。这又分两种情况：
    * 客户端继续发送数据，那么在收不到ACK后，重试一段时间会报超时ETIMEOUT错误。这个时间默认情况下比较长，我们可以设置超时时间。
    * 客户端不发送数据，那么检测不了服务器是否崩溃了。此时，SO_KEEPALIVE选项可以派上用场，在服务器主机崩溃后，能够及时检测出来。

5. 服务器主机崩溃后，又重启了。可以通过建立连接后，端开服务器网络，然后重启服务器，最后连上网络来模拟。这种情况，也需要使用SO_KEEPALIVE技术来检查服务器状况。

6. 服务器主机关机。这和上面第三点类似，客户端需要使用select, epoll等技术，监听多个描述符。
### 基本TCP套接字编程

基本的TCP套接字编程主要包含如下函数, 它们都定义在sys/socket.h头文件中：
- socket
- bind
- listen
- accept
- connect
- read
- write
- close
- getsockname
- getpeername

#### socket函数
socket函数的定义包含在sys/socket.h中：
```c
int socket(int family, int type, int protocol);
```
用它来新建一个套接字，返回套接字的描述符sockfd, 其中：
- family表示套接字所用的协议族，取值有：
    * AF_INET          IPv4协议的套接字
    * AF_INET6         IPv6协议的套接字
    * AF_LOCAL         Unix域协议套接字
    * AF_ROUTE         路由协议的套接字
    * AF_KEY           秘钥协议的套接字

- type表示套接字的类型，取值有：
    * SOCK_STREAM         字节流的套接字
    * SOCK_DGRAM          数据报套接字
    * SOCK_SEQPACKET      有序分组套接字
    * SOCK_RAW            原始套接字

- protocol参数表示传输协议类型，取值如下，一般传0，系统会根据前两个参数选择一个默认的配置的值
    * IPPROTO_TCP         TCP传输协议
    * IPPROTO_UDP         UDP传输协议
    * IPPROTO_SCTP        SCTP传输协议     

#### connect函数
客户端使用这个函数建立与服务器的连接。
```c
int connect(int sockfd, const struct sockaddr * servaddr, socklen_t addrlen);
```
对于TCP套接字来说，会激发内核的3路握手过程，连接成功或者出错后该函数才返回。出错的情况有:
1. 超时错误ETIMEDOUT：客户端发出SYN分节后，没有收到ACK，6s再发送一次，还没响应，24s后再发。总共等待75s后，仍没响应，就会返回超时错误。 当然有些系统会提供一个参数，来控制这个超时时间。
2. 拒绝连接错误ECONNREFUSED, 通常服务端会给SYN分节响应一个RST分节（reset），表示服务端对应的端口上没有进程在等待客户端连接，这是一种硬错误。

    RST分节是TCP发生错误时发送的一种TCP分节。有三种情况发生成这种RST分节：
    - 对应端口上没有正在等待连接的进程。
    - TCP想取消一个已有的连接。
    - TCP接收到一个根本不存在的连接上的分节。
3. 若客户发出的SYN分节，在中间的路由器上引发了destination unreachable的ICMP错误。客户端主机收到这种错误，认为是一种软错误，会执行第一种情况的重试。如果重试之后还不行，那么就会返回主机不可达(EHOSTUNREACH)或者网络不可达(ENETUNREACH)的错误。

#### bind函数
bind函数把本地的一个IP地址和端口号绑定到套接字上。它的函数签名和connect一样。
```c
int bind(int sockfd, const struct sockaddr * serv, socklen_t addrlen);
```
端口号和IP地址，可以只指定一个，也可以两个都指定，还可以两个都不指定。内核有一定的机制来选择一个地址或者临时端口。

这个函数可能返回的错误是：EADDRINUSE（"Address already in use"）。
套接字选项：SO_REUSEADDR, SO_REUSEPORT和这种情况有关系。

#### listen函数
```c
int listen(int sockfd, int backlog);
```
listen函数，是由TCP服务器调用的，客户端不需要调用。它主要做两件事情：
1. 把当前未连接的套接字转变成一个被动套接字，告诉内核，应该接受连接到这个套接字的请求。
2. 通过第二个参数backlog通知内核，当前监听套接字已完成和未完成连接队列总和的最大数。

内核为每个监听的套接字维护了两个队列：
- 未完成连接队列：此时的连接状态是：收到了客户端的SYN，正在完成TCP的3路握手过程，客户端处于SYN_SENT, 服务端处于SYN_RCVD状态。
- 已完成连接队列：此时的连接状态是：客户端和服务端已经完成TCP的3路握手，双方都处于ESTABLISHED状态。

关于这两个队列：
- backlog是两个队列的和的最大值。
- 不要把backlog设为0
- 已完成队列中的连接最大停留时间为一个RTT，这个值取决于特定的客户和服务器。比如这个值统计出来可能为187ms，当然单个的肯定是有大有小。
- 如果队列满了，新来的SYN，会被忽略，而不是响应RST，因为队列满的状态，可能不会持续很久，客户端没收到ACK，TCP会重发SYN。
- 已完成连接队列里的连接，如果有数据到来，如果连接没有accept，那么数据会存储到连接的接收缓冲区。
- 存在使用SYN泛滥的攻击，迫使backlog满了，导致正常的服务被拒绝。

#### accept函数
这个函数由服务端进程调用，从已完成队列的对头中返回一个连接。如果已完成队列为空，那么进程就被阻塞了（当然是假设套接字采用默认的阻塞模式）。
```c
int accept(int  sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);
```

这个函数的第二和第三个参数，是客户端的信息，是由内核填充并返回给用户进程的，如果对客户端的信息不感兴趣，可以传NULL。

#### close函数
close函数的默认行为是把一个TCP连接标记为已关闭，然后立马返回，进程不能再对它进行读写操作。
```c
int close(int sockfd);
```
但是在内核中TCP会尝试把已经排队等待发送的数据发送完，然后发送正常的TCP的4路挥手来终止连接。

TCP的这种默认行为可以通过SO_LINGER套接字选项来改变。

如果，一个套接字描述符在通过fork在父子进行之间共享，那么在一个进程中调用close，只是把描述符的引用计数减1，只有在引用计数为0后，才会真正的终止一个连接。

如果确实不管怎样，都想终止连接，那就调用shutdown函数。

#### getsockname和getpeername函数
这两个函数用来获取本地或者对端的协议地址，包括IP和端口号。不要被这两个函数名中的name给迷惑了，它们其实和名字没有关系，主要是用来获取IP地址和端口的。

需要这两个函数的理由：
- 调用connect前，没有bind一个本地地址，内核会选择一个地址，此时getsockname用来获取这个地址，当然是包括IP和端口号的。
- 客户端如果调用了bind，指定的端口号为0，那么使用getsockname来获取具体的端口号。
- 服务端调用bind，指定了一个统配IP地址，一般是0，也就是0.0.0.0，那么通过accept成功后，可以通过getsockname获取本地使用的IP地址。
- 服务端accept后，fork出子进程，那么子进程只能通过getpeername获取客户端的IP和端口。


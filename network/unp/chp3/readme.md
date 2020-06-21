### 套接字编程

#### 套接字地址结构
常见的有5种套接字地址结构。

1. IPv4对应的套接字
```c
struct in_addr {
    in_addr_t   s_addr;
}
struct sockaddr_in {
    uint8_t         sin_len;         // 本结构体的自身长度，固定为16字节 
    sa_family_t     sin_family;      // 为AF_INT
    in_port_t       sin_port;        // 16位的端口号，网络字节序
    struct in_addr  sin_addr;        // 32位的IPv4的地址，网络字节序
    char            sin_zero[8];     // 未使用
}
```
IPv4地址结构总长度为16字节：1 + 1 + 2 + 4 + 8。


2. IPv6对应的套接字
```c
struct in6_addr {
    uint8_t     s6_addr[16];               // 128位的IPv6的地址，采用的网络字节序
}
struct sockaddr_in6 {
    uint8_t             sin6_len;         // 本结构体的总的字节长度，固定为28字节
    sa_family_t         sin6_family;      // 为AF_INET6
    in_port_t           sin6_port;        // 16位端口号，也是网络字节序，0-65535

    uint32_t            sin6_flowinfo;    // 流信息，未定义
    struct  in6_addr    sin6_addr;        // IPv6的地址，  网络字节序

    uint32_t            sin6_scope_id;
}
```

IPv6地址结构总长为28字节：1 + 1 + 2 + 4 + 16 + 4，分别代表：
1  字节表示长度
1  字节表示协议，IPv6
2  字节表示端口号
4  字节流信息
16 字节IPv6的地址
4  字节的scope id

3. Unix本地套接字
```c
struct sockaddr_un { // un 表示unix

}
```

4. 数据链路套接字
```c
struct sockaddr_dl {  // dl 表示数据链路

}
```

5. 存储套接字  
是IPv6套接字API定义的一种新的通用的套接字地址结构
```
struct sockaddr_storage {
    uint8_t        ss_len;
    sa_family_t    ss_family;
}
```

#### 通用套接字地址结构
很多套接字函数中的参数是一个指向通用的套接字地址结构的指针。我们调用这些函数前需要做以下强制类型转换。这些套接字函数内部，能够根据结构体中的第二个字节sa_family_t属性(AF_INET, AF_INET6, AF_LOCAL, AF_LINK, AF_XXX)，区别出来该地址结构具体是5种类型中的哪一种。

通过的地址结构定义如下：
```c
struct sockaddr {
    uint8_t        sa_len;
    sa_family_t    sa_family;
    char           sa_data[14];
}
// 我们通常使用如下类型定义，给它取一个简短的别名为 SA
typedef struct sockaddr SA;
```


#### 套接字地址结构的传递

1. 有3个函数使用的套接字地址结构作为参数，是把地址结构从用户进程传到系统内核：
- connect
```c
struct sockaddr_in serv;
connect(sockfd, (SA*) &serv, sizeof(serv))
```
- bind
- sendto

2. 另有4个函数使用套接字地址结构作为参数，是把地址结构从系统内存传递到用户进程:
- accept
- recvfrom
- getsockname
- getpeername


#### 字节序

套接字地址结构中的端口号，IP地址等属性都是使用网络字节序来存储的。所以我们读写这些属性的时候，就需要对字节序进行转换。

字节序分为如下两种：
- 大端字节序（bid-endian），也就是高位存储在内存的起始位置
- 小端字节序（litter-endian），也就是低位存储在内存的起始地址

不同的操作系统默认使用的字节序可能是不一样的，我们称一个系统上使用的字节序为主机字节序（host byte order），这个字节序可能是大端，也可能是小端。

虽然不同操作系统可能使用不同的字节序，但是他们通过网络进行数据传输时，传输的数据必须使用相同的字节序。在网络上传输数据时使用的字节序我们就称之为网络字节序。

IP协议规定网络字节序使用 **大端字节序**。

有如下4个函数用来完成主机字节序和网络字节序之间的转换，它们定义在<netinet/in.h>中：
- uint16_t htons(uint16_t host16bitvalue);
- uint32_t htonl(uint32_t host32bitvalue);
- uint16_t ntohs(uint16_t net16bitvalue);
- uint32_t ntohl(uint32_t net32bitvalue);

从函数名字就可以看出具体对应的是哪种转换:
其中h表示host, n表示network，s表示short，16位，l表示long，32位。

#### IP地址转换函数
IP地址在套接字地址结构中存储的是一个整数，但是我们经常却是使用点分十进制来表示一个IP地址的。所以他们两者之间也需要转换，有如下一些方法来完成，它们定义在<arpa/inet.h>中：
- inet_aton
- inet_ntoa
- inet_addr

上面3个函数，只能完成IPv4的地址的转换。

以下两个比较新的函数，对IPv4和IPv6都适用：
- inet_pton
- inet_ntop


### 常用头文件

#### <string.h>

里面包含了大量处理字符串的函数，C字符串以空字符结尾。这些函数大都以str开头，如：

- strcpy
- strcmp
- strncpy

#### <arpa/inet>
包含如下IP地址转换些函数：
- inet_pton
- inet_ntop
- inet_aton
- inet_ntoa
- inet_addr

#### <netinet/in.h>
包含如下字节序转换函数：
- htons
- htonl
- ntohl
- ntohs
	TCP是传输层的一种协议。提供的是面向连接、可靠的、字节流的服务  

# 什么是socket？

所谓套接字(Socket)，就是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。一个套接字就是网络上进程通信的一端，提供了应用层进程利用网络协议交换数据的机制。从所处的地位来讲，==套接字上联应用进程，下联网络协议栈==，是应用程序通过网络协议进行通信的接口，是应用程序与网络协议进行交互的接口

 <img src="img/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%EF%BC%9ALinux%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E5%91%A2%E5%9F%BA%E7%A1%80API.img/image-20201202135748906.png" alt="image-20201202135748906" style="zoom:50%;" />

Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

# 主机字节序与网络字节序

大端字节序：一个整数的高位字节（23~31bit）存储在内存的低地址处，低位字节（0~7bit）存储在内存的高地址处

小端字节序：整数的高位字节存储在内存的高地址处，而低位字节存储在内存的低地址处

主机字节序：不同的芯片，所采用的数值存储方式不同：大端模式&小端模式，现代PC大多采用小端字节序，因此小端字节序又被称为主机字节序

网络字节序： 统一使用大端模式来表示数据  

> 在两台使用不同字节序的主机之间传递数据时，可能会出现冲突。所以，在将数据发送到网络时规定整形数据使用大端字节序，所以也把大端字节序称为网络字节序列。对方接收到数据后，可以根据自己的字节序进行转换。  
>
> ![image-20201203223424209](img/%E7%BD%91%E7%BB%9C%EF%BC%9ATCP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20201203223424209.png)

#### 字节序的转换

```c
#include <netinet/in.h>
uint32_t ntohl (uint32_t __netlong); // 网络字节序转化为主机字节序 long  //net to host long
uint16_t ntohs (uint16_t __netshort); // 网络字节序转化为主机字节序 short
uint32_t htonl (uint32_t __hostlong); // 主机字节序转化为网络字节序 long
uint16_t htons (uint16_t __hostshort); // 主机字节序转化为网络字节序 short
```

# 套接字的地址结构

运行在两个不同主机上的进程想要通信： 必须知道 IP地址 端口号  

```c
#include<bites/socket.h>

// 通用的地址结构
struct sockaddr
{
	sa_family_t sa_family;
	char sa_data[14];
};

// IPV4 专有的地址结构
struct sockaddr_in
{
	sa_family_t sin_family; // 地址簇 AF_INET
	uint16_t sin_port; // 端口号： 将主机字节序转化为网络字节序 0--1024 系统预留 1025 -- 4096 知名端口号 4097 -- 65535
	struct in_addr sin_addr;
};
struct in_addr
{
	uint32_t s_addr; // IP地址 以字符串形式来表示一个点分十进制。 IP地址的转化
};
```

sa_family 成员是地址族类型（sa_family_t） 的变量。地址族类型通常与协议族类型对应。常见的协议族和对应的地址族如下图所示：  

![image-20201203224241869](img/%E7%BD%91%E7%BB%9C%EF%BC%9ATCP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20201203224241869.png)

#### IP地址转化的方法  

通常，人们习惯用点分十进制字符串表示 IPV4 地址，但编程中我们需要先把它们转化为整数方能使用，下面函数可用于点分十进制字符串表示的 IPV4 地址和网络字节序整数表示的 IPV4 地址之间的转换：  

```c
#include <arpa/inet.h>
uint32_t inet_addr (const char *__cp); // 将点分十进制的字符串转化为uint32_t类型
char * inet_ntoa (struct in_addr __in); // 将struct in_addr类型的变量转化为char*字符串1
```

# TCP的网络接口  

#### 头文件

```c
#include <sys/types.h>   //Unix/Linux系统的基本系统数据类型的头文件，含有size_t，time_t，pid_t等类型
#include <sys/socket.h>  //以下socket方法相关的头文件
```

#### 创建socket套接字  

```c
int socket(int _domain, int _type, int _protocol);
```

返回值： 成功返回创建的套接字的文件描述符socket 失败返回-1

domain： 协议簇 AF_INET TCP/IP协议

type： 具体的协议 SOCK_STREAM --> tcp SOCK_DGRAM --> UDP

protocol: 在前两个值的协议基础下的一个具体协议，一般设置为0，表示使用默认协议  

#### 命令（绑定）socket套接字  

```c
int bind (int __fd, struct sockaddr * __addr, socklen_t __len);
```

bind()将 sockfd 与一个 socket 地址绑定，成功返回 0，失败返回-1  

fd： socket方法返回的套接字的文件描述符

addr： 服务器的地址结构变量的地址 需要类型强转

len： addr的长度  

#### 启动监听的方法

```c
int listen (int __fd, int __n);
```

listen()创建一个监听队列以存储待处理的客户连接

返回值： 成功返回0， 失败返回-1

fd： socket方法返回的套接字的文件描述符

n： 内核创建的用于维护已完成连接的客户端的个数： n+1  

#### 获取一个连接

```c
int accept (int __fd, struct sockaddr * __addr, socklen_t *__addr_len);
```

返回值： 成功返回描述这个连接的文件描述符， 失败返回-1

fd： socket创建的文件描述符

addr： 用于保存客户端的地址信息

addr_len： addr的长度  

#### 读取数据

```c
ssize_t recv (int __fd, void *__buf, size_t __n, int __flags);
```

recv()从本端的接收缓冲区中读取数据，如果接收缓冲区中没有数据，则 recv()方法会阻塞。返回值是实际读到的字节数，如果recv()返回值为 0， 说明对方已经关闭了 TCP 连接  

fd： 需要读取数据的文件描述符

buf： 读取的数据存储的缓冲区的首地址

n： 一次能够读取的数据长度，单位是字节

flag： 标志，默认给0  

#### 发送数据

```c
ssize_t send (int __fd, const void *__buf, size_t __n, int __flags);
```

send()方法用来向 TCP 连接的对端发送数据。 send()执行成功，只能说明将数据成功写入到发送端的发送缓冲区中，并不能说明数据已经发送到了对端。 send()的返回值为实际写入到发送缓冲区中的数据长度  

fd： 需要写入数据的文件描述符

buf： 写入的数据存储的缓冲区的首地址

n： 一次写入的真实数据长度，单位是字节

flag： 标志，默认给0  

#### 客户端发起连接的方法

```c
int connect (int __fd, struct sockaddr * __addr, socklen_t __len);
```

客户端需要通过此系统调用来主动与服务器建立连接  

返回值： 成功返回0， 失败返回-1

fd: socket创建的文件描述符

addr: 服务器的地址信息

len： addr的长度  

#### 关闭一个文件描述符  

```c
int close(int __fd);
```

# TCP服务器端的编程流程

 TCP 提供的是面向连接的、可靠的、字节流服务。 TCP 的服务器端和客户端编程流程如下： 

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9ATCP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20201203230013040.png" alt="image-20201203230013040" style="zoom:50%;" /> 

 TCP服务器端的编程流程如下：

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9ATCP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20201203221447623.png" alt="image-20201203221447623" style="zoom:50%;" />

==socket()==方法是用来创建一个套接字，有了套接字就可以通过网络进行数据的收发。这也是为什么进行网络通信的程序首先要创建一个套接字。创建套接字时要指定使用的服务类型，使用 tcp 协议选择流式服务（SOCK_STREAM）

==bind()==方法是用来指定套接字使用的 IP 地址和端口。 IP 地址就是自己主机的地址，如果主机没有接入网络，测试程序时可以使用回环地址“127.0.0.1”。端口是一个 16 位的整形值，一般 0-1024 为知名端口，如 http 使用的 80 号端口。这类端口一般用户不能随便使用。其次，1024-4096 为保留端口， 用户一般也不使用。 4096 以上为临时端口，用户可以使用。在 Linux上， 1024 以内的端口号，只有 root 用户可以使用。  

> 因为TCP是面向连接的，所以接下来用listen创建监听队列

==listen()==方法是用来创建监听队列。 监听队列有两种，一个是存放未完成三次握手的连接，一种是存放已完成三次握手的连接。 listen()第二个参数就是指定已完成三次握手队列的长度。  

==accept()==处理存放在 listen 创建的已完成三次握手的队列中的连接。每处理一个连接，则accept()返回该连接对应的套接字描述符。如果该队列为空，则 accept 阻塞。  

服务器端代码示例如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <string.h>
#include <unistd.h> //对于类 Unix 系统，unistd.h 中所定义的接口通常都是大量针对系统调用的封装（英语：wrapper functions），如 fork、pipe 以及各种 I/O 原语（read、write、close 等等）。

#include <sys/types.h>//Unix/Linux系统的基本系统数据类型的头文件
#include <sys/socket.h>//套接字接口之类的头文件
#include <netinet/in.h>//Internet地址族
#include <arpa/inet.h>//地址转换函数

int main()
{
	int listenfd = socket(AF_INET, SOCK_STREAM, 0);//创建套接字
	assert(-1 != listenfd);
    
	struct sockaddr_in addr; //套接字的地址结构，运行在两个不同主机上的进程想要通信： 必须知道 IP地址 端口号  
	memset(&addr, 0, sizeof(addr));
	addr.sin_family = AF_INET;
	addr.sin_port = htons(6000);
	addr.sin_addr.s_addr = inet_addr("127.0.0.1"); // 回环地址
    
	// bind方法失败的原因： 1、IP地址不正确 2、端口号不正确（没有使用权限， 端口号被其他进行使用）
	int res = bind(listenfd, (struct sockaddr*)&addr, sizeof(addr));//绑定套接字与地址结构
	assert(-1 != res);
    
	res = listen(listenfd, 5); //创建监听队列
	assert(-1 != res);
    
	while(1) // 循环接收不同客户端的链接
	{
		struct sockaddr_in cli_addr;
		socklen_t cli_len = sizeof(cli_addr);
		int c = accept(listenfd, (struct sockaddr*)&cli_addr, &cli_len);//接收连接
		if(c == -1)
		{
			printf("Get one client link fail\n");
			continue;
		} 
        while(1) //循环和一个客户端通讯
		{
			char buff[128] = {0};
			int n = recv(c, buff, 127, 0); // 如果没有数据到达则会阻塞，直到有数据或者客户端断开链接
			if(n <= 0)
			{
				printf("client will unlink\n");
				break;
			} 
            
      	  	printf("%d:%s\n", c, buff);
			send(c, "OK", 2, 0); //给客户端的回复
		} 
        close(c); // 服务器程序关闭接收的客户端链接
	} 
    
    close(listenfd); // 关闭该服务器程序前关闭监听的套接字
    
	exit(0);
}
```

# TCP客户端的编程流程

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9ATCP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20201203231606542.png" alt="image-20201203231606542" style="zoom:50%;" />

==socket()==方法是用来创建一个套接字，有了套接字就可以通过网络进行数据的收发。这也是为什么进行网络通信的程序首先要创建一个套接字。创建套接字时要指定使用的服务类型，使用 tcp 协议选择流式服务（SOCK_STREAM）

==connect()==方法一般由客户端程序执行，需要指定连接的服务器端的 IP 地址和端口。该方法执行后，会进行三次握手， 建立连接  

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9ATCP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20201203225611338.png" alt="image-20201203225611338" style="zoom:50%;" />

==send()==方法用来向 TCP 连接的对端发送数据。 send()执行成功，只能说明将数据成功写入到发送端的发送缓冲区中，并不能说明数据已经发送到了对端。 send()的返回值为实际写入到发送缓冲区中的数据长度。  

==recv()==方法用来接收 TCP 连接的对端发送来的数据。 recv()从本端的接收缓冲区中读取数据，如果接收缓冲区中没有数据，则 recv()方法会阻塞。返回值是实际读到的字节数，如果recv()返回值为 0， 说明对方已经关闭了 TCP 连接。  

==close()==方法用来关闭 TCP 连接。此时，会进行四次挥手。  

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9ATCP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20201203225846949.png" alt="image-20201203225846949" style="zoom:50%;" />

```c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <string.h>
#include <unistd.h>

#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h> //字节序的转换
#include <arpa/inet.h>  //IP地址转换

int main()
{
	int sockfd = socket(AF_INET, SOCK_STREAM, 0);//创建套接字
	assert(-1 != sockfd);
    
	struct sockaddr_in ser_addr;
	memset(&ser_addr, 0, sizeof(ser_addr));
	ser_addr.sin_family = AF_INET;
	ser_addr.sin_port = htons(6000);
	ser_addr.sin_addr.s_addr = inet_addr("127.0.0.1"); // 回环地址
    
	int res = connect(sockfd, (struct sockaddr*)&ser_addr, sizeof(ser_addr));//指定连接的服务器端的 IP 地址和端口
	assert(-1 != res);
    
	while(1)
	{
		printf("input: ");
		char buff[128] = {0};
		fgets(buff, 127, stdin);
		if(strncmp(buff, "end", 3) == 0)
		{
			break;
		} 
        
        send(sockfd, buff, strlen(buff) - 1, 0);
        
		memset(buff, 0, 128);
		recv(sockfd, buff, 127, 0);
		printf("%s\n", buff);
	} 
    
    close(sockfd);
	exit(0);
}
```

服务器和客户端的执行结果  

![image-20201203222206380](img/%E7%BD%91%E7%BB%9C%EF%BC%9ATCP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20201203222206380.png)
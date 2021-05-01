==UDP协议特点：无连接、不可靠、数据报服务==

## UDP编程流程

<img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9AUDP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20210103154716351.png" alt="image-20210103154716351" style="zoom:67%;" />

## UDP接口原型

```c
int recvfrom(int sockfd, void *buf, size_t size,int flag, 
             struct sockaddr *peer_addr, socklen_t * addr_len);

//peer_addr: 要来保存recvfrom接收到的数据是来自哪台主机的地址信息
//addr_len:  地址结构的长度
```

```cpp
int sendto(int sockfd, void * buf, size_t size, int flag,
           struct sockaddr *peer_addr, socklen_t addr_len);

//peer_addr: 用来指定数据要发送给谁（地址信息）
//addr_len:  地址信息的长度
```

## 代码示例

查看主机地址：ifconfig

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9AUDP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20210103170904035.png" alt="image-20210103170904035" style="zoom:67%;" />

服务器端代码：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <string.h>
#include <unistd.h>

#include <sys/socket.h>
#include <sys/types.h>
#include <arpa/inet.h>
#include <netinet/in.h>

int main()
{
	int sockfd = socket(AF_INET, SOCK_DGRAM,0);  //SOCK_DGRAM是UDP协议
	assert(-1 != sockfd);

	struct sockaddr_in ser_addr;
	memset(&ser_addr, 0, sizeof(ser_addr));
	ser_addr.sin_family = AF_INET;
	ser_addr.sin_port = htons(6000);
	ser_addr.sin_addr.s_addr = inet_addr("192.168.133.132");
	int res = bind(sockfd,(struct sockaddr *)&ser_addr,sizeof(ser_addr));

	//循环接收不同客户端的数据
	while(1)
	{
		char buff={0};
		struct sockaddr_in cli_addr;
		socklen_t cli_len=sizeof(cli_addr);

		int n = recvfrom(sockfd, buff, 127, 0, (struct sockaddr*)&cli_addr, &cli_len);
		if(n<=0)
		{
			break;
		}
		printf("%s: %d -- %s\n",inet_ntoa(cli_addr.sin_addr), ntohs(cli_addr.sin_port),buff);

		n=sendto(sockfd, "OK", 2,0,(struct sockaddr*)&cli_addr, cli_len);

		if(n<=0)
		{
			break;
		}
	}
	close(sockfd);
	exit(0);
}
```

客户端代码：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <assert.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main()
{
	int sockfd = socket(AF_INET, SOCK_DGRAM, 0);
	assert(sockfd != -1);

	struct sockaddr_in addr;
	memset(&addr, 0, sizeof(addr));
	addr.sin_family = AF_INET;
	addr.sin_port =htons(6000);
	addr.sin_addr.s_addr = inet_addr("192.168.133.132");
	
	while(1)
	{
		printf("please input: ");
		char buff[128] = {0};
		fgets(buff,127,stdin);

		if(strncmp(buff, "end",3)==0)
		{
			break;
		}
		int n = sendto(sockfd,buff,strlen(buff),0,(struct sockaddr*)&addr,sizeof(addr));
		if(n<=0)	break;
		memset(buff, 0, 128);
		int m = recvfrom(sockfd, buff,127,0,NULL,NULL);
		if(m<=0)	break;
		printf("%s\n",buff);
	}
	close(sockfd);
	exit(0);
}

```

运行结果：（上边代码在我的虚拟机运行有问题）

## <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9AUDP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20210105095148583.png" alt="image-20210105095148583" style="zoom:67%;" />

## UDP的报头结构

![image-20210105095312270](img/%E7%BD%91%E7%BB%9C%EF%BC%9AUDP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20210105095312270.png)

UDP的报头固定是8个字节

- UDP的报文段长度：表示这个UDP报文段的报头+数据部分的总长度，一个UDP报文段数据部分的总长度为总长度 - 8
- 冗余校验码：会对整个UDP数据报进行冗余校验

UDP的优势：

- 没有确认机制和超时重传机制，发送方发送报文段的效率就很高
- 头部固定部分比较小，一个UDP报文段所携带的上层协议的数据就比TCP多一点
- UDP的实现相对比较简单

## UDP的数据报服务

UDP数据报服务的特点：发送端应用程序每执行一次写操作，UDP模块就将其封装成一个UDP数据报发送。接收端必须及时针对每一个UDP数据报执行读操作，否则就会丢包。并且，如果用户没有指定足够的应用程序缓冲区来读取UDP数据，则UDP数据将被截断

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9AUDP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20210105100012450.png" alt="image-20210105100012450" style="zoom:50%;" />

- sendto和recvfrom是一一对应的
- sendto一次，底层就发送一个UDP报文段，对方就接收这一个UDP报文段
- 如果一次recvfrom没有将一个UDP报文段中的数据读取完成，则剩余的数据会被丢弃

测试数据的丢弃：（把服务器端recvfrom改成接收5个字节来测试）

<img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9AUDP%E7%BC%96%E7%A8%8B%E6%B5%81%E7%A8%8B.img/image-20210105100242497.png" alt="image-20210105100242497" style="zoom:67%;" />
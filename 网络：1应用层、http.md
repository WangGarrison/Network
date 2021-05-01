应用层协议

- HTTP：（HyperText Transfer Protocol）超文本传输协议，web所使用的协议，（区分HTML：超文本标记语言）

- FTP：文件传输协议，网盘，FTP站点
- TELNET：远程登录协议

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210105105843827.png" alt="image-20210105105843827" style="zoom:50%;" />

# HTTP协议

区分三个概念

- HTTP：超问文本传输协议，浏览器与后台的web服务器之间传递数据(html文件)时候的控制协议
- HTML：超文本标记语言，通过不同的标签来标识数据的不同样式或者作用
- HTTPS：安全的超文本传输协议，HTTP+SSL

HTTP与HTTPS的区别

- https协议需要到CA申请证书，一般免费证书较少，因而需要一定费用。
- http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl/tls加密传输协议。
- http和https使用的是完全不同的连接方式，用的端口也不一样，http用80端口，是网页服务器的访问端口，用于网页浏览，https默认端口号是443
- http的连接很简单，是无状态的；HTTPS协议是由SSL/TLS+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

# 浏览器和web服务器的通信流程

 浏览器与 web 服务器在应用层通信使用的是 http 协议（超文本传输协议），而 http 协议在传输层使用的是 tcp 协议。那么浏览器需要和 web 服务器三次握手建立连接后，才可以发送 http 请求报文，服务器收到请求报文后，向浏览器回复 http 应答报文。  

浏览器向服务器发起连接前，需要得到服务器的 IP 及端口。用户在浏览器中通常只输入网址（网站域名） ,浏览器会通过 DNS 服务查询获取到服务器的 IP 地址。 对于端口来讲，使用 http 协议的程序一般默认使用 80 端口。  

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210106111610505.png" alt="image-20210106111610505" style="zoom:50%;" />

DNS服务器：一般指域名服务器。DNS（Domain Name Server，域名服务器）是进行域名(domain name)和与之相对应的IP地址 (IP address)转换的服务器

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210105111344723.png" alt="image-20210105111344723" style="zoom:50%;" />

# 短连接和长连接

浏览器服务器建立连接后，如果两次以上的请求复用同一个 tcp 连接，则称之为长连接。如果浏览器发送一次请求报文，服务器回复一次应答就断开连接，下次交互再重新进行三次握手建立连接，那么就被称作短连接。使用长连接显然是更好一些，可以减少网络中的同步报文，减少了网络上为建立TCP连接导致的负荷，也使得服务器的响应速度变快  

- 短连接：close，一次http请求服务器应答之后就将这个连接断开

- 长连接：keep-alive，多个http请求可以共用同一个连接

HTTP请求和应答报头中Connection字段就是专门用于告诉对方一个请求完成之后该如何处理连接的，短连接用close，长连接用keep-alove

使用`netstat`命令可以查看连接状态

# 常见的web服务器

- Apache：简单、速度快、性能稳定，并可做代理服务器
- IIS：Internet Information Server，安全性、强大、灵活
- Nginx：小巧而高效，可以做高效的负载均衡和反向代理
- Tomcat：技术先进、性能稳定、免费

# HTTP的请求报头

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210106111738540.png" alt="image-20210106111738540" style="zoom:60%;" />

请求报文段实例：

![image-20210106113649907](img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210106113649907.png)

# HTTP的请求方法

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210106112818592.png" alt="image-20210106112818592" style="zoom:67%;" />

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210106112828858.png" alt="image-20210106112828858" style="zoom:67%;" />

# HTTP应答报头

 web服务器给浏览器回馈数据时的报头

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210106113211971.png" alt="image-20210106113211971" style="zoom:67%;" />

# http的应答状态

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210106113256221.png" alt="image-20210106113256221" style="zoom:50%;" />

 <img src="img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210106113308694.png" alt="image-20210106113308694" style="zoom:50%;" />

# C语言实现一个简单的web服务器

myweb.c

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <string.h>
#include <unistd.h>

#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

void DealLink(int fd)
{
    char buff[1024]={0};
    int n = recv(fd, buff, 1023, 0);
    if(n<=0) return;
    printf("%s", buff);
}

int main()
{
    int listenfd = socket(AF_INET, SOCK_STREAM, 0);
    assert(-1 != listenfd);
    
    struct sockaddr_in addr;
    memset(&addr, 0, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_port = htons(80);
    addr.sin_addr.s_addr = inet_addr("192.168.133.132");//Linux下使用ifconfig查看ip
    
    int res = bind(listenfd, (struct sockaddr*)&addr, sizeof(addr));
    assert(-1 != res);
    
    res = listen(listenfd,2);
    assert(-1 != res);
    
    while(1)
    {
        struct sockaddr_in cli;
        socklen_t len = sizeof(cli);
        
        int c = accept(listenfd, (struct sockaddr*)&cli, &len);
        if(c < 0)
        {
			continue;
        }
        printf("one new client link\n");
        
        DealLink(c);
        
        close(c); //短连接
    }
    close(listenfd);
    exit(0);
}
```

0~1023只有root用户才能使用，所以要用root用户使用80端口，运行的时候要使用：sudo ./myweb

启动myweb后，在浏览器中打开192.168.133.132

![image-20210105115358029](img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210105115358029.png)

服务器终端接收到的数据如下：

![image-20210105115446782](img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210105115446782.png)

作业：改进处理函数

![image-20210105115824506](img/%E7%BD%91%E7%BB%9C%EF%BC%9A1%E5%BA%94%E7%94%A8%E5%B1%82%E3%80%81http.img/image-20210105115824506.png)
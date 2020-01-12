# Linux Socket编程

## 什么是Socket？

Unix/Linux基本哲学之一就是“一切皆文件”，都可以通过open文件来获得文件描述符，然后通过这个文件描述符进行IO的操作，即读写操作read和write，完成后用close关闭。Socket可以视为网络通讯两端的描述符，在创建socket后，网络通讯的两端都是对socket进行读写操作来完成网络通讯，通讯结束后，用close关闭socket。这些都非常类似于文件描述符。

## TCP协议通讯的基本过程

目前，基于以太网的网络通讯主要有两大协议——TCP和UDP（当然还有其他协议）。TCP是面向连接的通讯，而UDP是面向无连接的协议，具体的含义，在了解了它们通讯的基本过程后就清楚了。我们先看基于Socket的TCP协议通讯的基本过程。

![Image](/tcp.png)

如上图所示，左边为客户端进程，右边为服务端进程。服务端进程首先用socket函数创建一个套接字socket，接着使用bind函数将本机的某个端口绑定在这个socket上，接着使用listen函数监听，等待客户端进程的接入，此时会阻塞进程以等待客户端的接入；一旦有客户端接入后，接着使用accept函数接受客户端的接入，accept函数如果执行成功，代表连接已经创建，就可以在这个连接的基础上进行读写操作（这就是面向连接的通讯）；然后可以用read或recv（receive）函数接受客户端发来的数据，此时也与文件读写操作类似，会阻塞进程直至接受数据完毕，也可以用write或send函数给客户端发送数据。

客户端进程首先用socket函数创建一个socket，然后在connect函数中指定服务端socket的地址和端口去连接服务端的socket，如果连接成功，表示连接已创建，之后的通讯都是基于这个连接的（这就是面向连接的通讯），之后与前面所述的服务端进程类似，可以用read/recv/write/send函数进行通讯，通讯完毕后执行close关闭。

先给个简单例子说明通讯过程

服务端代码
```
#include <sys/types.h>
#include <sys/socket.h>	  // 包含套接字函数库
#include <stdio.h>
#include <netinet/in.h>	  // 包含AF_INET相关结构
#include <arpa/inet.h>	  // 包含AF_INET相关操作的函数
#include <unistd.h>
#include <string.h>
#define PORT 3339

int setaddr(struct sockaddr_in* addr, int port)
{
	addr->sin_family = AF_INET;
	addr->sin_addr.s_addr = INADDR_ANY;
	addr->sin_port = htons(port);
	return  0;
}

int main()
{
	char sendbuf[256] = "OK";
	char cstr[32];
	char buf[256];
	int s_fd, c_fd;	      // 服务器和客户套接字标识符
	int s_len, c_len;     // 服务器和客户消息长度
	struct sockaddr_in s_addr;	// 服务器套接字地址
	struct sockaddr_in c_addr;	// 客户套接字地址

	s_fd = socket(AF_INET, SOCK_STREAM, 0);	// 创建套接字
	setaddr(&s_addr, PORT);
	s_len = sizeof(s_addr);
	bind(s_fd, (struct sockaddr *) &s_addr, s_len);	// 綁定套接字与设置的端口号
	listen(s_fd, 10);	// 监听状态，守候进程
	printf("请稍候，等待客户端发送数据\n");
	c_len = sizeof(c_addr);
	//接收客户端连接请求
	c_fd = accept(s_fd, (struct sockaddr *) &c_addr, (socklen_t *__restrict) &c_len);

	inet_ntop(AF_INET, &(c_addr.sin_addr), cstr, INET_ADDRSTRLEN);
	while (1) 
	{
		if (recv(c_fd, buf, 256, 0) > 0) //接收消息recv(c_fd,buf,256,0)>0
		{
			printf("收到客户端%s消息:\n %s\n", cstr, buf); //输出到终端
			if (strcmp(buf, "-1") == 0)
			{
				printf("close connection");
				strcpy(sendbuf, "quit");

				send(c_fd, sendbuf, sizeof(sendbuf), 0); //回复消息
				break;
			}
			else {
				send(c_fd, sendbuf, sizeof(sendbuf), 0); //回复消息
			}
		}
	}
	close(c_fd); // 关闭连接
}
```
客户端代码
```
#include <sys/types.h>
#include <sys/socket.h>						// 包含套接字函数库
#include <stdio.h>
#include <netinet/in.h>						// 包含AF_INET相关结构
#include <arpa/inet.h>						// 包含AF_INET相关操作的函数
#include <unistd.h>
#include <string.h>
#define PORT 3339

int setaddr(struct sockaddr_in* addr, char *sstr, int port)
{
	addr->sin_family = AF_INET;
	//addr->sin_addr.s_addr=INADDR_ANY;
	inet_pton(AF_INET, sstr, &(addr->sin_addr));
	addr->sin_port = htons(port);
	return  0;
}

int main(int argc, char *argv[]) {
	int sockfd;			// 客户端套接字标识符
	int len;			// 客户消息长度
	struct sockaddr_in addr;	// 客户端套接字地址
	int newsockfd;
	char buf[256] = "come on!";     //要发送的消息
	int len2;
	char rebuf[256];
	sockfd = socket(AF_INET, SOCK_STREAM, 0);	// 创建套接字

	if (argc != 2)
	{
		printf("wrong usage\n");
		return 1;
	}

	setaddr(&addr, argv[1], PORT);
	len = sizeof(addr);
	newsockfd = connect(sockfd, (struct sockaddr *) &addr, len);	//发送连接服务器的请求
	if (newsockfd == -1) {
		perror("连接失败");
		return 1;
	}
	len2 = sizeof(buf);
	while (1) {
		printf("请输入要发送的数据:");
		scanf("%s", buf);
		send(sockfd, buf, len2, 0); //发送消息
		if (recv(sockfd, rebuf, 256, 0) > 0)//接收新消息
		{ 
			printf("收到服务器消息:\n%s\n", rebuf);//输出到终端
			if (strcmp(rebuf, "quit") == 0)
			{
				printf("close connection\n");
				break;
			}
		}
	}
	close(sockfd);	 // 关闭连接
	return 0;
}
```

Windows客户端代码
```

#include <winsock2.h>
#include <stdio.h>
#include <Ws2tcpip.h>
#include <string.h>

#pragma comment(lib,"ws2_32.lib")

#define PORT 3339

int setaddr(struct sockaddr_in* addr, char *sstr, int port)
{
	addr->sin_family = AF_INET;
	inet_pton(AF_INET, sstr, &(addr->sin_addr));
	addr->sin_port = htons(port);
	return  0;
}

int main(int argc, char *argv[]) {
	int sockfd;						  // 客户套接字标识符
	int len;					      // 客户消息长度
	struct sockaddr_in addr;		  // 客户套接字地址
	int newsockfd;
	char buf[256] = "come on!";      //要发送的消息
	int len2;
	char rebuf[256];

	//初始化WSA
	WORD sockVersion = MAKEWORD(2, 2);
	WSADATA wsaData;
	if (WSAStartup(sockVersion, &wsaData) != 0)
	{
		return 0;
	}

	sockfd = socket(AF_INET, SOCK_STREAM, 0);	// 创建套接字

	if (argc != 2)
	{
		printf("wrong usage\n");
		puts("scw 服务端ip");
		return 1;
	}
	
	setaddr(&addr, argv[1], PORT);
	
	len = sizeof(addr);
	newsockfd = connect(sockfd, (struct sockaddr *) &addr, len);	//发送连接服务器的请求
	if (newsockfd == -1) {
		perror("连接失败");
		return 1;
	}
	len2 = sizeof(buf);
	while (1) {
		printf("请输入要发送的数据:");
		scanf_s("%s", buf, len2);
		send(sockfd, buf, len2, 0); //发送消息
		if (recv(sockfd, rebuf, 256, 0)>0)//接收新消息
		{
			printf("收到服务器消息:\n%s\n", rebuf);//输出到终端
			if (strcmp(rebuf, "quit") == 0)
			{
				printf("close connection\n");
				break;
			}
		}
	}
	closesocket(sockfd);	 // 关闭连接
	return 0;
}
```
 

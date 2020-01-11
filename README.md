# Linux Socket编程

## 什么是Socket？

Unix/Linux基本哲学之一就是“一切皆文件”，都可以通过open文件来获得文件描述符，然后通过这个文件描述符进行IO的操作，即读写操作read和write，完成后用close关闭。Socket可以视为网络通讯两端的描述符，在创建socket后，网络通讯的两端都是对socket进行读写操作来完成网络通讯，通讯结束后，用close关闭socket。这些都非常类似于文件描述符。

## TCP协议通讯的基本过程

目前，基于以太网的网络通讯有两大协议——TCP和UDP。TCP是面向连接的通讯，而UDP是面向无连接的协议，具体的含义，在了解了它们通讯的基本过程后就清楚了。我们先看基于Socket的TCP协议通讯的基本过程。

![Image](/tcp.png)

如上图所示，左边为客户端进程，右边为服务端进程。服务端进程首先用socket函数创建一个套接字socket，接着使用bind函数将本机的某个端口绑定在这个socket上，接着使用listen函数监听，等待客户端进程的接入，此时会阻塞进程以等待客户端的接入；一旦有客户端接入后，接着使用accept函数接受客户端的接入，accept函数如果执行成功，代表连接已经创建，就可以在这个连接的基础上进行读写操作（这就是面向连接的通讯）；然后可以用read或recv（receive）函数接受客户端发来的数据，此时也与文件读写操作类似，会阻塞进程直至接受数据完毕，也可以用write或send函数给客户端发送数据。

客户端进程首先用socket函数创建一个socket，然后在connect函数中指定服务端socket的地址和端口去连接服务端的socket，如果连接成功，表示连接已创建，之后的通讯都是基于这个连接的（这就是面向连接的通讯），之后与前面所述的服务端进程类似，可以用read/recv/write/send函数进行通讯，通讯完毕后执行close关闭。

```
if (isAwesome){
  return true
}
```

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/jjqtony/LinuxSocketProgramming/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.

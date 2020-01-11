# Linux Socket编程

## 什么是Socket？

Unix/Linux基本哲学之一就是“一切皆文件”，都可以通过open文件来获得文件描述符，然后通过这个文件描述符进行IO的操作，即读写操作read和write，完成后用close关闭。Socket可以视为网络通讯两端的描述符，在创建socket后，网络通讯的两端都是对socket进行读写操作来完成网络通讯，通讯结束后，用close关闭socket。这些都非常类似于文件描述符。

## TCP协议通讯的基本过程

目前，基于以太网的网络通讯有两大协议——TCP和UDP。TCP是面向连接的通讯，而UDP是面向无连接的协议，具体的含义，在了解了它们通讯的基本过程后就清楚了。我们先看基于Socket的TCP协议通讯的基本过程。

![Image](/tcp.png)

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

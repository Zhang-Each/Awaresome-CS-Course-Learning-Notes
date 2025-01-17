# CMU-15/440 Distributed Systems 2: Networking Communication

> 分布式系统课程15-440学习笔记2，这一节主要讲网络通信相关的内容。

进程之间的交互是分布式系统中的核心，不研究位于不同机器上的进程之间的通信的分布式系统研究是没有意义的。

## 网络链接Network Links

**网络链接**是网络通信中的基本组成单位，节点指的就是一台计算机(当然有的时候一台计算机上会分出多个虚拟节点)

![image-20220623172117716](https://raw.githubusercontent.com/Zhang-Each/Image-Bed/main/img/image-20220623172117716.png)

如果我们需要组建更复杂的网络，就需要用更复杂的组网方式，通过网线将多台计算机连接成网络。

![image-20220629235707467](https://raw.githubusercontent.com/Zhang-Each/Image-Bed/main/img/image-20220629235707467.png)

复用Multiplexing是计算机网络中一种很常见的技术，即多个链接共享同一个信道进行通信，交换机在这个过程中起到了很重要的作用。

![image-20220630225907847](https://raw.githubusercontent.com/Zhang-Each/Image-Bed/main/img/image-20220630225907847.png)

包交换(Packet Switching)是最常见的通信方式，一个自包含(关于自包含，我的理解是可以通过数据包本身的设定来读取里面所有的内容)的数据包从起始点发出，在网络中独立地进行传播，直到抵达目的地。

但是在信道被复用的情况下，通过同一个信道传输多个不同的包可能会导致网络过载，这个时候就需要通过缓冲区和冲突控制的设计来解决问题。当缓冲区也溢出的时候，需要丢弃掉一些数据包。

## 通信信道模型

### 常见的评价指标

对于一个通信信道，我们需要关注它的这样几个指标：

- 延迟Latency，即从起始点到终点所需要的时间
- 容量(吞吐量)Capacity，即网络中最多能容纳的字节数，也就是网络的带宽
- 抖动Jitter，代表了延迟的变化
- 丢包率，也可以认为是可信度

其中，数据包的延迟可以具体分成四个部分，分别是**传播延迟，发送延迟，处理延迟和队列延迟**，四种延迟产生的原因如下图所示。

![image-20220701233202203](https://raw.githubusercontent.com/Zhang-Each/Image-Bed/main/img/image-20220701233202203.png)

### 最基本的Stop&Wait协议

Stop&Wait是一种最基本的通信协议，它的工作方式就是，发送方发出一个包给接收方，并等待接收方发送确认(ACK)，下一个包需要等收到接收方的确认之后才可以发出。如果接收方在一个规定的时间内没有发回ACK，那么发送方就会紧凑型超时重传。

### 数据包具体长啥样？

Slides里面给了一个具体的例子。这是一个**Ethernet Packet** 

![image-20220702104511457](https://raw.githubusercontent.com/Zhang-Each/Image-Bed/main/img/image-20220702104511457.png)

一般来说一个数据包会包括起始地址和目标地址，以及一些关键的标记位，以及具体的数据，这里的数据可能会是更上一层的数据包。

### 关于因特网的小知识

因特网Internet的定义是a network of networks. 它将不同的网络组合到了一起成为了世界性的网络。它有着非常强的**异构性**，每个网络里面的地址形式、网络性能、网络协议的定义、路由的算法、具体使用的网络技术等内容都有可能不同，将这些复杂的网络组合成为一个通用的大网络，其难度也可想而知。

因特网通过Naming、Routing以及多种服务模型实现了计算机之间的互联。

![image-20220702111602514](https://raw.githubusercontent.com/Zhang-Each/Image-Bed/main/img/image-20220702111602514.png)

## 网络服务模型

网络服务模型就是对一个网络所提供的服务的描述。包括网络传输数据的保证、对发送失败的处理、网络的可信程度、网络冲突控制、发包顺序等等。

为了更好地对网络服务模型中的不同功能模块进行管理，现在的网络系统往往采用分层的设计方式，将整个网络分成若干层，每一层负责具体的目标。比如这是一个常见的四层模型：

![image-20220703233958980](https://raw.githubusercontent.com/Zhang-Each/Image-Bed/main/img/image-20220703233958980.png)

它将网络分成了四个层级，分别是网络硬件、主机到主机的链接层、应用到应用的信道和具体的应用程序。在OSI网络模型中，整个网络被划分成了7层。每一层提供的服务都依赖于上一层，并且向它的下一层提供特定的服务，，不同层之间通过网络结构的设计进行交互，这些具体的接口定义也就是网络协议。不同主机上的同层级也可以进行交互

在网络中，各种各样的数据包都可以进行传输，虽然传的内容各有不同，但是都可以共用一个网络，这个过程称为Multiplexing，网络的每一层可能会有多种不同的实现，比如传输层就可以有TCP/UDP等多种协议，为了让接收端能够感知每一层使用的协议，需要在每个数据包的头部用特定的内容来表示不同的协议，在应用程序具体处理的时候，就可以根据数据包头部的信息来判断数据哪一种协议。

### TCP和UDP

当我们设计网络协议的时候，我们需要思考，我们是否需要保证传输的可靠性，为此，我们应该选用什么类型的设计。

![image-20220706224912136](https://raw.githubusercontent.com/Zhang-Each/Image-Bed/main/img/image-20220706224912136.png)

TCP和UDP是OSI体系中两种不同的传输层协议，UDP只提供最基本的多路复用和多路解编功能，并且以用户数据报的形式发送数据，不保证发出的数据一定能到达指定的目标，并且也不需要指定发送的数据，每个数据报之间是独立的。

相比之下，TCP提供了颗心的端到端传输，通过有顺序的字节流来传输数据，并是一种面向链接的协议(Connection-oriented)，需要在建立端到端的链接之后才能进行数据的传输。

### Client-Server体系

客户端-服务端(Client-Server)体系，即C/S体系是最经典的网络服务架构之一，客户端负责从服务端获取信息并和用户进行交互，服务端则负责向客户端提供对应的服务，二者通过网络相连接。

![image-20220706233521420](https://raw.githubusercontent.com/Zhang-Each/Image-Bed/main/img/image-20220706233521420.png)


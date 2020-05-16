# 计算机网络

### 浏览器工作原理

![](./images/what_do_i_do_when_type_url.jpg)

#### 浏览器格式化检查
  
浏览器对网址进行了格式化检验，首先确定这是一个有效的网址，才会进入下一步

#### DNS解析

##### 为什么需要DNS解析

DNS保存了互联网上所有域名与IP的映射关系，我们可以通过主机名获取对应主机的IP地址。那么为什么需要DNS解析这部分？这得分成两个子问题，为什么面向用户的部分需要使用域名，传输层却需要IP地址来标示一台主机。首先IP地址的长度为32比特(4字节），使用域名最少也要几十个字节，最长可达到255字节，如果传输层直接用域名增加了路由器的负担,传送数据也会花费更长的时间 ，整体效率很低。对于面向用户的应用层用IP地址来代替服务器名称也是能够正常工作的，但是,要记住一串由数字组成的IP地址显然难以记忆。

> 如果Web服务器使用了虚拟主机功能,有可能无法通过IP地址来访问。因为虚拟主机是寄存在服务器上的一个或多个没有实体的服务器，访问虚拟主机的域名的时候，先根据DNS解析的IP访问到实体主机，然后实体主机再根据域名把连接转发给对应的虚拟主机，DNS解析的IP只是实体主机的IP

##### DNS分层结构

我们日常解除的域名是分层的以 https://www.github.com/ 为例子: com 为顶级域名，github 为二级域名，www 为三级域名。相应的DNS结构也是采用分级树状系统创建域名数据库，从而提供域名解析服务。

![](./images/dns_tree_struct.png)

- ****根域名服务器**** 是域名解析体系的核心，如果要查询域名，均需要从根域名服务器开始查询。全球 IPv4 的根域名服务器只有13台，1个为主根服务器，放置在美国。其余12个均为辅根服务器，其中9个放置在美国，欧洲2个位于英国和瑞典，亚洲1个，位于日本。实际上到目前为止，所谓的只有13台根域名服务器，只是逻辑上的，分别编号从a到m，而物理上的根域名服务器已远不止 13 台。可以通过如下网站查看根域名服务器网站分布情况[https://root-servers.org](https://root-servers.org)

- ****顶级域名服务器****
根域名服务器负责返回顶级域 (如 .com 等) 的权威域名服务器地址。在拿到顶级DNS服务器地址后，再向顶级域名服务器发起查询请求，其返回一级域 (如 example.com) 的权威域名服务器地址

- ****次级域名服务器****

##### DNS解析流程

- 查询浏览器自身的DNS缓存：
首先浏览器会去搜索自身的DNS缓存，如果不存在对应缓存或者缓存过期了，就进行下一步。

- 查询操作系统自身的DNS缓存：
如果上一步没有找到则浏览器就会搜索操作系统自身的缓存，没有找到或者过期则解析结束

- 查询本地的hosts文件：
若操作系统的缓存也没有找到或失效，浏览器就会去读取本地的hosts文件（Hosts文件也可以建立域名到IP地址的绑定关系，可以通过编辑Hosts文件来达到名称解析的目的。 例如，我们需要屏蔽某个域名时，就可以将其地址指向一个不存在IP地址，以达到屏蔽的效果）。

- 浏览器发起一个DNS的系统调用：
hosts中没有找到对应的配置项的话，浏览器向本地主控DNS服务发起一个DNS的调用。

###### 客户端如何向DNS服务器发送请求

向DNS服务器发送请求的过程如下图所示：

![](./images/send_dns_request.jpg)

DNS解析的实际上就是调用 _gethostbyname_ 方法并将web服务的域名传入即可。调用解析器后,解析器会向运营商提供的DNS服务器发送查询消息。
运营商服务会先查找自身缓存找到对应条目，查看是否过期，如果没有过期则解析成功，若没找到对应条目，主控服务器会代替浏览器发起一个迭代的DNS解析的请求，先查找根域的，运营商服务器拿到域名的IP，返回给操作系统的内核，同时在自己的缓存区缓存一份，操作系统内核从DNS服务商拿来的IP地址返回给浏览器。浏览器再向 Web 服务器发送消息时,只要从该内存地址取出 IP地址,将它与 HTTP 请求消息一起交给操作系统.

那么DNS服务器的IP地址又是怎么确定的呢？其实DNS 服务器的IP地址是作为TCP/IP的一个设置项目事先设置好的,不需要再去查询了。

###### DNS服务器收到请求后的处理逻辑

![](./images/dns_server_process.png)

客户端发给DNS服务器的信息包括：

- ****域名****：服务器、邮件服务器(邮件地址中@后面的部分)的名称 。
- ****Class****：用来识别网络的信息（在最早设计 DNS 方案时，DNS 在互联网以外的其他网络中的应用也被考虑到了。不过,如今除了互联网并没有其他的网络了，因此 Class 的值永远是代表互联网的 IN ）。
- ****记录类型**** ：表示域名对应何种类型的记录（例如，A ：表示域名对应的是 IP 地址,MX ：表示域名对应的是邮件服务器）

DNS服务器内部有一张表专门用来存储这些映射关系，那么这张表里面的最初条目是从哪里获取到的呢？这就涉及运营商DNS服务器发起的递归迭代请求。

运营商的服务器里面安装的DNS服务器程序的配置文件中保存了根域服务的IP地址。运营商DNS服务器检查本地对应缓存无数据后，运营商服务从已经配置好的信息中拿到根域名的IP地址，向根DNS服务器发起迭代查询请求。
根DNS服务器告诉运营商DNS服务器该域名已授权给某个顶级域名区域（比如com区）管理，应去对应的顶级DNS服务器查询，并给出了对应顶级DNS服务器的IP地址。运营商DNS先将目标顶级DNS服务器及其IP地址加入到缓存，下次DNS请求，在缓存未过期的情况下，直接往目标顶级DNS服务器发查询请求，不再向根DNS服务器请求。
紧接着运营商服务器根据根DNS服务器响应的结果向某个顶级DNS服务器发起迭代查询请求。目标顶级DNS服务器告诉运营商DNS服务器该域名已授权给某个下一级DNS域名区管理，，并给出下一级DNS域名区管理DNS服务器IP地址。运营商DNS服务器将下一级DNS域名区管理DNS服务器IP地址加入到缓存，下次DNS请求，在缓存未过期的情况下，直接往对应 DNS服务器发查询请求，不再向根DNS服务器及顶级DNS服务器请求。运营商DNS服务器根据下一级DNS域名区管理DNS服务器IP地址向下一级DNS域名区管理DNS服务器发起迭代查询请求。下一级DNS域名区管理DNS服务器返回最终的域名对应的IP地址。运营商DNS服务器将最终的IP地址加入到上面提到的表格上。下次相同的DNS请求，在缓存未过期的情况下，运营商DNS直接给出该域名的IP地址，不再向权威DNS服务器查询，运营商DNS将该域名对应的IP地址返回给客户端。

![](./images/search_ip.png)

  - [DNS域名解析解剖](https://zhuanlan.zhihu.com/p/86504259)
  - [DNS 原理入门](http://www.ruanyifeng.com/blog/2016/06/dns.html)
  - [DNS入门：域名解析流程详解](https://zhuanlan.zhihu.com/p/38499577)
  - [DNS 域名解析系统概述](https://blog.konghy.cn/2019/08/06/dns-overview/)
  - [DNS简单总结](https://github.com/hellorocky/blog/blob/master/network/5.dns%E7%AE%80%E5%8D%95%E6%80%BB%E7%BB%93.md)

#### 推荐文章：

- [当你在浏览器中输入了本站网址并回车后，产生了哪些技术步骤？](https://zhuanlan.zhihu.com/p/96505045)
- [在浏览器输入 URL 回车之后发生了什么](https://wsgzao.github.io/post/url/)
- [经典面试题：从 URL 输入到页面展现到底发生什么？](https://juejin.im/post/5c773dd251882519610194c1)
- [史上最全！图解浏览器的工作原理](https://www.infoq.cn/article/CS9-WZQlNR5h05HHDo1b)
- [图解浏览器的基本工作原理](https://zhuanlan.zhihu.com/p/47407398)
- [浏览器的工作原理](https://github.com/creeperyang/blog/issues/46)
- [深入浅出浏览器渲染原理](https://blog.fundebug.com/2019/01/03/understand-browser-rendering/)


#### 分层网络结构

![](./images/network_layer.jpg)

- ****物理层****
　物理层是参考模型的最低层。物理层的作用是实现相邻计算机节点之间[比特流的透明传送]，尽可能[屏蔽掉具体传输介质和物理设备的差异]。
  关键字：****比特流**** ，****屏蔽掉具体传输介质和物理设备的差异****

- ****数据链路层****
  在计算机网络中由于各种干扰的存在，物理链路是不可靠的,数据链路层的主要功能是：通过各种****差错控制**** 、****流量控制****等控制协议，将有差错的物理信道变为无差错的、能可靠传输数据帧的数据链路。数据链路层的具体工作是接收来自物理层的比特流形式的数据，并封装成帧，传送到上一层；同样，也将来自上层的数据帧，拆装为比特流形式的数据转发到物理层。
  关键字：****帧**** ，****无差错的、能可靠传输数据帧****

- ****网络层****
  网络层将数据链路层的数据在这一层被转换为数据包再通过路由选择算法将数据包从一个网络设备传送到另一个网络设备。一般地，数据链路层是解决同一网络内节点之间的通信，而网络层主要解决不同子网间的通信。例如在广域网之间通信时，必然会遇到路由（即两节点间可能有多条路径）选择问题。
  关键字：****数据包**** ，****不同子网间的通信****

- ****传输层****
  向用户提供可靠的端到端的差错和流量控制，保证报文的正确传输，该层常见的协议：TCP/IP中的TCP协议和UDP协议。传输层提供会话层和网络层之间的传输服务，这种服务从会话层获得数据，并在必要时，对数据进行分割。然后，传输层将数据传递到网络层，并确保数据能正确无误地传送到网络层。
  关键字：****报文**** ，****端到端的差错和流量控制****

- ****会话层****
  会话层主要功能是用来管理网络设备的会话连接，细分为三大功能：
  ****建立会话****：A，B两台网络设备之间要通信，要建立一条会话供他们使用，在建立会话的过程中也会有身份验证，权限鉴定等环节；
  ****保持会话****：通信会话建立后，通信双方开始传递数据，当数据传递完成后，OSI会话层不一定会立刻将两者这条通信会话断开，它会根据应用程序和应用层的设置对该会话进行维护，在会话维持期间两者可以随时使用这条会话传数据；
  ****断开会话****：当应用程序或应用层规定的超时时间到期后，OSI会话层才会释放这条会话。或者A、B重启、关机、手动执行断开连接的操作时，OSI会话层也会将A、B之间的会话断开。

- ****表示层****
　 表示层它用来对来自应用层的命令和数据进行解释，对各种语法赋予相应的含义，并按照一定的格式传送给会话层。其主要功能是处理用户信息的表示问题，如编码、数据格式转换和加密解密等。

- ****应用层****
　应用层的功能是直接向用户提供某项服务，完成用户希望在网络上完成的各种工作。

![](./images/OSI.png)
![](./images/OSI_Layer.png)
![](./images/layer_comm.png)

下面是YouTobe上找到的一个比较好的视频[『初探資訊』OSI 網路通訊架構](https://www.youtube.com/watch?v=kyARqyCjoYY)

##### OSI七层网络模型与TCP/IP 四层网络结构对应关系

![](./images/7_layer_4_layer.png)
![](./images/OSI-TCP-vs.png)


#### 传输包的封装解封装

![](./images/TCP-IP-package.png)


#### 数据包的传输

![](./images/how-data-is-processed-in-OSI-and-TCPIP-models.png)
![](./images/Internet_package.png)


#### 传输层协议 - 如何在两个应用直接可靠传输

#### TCP协议的特点

- ****面向连接****:
  在服务端和客户端进行数据通信之前必须在两端通过三次握手建立连接。为数据的可靠传输打下基础。TCP的三次握手和四次挥手会在下面进行介绍：

- ****可靠传输****：
  对于可靠传输，判断丢包，误码靠的是TCP的段编号以及确认号。TCP为了保证报文传输的可靠，就给每个包一个序号，同时序号也保证了传送到接收端实体的包的按序接收。然后接收端实体对已成功收到的字节发回一个相应的确认(ACK)；如果发送端实体在合理的往返时延(RTT)内未收到确认，那么对应的数据将会被重传。

- ****面向字节流****：
  TCP不像UDP一样那样一个个报文独立地传输，而是在不保留报文边界的情况下以字节流方式进行传输。

- ****仅支持单播传输****：
  每条TCP传输连接只能有两个端点，只能进行点对点的数据传输，不支持多播和广播传输方式。
  
- ****提供拥塞控制****：
  当网络出现拥塞的时候，TCP能够减小向网络注入数据的速率和数量，缓解拥塞

#### TCP帧格式

不携带选项(option)的TCP头长为20bytes，携带选项的TCP头最长可到60bytes。

![](./images/tcp_package_format.png)

****TCP各字段意义****:

- ****源端口(Source Port)****：16位的源端口表示发送方应用程序对应的端口。
- ****目的端口(Destination port)****：16位的目的端口域用来指明报文接收计算机上的应用程序地址接口。
- ****序列号（SequenceNumber）****：32位的序列号标识了TCP报文中第一个byte在对应方向的传输中对应的字节序号。通过序列号，TCP接收端可以识别出重复接收到的TCP包，从而丢弃重复包，同时对于乱序数据包也可以依靠系列号进行重排序，进而对高层提供有序的数据流。简而言之就是用于去重和排序。
- ****应答号(Acknowledgment Number)****：32位的ACK Number标识了报文发送端期望接收的字节序列。如果设置了ACK控制位，这个值表示一个准备接收的包的序列码，注意是准备接收的包，比如当前接收端接收到一个净荷为12byte的数据包，SN为5，则会回复一个确认收到的数据包，如果这个数据包之前的数据也都已经收到了，这个数据包中的ACK Number则设置为12+5=17，表示之前的数据都已经收到了，准备接受SN=17的数据包。
- ****头长(Header Length)****：4位包括TCP头大小，指示TCP头的长度，即数据从何处开始。
- ****保留(Reserved)位****：4位值域，这些位必须是0。
- ****标志(Code Bits)位****：8位标志位
  - CWR(Congestion Window Reduce)：拥塞窗口减少标志，用来表明它接收到了设置ECE标志的TCP包。并且发送端在收到消息之后已经通过降低发送窗口的大小来降低发送速率。
  - ECE(ECN Echo)：ECN响应标志被用来在TCP3次握手时表明一个TCP端是具备ECN功能的。在数据传输过程中也用来表明接收到的TCP包的IP头部的ECN被设置为11。
  - URG(Urgent)：该标志位置位表示紧急(The urgent pointer) 标志有效。
  - ACK：取值1代表Acknowledgment Number字段有效，这是一个确认的TCP包，取值0则不是确认包。
  - PSH(Push)：该标志置位时，一般是表示发送端缓存中已经没有待发送的数据，接收端不将该数据进行队列处理，而是尽可能快将数据转由应用处理。
  - RST(Reset)：用于reset相应的TCP连接。通常在发生异常或者错误的时候会触发复位TCP连接。
  - SYN：同步序列编号(Synchronize Sequence Numbers)有效。该标志仅在三次握手建立TCP连接时有效。
  - FIN(Finish)：表示发送方没有数据要传输了。
- ****窗口大小(Window Size)****：16位，该值指示了从Ack Number开始还愿意接收多少byte的数据量，也即用来表示当前接收端的接收窗还有多少剩余空间，用于TCP的流量控制。
- ****校验位(Checksum)****：16位TCP头。发送端基于数据内容计算一个数值，接收端要与发送端数值结果完全一样，才能证明数据的有效性。接收端checksum校验失败的时候会直接丢掉这个数据包。CheckSum是根据伪头+TCP头+TCP数据三部分进行计算的。
- ****紧急数据指针（Urgent Pointer）****：16位，指向后面是优先数据的字节，在URG标志设置了时才有效。如果URG标志没有被设置，紧急域作为填充。
- ****选项(Option)****：长度不定，但长度必须以是32bits的整数倍。常见的选项包括MSS、SACK、Timestamp等等。


#### UDP特点

- ****面向无连接****：
  不需要三次握手建立连接，想发数据随时开始，

- ****不可靠传输****：
  收到什么数据就传递什么数据，它只是数据报文的搬运工，不会对数据报文进行任何拆分和拼接操作。在发送端，应用层将数据传递给传输层的UDP协议，UDP只会给数据增加一个UDP头标识下这是UDP协议，然后就传递给网络层了，在接收端，网络层将数据传递给传输层，UDP只去除IP报文头就传递给应用层，不会任何拼接操作。发送数据也不会关心对方是否已经正确接收到数据了，网络环境时好时坏的情况下，UDP因为没有拥塞控制，一直会以恒定的速度发送数据。即使网络条件不好，也不会对发送速率进行调整。这样实现的弊端就是在网络条件不好的情况下可能会导致丢包。

- ****头部开销小，传输报文数据高效实时****
  UDP的头部开销小，只有8字节，相比TCP的至少二十字节要少得多，在传输数据报文时是很高效的

- ****UDP是面向报文的****

- ****支持单播，多播，广播的功能****

#### UDP帧格式

UDP报文头长度是8个字节

![](./images/udp_frame.png)

****UDP各字段意义****:

- ****源端口****：这个字段表示发送数据报的应用程序所使用的 UDP 端口。接收端的应用程序利用这个字段的值作为发送响应的目的地址。这个字段是可选的，所以发送端的应用程序不一定会把自己的端口号写入该字段中。如果不写入端口号，则把这个字段设置为 0。这样，接收端的应用程序就不能发送响应了。
- ****目的端口****：接收端计算机上 UDP 数据使用的端口，占据 16 位。
- ****长度****：该字段占据 16 位，表示 UDP 数据报长度，包含UDP报文头和UDP数据长度。
- ****校验值****：该字段占据 16 位，可以检验数据在传输过程中是否被损坏。

#### TCP 和 UDP帧的区别

![](./images/TCP_vs_UDP.jpeg)
![](./images/tcp-vs-udp.svg)
![](./images/tcp_vs_udp.png)

#### TCP中的一些关键技术

TCP 协议的主要目标就是保证数据的可靠传输，避免数据的破坏、丢包、重复以及分片顺序混乱等现象出现。让用户在上层看来是一个连续的数据流。为了达到这个目的，TCP 采用了三次握手四次挥手，窗口控制，流量控制，拥塞控制，重发机制等技术。这部分就来重点介绍下这部分技术。

#### 1. 连接控制 - TCP三次握手和四次挥手

##### 三次握手

三次握手是指建立一个TCP连接时，需要客户端和服务器之间总共发送3个包，它的作用就是在进行数据传输之前让双方都能明确自己和对方的收、发能力是正常的。实质上其实就是连接服务器指定端口，建立TCP连接，并同步连接双方的序列号和确认号，交换TCP窗口大小信息。

![](./images/3_time_shake.jpeg)


刚开始客户端处于 Closed 的状态，服务端处于 Listen 状态。

- 第一次握手：建立连接。客户端发送连接请求报文段，将SYN位置为1，Sequence Number为x；然后，客户端进入****SYN_SEND****状态，等待服务器的确认,服务器收到SYN报文段。
  这个阶段客户端发送网络包，服务端收到了。这样服务端就能得出结论：客户端的发送能力、服务端的接收能力是正常的。而客户端由于这个阶段服务端没有返回数据所以无法知道它的信息是否发送成功，服务端接收能力是否正常。

- 第二次握手：服务器收到客户端的SYN报文段，需要对这个SYN报文段进行确认，设置Acknowledgment Number为x+1(Sequence Number+1)；同时，自己还要发送SYN请求信息，将SYN位置为1，Sequence Number为y；服务器端将上述所有信息放到一个报文段（即SYN+ACK报文段）中，一并发送给客户端，此时服务器进入SYN_RECV状态，客户端收到该回复信息。
  这个阶段：服务端发包，客户端收到了。这样客户端就能得出结论：服务端的接收、发送能力，客户端的接收、发送能力是正常的。服务端目前只知道接收能力正常。
  
- 第三次握手：客户端收到服务器的SYN+ACK报文段。然后将Acknowledgment Number设置为y+1，向服务器发送ACK报文段，
  这个阶段：服务端能得出结论：客户端的接收、发送能力，服务端的发送、接收能力是正常的。这个报文段发送完毕以后，客户端和服务器端都进入ESTABLISHED状态，完成TCP三次握手。

经历了上面的三次握手过程，客户端和服务端都确认了自己的接收、发送能力是正常的。之后就可以正常通信了。

![](./images/3_time_shake_1.jpeg)

****两次握手会有什么问题：****

如客户端发出连接请求，但因连接请求报文丢失而未收到确认，于是客户端再重传一次连接请求。后来收到了确认，建立了连接。数据传输完毕后，就释放了连接，客户端共发出了两个连接请求报文段，其中第一个丢失，第二个到达了服务端，但是第一个丢失的报文段只是在某些网络结点长时间滞留了，延误到连接释放以后的某个时间才到达服务端，此时服务端误认为客户端又发出一次新的连接请求，于是就向客户端发出确认报文段，同意建立连接，不采用三次握手，只要服务端发出确认，就建立新的连接了，此时客户端忽略服务端发来的确认，也不发送数据，则服务端一致等待客户端发送数据，浪费资源。

****半连接队列****
服务器第一次收到客户端的 SYN 之后，就会处于 SYN_RCVD 状态，服务器会把此种状态下请求连接放在一个队列里，我们把这种队列称之为半连接队列。服务器发送完SYN-ACK包，如果未收到客户确认包，服务器进行首次重传，等待一段时间仍未收到客户确认包，进行第二次重传。如果重传次数超过系统规定的最大重传次数，系统将该连接信息从半连接队列中删除。每次重传等待的时间不一定相同，一般会是指数增长，例如间隔时间为 1s，2s，4s，8s……。

****建立连接时候的初始序列号是固定的吗****

ISN随时间而变化，因此每个连接都将具有不同的ISN。ISN可以看作是一个32比特的计数器，每4ms加1 。三次握手的其中一个重要功能是客户端和服务端交换 ISN，以便让对方知道接下来接收数据的时候如何按序列号组装数据。如果 ISN 是固定的，攻击者很容易猜出后续的确认号，从而发起网络攻击。

****三次握手过程中可以携带数据吗****

其实第三次握手的时候，是可以携带数据的。但是，第一次、第二次握手不可以携带数据。首先我们必须明确一点，服务端的存在是为了替客户端处理数据，所以最终的数据还是要返回给客户端的，如果还没法确认客户端的收发能力，服务端接受数据是没有意义的，我们回过头从服务端的角度看下三次握手的情况：第一次服务端只能知道客户端有发送数据的能力，第二次服务端依旧只知道客户端发送数据的能力，并不知道客户端接收数据的能力。所以这种情况下接收数据是没有意义的。

****什么是SYN攻击****

SYN攻击就是客户端在短时间内伪造大量不存在的IP地址，并向服务端断地发送SYN包，服务端则回复确认包，并等待客户端确认，由于源地址不存在，因此服务端需要不断重发直至超时，这些伪造的SYN包将长时间占用未连接队列，导致正常的SYN请求因为队列满而被丢弃，从而引起网络拥塞甚至系统瘫痪。这也是为什么当第三次握手失败时，服务器并不会重传ack报文，而是直接发送RST报文段，进入CLOSED状态。这样做的目的是为了防止SYN攻击。

##### 四次挥手

![](./images/4_time_wave.jpeg)

建立一个连接需要三次握手，而终止一个连接要经过四次挥手。这是由TCP的半关闭造成的。所谓的半关闭，其实就是TCP提供了连接的一端在结束它的发送后还能接收来自另一端数据的能力。也就是说这里要完成的任务是保证两边在关闭前都确保无数据发送给对方的诉求，并且已经完成了各自的收尾工作。

![](./images/4_time_wave_1.jpeg)

- ****第一次挥手****：客户端(可以是客户端，也可以是服务器端,这里以客户端先发起终止为例子) 发送一个 FIN 报文，报文中会指定一个序列号。此时客户端处于 FIN_WAIT1 状态。即发出连接释放报文段（FIN=1，序号seq=u），并停止再发送数据，主动关闭TCP连接，进入FIN_WAIT1（终止等待1）状态，等待服务端的确认。这表示客户端没有数据要发送给服务端了。
- ****第二次挥手****：服务端收到 FIN 之后，会发送 ACK 报文，且把客户端的序列号值 +1 作为 ACK 报文的序列号值，表明已经收到客户端的报文了，此时服务端处于 CLOSE_WAIT 状态。即服务端收到连接释放报文段后即发出确认报文段（ACK=1，确认号ack=u+1，序号seq=v），服务端进入CLOSE_WAIT（关闭等待）状态，此时的TCP处于半关闭状态，客户端到服务端的连接释放。客户端收到服务端的确认后，进入FIN_WAIT2（终止等待2）状态，等待服务端发出的连接释放报文段。这时候表示服务端告诉客户端：我同意你的关闭请求.
- ****第三次挥手****：如果服务端也想断开连接了，和客户端的第一次挥手一样，发给 FIN 报文，且指定一个序列号。此时服务端处于 LAST_ACK 的状态。
即服务端没有要向客户端发出的数据，服务端发出连接释放报文段（FIN=1，ACK=1，序号seq=w，确认号ack=u+1），服务端进入LAST_ACK（最后确认）状态，等待客户端的确认。这表示服务端没有数据要发送给客户端了。
- ****第四次挥手****：客户端收到 FIN 之后，一样发送一个 ACK 报文作为应答，且把服务端的序列号值 +1 作为自己 ACK 报文的序列号值，此时客户端处于 TIME_WAIT 状态。需要过一阵子以确保服务端收到自己的 ACK 报文之后才会进入 CLOSED 状态，服务端收到 ACK 报文之后，就处于关闭连接了，处于 CLOSED 状态。
即客户端收到服务端的连接释放报文段后，对此发出确认报文段（ACK=1，seq=u+1，ack=w+1），客户端进入TIME_WAIT（时间等待）状态。此时TCP未释放掉，需要经过时间等待计时器设置的时间2MSL后，客户端才进入CLOSED状态。这时候客户端的行为是：我收到了你的请求我等你2MSL后关闭，服务端收到这个消息后就关闭了。

所以整个过程可以通过如下对话来总结：

客户端：我没有数据要传的了
服务端：收到，我同意你那边的关闭请求,我这边的数据还没处理完所以现在不能直接关闭，你先等下我处理完成后会告诉你。
..... 等待中 ......
服务端：收尾工作已经处理完毕，我这边已经没有数据需要传递给你了。
客户端：收到，我这边会等2MSL后关闭。（服务端收到该消息后就立刻关闭，客户端等待2MSL后关闭）

对于这个地方有个比较困惑的问题：

- [为什么TCP4次挥手时等待为2MSL](https://www.zhihu.com/question/67013338)

2MSL等待的结果是在这个期间，这个连接两端的插口（客户的IP地址和端口号，服务器的IP地址和端口号）不能再被使用。这个连接只能在2MSL结束后才能再被使用。

那么为什么要等待2MSL 首先我们先要弄明白什么是MSL，MSL是Maximum Segment Lifetime的英文缩写，可译为“最长报文段寿命”，它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。而这里的2MSL = 去向ACK消息最大存活时间（MSL) + 来向FIN消息的最大存活时间(MSL) 。

客户端第四次握手发送ACK从客户端到服务端最多经过1MSL，超过这个时间服务端会重发FIN。服务端重发的FIN最多经过1MSL到达客户端。等待2MSL就保证了，如果由于某些原因导致客户端发出的ACK不能到达服务端，这时候服务端重发的FIN报文一定能够被客户端收到。

![](./images/3_shake_4_wave.png)

[面试官，不要再问我三次握手和四次挥手](https://yuanrengu.com/2020/77eef79f.html)


#### 2. 窗口控制

- ****为什么需要窗口****
  我们知道TCP的每个数据包都应该有ACK应答,如下图所示,这其实是一种串行模式，虽然可以保证数据的可靠传输，但是资源并没被合理利用：
  ![](./images/send_ack.png)
  所以我们需要思考发送端必须是前一个数据包收到应答后才能发送后一个数据包吗。其实不是的，我们可以通过在发送方和接收方增加缓存的方式来解决这个问题，发送方将数据发送出去后在等到确认应答返回之前，将已发送的数据保留在缓存区中。如果按期收到确认应答，此时数据就可以从缓存区清除。这个缓存区域就是所谓的窗口。窗口大小就是指无需等待确认应答，就可以继续发送数据的最大值。
  ![](./images/send_window.png)
  
  使用窗口的另一个优点就是****累计应答****,如上图所示ACK 600 确认应答报文丢失，也没关系，因为可以通过下一个确认应答进行确认，只要发送方收到了 ACK 700 确认应答，就意味着 700 之前的所有数据都收到了。
  除此之外窗口还有一个用途是用于对传输层进行流控的一种措施，接收方通过通告发送方自己的窗口大小，从而控制发送方的发送速度，从而达到防止发送方发送速度过快而导致自己被淹没的目的。总的来说窗口的作用一是通过缓存将原先的串行形式转换为并行模式，二是作为流控手段调节发送方的发送速度。

- ****发送窗口和接收窗口的结构****
  
  我们来进一步看下发送方和接收方的窗口结构：

  ![](./images/send_window_size.png)
  整个缓存区域可以分成四个部分：
  * 已发送并收到ACK确认信号的请求
  * 已发送但并未收到ACK确认信号的请求
  * 未发送但在接收方处理能力范围之内的请求空位
  * 未发送但在接收方处理能力范围之外的请求空位
  
  TCP协议中用了三个字段来标示这四个区间：
  * ****SND.WND****：表示发送窗口的大小，大小由接收方指定的；
  * ****SND.UNA****：是一个绝对指针，它指向的是已发送但未收到确认的第一个字节的序列号，也就是 #2 的第一个字节。
  * ****SND.NXT****：也是一个绝对指针，它指向未发送但可发送范围的第一个字节的序列号，也就是 #3 的第一个字节。
  #4 的第一个字节是个相对指针，它需要 SND.NXT 指针加上 SND.WND 大小的偏移量，就可以指向 #4 的第一个字节了。

  可用窗口大小就等于SND.WND -（SND.NXT – SND.UNA）：
  我们前面提到了TCP 的 Window是一个16bit位字段，它代表的是窗口的字节容量，也就是TCP的标准窗口最大为2^16-1=65535个字节。另外在TCP的选项字段中还包含了一个TCP窗口扩大因子，option-kind为3，option-length为3个字节，option-data取值范围0-14。窗口扩大因子用来扩大TCP窗口，可把原来16bit的窗口，扩大为31bit。

  ![](./images/receive_window.png)

  接收窗口相对简单一些它包含三个部分：
  - #1 + #2 是已成功接收并确认的数据（等待应用进程读取）；
  - #3 是未收到数据但可以接收的数据；
  - #4 未收到数据并不可以接收的数据；
  
  三个接收部分，使用两个指针进行划分:
  - RCV.WND：表示接收窗口的大小，它会通告给发送方。
  - RCV.NXT：是一个指针，它指向期望从发送方发送来的下一个数据字节的序列号，也就是 #3 的第一个字节。

  接下来我们以动态的视角看下整个窗口滑动的效果：
  ![](./images/slice_window.png)
  ![](./images/slice.jpg)

- ****发送窗口与接收窗口关系****
  
  TCP是双工的协议，会话的双方都可以同时接收、发送数据。TCP会话的双方都各自拥有一个“发送窗口”和一个“接收窗口”。各自的“发送窗口”则要求取决于对端通告的“接收窗口”。
  ![](./images/snd_receiv_window_size.jpg)

#### 3. 流量控制

  前面介绍了窗口控制的概念的时候提到了窗口其中的一个用途是用于对传输层进行流量控制，接收方通知发送方自己的窗口大小，发送方根据收到的接收方发过来的窗口大小，控制请求的发送速度，从而达到防止发送方发送速度过快而导致自己被淹没的目的，接受方一旦被发送端发送过来的数据淹没就会触发发送方重发机制，从而导致网络流量的无端的浪费。为了解决这种现象发生，TCP 提供一种机制可以让发送方根据接收方的实际接收能力控制发送的数据量，这就是所谓的流量控制。
  随着数据的传输，窗口会发生移动，有三种运动方式：
  - ****关闭****：窗口左边界右移。发送数据得到 ACK 时窗口减小。
  - ****打开****：窗口右边界右移。当接收方可用缓存增大时，窗口增大。
  - ****收缩****：窗口右边界左移。一般来说不应发生，但必须能够处理此问题。
  不存在左边界左移，因为窗口左边是已确认的数据，不可能会再次发送。当 ACK 号不断增大，窗口大小保持不变时，则称窗口向前滑动。 

##### 流量控制过程

  ![](./images/liuliang_1.png)

  从上图中可以很明显看出整个流量控制的过程，每次发送方在发送完数据还没收到回复的时候窗口会缩小，一旦收到回复的话便会增大窗口。上面是窗口大小不变的理想环境下的情况，但是实际上窗口对应的缓冲区都是放在操作系统内存缓冲区中的，会被操作系统调整。也就是说在系统资源紧张的时候有可能会收回一些内存缓冲区域，就会相应减少窗口区域。如果在未将窗口大小通知给对方之前，系统回收了对应作为窗口的缓冲区域，就会导致对方没有感知到窗口的缩小而继续发送大于接收方实际窗口的数据，从而导致数据的丢失。为了防止这种情况发生，TCP 规定是不允许同时减少缓存又收缩窗口的，而是采用先收缩窗口，过段时间再减少缓存，这样就可以避免了丢包情况。

##### 窗口关闭带来的死锁现象以及解决方案

窗口关闭带来的死锁现象,如下图所示：
![](./images/close_window_01.png)
它指的是当发生窗口关闭时，接收方处理完数据后，会向发送方通告一个窗口非 0 的 ACK 报文，如果这个通告窗口的 ACK 报文在网络中丢失了，由于发送方没感知到接收方窗口腾出空的信号，所以发送方不会向接收方发送数据，同时接收方也一直在等待发送方发数据，所以两边都陷入了僵持阶段，造成死锁。为了解决这个问题，可以引入窗口探测报文：
![](./images/close_window_02.png)

TCP 为每个连接设有一个持续定时器，只要TCP连接一方收到对方的零窗口通知，就启动持续计时器。如果持续计时器超时，就会发送窗口探测报文，而对方在确认这个探测报文时，给出自己现在的接收窗口大小。如果接收窗口仍然为 0，那么收到这个报文的一方就会重新启动持续计时器，如果接收窗口不是 0，那么死锁的局面就可以被打破了。窗口探查探测的次数一般为3次，每次大约 30-60 秒。如果 3 次过后接收窗口还是 0 的话，有的 TCP 实现就会发 RST 报文来中断连接。

##### 糊涂窗口与粘包拆包

如果接收方太忙了，来不及取走接收窗口里的数据，那么就会导致发送方的发送窗口越来越小。到最后，如果接收方腾出几个字节并告诉发送方现在有几个字节的窗口，发送方也会义无反顾地发送这几个字节，这就是糊涂窗口综合症。

所以糊涂窗口综合症的原因是由发送方和接收方共同造成的：
接收方可以通告一个小的窗口，发送方对应发送小数据。从而造成资源的浪费。

要解决糊涂窗口综合症，就解决上面两个问题就可以了

- 让接收方不通告小窗口给发送方
  当窗口大小Min(MSS，缓存空间/2) 向发送方通告窗口为 0，阻止发送方再发数据过来。当接收方处理了一些数据后，窗口大小 >= MSS，或者接收方缓存空间有一半可以使用，就可以把窗口打开让发送方发送数据过来。

- 让发送方避免发送小数据
  等到窗口大小 >= MSS 或是 数据大小 >= MSS 或者收到之前发送数据的 ack 回包的时候开始发送数据。同时为了满足将多个小数据包合成一个大数据包发送，就涉及到了粘包处理。

##### 什么是分包与粘包

![](./images/package_unpackage.gif)
![](./images/package_unpackage.jpg)

TCP是个字节流协议，是没有界限的一串数据。TCP底层并不了解上层业务数据的具体含义只保证数据可靠得从一个端传输到另一个端，并且应用层数据在送到TCP层到时候发送端为了将多个包，更有效的发到对方，使用了优化方法（Nagle算法）将多次间隔较小、数据量小的数据，合并成一个大的数据块，然后进行封包。同时由于套接字缓冲区，或者MSS(Max Segment Size),以太网帧的负载 MTU（Maximum Transmission Unit）的限制对于发送较大块数据的时候会先讲数据进行分割后发出。所以在业务上认为，一个完整的包可能会被TCP拆分成多个包进行发送，也有可能把多个小的包封装成一个大的数据包发送，这就是所谓的TCP粘包和拆包问题。粘包问题只会在流传输中出现，UDP不会出现粘包，因为它有消息边界。

粘包情况有两种，一种是粘在一起的包都是完整的数据包，另一种情况是粘在一起的包有不完整的包。不是所有的粘包现象都需要处理，若传输的数据为不带结构的连续流数据，则不必把粘连的包分开。

##### 造成粘包的原因

出现粘包现象的原因是多方面的，它既可能由发送方造成，也可能由接收方造成：

正如上面提到的发送方引起的粘包是由TCP协议本身造成的，TCP为提高传输效率，发送方往往要收集到足够多的数据后才发送一包数据。若连续几次发送的数据都很少，通常TCP会根据优化算法把这些数据合成一包后一次发送出去，这样接收方就收到了粘包数据。而接收方引起的粘包是由于接收方用户进程不及时接收数据，从而导致粘包现象。这是因为接收方先把收到的数据放在系统接收缓冲区，用户进程从该缓冲区取数据，若下一包数据到达时前一包数据尚未被用户进程取走，则下一包数据放到系统接收缓冲区时就接到前一包数据之后，而用户进程根据预先设定的缓冲区大小从系统接收缓冲区取数据，这样就一次取到了多包数据。

##### 如何解决粘包问题

上面我们提到了UDP是没有半包粘包的问题是因为它有边界协议，消息是有格式的。所以为解决粘包问题我们也可以在应用层上通过类似的解决方案，解决半包粘包的问题其实就是定义消息边界的问题。

- ****长度边界****
  应用层在发送消息的时候指定每个消息的固定长度，比如固定每个消息为1K，那么当发送时消息不满1K时，用固定的字符串填充，当接收方读取消息的时候，每次也截取1k长度的流作为一个消息来解析。这种方式的问题在于应用层不能发送超过1K大小的数据，所以使用这种方式的前提知道了消息大小会在哪个范围之内，如果不能确定消息的大小范围不太适合用这种方式，这样会导致大的消息发出去会有问题，小的消息又需要大量的数据填充，不划算。

- ****符号边界****
  应用层在发送消息前和发送消息后标记一个特殊的标记符，比如 &符号，当接收方读取消息时，根据&符号的流码来截取消息的开始和结尾。这种方式的问题在于发送的消息内容里面本身就包含用于切分消息的特殊符号，所以在定义消息切分符时候尽量用特殊的符号组合。
  
- ****组合边界****
  这种方式先是定义一个Header+Body格式，Header消息头里面定义了一个开始标记+一个内容的长度，这个内容长度就是Body的实际长度，Body里面是消息内容，当接收方接收到数据流时，先根据消息头里的特殊标记来区分消息的开始，获取到消息头里面的内容长度描述时，再根据内容长度描述来截取Body部分。

- ****利用TCP内置的机制****
  TCP提供了强制数据立即传送的操作指令push，TCP软件收到该操作指令后，就立即将本段数据发送出去，而不必等待发送缓冲区满

- ****接收方新增加预处理线程****
  接收方创建一个预处理线程，对接收到的数据包进行预处理，将粘连的包分开。


#### 4. 拥塞控制

##### 拥塞控制和流量控制的区别

前面介绍了流量控制，接下来在介绍拥塞控制之前对二者进行简单的区分：

****流量控制****是作用于接收者的，目标是调整两个连接的网络状况，接收方通知发送方自己的窗口大小，发送方根据收到的接收方发过来的窗口大小，控制请求的发送速度，从而达到防止发送方发送速度过快而导致接收方被淹没的目的。

而****拥塞控制****是作用于发送方，目标是调整整个网络的网络环境，我们知道计算机网络都处在一个共享的环境，任何一台机器都不能长期占用大量的网络资源，每个网络的结点都有义务做到防止过多的数据注入到网络中，避免出现网络负载过大的情况。在网络出现拥堵时，如果继续发送大量数据包，可能会导致数据包时延、丢失等，这时 TCP 就会重传数据，但是一重传就会导致网络的负担更重，于是会导致更大的延迟以及更多的丢包，这个情况就会进入恶性循环被不断放大，甚至导致整个网络瘫痪。

##### 拥塞窗口

拥塞窗口cwnd是发送方维护的一个的状态变量，它会根据网络的拥塞程度动态变化的。只要网络中没有出现拥塞，cwnd 就会增大，一旦网络中出现了拥塞cwnd就减小。判断是否网络拥塞的方法也很简单，只要发送方没有在规定时间内接收到ACK应答报文，也就是发生了超时重传，就会认为网络出现了拥塞。由于拥塞控制的存在发送方的最终窗口大小取决于拥塞窗口和流量控制窗口的最小值。

##### 拥塞控制算法

拥塞控制主要包含三个阶段四个算法：

- 慢启动阶段

慢启动阶段在包个数到达慢启动门限（ssthresh = 65535 字节）之前包的个数是指数性的增长。
- 当 cwnd < ssthresh 时，使用慢启动算法。
- 当 cwnd >= ssthresh 时，就会使用拥塞避免算法

整个过程如下图所示：

![](./images/slow_grow_01.png)


- 拥塞避免阶段

进入拥塞避免算法后，每当收到一个 ACK 时，cwnd 增加 1/cwnd。所以，拥塞避免算法就是将原本慢启动算法的指数增长变成了线性增长，还是增长阶段，但是增长速度缓慢了一些。随着cwnd的不断增加，网络就会慢慢进入了拥塞的状况了，于是就会出现丢包现象，这时就需要对丢失的数据包进行重传。这时候就会进入拥塞发生阶段。
![](./images/stuck_void02.png)


- 拥塞发生阶段

拥塞发生阶段也就是所处的网络出现拥塞导致的丢包现象，这时候需要对数据包重传，重传机制主要有两种：

- 超时重传
- 快速恢复

- 超时重传

当****发生网络超时****的时候会触发超时重传，它会将启动门限ssthresh设置为cwnd/2，并将cwnd重置为1。接着，就重新开始慢启动，慢启动是会突然减少数据流。有可能会造成网络卡顿。
![](./images/net_work_reset.png)

- 快速恢复

当接收方发现****丢了一个中间包****的时候，发送三次前一个包的 ACK，于是发送端就会快速地重传，不必等待超时再重传。TCP 认为对方还能收到3个重复ACK说明网络也不那么糟糕，所以快速恢复cwnd和ssthresh进行如下更新：

```
cwnd = cwnd/2 
ssthresh = cwnd;
```
紧接着进入快速恢复算法如下：

拥塞窗口 cwnd = ssthresh + 3
重传丢失的数据包，接着进入了拥塞避免算法。

- ![](./images/quick_start.png)


![](./images/busy_net_control_01.png)
![](./images/busy_net_control_02.png)
![](./images/busy_net_control_03.png)

#### 5. 重发机制

前面的流量控制和拥塞控制的主要是用于缓解由于网络的原因导致的数据丢失，但是并不能根本上解决网络丢失的问题，TCP协议要保证所有的数据包都要到达，必需要借助重传确认应答机制。

在传输过程中每发送一个片段，就会开启一个重传倒计时定时器，并将该片段的一份复制放置于重传队列，重传队列按照重传计时器的剩余时间来排序，如果在计时器超时之前收到确认应答，则将该片段从重传队列中移除。

如果在超时之前没有收到确认信号，则自动重传信号。重传之后该片段还是保留在重传队列中，重传计时器会被重启，这样不断重复一定次数，如果还没有成功则判定为传输失败。

那我们如何知道是否确认对方是否有收到对应的片段？这就要靠TCP的确认机制：接收方在收到每个数据包之后都会回复一个ACK确认号，ACK不能跳着确认，只能确认最大的连续收到的包，也就是说ACK确认号表明所有低于该编号的sequence number已经被发送该编号的设备接收。所以每次收到一个ACK确认号后就可以将重传队列中小于该确认号的片段移除。

从接收方角度来看，如果前一个片段A丢失了，但是后一个片段B收到了，那么接收方将片段B放在接收缓存中不需要进行确认,而是要等到A收到之后回复。这就是所谓的累积确认，累积确认机制也有比较明显的缺点。假设服务器被告知它有一个10000字节窗口，20个片段每个片段500字节。第一个片段丢失了，其他19个被接收到了。但是由于第一个片段从没有接收到，其他19个也无法确认。

为了解决上面提到的其他19个也无法确认的问题，就涉及到处理未确认片段的策略：

有四种主要的策略：

- ****仅重传超时片段****：
仅重传超时的片段，希望其他片段都能够成功接收。如果该片段之后的其他片段实际上接收到了，这一方式是最佳的。

- ****重传所有片段****：
无论何时一个片段超时了，不仅重传该片段，还有所有其他尚未确认的片段。这一方式确保了任何时间都有一个等待确认的停顿时间，在所有未确认片段丢失的情况下，会刷新全部未确认片段，以使对端设备多一次接收机会。这一过程完成之后，任一设备都可以在常规TCP片段中使用SACK选项。这一选项包含一个关于已接收但未确认片段数据sequence number范围的列表。

上面两种方式也就是前面提到的超时重传，它有好也有不好，第一种会节省带宽，但是慢，第二种会快一点，但是会浪费带宽，也可能会有无用功。但总体来说都不好。因为都在等timeout。

- ****快速重传机制****：

为了解决超时重传的缺点，TCP引入了快速重传的机制，快速重传机制不以时间驱动，而以数据驱动重传。也就是说，如果包没有连续到达，就ACK最后那个可能被丢了的包，如果发送方连续收到3次相同的ACK，就重传。快速重传机制的好处是不用等超时了再重传。

![](./images/FASTIncast.png)

比如上面例子：如果发送方发出了1，2，3，4，5份数据，第一份先到送了，于是就ACK回2，结果2因为某些原因没收到，3到达了，于是还是ACK回2，后面的4和5都到了，但是还是ACK回2，因为2还是没有收到，于是发送端收到了三个ACK=2的确认，知道了2还没有到，于是就马上重转2。然后，接收端收到了2，此时因为3，4，5都收到了，于是ACK回6。

Fast Retransmit只解决了一个问题，就是timeout的问题，它依然面临一个艰难的选择，就是，是重传之前的一个还是重传所有的问题。对于上面的示例来说，是重传#2呢还是重传#2，#3，#4，#5呢？因为发送端并不清楚这连续的3个ack(2)是谁传回来的？也许发送端发了20份数据，是#6，#10，#20传来的呢。这样，发送端很有可能要重传从2到20的这堆数据

- ****选择性确认 SACK 方法****：

前面的问题的关键在于无法确认非连续片段。要解决这个问题必须添加允许设备分别确认非连续片段的功能。这一功能称为选择确认（selective acknowledgment, SACK）。
如果要使用这个功能必须连接的两方设备同时支持这一功能，这个确定过程在连接的时候通过SYN片段来协商是否允许使用SACK。如果该片段已被选择确认过，则该片段中的SACK比特位置为1。该设备使用图2中激进方式的改进版本，一个片段重传之后，之后所有SACK比特位非1的片段都会被重传。

![](./images/tcp_sack_example-1024x577.jpg)

Duplicate SACK – 重复收到数据的问题

Duplicate SACK又称D-SACK，其主要使用了SACK来告诉发送方有哪些数据被重复接收了
D-SACK使用了SACK的第一个段来做标志，

```
如果SACK的第一个段的范围被ACK所覆盖，那么就是D-SACK
如果SACK的第一个段的范围被SACK的第二个段覆盖，那么就是D-SACK
```

- ****推荐文章****
  - [TCP协议的滑动窗口具体是怎样控制流量的？](https://www.zhihu.com/question/32255109)
  - [你还在为 TCP 重传、滑动窗口、流量控制、拥塞控制发愁吗？看完图解就不愁了](https://www.javashitang.com/?p=577)
  - [TCP 的那些事儿（上）](https://coolshell.cn/articles/11564.html)
  - [TCP 的那些事儿（下）](https://coolshell.cn/articles/11609.html)
  - [这一次,让我们再深入一点 - TCP协议](https://juejin.im/post/5a49d95af265da430a50ed8c)
  - [TCP 进阶](https://juejin.im/post/5a49d95af265da430a50ed8c)
  - [TCP协议 读后感](https://www.todayios.com/impression-of-tcp-in-geekbang/)
  - [一篇文章带你熟悉 TCP/IP 协议（网络协议篇二）](https://juejin.im/post/5a069b6d51882509e5432656)
  - [一篇文章带你详解 HTTP 协议（网络协议篇一）](https://www.jianshu.com/p/6e9e4156ece3)
  - [网络编程懒人入门](https://yq.aliyun.com/articles/670062?spm=a2c4e.11153940.0.0.72e87584UZ2aDo)
  - 
#### IP协议 - 如何找到最合适的路径将数据送到

要标示一个应用和另一个应用的连接，会涉及到MAC地址，IP地址及端口。其中MAC地址具有设备唯一性，是物理地址，而IP地址是逻辑上的地址，一台机器有一个IP地址，但是一台主机上可能有多个应用所以，要标示某台机器上的某个应用就需要借助于端口，每个设备会为需要的应用分配单独的端口。接下来会重点从如下几个方面介绍IP协议相关的内容：

- ****IP地址MAC地址及端口划分**** :介绍IP地址的组成，IP头的组成，MAC地址相关，以及应用端口分配
- ****IP 相关技术**** : 介绍DNS，ARP，DHCP等和IP协议相关的技术。
- ****路由过程**** : 介绍一个包怎么从一个应用送往另一个应用。

##### IP 相关技术

- ****DNS****

域名和IP地址都可以唯一对应一台主机，我们可以使用它们中的任意一种形式来访问网络，但是由于IP地址难以记忆，所以就需要一个同样能够唯一标示一个主机地址的域名，但是我们协议部分使用的还是设备的IP地址，我们往浏览器中输入需要访问网站域名的时候，就需要有个中间对象将域名转换为IP地址，DNS就是充当这一对象，DNS如同网络中一个庞大的通讯录，它将自身具有意义的域名转换成不容易记住的IP地址。关于DNS详细的技术细节已经在前面介绍过了，大家可以查阅之前的内容。

![](./images/dns.png)

- ****ARP****

在应用层我们输入某个网站对应的域名地址后，经过DNS解析转换为IP地址，有了IP地址就可以向这个目标地址发送IP数据报。但是在底层数据链路层,进行实际通信时,还需要知道每个IP地址所对应的MAC地址，ARP协议就是用于通过目标IP地址，定位下一个接收数据包的网络设备的MAC地址。如果目标主机处在同一个数据链路上，那么可以直接得到目标主机的MAC地址，否则会得到下一条路由器的MAC地址。

要通过ARP协议获得下一级的MAC地址，首先源主机需要通过广播发送一个ARP请求包：“我要与IP地址为xxx的主机通话，谁知道它的 MAC地址？”。
数据链路上的所有主机都会收到这条消息并检查自己的IP地址，如果与ARP请求包中的IP地址一致，主机就会发送ARP响应包：“我就是IP地址为xxx的主机，我的 MAC地址是xxxx”。

在实际的使用过程中，每次往目标主机发送数据都要使用ARP是很低效的，通常的做法是把获取到的MAC地址缓存一段时间。一般来说，一旦源主机向目标地址发送一个数据包，接下来继续发送多次的概率非常大，因此这种缓存非常容易命中。

当下一次发送ARP请求或超过一定时间后，缓存就会失效，从而保证了即使 MAC地址与IP地址的对应关系发生了变化，数据包依然能够被正确的发往目标地址的目的。

![](./images/arp.png)

- ****ICMP****

ICMP(互联网控制消息协议（Internal Control Message Protocol）)，与IP协议一样同属TCP/IP模型中的网络层，并且ICMP数据包是包裹在IP数据包中的,它的作用是报告一些网络传输过程中的错误以及做一些同步工作。ICMP数据包有许多类型。每一个数据包只有前4个字节是相同域的，剩余的字段有不同的数据包类型的不同而不同。

![](./images/icmp.png)

类型字段：指明该数据包属于什么类型（大分类），长度1个字节。
代码字段：指明数据包属于大类里面的哪个小类，长度1个字节。类型字段与代码字段共同决定ICMP数据包类型，以及后续字段含义。
校验和： 指明该数据包的校验和，长度2个字节。该校验和覆盖整个ICMP数据包。

![](./images/icmp_code.png)

![](./images/icmp_image.png)

- ****DHCP****

DHCP（Dynamic Host Configuration Protocol）：动态主机配置协议，是一个局域网的网络协议，使用UDP协议工作， 主要有两个用途：

- 给内部网络或网络服务供应商自动分配IP地址
- 给用户或者内部网络管理员作为对所有计算机作中央管理的手段。

在IP网络中，每个连接网络的设备都需要分配唯一的IP地址。DHCP 使网络管理员能从中心结点监控和分配IP地址。当某台计算机移到网络中的其它位置时，能自动收到新的IP地址，而不用在每台计算机上单独配置固定的IP地址，这对移动设备普的今天显得尤为重要。

工作原理：

- ****发现阶段**** 即DHCP客户机寻找DHCP服务器的阶段。
DHCP客户机以广播方式（因为DHCP服务器的IP地址对于客户机来说是未知的）发送DHCP discover 发现信息来寻找DHCP服务器，网络上每一台安装了 TCP/IP协议的主机都会接收到这种广播信息，但只有DHCP服务器才会做出响应。

- ****提供阶段**** 即DHCP服务器提供IP地址的阶段。在网络中接收到DHCP discover发现信息的DHCP服务器都会做出响应，它从尚未出租的IP地址中挑选一个分配给DHCP客户机，向DHCP客户机发送一个包含出租的IP地址和其他设置的DHCP offer提供信息。

- ****选择阶段**** 即DHCP客户机选择某台DHCP服务器提供的IP地址的阶段。如果有多台DHCP服务器向DHCP客户机发来的DHCP offer提供信息，则DHCP客户机****只接受第一个收到的DHCP offer提供信息****，然后它以广播方式回答一个DHCP request请求信息，****该信息中包含向它所选定的DHCP服务器请求IP地址的内容****。之所以要以广播方式回答，是为了通知所有的DHCP服务器，他将选择某台DHCP服务器所提供的IP地址。

- ****确认阶段**** 即DHCP服务器确认所提供的IP地址的阶段。当DHCP服务器收到DHCP客户机回答的DHCP request请求信息之后，它便向DHCP客户机发送一个包含它所提供的IP地址和其他设置的DHCP ack确认信息，告诉DHCP客户机可以使用它所提供的IP地址。然后DHCP客户机便将其TCP/IP协议与网卡绑定，另外，除DHCP客户机选中的服务器外，其他的DHCP服务器都将收回曾提供的IP地址。

- ****重新登录**** 以后DHCP客户机每次重新登录网络时，就不需要再发送DHCP discover发现信息了，而是直接发送包含前一次所分配的IP地址的DHCP request请求信息。当DHCP服务器收到这一信息后，它会尝试让DHCP客户机继续使用原来的IP地址，并回答一个DHCP ack确认信息。如果此IP地址已无法再分配给原来的DHCP客户机使用时（比如此IP地址已分配给其它DHCP客户机使用），则DHCP服务器给DHCP客户机回答一个DHCP nack否认信息。当原来的DHCP客户机收到此DHCP nack否认信息后，它就必须重新发送DHCP discover发现信息来请求新的IP地址。

- ****更新租约**** DHCP服务器向DHCP客户机出租的IP地址一般都有一个租借期限，期满后DHCP服务器便会收回出租的IP地址。如果DHCP客户机要延长其IP租约，则必须更新其IP租约。DHCP客户机启动时和IP租约期限过一半时，DHCP客户机都会自动向DHCP服务器发送更新其IP租约的信息。

![](./images/dhcp.jpg)

- ****NAT****
  
网络地址转换协议 (Network Address Translation) 在计算机网络中是一种在IP数据包通过路由器或防火墙时重写来源IP地址或目的IP地址的技术。这种技术被普遍使用在有多台主机但只通过一个公有IP地址访问互联网的私有网络中。
NAT最初的发展是为了解决IPv4地址短缺而流行起来的。NAT在家庭和小型办公室网络连接上的路由器成为了一个标准特征，因为对他们来说，申请独立的IP地址的代价要高于所带来的效益。

在一个典型的配置中，一个本地网络使用一个专有网络的指定子网（比如192.168.x.x或10.x.x.x）和连在这个网络上的一个路由器。这个路由器占有这个网络地址空间的一个专有地址（比如192.168.0.1），同时它还通过一个或多个因特网服务提供商提供的公有的IP地址连接到因特网上。当信息由本地网络向因特网传递时，源地址从专有地址转换为公用地址。由路由器跟踪每个连接上的基本数据，主要是目的地址和端口。当有回复返回路由器时，它通过输出阶段记录的连接跟踪数据来决定该转发给内部网的哪个主机；如果有多个公用地址可用，当数据包返回时，TCP或UDP客户机的端口号可以用来分解数据包。对于因特网上的通信，路由器本身充当源和目的。流行在网络上的一种看法认为，IPv6的广泛采用将使得NAT不再需要，因为NAT只是一个处理IPv4的地址空间不足的方法。

![](./images/static-nat.jpg)

- ****VLSM****

VLSM的作用就是在类的IP地址的基础上，****从它们的主机号部分借出相应的位数来做网络号****，也就是增加网络号的位数。各类网络可以用来再划分子网的位数为：A类有二十四位可以借，B类有十六位可以借，C类有八位可以借（可以再划分的位数就是主机号的位数。实际上不可以都借出来，因为IP地址中必须要有主机号的部分，而且主机号部分剩下一位是没有意义的，所以在实际中可以借的位数是在上面那些数字中再减去2，借的位作为子网部分）

- ****CIDR****

CIDR主要是一个按位的、基于前缀的，用于解释IP地址的标准。它通过把多个地址块组合到一个路由表表项而使得路由更加方便。这些地址块叫做CIDR地址块。当用二进制表示这些地址时，它们有着在开头部分的一系列相同的位。IPv4的CIDR地址块的表示方法和IPv4地址的表示方法是相似的：由四部分组成的点分十进制地址，后跟一个斜线，最后是范围在0到32之间的一个数字：A.B.C.D/N。点分十进制的部分和IPv4地址一样是一个被分成四个八位位组的32位二进制数。斜线后面的数字就是前缀长度，因为IPv4地址的长度总是32位，N位长的CIDR前缀就意味着地址里前N位匹配，后32-N位不匹配。这些位有2^(32-N)种不同的组合，即2^(32-N)个IPv4地址与CIDR地址块的前缀匹配。前缀越短就能匹配越多的地址，越长就匹配得越少。一个地址可能与多个长度不同的CIDR前缀匹配。

![](./images/CIDR-IPv4-Subnet-Mask-Cheat-Sheet.png)
  
- ****IP隧道****
  
IP隧道技术是路由器把一种网络层协议封装到另一个协议中以跨过网络传送到另一个路由器的处理过程，发送路由器将被传送的协议包进行封装，经过网络传送，接受路由器解开收到的包，取出原始协议，而在传输过程中的中间路由器并不在意封装的协议是什么。隧道技术是一种点对点的链接，因而必须在链接的两端配置隧道协议。

![](./images/pp.jpg)

下面是关于VPN/L2TP/IPSec/PPTP的介绍视频：

- [L2TP/IPSec VPN Protocol vs PPTP - Which One Is Best?](https://www.youtube.com/watch?v=5pCYnFNxugE)

- [What is a VPN and How Does it Work?](https://www.youtube.com/watch?v=_wQTRMBAvzg)

- [VPN翻墙到底是什么原理？](https://www.youtube.com/watch?v=ZT-q6mJ-e3g)

##### IP地址MAC地址及端口划分

IP协议是一个****无连接的服务****，负责从源地址到目的地址之间传送数据报，为了适应不同网络对分组大小的要求，还可以对上层报文进行分割。

- ****公网IP地址****:
公网IP地址是可以被直接路由查找到的、并需要向IP地址管理机构申请、注册、购买，且保证全球唯一的地址。它就相当于身份证号.

- ****私网IP地址****:又称为局域网IP地址,它是仅可以在用户的局域网内部使用的IP地址，且可以重复使用，无需向IP地址管理机构申请、注册、购买的地址。

缓解公网IP地址枯竭的途径：

- ****NAT 网络地址转换 (Network Address Translation)****
- ****VLSM 可变长子网掩码 (Variable Length Subnet Mask)****
- ****CIDR 无类域间路由 (Classless Inter-Domain Routing)****

##### IPv4 地址结构 

IPv4使用32位4字节地址,理论上整个地址空间可容纳2^32^（约43亿）个地址。32个字节中共分成两大部分，网络地址和主机地址：

![](./images/ip_submarsk.png)

网络部分中使用部分位数来区分网络地址类别，IPv4共分为 A,B,C,D,E五组：

![](./images/ip_address.jpg)


它们都有不同的网络类别长度，剩余部分就用来标识主机。网络ID用来确定不同类型的网络里面有的网络数量，而主机ID用来确定每个网络里面的IP地址数。

****_A类地址_****

网络ID占最高一个字节，主机ID则占用剩余的三个字节,A类地址的最高位固定为0,网络ID全为0的地址称为保留地址,全为1的（十进制的127）是本地环路地址。A类网络可构建的网络数量最少，但每个网络中拥有的地址数量是最多的.

****_B类地址_****

B类地址网络ID扩展到了16位，前两位固定为10

****_C类地址_****

B类地址网络ID扩展到了24位，前三位固定为110

****_D类地址_****

D类地址属于组播地址，用于组播通信。源主机只需要发送一次数据就可以使对应的组中所有主机都接受到这份数据。前4位固定为1110

****_E类地址_****

该地址是IANA（互联网号码分配局）保留地址，不分配给用户使用

从整体上看所有地址分成三大类：A、B、C类地址是单播地址，D类为组播地址，E类为保留地址

下面是一个整体的概图：

![](./images/ipv4.png)

##### IPv6 地址结构 

IPv6的地址长度是128位（bit）,将这128位的地址按每16位划分为一个段，将每个段转换成十六进制数字，并用冒号隔开:

```
例如：2000:0000:0000:0000:0001:2345:6789:abcd
```

_地址压缩_：

_前导零压缩法_：

将每一段的前导零省略，但是每一段都至少应该有一个数字

```
例如：2000:0:0:0:1:2345:6789:abcd
```

_双冒号法_：

如果一个以冒号十六进制数表示法表示的IPv6地址中，如果几个连续的段值都是0，那么这些0可以简记为::。每个地址中只能有一个::。

```
例如：2000::1:2345:6789:abcd
```

![](./images/ipv6_address.png)

##### IP帧结构

![](./images/ipv4-ipv6-620x349.gif)


##### 从宏观看IP帧，TCP/UDP帧，MAC地址，端口在数据通信过程中的作用：

##### 路由过程

- ****推荐文章****

- [IP协议相关技术--图解TCP/IP](https://blog.csdn.net/sinat_37138973/article/details/72767558)
- [IP地址](https://juejin.im/post/5c8e3e426fb9a070ab7e1a73)
- [一文看懂IP地址：含义、分类、子网划分、查与改、路由器与IP地址](https://network.51cto.com/art/201911/605681.htm)
- [网络编程懒人入门(十一)：一文读懂什么是IPv6](https://zhuanlan.zhihu.com/p/132411396)
- [协议森林](https://www.cnblogs.com/vamei/archive/2012/12/05/2802811.html)
- [ICMP协议详解](https://blog.csdn.net/baidu_37964071/article/details/80514340)
- [协议森林06 瑞士军刀 (ICMP协议)](https://www.cnblogs.com/vamei/archive/2012/12/05/2801991.html)
- [DHCP协议原理及其实现流程](https://blog.csdn.net/wuruixn/article/details/8282554)
- [IP 隧道技术：基础篇](https://blog.csdn.net/bat603/article/details/779842)

#### 应用层协议

##### HTTP、HTTPS、HTTP2, SPDY、QUIC

##### SSL/TLS

##### Nginx

##### CDN

##### WebSocket

##### 音视频相关协议

##### 网络上的硬件设备简介

- 中继器
- 集线器
- 网关
- 网桥
- 路由器
- 交换器 
- 调制解调器

#### 网络安全

#### 网络缓存技术

#### 网站架构

#### 前端常见问题

##### GraphQL/REST 

#### 浏览器原理

- [浏览器工作原理-webkit内核研究](https://juejin.im/entry/5a9a379af265da239d48c027)
- [深入剖析 WebKit](https://ming1016.github.io/2017/10/11/deeply-analyse-webkit/)
- [「一道面试题」输入URL到渲染全面梳理上-网络通信篇](https://juejin.im/post/5e9c48b2f265da47c558566b)
- [「一道面试题」输入URL到渲染全面梳理中-页面渲染篇](https://juejin.im/post/5e9f1db86fb9a03c85463560)
- [「一道面试题」输入URL到渲染全面梳理下-总结篇](https://juejin.im/post/5ebabbf96fb9a043586c8f9e)
- [HTTP 的前世今生：一次性搞懂 HTTP、HTTPS、SPDY、HTTP2](https://juejin.im/post/5be935f2e51d4570813b8cf0)

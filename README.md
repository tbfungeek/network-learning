# 计算机网络

![](./images/header.jpg)

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
  在计算机网络中由于各种干扰的存在，物理链路是不可靠的,数据链路层的主要功能是：通过各种****差错控制*** 、****流量控制****等控制协议，将有差错的物理信道变为无差错的、能可靠传输数据帧的数据链路。数据链路层的具体工作是接收来自物理层的比特流形式的数据，并封装成帧，传送到上一层；同样，也将来自上层的数据帧，拆装为比特流形式的数据转发到物理层。
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


下面是YouTobe上找到的一个比较好的视频[『初探資訊』OSI 網路通訊架構](https://www.youtube.com/watch?v=kyARqyCjoYY)


##### OSI七层网络模型与TCP/IP 四层网络结构对应关系

![](./images/7_layer_4_layer.png)


#### TCP/IP 协议 传输层连接建立

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

#### 传输包的封装。解封装

#### 数据包的传输



#### HTTP、HTTPS、SPDY、HTTP2

- [HTTP 的前世今生：一次性搞懂 HTTP、HTTPS、SPDY、HTTP2](https://juejin.im/post/5be935f2e51d4570813b8cf0)


#### 浏览器原理

- [浏览器工作原理-webkit内核研究](https://juejin.im/entry/5a9a379af265da239d48c027)
- [深入剖析 WebKit](https://ming1016.github.io/2017/10/11/deeply-analyse-webkit/)




http://c.biancheng.net/tcp_ip/




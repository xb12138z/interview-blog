# network
## TCP

1.tcp建立的三个过程：

建立连接、传输数据、断开连接

2.TCP数据头：

源端口16bits

目的端口16bits

Sequence Number 32bits                     本机包的序号（本次发送包的序号）

Acknowledgment Number 32bits         对方机的确认信号（表示序号前的包全部收到）

Header Leng 4bits

Resv 4bits

标识位（8个）

windowSize  16bits   告诉对方我的存储大小（我能收多少）

TCP Checksum  16bits

Urgent Pointer  16bits


3..建立连接过程：三次握手

客户端                                    服务器

    ->syn、seqnum

                   suqnum、syn、ack、acknum<-

   ->acknum、ack  

服务器收到第一次 SYN：进入 半连接队列（SYN队列）

完成三次握手：进入全连接队列（accept队列）

半连接队列：syn队列（不限制长度的话，会被攻击syn泛洪）【listen（fd，backlog）backlog是队列限制长度】

全连接队列：accept队列     clientfd = accept（listenfd ， &addr ， &addrlen）；

4.·网络编程、

    网络原理（计算机网络-tcp、ip协议栈）


![项目截图](./net.jpg "项目截图")

一个TCP数据包到达服务器后，首先由网卡接收，通过DMA写入内存并触发中断，进入Linux内核网络栈。数据依次经过二层、三层、四层处理，在TCP层根据四元组找到对应socket，将数据放入接收缓冲区，并唤醒阻塞的
应用进程。应用通过read或recv系统调用将数据从内核拷贝到用户空间进行处理。


5.Posix API

open、read、write、close、seek、recv、send、connect、accept

6.超时重传：

规定的时间不是固定的时间，是根据回报的时间和网络状态算出来的

7.断开连接（四次挥手）

各自发送fin，回复ack

TIme_wait：

tcb.status = TIME_WAIT;

set_timer();


![项目截图](./state.jpg "项目截图")

## UDP
1.包头：（8字节）

Source Port 

Destination Port

Length

CheckSum



## 面试题
1.三次握手过程：

核心目的：

(1)确认双方都有发送和接收能力

(2)同步初始序列号（ISN）

(3)建立可靠连接


第一次握手：客户端发送SYN = 1、seqnum = x（客户端初始序列号

状态：客户端：SYN_SENT

第二次握手：服务器发送SYN = 1、ACK = 1、seqnum = y、acknum = x + 1

状态：服务器：SYN_RCVD

第三次握手：客户端发送ACK = 1、seqnum = x + 1、acknum = y + 1

状态：双方进入 ESTABLISHED

如果两次握手的话：如果客户端发的 SYN 因网络延迟滞留，服务器会误以为客户端要建立连接。第三次握手的作用：防止已失效连接请求报文段突然又传到服务器

2.四次挥手过程

关闭连接必须两边都关闭

第一次握手 ： 客户端 → 服务器   FIN = 1  seq = u

客户端进入：FIN_WAIT_1

第二次握手 ： 服务器 → 客户端  ACK = 1 ack = u + 1

服务器进入：CLOSE_WAIT      客户端进入：FIN_WAIT_2

⚠ 说明：服务器此时可能还有数据要发、所以不能马上关闭

第三次握手 ： 服务器 → 客户端  FIN = 1  seq = v

服务器进入：LAST_ACK

第四次握手 ： 客户端 → 服务器  ACK = 1 ack = v + 1

客户端进入：TIME_WAIT     服务器进入：CLOSED



3.大量close_wait的原因

close_wait状态   -   收到一段发出的close信号后，返回ack后，本端没有调用close（）函数。

1.在0 = recv后需要清除临时数据后才可以进行close函数的调用              --      没有及时的清除临时数据。 解决方法 ： 清除临时数据放到异步的线程中进行清除   /   先发close再清除临时数据



4.time_wait状态持续时间

客户端要等待 2MSL。   （MSL：最大报文生存时间。）

原因：

    保证最后一个ACK到达  如果服务器没收到，会重发FIN

    防止旧连接报文影响新连接

5.udp并发的实现

并发：同时承载的用户数量

你的项目，并发量怎么样？

1.内部测试

2.真实情况



并发的实现：

1.sendto发送给 udpserver8000

2.udp server recvfrom接收user1的数据

3.分配fd以及另外的端口，回发数据给user1



6.tcp首部长度，有哪些字段

源端口16bits

目的端口16bits

Sequence Number 32bits                     序列号（本次发送包的序号）

Acknowledgment Number 32bits         确认号（表示序号前的包全部收到）

Header Leng 4bits      首部长度

Resv 4bits    保留位

标识位（8个） SYN、ACK、FIN、RST（重新连接）、PSH（立即推送）、URG（紧急数据）

windowSize  16bits   告诉对方我的存储大小（我能收多少）

TCP Checksum  16bits    检测数据是否损坏

Urgent Pointer  16bits    URG为1时表示紧急数据位置

Options 可变    ——     常见 TCP Option：

- MSS（最大报文段）

- Window Scale（窗口扩大）

- SACK（选择确认）

- Timestamp（时间戳）

因为 Option 存在，TCP 首部才会变成 20~60 字节。

7.tcp的listen参数backlog的含义

backlog是队列限制长度

8.accpect发生在三次握手的那一步骤

在三次握手之后

9.tcp/udp的区别

(1)tcp 是 stream ， udp是dgram

(2)tcp保证顺序，udp不保证，实时性更好

(3).tcp保证有拥塞控制，udp没有

(4)tcp一个客户端对应一个soctfd  ， udp直接监听一个sockfd，所有客户端共用一个连接

(5)实时性要求高的会选择udp，迅雷下载，大量传输时选择udp。


10滑动窗口如何实现

TCP发送方有：发送窗口 = min(拥塞窗口cwnd, 接收窗口rwnd)

窗口滑动的本质：

随着ACK到来、左边界右移

11.epoll与select的区别

select/poll/epoll

1.select(maxfd , rfds , wfds , efds, timeout); //最大fd数量，可读，可写，错误，轮询时间。   timeout==    0为立即读取 、 -1为等待时间触发  、 >0 等待固定时长立即返回

每次select调用，需要把rfds、wfds复制到内核    需要通过循环遍历

2.poll（pfds，length，timeout）

每次poll调用，需要把pfds复制到内核空间，之后循环遍历【与select一致】

3.epoll    --    Linux服务器

int epfd  =  epoll_create(size);//创建一个总集

epoll_ctl(epfd ,OPERA , fd ,event);//再总集中增加一个一个的节点

epoll_wait(epfd,events,length,timeout);//把就绪的节点带出来

select要将关注的全部io都copy进去，然后判断状态后全部带出

而epoll只需要一次将全部io都放进去，需要用的时候，带出就绪节点即可

epoll

12.dpdk对传统网络做了哪些改变

网络io -> dpdk

磁盘io -> spdk

旁路技术 -   正常tcp从网卡到协议栈需要进行一次复制，从协议栈到server中需要再进行一次复制。

dpdk将网卡的接收直接映射到了内存中

13.全连接队列和半连接队列：

服务器收到第一次 SYN：进入 半连接队列（SYN队列）

完成三次握手：进入全连接队列（accept队列）

半连接队列：syn队列（不限制长度的话，会被攻击syn泛洪）【listen（fd，backlog）backlog是队列限制长度】

全连接队列：accept队列     clientfd = accept（listenfd ， &addr ， &addrlen）；

14.拥塞四个经典算法：

慢启动：从小到大试探网路能力

拥塞避免：当前发送窗口超过慢启动阈值时，进入拥塞避免阶段。防止突然拥塞

快重传：如果收到三个重复的ACK（因为受到后序包后 会ACK会显示为没收到的包），说明该包丢了可以立即重传丢包，不用等超时。

快恢复： 触发快重传 后    ssthresh = cwnd / 2    cwnd = ssthresh + 3。避免重新慢启动。
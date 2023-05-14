# ospf基础

ospf学习笔记
<!--more-->
# ospf基础

### **Router ID：**

-   企业网中的设备少则几台多则几十台甚至几百台，每台路由器都需要有一个唯一的ID用于标识自己。
-   Router ID是一个32位的无符号整数，其格式和IP地址的格式是一样的，RouterID选举规则如下：
    1.  手动配置OSPF路由器的Router ID（通常建议手动配置）；
    2.  如果没有手动配置Router ID，则路由器使用Loopback接口中**最大的IP地址**作为 Router ID；
    3.  如果没有配置Loopback接口，则路由器使用物理接口中**最大的IP地址**作为Router ID。
-   OSPF的路由器Router ID重新配置后，可以通过重置OSPF进程来更新Router ID。

注：router ID只是格式为ip地址格式，只是作为名称地址可以不存在

### **邻居建立过程：**

![](/post-images/1683042032465.png)

-   OSPF路由器之间在交换链路状态信息之前，首先需要彼此建立邻居关系，通过Hello报文 实现。
    -   OSPF协议通过Hello报文可以让互联的路由器间自动发现并建立邻居关系，为后续可
    -   在形成邻居关系过程中，路由器通过Hello报文完成一些参数的协商。
    -   邻居关系建立后，周期性的Hello报文发送还可以实现邻居保持的功能，在一定时间 内没有收到邻居的Hello报文，则会中断路由器间的OSPF邻居关系。

        达性信息的同步作准备。
        两种方式：组播224.0.0.5 224.0.0.6、手工单播
-   LSA链路状态信息
    -   链路类型
    -   接口ip地址及掩码
    -   链路上所连接的邻居路由器
    -   链路的带宽（开销）

ospf 度量方式

ospf 报文类型

![](/post-images/1683042055018.png)

路由信息仅在第4种里，其他的都是摘要信息

DR角色不会被抢占，需手动重启进程

ospf的lsdb同步

![](/post-images/1683042067477.png)

-   状态含义：
    -   ExStart：邻居状态变成此状态以后，路由器开始向邻居发送DD报文。Master/Slave  关系是在此状态下形成的，初始DD序列号也是在此状态下确定的。在此状态下发送 的DD报文不包含链路状态描述。
    -   Exchange：在此状态下，路由器与邻居之间相互发送包含链路状态信息摘要的DD报 文。
    -   Loading：在此状态下，路由器与邻居之间相互发送LSR报文、LSU报文、LSAck报 文。
    -   Full：LSDB同步过程完成，路由器与邻居之间形成了完全的邻接关系。
-   LSDB同步过程如下：
    -   RTA和RTB的Router ID分别为[1.1.1.1和2.2.2.2并且二者已建立了邻居关系]。当RTA  的邻居状态变为ExStart后，RTA会发送第一个DD报文。此报文中，DD序列号被随 机设置为X，I-bit设置为1，表示这是第一个DD报文，M-bit设置为1，表示后续还有 DD报文要发送，MS-bit设置为1，表示RTA宣告自己为Master。
    -   当RTB的邻居状态变为ExStart后，RTB会发送第一个DD报文。此报文中，DD序列号 被随机设置为Y（I-bit=1，M-bit=1，MS-bit=1，含义同上）。由于RTB的Router  ID较大，所以RTB将成为真正的Master。收到此报文后，RTA会产生一个 Negotiation-Done事件，并将邻居状态从ExStart变为Exchange。
    -   当RTA的邻居状态变为Exchange后，RTA会发送一个新的DD报文，此报文中包含了 LSDB的摘要信息，序列号设置为RTB在步骤2中使用的序列号Y，I-bit=0，表示这不 是第一个DD报文，M-bit=0，表示这是最后一个包含LSDB摘要信息的DD报文，  MS-bit=0，表示RTA宣告自己为Slave。收到此报文后，RTB会产生一个 Negotiation-Done事件，并将邻居状态从ExStart变为Exchange。
    -   当RTB的邻居状态变为Exchange后，RTB会发送一个新的DD报文，此报文包含了 LSDB的摘要信息，DD序列号设置为Y+1, MS-bit=1，表示RTB宣告自己为Master。
    -   虽然RTA不需要发送新的包含LSDB摘要信息的DD报文，但是作为Slave，RTA需要 对Master发送的每一个DD报文进行确认。所以，RTA向RTB发送一个新的DD报文，  序列号为Y+1，该报文内容为空。发送完此报文后，RTA产生一个Exchange-Done  事件，将邻居状态变为Loading。RTB收到此报文后，会将邻居状态变为Full（假设 RTB的LSDB是最新最全的，不需要向RTA请求更新）。
        -   如果需要更新：

           ![](/post-images/1683042094446.png)
        -   RTA开始向RTB发送LSR报文，请求那些在Exchange状态下通过DD报文发现的、并 且在本地LSDB中没有的链路状态信息。
            -   RTB向RTA发送LSU报文，LSU报文中包含了那些被请求的链路状态的详细信息。
        -   RTA向RTB发送LSAck报文，作为对LSU报文的确认。RTB收到LSAck报文后，双方便 建立起了完全的邻接关系。
    -   从建立邻居关系到同步LSDB的过程较为复杂，错误的配置或设备链路故障都会导致无法完 成LSDB同步。为了快速排障，最关键的是要理解不同状态之间切换的触发原因。

        RTA在完成LSU报文的接收之后，会将邻居状态从Loading变为Full。

ospf邻居状态机制

![](/post-images/1683042130529.png)

-   这是形成邻居关系的过程和相关邻居状态的变换过程。
    -   Down：这是邻居的初始状态，表示没有从邻居收到任何信息。在NBMA网络上，此 状态下仍然可以向静态配置的邻居发送Hello报文，发送间隔为PollInterval，通常和 Router DeadInterval间隔相同。
    -   Attempt：此状态只在NBMA网络上存在，表示没有收到邻居的任何信息，但是已经 周期性的向邻居发送报文，发送间隔为HelloInterval。如果Router DeadInterval间 隔内未收到邻居的Hello报文，则转为Down状态。
    -   Init：在此状态下，路由器已经从邻居收到了Hello报文，但是自己不在所收到的 Hello报文的邻居列表中，表示尚未与邻居建立双向通信关系。在此状态下的邻居要 被包含在自己所发送的Hello报文的邻居列表中。
    -   2-Way Received：此事件表示路由器发现与邻居的双向通信已经开始（发现自己在 邻居发送的Hello报文的邻居列表中）。Init状态下产生此事件之后，如果需要和邻居 建立邻接关系则进入ExStart状态，开始数据库同步过程，如果不能与邻居建立邻接 关系则进入2-Way。
    -   2-Way：在此状态下，双向通信已经建立，但是没有与邻居建立邻接关系。这是建立 邻接关系以前的最高级状态。
    -   1-Way Received：此事件表示路由器发现自己没有在邻居发送Hello报文的邻居列 表中，通常是由于对端邻居重启造成的。
    -   ExStart：这是形成邻接关系的第一个步骤，邻居状态变成此状态以后，路由器开始 向邻居发送DD报文。主从关系是在此状态下形成的；初始DD序列号是在此状态下决 定的。在此状态下发送的DD报文不包含链路状态描述。
    -   Exchange：此状态下路由器相互发送包含链路状态信息摘要的DD报文，描述本地 LSDB的内容。
    -   Loading：相互发送LS Request报文请求LSA，发送LS Update通告LSA。
    -   Full：两台路由器的LSDB已经同步。

DR与BDR选举

规则：基于接口，接口的优先级越大越优先，优先级相等 router ID越大越优先

优先级为0不参与选举

.6只有dr或bdr接收

.5给所有的ospf接收

### **邻接建立过程：**

路由同步后才是邻接关系


---

> 作者: WAKE  
> URL: https://weiqinke.com/ospf-ji-chu/  


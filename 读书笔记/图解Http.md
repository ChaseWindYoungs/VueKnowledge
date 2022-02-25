# 一、**了解 Web 及网络基础**

Web 是建立在何种技术之上，以及 HTTP 协议是如何诞 生并发展的

HTTP（HyperText Transfer Protocol，超文本传输协议）

### 1、网络基础 **TCP/IP**

通常使用的网络（包括互联网）是在 TCP/IP 协议族的基础上运作 的。而 HTTP 属于它内部的一个子集。

- 1）**TCP/IP** 协议族
    - 计算机与网络设备要相互通信，双方就必须基于相同的方法、或者规则，统称为 协议（protocol）
    - 而 **TCP/IP** 是互联网相关的各类协议族的总称

- 2）**TCP/IP** 的分层管理
    
    
    族按层次分别分 为以下 4 层：应用层、传输层、网络层和数据链路层
    
    优点：逻辑清晰，修改与设计简单，自由度高，
    
    **应用层**：
    
    - 应用层决定了向用户提供应用服务时通信的活动。
    - FTP（File Transfer Protocol，文件传输协议）
    - DNS（Domain Name System，域名系统）
    - HTTP（HyperText Transfer Protocol，超文本传输协议）
    
    **传输层** 
    
    - 传输层对上层应用层，提供处于网络连接中的两台计算机之间的数据 传输。
    - TCP（Transmission Control Protocol，传输控制协议）
    - UDP（User Data Protocol，用户数据报 协议）
    
    **网络层**（又名网络互连层）
    
    - 来处理在网络上流动的数据包。数据包是网络传输的最小数 据单位。
    
    **链路层**（又名数据链路层，网络接口层）
    
    - 来处理连接网络的硬件部分，操作系统、硬件的设备驱，网卡，光纤，等硬件上的范畴均在 链路层的作用范围之内。

- 3）**TCP/IP** 通信传输流
    
    
    TCP/IP 协议族进行网络通信时，会通过分层顺序与对方进行通信。
    
    发送端从应用层往下走，接收端则往应用层往上走。
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6eee1339-d36f-4de1-94d4-6c3538e1795e/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/69e6688e-6e97-4161-ae84-9e5afa2f7c81/Untitled.png)
    
    发送端在层与层之间传输数据时，每经过一层时必定会被打上一个该 层所属的首部信息。
    
    反之，接收端在层与层传输数据时，每经过一层 时会把对应的首部消去。 
    
    这种把数据信息包装起来的做法称为封装（encapsulate）。
    

- 4）与 **HTTP** 关系密切的协议 **: IP**、**TCP** 和 **DNS**
    
    负责传输的 **IP** 协议（网络层）
    
    IP（Internet Protocol）网际协议作用是把各种数据包传送给对方，
    
    IP 地址 和 MAC 地址 是保证传送的两个重要条件
    
    IP 地址指明了节点被分配到的地址，MAC 地址是指网卡所属的固定 地址。
    
    IP 地址可以和 MAC 地址进行配对。
    
    IP 地址可变换，但 MAC 地址基本上不会更改
    
    ARP 协议（Address Resolution Protocol）
    
    ARP 是一种用以解析地址的协议，根据通信方 的 IP 地址就可以反查出对应的 MAC 地址。
    
    确保可靠性的 **TCP** 协议（传输层）
    
    提供可靠的字节流服务
    
    为了准确无误地将数据送达目标处，TCP 协议采用了三次握手 （three-way handshaking）策略
    
    TCP 的握手标志（flag） —— SYN（synchronize） 和 ACK（acknowledgement）
    
    负责域名解析的 **DNS** 服务（应用层）
    
    提供通过域名 查找 IP 地址，或逆向从 IP 地址反查域名的服务
    

- 5）**URI** 和 **URL**
    
    URI（Uniform Resource Identifier，统一资源标识符）
    
    是由某个协议方案表示的资源的定位标识符。协议方案是指访问资源所使用的协议类型名称。
    
    URL（Uniform Resource Locator，统一资源定位符）
    
    URL表示资源的地点（互联 网上所处的位置）
    
    URL是 URI 的子集
    
    **URI** 格式
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/54f6aee6-9f17-4689-9438-9e31ab090a88/Untitled.png)
    

### 2、**简单的 HTTP 协议**

**HTTP** 协议用于客户端和服务器端之间的通信，通过请求和响应的交换达成通信

**HTTP** 是不保存状态的协议，是为了 更快地处理大量事务，确保协议的可伸缩性

告知服务器意图的 **HTTP** 方法

**GET** ：获取资源

求访问已被 URI 识别的资源

**POST**：传输实体主体

**PUT**：传输文件

**DELETE**：删除文件

**OPTIONS**：询问支持的方法

来查询针对请求 URI 指定的资源支持的方法

**HEAD**：获得报文首部

和 **GET** 一样，但不返回报文主体

**TRACE**：追踪路径

**CONNECT**：要求用隧道协议连接代理
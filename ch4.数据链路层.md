# 数据链路层

## 数据链路层概述

+ 作用：实现了相邻节点之间的通信，为网络层实现主机之间的通信提供了支持

+ 数据链路层信道类型：

	+ 点对点信道：使用一对一的点对点通信方式
	+ 广播信道：使用一对多的广播通信方式

+ 物理链路与数据链路：

	+ 物理链路：简称链路，指的是相邻节点间无源的物理线路，是一条通路的组成部分

	+ 数据链路：链路加上实现通信协议的软硬件

		>  数据链路（逻辑链路） = 物理链路 + 通信规程 

	+ 适配器/网卡：包括数据链路层和物理层

+ 帧：数据链路层协议数据单元

	+ 发送方把网络层交下来的IP数据报添加首部和尾部封装成帧
	+ 接收方从帧中取出IP数据报交给网络层

+ 主要功能：

	+ 数据链路的建立、拆除和管理
	+ 封装成帧
	+ 透明传输
	+ 差错控制
	+ 流量控制

## 三个基本问题

### 封装成帧

+ 在一段数据的前后分别添加首部和尾部
+ 首部和尾部重要作用就是帧定界；首部中还含有控制信息
+ 用控制字符作为帧定界符，例如可以使用特殊的帧定界符SOH、EOT

### 透明传输

+ 透明：一个实际存在的事物看起来像不存在一样

+ 无论发送什么样的比特组合数据，都可以没有差错的通过数据链路层
+ 原因：数据中存在和SOH或EOT一样的数据，错误找到帧的边界

+ 解决办法：字节填充和字符填充
	+ 在SOH和EOT前加一个转义字符ESC，如果ESC也在数据当中，则在ESC前再加一个ESC

### 差错检测

+ 比特差错：1变成0，0变成1
+ 误比特率或误码率是衡量通信链路的标准
+ 使用循环冗余检验CRC，码的检错和纠错能力是用信息量的冗余度换取的
+ 在数据后添加的冗余码称为帧检验序列（FCS），FCS与CRC不同，CRC是检错方法，FCS是冗余码，FCS由CRC得来，但CRC不是得来FCS的唯一方法
+ 使用CRC只能做到无差错接收，即接收到的帧没有差错；不能实现无差错传输或可靠传输

## 点对点信道的数据链路层协议

### 串行线路IP（SLIP）

+ 在串行线路支持TCP/IP协议的点对点协议，不能收发IP数据报
+ 允许主机、路由器混连
+ 线路速率非常低
+ 为个人用户上网提供拨号IP模式，为行业用户提供专线IP模式
+ 存在的问题：没有寻址功能；数据帧中没有类型标记；没有差错检测和校正功能

### 点对点协议PPP

+ 能在多种链路上运行

+ 应满足的需求：简单（首要的要求）、封装成帧、透明性、多种网络层协议、多种类型链路、差错检测、检测连接状态、最大传送单元、网络层地址协商、数据压缩协商

	不需要的功能：纠错、流量控制、设置需要、多点线路、半双工或单工

+ 每收到一个帧，CRC检验，如果正确则接收，错误则丢弃

+ 使用场景：用户到ISP的链路使用；PPPoe实现了传统以太网没有身份验证、加密以及压缩的功能

+ 组成：

	+ 将IP数据报封装到串行链路的方法
	+ 链路控制协议LCP：建立配置测试链路，身份验证
	+ 网络控制协议NCP

+ 透明传输问题：
	+ 异步传输：字节填充：出现0x7E转为(0x7D,0x5E)，0x7D转为(0x7D,0x5D)，数值小于0x20变为(0x7D,控制字符+0x20)
	+ 同步传输：零比特填充：连续5个1填入一个0
+ 工作过程：包含物理层和网络层内容
	1. 用户拨号接入ISP，路由器确认，建立物理连接
	2. 发送LCP分组
	3. NCP给主机分配临时IP
	4. 通信完毕，NCP释放连接，收回IP地址，LCP释放数据链路层连接，最后释放物理层连接
+ 鉴别：验证通信双方的确是自己的通信对象，
	+ 口令鉴别协议PAP：两次握手、明文传送、被鉴别方首先发起请求
	+ 质询握手鉴别协议CHAP：三次握手，主鉴别方首先发起请求、加密传送

## 广播信道的数据链路层协议

+ 局域网使用广播信道

### Ethernet II

+ DIX Ethernet II是世界上第一个局域网规约

+ IEEE 802.3是IEEE第一个局域网标准，致力研究物理层和MAC层

+ 媒体接入控制（MAC）子层：与接入到传输媒体有关的内容都放在MAC子层

	逻辑链路控制子层（LLC）：负责没有中间交换节点的两站之间的数据帧传输

+ MAC地址：也叫硬件地址或物理地址

	+ 48位，6字节
	+ 前3位，称为组织唯一标识符，管理机构向厂家分配
	+ 后3位，扩展唯一标识符，由厂家自行指派，必须保证没有重复地址

+ Ethernet II帧格式

	+ 目的地址、源地址：各6字节
	+ 类型：2字节，用来标志数据是由哪个协议负责
	+ 数据：范围在46-1500字节，如果小于46字节，要在后面添加填充字段
	+ 在帧的最前面加硬件生成的8字节，前7字节是前同步码，实现MAC帧比特同步。第8字节是帧开始定界符

+ 无效MAC帧：对于无效MAC帧直接丢弃，不负责重传

	+ 数据字段长度与长度字段值不一致
	+ 长度不是整数字节
	+ FCS有错
	+ 数据字段长度不在46-1500间

+ 计算机通过适配器和局域网通信：

	+ 适配器的作用：数据帧封装解封装、串并转换、编译码、数据缓存、数据过滤

+ 局域网利用总线通信，易于实现广播通信，通过目的地址实现一对一通信，缺点是数据会碰撞

### CSMA/CD

+ 媒体共享技术：

	+ 静态划分信道，代价高不适合局域网
	+ 动态媒体介入控制：随机接入、受控接入
		+ 随机接入中的集中控制，通过主站探询从站、选择从站

+ 局域网采取无连接工作方式、发送的数据都是用曼彻斯特编码

	> 优点：具有自同步功能
	>
	> 缺点：频带比原始基带信道增加一倍

+ CSMA/CD协议：载波监听多址接入技术/碰撞检测

	+ 载波监听：发送数据之前检测总线上是否有其他计算机发送数据，发送之中也不断检测

	+ 多址接入：计算机以多址接入接在一根总线上

	+ 碰撞检测：便发送数据边检测信号电压大小，即检测碰撞（冲突）

	+ 争用期（碰撞窗口）：2t，两倍的端到端传播时延

	+ 在发送数据帧后，最多经过一个争用期就可以知道数据帧是否发生碰撞。以太网规定51.2$\mu s$为争用期

	+ 如果发生碰撞，适配器立即停止发送，等待随机事件后再次发送，随机事件采用截断二进制指数退避算法；

		发生碰撞后继续发送若干比特的人为干扰信号，让所有用户都知道发生了碰撞

	+ 不能全双工通信的原因：

		1. 每个站点发送数据之后的一段时间内，存在着遭遇碰撞的可能性
		2. 发送的不确定性是平均通信量远小于以太网的最高数据率

	+ 信道利用率：以太网的信道利用率不能达到100%

		+ 成功发送一帧占用信道时间$T_0 + t$，发送完毕后还要多一个端到端时延$t$ 
		+ 定义参数a，以太网单程端到端时延与帧的发送时间之比；如果$a \to0$表示一发生碰撞就可以检测；a越大争用期时间越长，利用率越低
		+ 极限信道利用率$S_{max} = \frac{T_0}{T_0 + t} = \frac{1}{a+1 }$

## 扩展局域网

+ 网络的中介设备：
	+ 连接设备：建立物理网络连接，不改变数据传输路径（网卡、放大器、集线器）
	+ 互联设备：网络中搬运数据，将数据引导到网络中的特定位置，将数据转换为另一种格式（网桥、交换路由器）

### 在物理层扩展局域网

+ 光纤和光纤调制解调器
+ 集线器：将多个局域网网段连成更大的、多级星型结构的局域网
	+ 优点：是不同碰撞域的局域网可以垮碰撞域通信、扩大局域网覆盖的地理范围
	+ 缺点：碰撞域增大，吞吐量未提高；所有计算机都处于同一个碰撞域和广播域

### 数据链路层扩展局域网

+ 网桥：
	+ 网桥工作在数据链路层
	+ 根据MAC帧目的地址对收到的帧进行转发和过滤
	+ 隔离碰撞域
+ 以太网交换机：
	+ 本质上是多端口网桥
	+ 全双工方式
	+ 具有并行性，同时连通多对端口，使多对主机同时通信
	+ 用硬件转发，比用软件转发的网桥速度快
	+ 每个端口处于独立的碰撞域中，但位于同一个广播域
	+ 优点：性能远远超过集线器；用户独享带宽，增加容量；接入设备不需要改动；使用存储转发方式可以支持不同速率设备连接
	+ 帧交换表通过自学习算法建立，即插即用
	+ 缺点：会出现环路问题，导致广播风暴和MAC地址不稳定；
	+ 解决环路问题：指定生成树协议STP，不改变网络实际拓扑，逻辑上切断某些链路，形成无环路的树状结构

### 虚拟局域网VLAN

+ 是局域网给用户提供的服务，不是新型局域网

+ VLAN由一些局域网网段构成的与物理位置无关的逻辑组

+ VLAN的主要作用是隔离广播域

+ VLAN = 广播域 + 逻辑网段

+ 划分方法：

	+ 基于交换机端口：

		优点：定义成员简单

		缺点：不允许成员移动

	+ 基于MAC地址：

		优点：允许用户移动

		缺点：需要输入和管理大量MAC地址，如果MAC地址改变也需要重新配置

	+ 基于协议类型：根据以太网帧第三个字段

		优点：允许用户移动，不需要额外的标签，减小网络流量

		缺点：检查每一帧的协议字段，消耗时间

	+ 基于IP子网地址：根据类型字段和源IP地址

	+ 基于高层应用或服务

+ 以太网帧格式：加入4字节的标识符，称为VLAN标记（IEEE 802.1Q/802.1P）

+ 交换机端口：

	+ Access端口：只属于一个VLAN；发送不代标签报文，主机相连
	+ Trunk端口：属于多个VLAN，发送带标签报文，路由器相连
	+ Hybrid端口：与Trunk类似，但允许不打标签，都可

## ARP

+ IP数据报交付到主机需要IP地址和物理地址

+ ARP构造映射表，由IP地址指向物理地址

+ 工作原理：

	+ 主机A广播发送ARP请求分组
	+ 主机B向A单播发送ARP响应分组

+ 高速缓存技术：

	+ ARP高速缓存区，用于存放最近使用的IP地址到物理地址的映射记录
	+ ARP给每项配备计时器，超时后将其删除
	+ 可以提高网络运行的效率

+ 命令：-a/g 显示缓存所有内容；-d删除某一项；-s绑定IP和MAC

+ 点对点链路不适用ARP

+ 特殊ARP：

	+ 代理ARP：跨网段ARP时，路由器将自己的物理地址返回，实现物理地址代理

	+ 开启代理ARP可能会导致地址冲突，产生不稳定的故障现象

	+ 代理ARP还可能会被ARP欺骗，填写错误的IP-MAC对应关系（伪冒网关、欺骗网关、欺骗终端用户、洪泛攻击），防御措施：网关防御、接入设备防御、客户端防御

	+ 免费ARP：查找自己的IP地址，用来获取网络接口的MAC地址；用于检测IP地址冲突；实现双机主备系统

		

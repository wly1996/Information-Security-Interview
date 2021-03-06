# 计算机网络

无论是计算机的哪一个岗位，计算机网络都是一个必考的问题。对于信息安全岗位来说，需要将计算机网络烂熟于心。一般常考的问题集中在：TCP/UDP，三次握手及四次挥手，HTTP与HTTPS。下面将对常见的问题进行一一解答。

## OSI七层模型

1984年，ISO发布了SO/IEC7498标准，定义了计算机网络的七层框架，也是目前业界公认的网络结构。

1. 物理层：物理层作为OSI中最低的一层，也是最基础的一层。物理层的作用是确保原始的数据可以在各种物理媒体上传输。通俗的来说，物理层可以理解为各种物理设备，如光纤，各种网线，网卡，集线器等。
2. 数据链路层：在两个网络实体之间提供数据链路连接的建立，维持和释放管理。构成数据链路数据的基本单元叫做帧（frame）。数据链路层的基本功能有链路管理，帧同步，流量控制，差错控制，透明传输和寻址。
   - 链路管理：当网络中的两个节点要进行通信时，数据的发送方必须确知接收方是否已处在接受的状态。为此通信双方必须要先交换一些必要的信息，以建立一条基本的数据链路。在传输数据时要维持数据链路，在通信完毕时要释放数据链路。
   - 帧同步：为了能实现数据有效的差错控制，在传输中发生差错后只将有错的数据进行重发。帧的数据结构必须设计成使接收方能够明确的从物理层收到的比特流中对其进行识别，也能从比特流中区分出帧的起始与终止。可以采用字节计数法、字符填充的首尾定界符法、使用比特填充的首尾标志法、违法编码法等。
   - 流量控制：为了确保数据通信的有序进行，还可以避免通信过程中不会因为接收方来不及接受而造成的数据丢失。对发送方的数据流量进行控制，使其发送速度不会超过接收方所能承受的能力。
   - 差错控制：在传输过程中可能会存在一些传输错误，为了确保通信的准确，又必须使得这些错误发生的机率尽可能低。通常引入计时器（TImer）来限定接收方发回反馈信息的时间间隔，当发送方发送一帧的时候同时也启动计数器，如果在限定时间内未收到回应，则判定传输错误，进行重传。为了防止发送多次，对每个帧进行编号，这样就可以通过编号来判定哪些帧传输错误。
   - 透明传输：当传输的数据中存在控制字符时，为了使接收方不会将其认为是控制信息，保证传输的数据比特流是任意的，透明的，即“传输透明性”。
   - 寻址：每一个网络设备都有一个独一无二的MAC地址，在数据链路层通过MAC地址来寻址。
3. 网络层：通过路由选择算法，为IP分组通过通信子网从原主机到目的主机选择一条最合适的传播路径。包括分组交换、路由、差错控制、流量控制、拥塞控制。其中包含的协议主要有IP协议，ARP协议，ICMP协议等。
   
   - IP协议：在IPv4地址中，使用32位长度的地址。IP地址=网络地址+主机地址。为了判断IP地址中的网络地址，IP协议还引入了子网掩码，IP地址和子网掩码通过按位与运算后可以得到网络地址。IP地址的分类如下：
   
     | 类型 |                组成方式                |            范围             |
     | :--: | :------------------------------------: | :-------------------------: |
     |  A   |  0 ｜ 网络号（7位） ｜ 主机号（24位）  |  1.0.0.0 - 127.255.255.255  |
     |  B   | 10 ｜ 网络号（14位） ｜ 主机号（16位） | 128.0.0.0 - 191.255.255.255 |
     |  C   | 110 ｜ 网络号（21位） ｜ 主机号（7位） | 192.0.0.0 - 223.255.255.255 |
     |  D   |        1110 ｜ 组播地址（28位）        | 224.0.0.0 - 239.255.255.255 |
     |  E   |    11110 ｜ 保留用于实验和将来使用     | 240.0.0.0 - 247.255.255.255 |
   
     IP报文的头部各字段意义按顺序如下：
   
     |        名称        |                             意义                             |
     | :----------------: | :----------------------------------------------------------: |
     |    版本（4位）     |                     该字段定义IP协议版本                     |
     |  头部长度（4位）   | 该字段定义数据包协议头长度，表示协议头部具有32位字长的数量。协议头最小值为5，最大值为15 |
     |    服务（8位）     | 该字段定义上层协议对处理当前数据报所期望的服务质量，并对数据报按照重要性级别进行分配。前3位成为优先位，后面4位成为服务类型，最后1位没有定义。这些8位字段用于分配优先级、延迟、吞吐量以及可靠性。 |
     |   总长度（16位）   | 该字段定义整个IP数据报的字节长度，包括协议头部和数据。其最大值为65535字节。以太网协议对能够封装在一个帧中的数据有最小值和最大值的限制（46~1500个字节） |
     |    标识（16位）    | 该字段包含一个整数，用于识别当前数据报。当数据报分段时，标识字段的值被复制到所有的分段之中。该字段由发送端分配帮助接收端集中数据报分段。 |
     |    标记（3位）     | 该字段由3位字段构成，其中最低位（MF）控制分段，存在下一个分段置为1，否则置0代表该分段是最后一个分段。中间位（DF）指出数据报是否可进行分段，如果为1则机器不能将该数据报进行分段。第三位即最高位保留不使用，值为0。 |
     |  分段偏移（13位）  | 该字段指出分段数据在源数据报中的相对位置，支持目标IP适当重建源数据。 |
     |  生存时间（8位）   | 该字段是一种计数器，在丢弃数据报的每个点值依次减1直至减少为0。这样确保数据报拥有有限的环路过程（即TTL），限制了数据报的寿命。 |
     |    协议（8位）     | 该字段指出在IP处理过程完成之后，有哪种上层协议接收导入数据报。这个字段的值对接收方的网络层了解数据属于哪个协议很有帮助 |
     | 头部校验和（16位） | 该字段帮助确保IP协议头的完整性。由于某些协议头字段的改变，这就需要对每个点重新计算和检验。计算过程是先将校验和字段置为0，然后将整个头部每16位划分为一部分，将个部分相加，再将计算结果取反码，插入到校验和字段中 |
     |   源地址（32位）   | 源主机IP地址，该字段在IPv4数据报从源主机到目的主机传输期间必须保持不变 |
     |  目的地址（32位）  | 目标主机IP地址，该字段在IPv4数据报从源主机到目的主机传输期间同样必须保持不变 |
   
     
##### 定义
* UDP（User DatagramProtocol）用户数据报协议

* 不可靠的、无连接的服务，传输效率高（发送前时延小），一对一、一对多、多对一、多对多、面向报文，尽最大努力服务，无拥塞控制。使用UDP的应用：域名系统 (DNS)；视频流；IP语音(VoIP)。

##### 特性
* ###### 无连接的服务
> UDP是一个非连接的协议，传输数据之前源端和终端不建立连接，双方没有专有的通信通道。当发送端想传送数据时就简单地把数据扔到网络上，并不能保证他们能到达目的地。接收端由于没有与发送端建立专用的通信通道，因此接收数据时并不能确定是有谁发来的数据。

* ###### 面向报文
> * 发送方的UDP对应用程序交下来的报文，在添加首部后就向下交付给IP层。既不拆分，也不合并，而是保留这些报文的边界。也就是说应用层交给UDP多长的报文，UDP就照样发送，即一次发送一个报文。
> * 不同于TCP有缓存机制。TCP是将发送的数据都看成字节流，根据字节流在缓冲区存储的大小来决定是否发送，一次发送的信息不一定是整个报文。
> * 使用UDP发送信息，应用程序必须选择合适大小的报文。若报文太长，则IP层需要分片，降低效率。若太短，会是IP太小。

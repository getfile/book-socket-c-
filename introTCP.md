##### 定义

* TCP（TransmissionControl Protocol）传输控制协议。
	是一种可靠的、面向连接的协议（eg:打电话）、传输效率低全双工通信（发送缓存&接收缓存）、面向字节流。使用TCP的应用：Web浏览器；电子邮件、文件传输程序。

##### 特性

* ######是面向连接的协议
	> 也就是说，在收发数据前，必须和对方建立可靠的连接。一个TCP连接必须要经过三次“对话”才能建立起来。接收双方独占一个通道。


* ######面向字节流
	> TCP将应用程序看成是一连串的无结构的字节流。每个TCP套接口有一个发送缓冲区，如果字节流太长时，TCP会将其拆分进行发送。
	当字节流太短时，TCP会等待缓冲区中的字节流达到一定程度时再构成报文发送出去，TCP发给对方的数据，对方在收到数据时必须给矛确认，
	只有在收到对方的确认时，本方TCP才会把TCP发送缓冲区中的数据删除。


* ######3次握手建立连接，4次握手释放连接
![](https://img-blog.csdn.net/20180512110847765)
	> ACK：TCP报头的控制位之一，表示确认号是否有效。只有当ACK=1时，确认号才有效，当ACK=0时，确认号无效，这时会要求重传数据，保证数据的完整性。

	> 确认号：用它来告诉发送端发送过来的序列号之前的数据段都收到了。比如，确认号为X，则表示前X-1个数据段都收到了。
	
	> SYN：同步序列号，TCP建立连接时将这个位置1。

	> FIN：发送端完成发送任务位，当TCP完成数据传输需要断开时，提出断开连接的一方将这位置1。
 


 #####TCP建立连接三次握手过程
![](https://img-blog.csdn.net/20180512111108972)
1. 主机A通过向主机B发送一个含有同步序列号的标志位的数据段给主机B ，向主机B 请求建立连接，通过这个数据段，主机A告诉主机B 两件事：我想要和你通信；你可以用哪个序列号作为起始数据段来回应我。
2. 主机B 收到主机A的请求后，用一个带有确认应答(ACK)和同步序列号(SYN)标志位的数据段响应主机A，也告诉主机A两件事：我已经收到你的请求了，你可以传输数据了；你要用哪个序列号作为起始数据段来回应我。
3. 主机A收到这个数据段后，再发送一个确认应答，确认已收到主机B 的数据段：我已收到回复，我现在要开始传输实际数据了。



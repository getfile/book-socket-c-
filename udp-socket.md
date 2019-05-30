#### 简介
Socket实现UDP的基本步骤如下：

##### （1）创建一个Socket对象

	Socket mySocket = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);
		AddressFamily 寻址类型
			AddressFamily.InterNetwork代表IPV4。
		SocketType 套接字类型
			SocketType.Dgram表示使用数据报协议。
		ProtocolType 协议类型
			ProtocolType.Udp表示使用UDP协议

但需要注意的是套接字类型与协议类型并不是可以随便组合。如下表

| SocketType | ProtocolType | 描述 |
|:-----|:-----|:-----|
| Stream（使用字节流） | Tcp | 面向连接 |
| Dgram（使用数据报）|Udp|面向无连接|
| Raw|Icmp|网际消息控制|
| Raw|Raw|基础传输协议|


##### （2）将创建的套接字对象与本地IPEndPoint进行绑定。

完成上述步骤后，那么创建的套接字就能够在本地IPEndPoint上接收流入的UDP数据包，或者将流出的UDP数据包通过本地IPEndPoint发送到网络中其他任意设备。

	mySocket.Bind(EndPointlocalEP)//绑定本地IP

	注意：
	1、绑定的目的主要是绑定端口，接收方指定在本机的哪个端口接收数据。
	2、并不是所有机器都需要bind：

（1）A机仅发送数据，B机仅用来接收A发送的数据。如下图所示：

![下图](https://img-blog.csdn.net/2018051211264373)
			
* 此时A不用绑定本机IP（也可以绑定），B必须绑定本机IP，否则报错。
* 因为A只需要发送，不需要接收，从而就不需要监听本机的特定端口来接收UDP数据报。B机必须要绑定本机IP。因为A在SendTo时会向指定的端口发送UDP数据报，B机需要监听本机相应端口，才能收到A发送过来的数据。

（2）B发消息A接收，A接收到消息后发送消息B接收。如下图所示：

![下图](https://img-blog.csdn.net/20180512112734396) 

* 此时A必须绑定，B不需要绑定（也可以绑定）。
* A需要在程序启动时，首先监听特定端口等待接收信息，因此需要绑定。
* B首先发送数据，不需要在指定的UDP端口等待流入的数据。B在发送时socket会随机指定B机的端口进行发送。
* A收到数据，同时会获取B端的发送IP与端口。即ReceiveFrom(byte[] data , ref EndPoint Remote)中的Remote就是B端socket发送数据时使用的IP与端口信息。
* A再发送数据时会用收到的B机的端口进行发送。即SendTo(byte[] data , EndPointRemote)，Remote就是上面ReceiveFrom方法获取的Remote。
* 此时B在接收时socket.ReceiveFrom监听的IP与端口就是发送数据时使用的IP与端口。

总结：
通常，接收方需要绑定端口，发送方发送数据时指定的端口（要发送到接收方的哪个端口）应与接收方监听的端口一致。
绑定的目的是为了监听本机的接收数据的端口。发送方发送数据的时候会明确告知往接收方的哪个端口发送。


##### （3）发送数据

* 使用UDP进行通信时，不需要连接。因为异地的主机之间没有建立连接，所以UDP不能使用标准的Send()和Receive()套接字方法，而是使用两个其他的方法：
		SendTo(byte[] data , EndPoint Remote)和ReceiveFrom(byte[] data , ref EndPoint Remote)。
		SendTo()方法指定要发送的数据，和目标机器的IPEndPoint。该方法有多种不同的使用方法，可以根据具体的应用进行选择，但是至少要指定数据包和目标机器。如下：
			SendTo(byte[] data , EndPointRemote)


* <1>广播发送：
	* 在发送前应对socket进行设置，如下
		socket.SetSocketOption(SocketOptionLevel.Socket,SocketOptionName.Broadcast,1);
	* 广播消息的目的IP地址是一种特殊IP地址，称为广播地址。
	* 广播地址由IP地址网络前缀加上全1主机后缀组成，如：192.168.1.255是192.169.1.0这个网络的广播地址;130.168.255.255是130.168.0.0这个网络的广播地址。向全部为1的IP地址（255.255.255.255）发送消息的话，那么理论上全世界所有的联网的计算机都能收得到了。但实际上不是这样的，一般路由器上设置抛弃这样的包，只在本地网内广播，所以效果和向本地网的广播地址发送消息是一样的。
	* 通常EndPoint  Remote = new IPEndPoint(IPAddress.Broadcast,9050)
	* 其中端口号要确定好。


* <2>一对一发送
	* 一对一发送无需设置socket
	* EndPoint Remote需要制定接收机的IP与接收端口。


##### （4）接收数据

* ReceiveFrom()方法同SendTo()方法类似，但是使用EndPoint对象声明的方式不一样。利用ref修饰。接收方接收到数据后Remote为发送方的IP与端口信息。
* 如果发送方没有绑定，则发送方在发送的时候，操作系统会随机指定一个端口，接收方的Remote就是发送方随机确定的IP与端口。
* ReceiveFrom()方法是一个阻塞方法，会一直等待，直到接收到数据才会往下执行程序。
		int ReceiveFrom(byte[] data ,ref EndPointRemote)
			返回的是接收到的byte[]长度。例如发送方发送了一个长度为1024的byte[]，接收方接收到了1024长度的byte[]，则返回1024。
			参数data的长度不能小于发送方发送的一包数据的长度。例如发送方发送了1024长度的byte[]。则data的长度应大于等于1024，否则由于长度不够会报错。
* UDP应用不是严格意义上的真正的服务器和客户机，而是平等的关系，即没有主与次的关系。
* 通常，我们都是将广播发送方当做服务端，广播接收方当做客户端。

#### 程序示例
##### 实现广播功能
	
服务端发送广播、客户端接收广播。

* 服务端代码：

```cpp
static void Main(string[] args)
{
	sock = newSocket(AddressFamily.InterNetwork, SocketType.Dgram,ProtocolType.Udp);
	// 如果是广播通常这样设定发送地址，一定要与接收端绑定的端口一致！
    iep1 = newIPEndPoint(IPAddress.Broadcast, 9050);
    stringhostname = Dns.GetHostName();
	data =Encoding.ASCII.GetBytes(hostname);
	// 设置socket，否则程序报错
	sock.SetSocketOption(SocketOptionLevel.Socket,SocketOptionName.Broadcast,1);
    Thread t = newThread(BroadcastMessage);
    t.Start();
	Console.ReadKey();          
}

private static void BroadcastMessage()
{
    while (true)
	{
      //向iep1发送广播数据
      sock.SendTo(data, iep1);
      Thread.Sleep(2000);
	}         
}
```

* 客户端代码：

```cpp
Socket sock = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);

//获取本机所有IP进行监听
//IPAddress.Any表示本机ip，换言之，如果服务器绑定此地址，则表示侦听//本机所有ip对应的那个端口（本机可能有多个ip或只有一个ip）
IPEndPoint iep =new IPEndPoint(IPAddress.Any, 9050);

//绑定本机端口，用于接收UDP数据包
sock.Bind(iep);

//iep可以是任意IP,因为有多网卡情况的存在，在ReceiveFrom后会被赋值为发送端的地址
EndPoint ep = (EndPoint)iep;
Console.WriteLine("Ready to receive…");
byte[] data = new byte[1024];

//是一个阻塞方法，会一直等待数据，直到接收到数据才会往下执行
//返回的地址ep是路由器地址，因为是由路由器转发的信息。
int recv = sock.ReceiveFrom(data, ref ep);
string stringData = Encoding.ASCII.GetString(data, 0, recv);           
Console.WriteLine("received: {0} from: {1}",stringData,ep.ToString());
sock.Close();          
Console.ReadKey();
```

##### 实现一对一通信

* 服务器端代码：

```cpp
int recv;
byte[] data = new byte[1024];

//得到本机IP，设置TCP端口号        
IPEndPoint ip = new IPEndPoint(IPAddress.Any, 8001);
Socket newsock = new Socket(AddressFamily.InterNetwork,SocketType.Dgram, ProtocolType.Udp);

//绑定网络地址，这里之所以绑定，是因为要侦听客户端发送来的数据
newsock.Bind(ip);
Console.WriteLine("This is a Server, host name is{0}", Dns.GetHostName());

//等待客户机连接
Console.WriteLine("Waiting for a client");

//得到客户机IP
EndPoint Remote = new IPEndPoint(IPAddress.Any, 0);

//获得的Remote是客户端地址
recv = newsock.ReceiveFrom(data, ref Remote);
Console.WriteLine("Message received from {0}:", Remote.ToString());
Console.WriteLine(Encoding.ASCII.GetString(data, 0,recv));

//客户机连接成功后，发送信息
string welcome = "你好 ! ";

//字符串与字节数组相互转换
data = Encoding.ASCII.GetBytes(welcome);

//发送信息
newsock.SendTo(data, data.Length, SocketFlags.None,Remote);
newsock.close();
```

* 客户端代码：

```cpp
byte[] data = new byte[1024];
string input, stringData;
Console.WriteLine("This is a Client, host name is{0}", Dns.GetHostName());

//设置为服务端IP与端口
IPEndPoint ip = new IPEndPoint(IPAddress.Parse("127.0.0.1"),8001);

//定义网络类型，数据连接类型和网络协议UDP
Socket server = new Socket(AddressFamily.InterNetwork,SocketType.Dgram, ProtocolType.Udp);
string welcome = "你好! ";
data = Encoding.UTF8.GetBytes(welcome);

//向服务端发送数据，此时客户端通过本机哪个IP与端口发送数据是随机的，
//该信息被存放在socket中。
//服务端接收到信息会得到客户端的发送地址，服务端利用这个地址向客户端
//发送消息，客户端直接利用这个socket就可以接收数据，从而无需bind。
server.SendTo(data, data.Length, SocketFlags.None, ip);
EndPoint Remote = new IPEndPoint(IPAddress.Any, 0);
data = new byte[1024];

//获得的Remote是服务端地址
int recv = server.ReceiveFrom(data, ref Remote);
Console.WriteLine("Message received from {0}:", Remote.ToString());
Console.WriteLine(Encoding.UTF8.GetString(data, 0,recv));
Console.WriteLine("Stopping Client.");
server.Close();
```

*  * 注意：该例子是服务端接收到客户端的信息后，直接利用receiveFrom获取的客户端IP向客户端发送消息。因此，客户端利用发送时的IP与端口进行信息的接收，无需bind。

*  * 如果服务端与客户端指定了各自的接收端口，那么双方都需要绑定各自的端口进行监听。举例：

		服务端(IP：172.1.1.1监听端口：8001)
			IPEndPoint Local = new IPEndPoint(IPAddress.Any, 8001);
			IPEndPoint Remote = new IPEndPoint(IPAddress. Parse("127.1.1.2"),8002);
			socket.Bind(Local);
			//向客户端绑定的地址发送信息
			socket.SendTo(data,Remote);

		客户端(IP：172.1.1.2 监听端口：8002)
			IPEndPoint Local = new IPEndPoint(IPAddress.Any, 8001);
			IPEndPoint Remote = new IPEndPoint(IPAddress. Parse("127.1.1.2"),8002);
			socket.Bind(Local);
			//向服务端绑定的地址发送信息
			socket.SendTo(data,Remote);
	

*  * 一旦socket绑定了IP与端口，那么会通过该端口收发数据。

*  * 如果想发送与接收绑定不同的地址，则需要在一个程序中声明两个socket，绑定不同的地址，一个socket用于收，一个socket用于发。

#####  实现一对多通信（组播）

* IP组播通信需要一个特殊的组播地址，IP组播地址是一组D类IP地址，范围从224.0.0.0 到239.255.255.255。其中还有很多地址是为特殊的目的保留的。224.0.0.0到224.0.0.255的地址最好不要用，因为他们大多是为了特殊的目的保持的（比如IGMP协议）。

* IGMP是IP组播的基础。在IP协议出现以后为了加入对组播的支持，IGMP产生了。IGMP所做的实际上就是告诉路由器，在这个路由器所在的子网内有人对发送到某一个组播组的数据感兴趣，这样当这个组播组的数据到达后面，路由器就不会抛弃它，而是把他转送给所有感兴趣的客户。假如不同子网内的A和B要进行组播通信，那么位于AB之间的所有路由器必须都要支持IGMP协议，否则AB之间不能进行通信。 

* 使用Socket实现任意源组播
		利用C#实现UDP组播的基本步骤为：
		（1）建立socket；
		（2）socket和端口绑定；
		（3）加入一个组播组；
		（4）通过sendto / recvfrom进行数据的收发；
		（5）关闭socket。


* 下面是简单的示例：

*	* 发送示例：

```cpp
byte[] data = new byte[1024];
string hostname = Dns.GetHostName();       
string welcome = hostname+ ":你好 ! ";
data = Encoding.UTF8.GetBytes(welcome);

//声明组播组IP
IPAddress ip = IPAddress.Parse("226.1.1.2");
Socket s = new Socket(AddressFamily.InterNetwork,SocketType.Dgram, ProtocolType.Udp);

//对socket进行设置
s.SetSocketOption(SocketOptionLevel.IP,SocketOptionName.MulticastTimeToLive, 1);

//确定组播地址，注意端口号要与接收端统一
IPEndPoint ipep = new IPEndPoint(ip, 5000);
while (true)
{
	Thread.Sleep(2000);
	s.SendTo(data,data.Length, SocketFlags.None, ipep);
}
s.Close();
```

*  * 接收示例：

```cpp
byte[] data = new byte[1024];
int recv = 0;
Socket s = new Socket(AddressFamily.InterNetwork,SocketType.Dgram, ProtocolType.Udp);

//绑定本机地址，注意端口号要与发送端多播端口号一致
IPEndPoint ipep = new IPEndPoint(IPAddress.Any, 5000);
s.Bind(ipep);

//加入到多播组
s.SetSocketOption(SocketOptionLevel.IP,SocketOptionName.AddMembership,
            newMulticastOption(IPAddress.Parse("226.1.1.2"), IPAddress.Any));

//定义任意地址用于返回发送方的地址
IPEndPoint sender = new IPEndPoint(IPAddress.Any, 0);
EndPoint Remote = (EndPoint)sender;
while (true)
{
	//放回的是路由器的地址，而不是发送机器的地址，这是因为是由路由器转发的
	//信息
	recv =s.ReceiveFrom(data, ref Remote);
	Console.WriteLine(Encoding.UTF8.GetString(data, 0, recv));
}

s.Close();
Console.ReadKey();
```

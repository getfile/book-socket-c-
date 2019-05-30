
三、Socket编程
---
# 1、 UDP通信

## 1.1 采用Socket实现UDP
### 1.1.1简介
Socket实现UDP的基本步骤如下：

#### （1）创建一个Socket对象

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


#### （2）将创建的套接字对象与本地IPEndPoint进行绑定。

完成上述步骤后，那么创建的套接字就能够在本地IPEndPoint上接收流入的UDP数据包，或者将流出的UDP数据包通过本地IPEndPoint发送到网络中其他任意设备。

	mySocket.Bind(EndPointlocalEP)//绑定本地IP

	注意：
	1、绑定的目的主要是绑定端口，接收方指定在本机的哪个端口接收数据。
	2、并不是所有机器都需要bind：

（1）A机仅发送数据，B机仅用来接收A发送的数据。如下图所示：

![下图](https://img-blog.csdn.net/2018051211264373)
			
此时A不用绑定本机IP（也可以绑定），B必须绑定本机IP，否则报错。
因为A只需要发送，不需要接收，从而就不需要监听本机的特定端口来接收UDP数据报。B机必须要绑定本机IP。因为A在SendTo时会向指定的端口发送UDP数据报，B机需要监听本机相应端口，才能收到A发送过来的数据。

（2）B发消息A接收，A接收到消息后发送消息B接收。如下图所示：

![下图](https://img-blog.csdn.net/20180512112734396) 

此时A必须绑定，B不需要绑定（也可以绑定）。
A需要在程序启动时，首先监听特定端口等待接收信息，因此需要绑定。
B首先发送数据，不需要在指定的UDP端口等待流入的数据。B在发送时socket会随机指定B机的端口进行发送。
A收到数据，同时会获取B端的发送IP与端口。即ReceiveFrom(byte[] data , ref EndPoint Remote)中的Remote就是B端socket发送数据时使用的IP与端口信息。
A再发送数据时会用收到的B机的端口进行发送。即SendTo(byte[] data , EndPointRemote)，Remote就是上面ReceiveFrom方法获取的Remote。
此时B在接收时socket.ReceiveFrom监听的IP与端口就是发送数据时使用的IP与端口。

总结：
通常，接收方需要绑定端口，发送方发送数据时指定的端口（要发送到接收方的哪个端口）应与接收方监听的端口一致。
绑定的目的是为了监听本机的接收数据的端口。发送方发送数据的时候会明确告知往接收方的哪个端口发送。


#### （3）发送数据

	使用UDP进行通信时，不需要连接。因为异地的主机之间没有建立连接，所以UDP不能使用标准的Send()和Receive()套接字方法，而是使用两个其他的方法：SendTo(byte[] data , EndPoint Remote)和ReceiveFrom(byte[] data , ref EndPoint Remote)。

		SendTo()方法指定要发送的数据，和目标机器的IPEndPoint。该方法有多种不同的使用方法，可以根据具体的应用进行选择，但是至少要指定数据包和目标机器。如下：

	SendTo(byte[] data , EndPointRemote)

	<1>广播发送：

	在发送前应对socket进行设置，如下

	socket.SetSocketOption(SocketOptionLevel.Socket,SocketOptionName.Broadcast,1);

	广播消息的目的IP地址是一种特殊IP地址，称为广播地址。

	广播地址由IP地址网络前缀加上全1主机后缀组成，如：192.168.1.255是192.169.1.0这个网络的广播地址;130.168.255.255是130.168.0.0这个网络的广播地址。向全部为1的IP地址（255.255.255.255）发送消息的话，那么理论上全世界所有的联网的计算机都能收得到了。但实际上不是这样的，一般路由器上设置抛弃这样的包，只在本地网内广播，所以效果和向本地网的广播地址发送消息是一样的。

	通常EndPoint  Remote = new IPEndPoint(IPAddress.Broadcast,9050)

	其中端口号要确定好。

	<2>一对一发送

	一对一发送无需设置socket

	EndPoint Remote需要制定接收机的IP与接收端口。

#### （4）接收数据

	ReceiveFrom()方法同SendTo()方法类似，但是使用EndPoint对象声明的方式不一样。利用ref修饰。接收方接收到数据后Remote为发送方的IP与端口信息。

	如果发送方没有绑定，则发送方在发送的时候，操作系统会随机指定一个端口，接收方的Remote就是发送方随机确定的IP与端口。

	ReceiveFrom()方法是一个阻塞方法，会一直等待，直到接收到数据才会往下执行程序。

	int ReceiveFrom(byte[] data ,ref EndPointRemote)

	int返回参数返回的是接收到的byte[]长度。例如发送方发送了一个长度为1024的byte[]，接收方接收到了1024长度的byte[]，则返回1024。

	参数data的长度不能小于发送方发送的一包数据的长度。例如发送方发送了1024长度的byte[]。则data的长度应大于等于1024，否则由于长度不够会报错。

	UDP应用不是严格意义上的真正的服务器和客户机，而是平等的关系，即没有主与次的关系。

	通常，我们都是将广播发送方当做服务端，广播接收方当做客户端。

### 1.1.2 程序示例
#### 1.1.2.1 实现广播功能
	服务端发送广播、客户端接收广播。

服务端代码：
static void Main(string[] args)
{
   sock = newSocket(AddressFamily.InterNetwork, SocketType.Dgram,ProtocolType.Udp);

//如果是广播通常这样设定发送地址，一定要与接收端绑定的端口一致！

    iep1 = newIPEndPoint(IPAddress.Broadcast, 9050);

    stringhostname = Dns.GetHostName();

   data =Encoding.ASCII.GetBytes(hostname);

//设置socket，否则程序报错

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

 

客户端代码：

Socket sock = new Socket(AddressFamily.InterNetwork,

             SocketType.Dgram, ProtocolType.Udp);

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

#### 1.1.2.2 实现一对一通信
服务器端代码：

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

 

客户端代码：

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

 

注意：该例子是服务端接收到客户端的信息后，直接利用receiveFrom获取的客户端IP向客户端发送消息。因此，客户端利用发送时的IP与端口进行信息的接收，无需bind。

 如果服务端与客户端指定了各自的接收端口，那么双方都需要绑定各自的端口进行监听。举例：

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

 

一旦socket绑定了IP与端口，那么会通过该端口收发数据。

如果想发送与接收绑定不同的地址，则需要在一个程序中声明两个socket，绑定不同的地址，一个socket用于收，一个socket用于发。

#### 1.1.2.3 实现一对多通信（组播）
IP组播通信需要一个特殊的组播地址，IP组播地址是一组D类IP地址，范围从224.0.0.0 到239.255.255.255。其中还有很多地址是为特殊的目的保留的。224.0.0.0到224.0.0.255的地址最好不要用，因为他们大多是为了特殊的目的保持的（比如IGMP协议）。

 IGMP是IP组播的基础。在IP协议出现以后为了加入对组播的支持，IGMP产生了。IGMP所做的实际上就是告诉路由器，在这个路由器所在的子网内有人对发送到某一个组播组的数据感兴趣，这样当这个组播组的数据到达后面，路由器就不会抛弃它，而是把他转送给所有感兴趣的客户。假如不同子网内的A和B要进行组播通信，那么位于AB之间的所有路由器必须都要支持IGMP协议，否则AB之间不能进行通信。 

1 使用Socket实现任意源组播

 利用C#实现UDP组播的基本步骤为：

 （1）建立socket；

 （2）socket和端口绑定；

 （3）加入一个组播组；

 （4）通过sendto / recvfrom进行数据的收发；

 （5）关闭socket。

 下面是简单的示例：

（1）   发送示例：

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

 

 

 

（2）接收示例：

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

 

## 1.2 采用UdpClient实现UDP
1.2.1简介
（1）创建对象

绑定指定的端口进行数据的接收与发送：

UdpClient  udpClient= new UdpClient(11000);

使用随机的端口进行数据的接收与发送：

UdpClient  udpClient= new UdpClient(0);

使用随机端口进行发送，每次发出去的信息端口号码都是不同的，这样接收方要把信息发回来，就没办法了。

绑定指定终结点，进行数据的发送：

UdpClient  udpClient= new UdpClient(IPEndPoint);

 

（2）发送数据

发送的时候创建一个结点，来指定接收方的地址

Send(byte[],Int32,IpEndPoint)

Send(byte[],Int32,String,Int32)

Send(byte[],Int32)

在使用第三个send方法时由于没有指定远端地址，需要在之前使用Connect方法来指定远端地址。

 

 

（3）接收信息

byte[] Receive(IpEndPoint)

该方法是一个阻塞方法，如果没有收到数据，会一直在该方法上等待。

1.2.2程序示例
1.1.2.1实现广播功能
发送端：

其实UDP广播就是向255.255.255.255发送数据，接收端只需绑定UDP广播的端口号即可。发送端，发送的地址，255.255.255.255：Port，即，IPAddress.Broadcast：Port。

 

UdpClient UDPsend = new UdpClient(newIPEndPoint(IPAddress.Any, 0));

//其实IPAddress.Broadcast 就是 255.255.255.255

IPEndPoint endpoint = newIPEndPoint(IPAddress.Broadcast, 8080);

byte[] buf = Encoding.Default.GetBytes("This is UDPbroadcast");

UDPsend.Send(buf, buf.Length, endpoint);

 

接收端：

接收端仅需绑定发送端发送时指定的端口，便可以接收广播信息

IPEndPoint LocalIp = new IPEndPoint(IPAddress.Any, 8080);

//任意地址，用于Receive中的ref参数，返回发送端的地址。

IPEndPoint Remote = new IPEndPoint(IPAddress.Any, 0);

UdpClient udpClient = new UdpClient(LocalIp);

//Receive方法返回byte数组。

byte[] d = udpClient.Receive(ref Remote);

string s = Encoding.UTF8.GetString(d, 0, d.Length);

 

 

1.1.2.2实现一对一通信
发送端：

//定义接收端的地址

IPEndPoint targetIp = newIPEndPoint(IPAddress.Parse("192.168.2.119"), 8001);

UdpClient udpClient= new UdpClient(0);

byte[] data = Encoding.UTF8.GetBytes(“Hello”);

//发送信息，第二个参数为发送的字节数

udpClient.Send(data, data.Length, targetIp);

 

接收端：

//定义本地地址，用于不断监听该地址收到的信息。该地址与发送端的发送地址//一致。

IPEndPoint LocalIp = new IPEndPoint(IPAddress.Any,8001);

//任意地址，用于Receive中的ref参数，返回发送端的地址。

IPEndPoint Remote = new IPEndPoint(IPAddress.Any, 0);

UdpClient udpClient = new UdpClient(LocalIp);

//Receive方法返回byte数组。

byte[] d = udpClient.Receive(ref Remote);

string s = Encoding.UTF8.GetString(d, 0, d.Length);

 

1.1.2.3实现一对多通信（组播）
IP组播通信需要一个特殊的组播地址，IP组播地址是一组D类IP地址，范围从224.0.0.0 到239.255.255.255。其中还有很多地址是为特殊的目的保留的。224.0.0.0到224.0.0.255的地址最好不要用，因为他们大多是为了特殊的目的保持的（比如IGMP协议）。

因此如果设定的IP不符合D类IP规则，则程序报错。

发送端：

//定义组播地址，注意一定要是D类广播

IPEndPoint targetIp = new IPEndPoint(IPAddress.Parse("226.1.1.2"),8001);

UdpClient udpClient= new UdpClient(0);

byte[] data = Encoding.UTF8.GetBytes(“Hello”);

//发送信息，第二个参数为发送的字节数

udpClient.Send(data, data.Length, targetIp);

 

接收端：

//定义本地地址，注意端口一定与发送端发送时指定的端口一致

IPEndPoint LocalIp = new IPEndPoint(IPAddress.Any,8001);

//任意地址，用于Receive中的ref参数，返回发送端的地址。

IPEndPoint Remote = new IPEndPoint(IPAddress.Any, 0);

UdpClient udpClient = new UdpClient(LocalIp);

//加入组播，与发送端定义的组播地址一致，必须符合D类IP地址，否则报错

udpClient.JoinMulticastGroup(IPAddress.Parse("226.1.1.2"));

//Receive方法返回byte数组。

byte[] d = udpClient.Receive(ref Remote);

string s = Encoding.UTF8.GetString(d, 0, d.Length);

//离开该多播组

udpClient.DropMulticastGroup(IPAddress.Parse("226.1.1.2"));

# 2、 TCP通信

## 2.1 采用Socket实现TCP

### 2.1.1 同步通信
#### 2.1.1.1 简介
服务端：

	服务端的主要职责是处理各个客户端发送来的数据，因此在客户端的Socket编程中需要使用两个线程来循环处理客户端的请求，一个线程用于监听客户端的连接情况，一个线程用于监听客户端的消息发送。

	基本流程
	•	创建套接字
	•	绑定套接字的IP和端口号——Bind(EndPoint)
	•	将套接字处于监听状态等待客户端的连接请求——Listen(int) 监听端口，其中parameters表示最大监听数。
	•	阻塞方法，当请求到来后，接受请求并返回本次会话的套接字——Accept()接受客户端链接，并返回一个新的链接，用于处理同客户端的通信问题。
	•	使用返回的套接字和客户端通信——Send()/Receive()
			Send(byte[]) 简单发送数据。
			Send(byte[],SocketFlag)   使用指定的SocketFlag发送数据。
			Send(byte[],int, SocketFlag)   使用指定的SocketFlag发送指定长度数据。
			Send(byte[], int, int,SocketFlag)  使用指定的SocketFlag，将指定字节数的数据发送到已连接的socket(从指定偏移量开始)。

			Receive(byte[])  简单接受数据。
			Receive(byte[],SocketFlag)    使用指定的SocketFlag接受数据。
			Receive(byte[], int, SocketFlag)    使用指定的SocketFlag接受指定长度数据。
			Receive (byte[], int,int, SocketFlag)    使用指定的SocketFlag，从绑定的套接字接收指定字节数的数据，并存到指定偏移量位置的缓冲区。
	•	返回，再次等待新的连接请求
	•	关闭套接字

客户端：

	客户端相对于服务端来说任务要轻许多，因为客户端仅仅需要和服务端通信即可，可是因为在和服务器通信的过程中，需要时刻保持连接通畅，因此同样需要两个线程来分别处理连接情况的监听和消息发送的监听。

	基本流程
	•	创建套接字保证与服务器的端口一致
	•	向服务器发出连接请求——Connect(EndPoint)连接远程服务器，如果没有相应的远程等待连接，直接返回错误。
	•	和服务器端进行通信——Send()/Receive()
	•	关闭套接字

#### 2.1.1.2 程序示例
##### 2.1.1.2.1 简单的单次通信
服务端接收客户端信息并向客户端返回信息

服务端：
```
static void Main()
{
	try{
		IPEndPoint ipe = new IPEndPoint(IPAddress.Any,8001);
		Socket s = newSocket(AddressFamily.InterNetwork, SocketType.Stream,ProtocolType.Tcp);//创建一个Socket类
		s.Bind(ipe);//绑定
		s.Listen(0);//开始监听

		Socket temp = s.Accept();//等待客户端连接
		byte[] recvBytes = new byte[1024];
		int bytes = temp.Receive(recvBytes);//从客户端接受信息，返回接收的数据长度
		string recvStr = Encoding.UTF8.GetString(recvBytes,0, bytes);
		Console.WriteLine("Server GetMessage:{0}", recvStr);
		string sendStr = "Ok!Client SendMessage Sucessful!";
		byte[] bs = Encoding.UTF8.GetBytes(sendStr);
		temp.Send(bs);//向客户端发送数据
		temp.Close();
		s.Close();
	}
	catch (ArgumentNullException e){
		Console.WriteLine("ArgumentNullException:{0}", e);}
	catch (SocketException e){
		Console.WriteLine("SocketException: {0}", e);
	}

	Console.WriteLine("Press Enter to Exit");
	Console.ReadLine();
}
```

客户端：
```
static void Main()
{
	try{
		IPEndPoint ipe =new IPEndPoint IPAddress.Parse(“192.168.2.119”),8001);//服务器地址
		Socket c = new Socket(AddressFamily.InterNetwork,SocketType.Stream,ProtocolType.Tcp);//创建一个Socket
		c.Connect(ipe);//连接到服务器
		byte[] bs =Encoding.UTF8.GetBytes("hello!This is a socket test");
		c.Send(bs);//向服务端发送信息
		byte[] recvBytes= new byte[1024];
		int bytes= c.Receive(recvBytes);//从服务器端接受返回信息
		string recvStr +=Encoding.UTF8.GetString(recvBytes, 0, bytes);
		Console.WriteLine("Client Get Message:{0}", recvStr);//显示服务器返回信息
		c.Close();
	}
	catch(ArgumentNullException e){
		Console.WriteLine("ArgumentNullException:{0}", e);
	}
	catch(SocketException e){
		Console.WriteLine("SocketException:{0}", e);
	}

	Console.WriteLine("Press Enter to Exit");
	Console.ReadLine();
}
``` 

##### 2.1.1.2.1 多次通信
```
public class TCPServer 
{ 
	private byte[]result = new byte[1024]; 
	// 最大的监听数量 
	privateint maxClientCount;   
	// IP地址 
	privatestring ip; 
	// 端口号 
	privateint port; 
	// 客户端列表 
	privateList<Socket> ClientSockets;   
	// IP终端 
	privateIPEndPoint ipEndPoint; 
	// 服务端Socket 
	privateSocket ServerSocket; 
	// 当前客户端Socket 
	privateSocket ClientSocket; 

	public TCPServer(int port, int count) 
	{ 
		this.ip =IPAddress.Any.ToString(); 
		this.port = port; 
		this.maxClientCount=count; 
		this.ClientSockets = newList<Socket>();
		//初始化IP终端 
		this.ipEndPoint = newIPEndPoint(IPAddress.Any, port); 
		//初始化服务端Socket  
		this.ServerSocket = newSocket(AddressFamily.InterNetwork,SocketType.Stream,ProtocolType.Tcp); 
		//端口绑定 
		this.mServerSocket.Bind(this.ipEndPoint); 
		//设置监听数目 
		this.ServerSocket.Listen(maxClientCount); 
	} 

	// 定义一个Start方法将构造函数中的方法分离出来 
	public void Start() 
	{ 
		//创建服务端线程，实现客户端连接请求的循环监听 
		var ServerThread = new Thread(this.ListenClientConnect); 
		//服务端线程开启 
		ServerThread.Start(); 
	} 

	// 监听客户端链接 
	private void ListenClientConnect() 
	{ 
		//设置循环标志位 
		while(true) 
		{ 
			//获取连接到服务端的客户端 
			this.ClientSocket = this.ServerSocket.Accept(); 
			//将获取到的客户端添加到客户端列表 
			this.ClientSockets.Add(this.ClientSocket); 
			//向客户端发送一条消息 
			this.SendMessage(string.Format("客户端{0}已成功连接到服务器",this.ClientSocket.RemoteEndPoint)); 
			//创建客户端消息线程，实现客户端消息的循环监听 
			var ReveiveThread = newThread(this.ReceiveClient); 
			//注意到ReceiveClient方法传入了一个参数 
			//实际上这个参数就是此时连接到服务器的客户端 
			//即ClientSocket 
			ReveiveThread.Start(this.ClientSocket); 
		} 
	} 

	// 接收客户端消息的方法 
	private void ReceiveClient(objectobj) 
	{ 
		Socket_ClientSocket = (Socket)obj; 
		while(true) 
		{ 
			try 
			{ 
				//获取数据长度 
				int receiveLength = _ClientSocket.Receive(result); 
				//获取客户端消息 
				string clientMessage =Encoding.UTF8.GetString(result, 0, receiveLength); 
				//服务端负责将客户端的消息分发给各个客户端 
				this.SendMessage(string.Format("客户端{0}发来消息:{1}",_ClientSocket.RemoteEndPoint,clientMessage));
			} 
			catch (Exception e) 
			{ 
				//从客户端列表中移除该客户端 
				this.ClientSockets.Remove(_ClientSocket); 
				//向其它客户端告知该客户端下线 
				this.SendMessage(string.Format("服务器发来消息:客户端{0}从服务器断开,断开原因{1}",_ClientSocket.RemoteEndPoint,e.Message)); 
				//断开连接 
				_ClientSocket.Shutdown(SocketShutdown.Both); 
				_ClientSocket.Close(); 
				break; 
			} 
		}   
	} 
 
	// 向所有的客户端群发消息 
	public void SendMessage(string msg) 
	{ 
		//确保消息非空以及客户端列表非空 
		if (msg == string.Empty || this.ClientSockets.Count<= 0) return; 
		//向每一个客户端发送消息 
		foreach (Socket s in this.ClientSockets) 
		{ 
			s.Send(Encoding.UTF8.GetBytes(msg)); 
		} 
	} 

	// 向指定的客户端发送消息 
	public void SendMessage(string ip,intport,string msg) 
	{ 
		//构造出一个终端地址 
		IPEndPoint_IPEndPoint = new IPEndPoint(IPAddress.Parse(ip), port); 
		//遍历所有客户端 
		foreach (Socket s in ClientSockets) 
		{ 
			if (_IPEndPoint ==(IPEndPoint)s.RemoteEndPoint) 
			{ 
				s.Send(Encoding.UTF8.GetBytes(msg)); 
			} 
		} 
	} 
} 
```

### 2.1.2 异步通信
#### 2.1.1.1 简介
（1）BeginAccept(AsynscCallBack,object)
	开始一个异步的接受一个连接尝试。
	object：通常传入的是服务Socket对象实例。
	AsynscCallBack：回调函数，该回调函数中必须包含EndAccept方法。

	应用程序调用BegineAccept方法后，系统会使用单独的线程执行指定的回调方法并在EndAccept上一直处于阻塞状态，直至监测到挂起的链接。EndAccept会返回新的socket对象。供你来同远程主机数据交互。调用BeginAccept当希望原始线程阻塞的时候，请调用WaitHandle.WaitOne方法。当需要原始线程继续执行时请在回调方法中使用ManualResetEvent的set方法。

（2）Socket EndAccept（IAsyncResult）
	重点是返回一个Socket与客户端进行通信。IAsyncResult作用不大。

（3）BeginConnect（EndPoint,AsyncCallBack, Object）
	EndPoint：指定想要连接的服务端地址。
	AsynscCallBack：回调函数，该回调函数中必须包含EndConnect方法。
	object：通常传入的是客户端Socket对象实例。

（4）void EndConnect（IAsyncResult）
	无返回值，在为连接成功前一直在该方法上等待。

（5）BeginReceive(byte[] buffer,int offset, int size, SocketFlags, AsyncCallbac, object)
	buffer:用于存放接收信息。
	offset：指定buffer的其实存储位置。
	size：指定接收的信息byte长度。
	SocketFlags：socket标志位。
	AsynscCallBack：回调函数，该回调函数中必须包含EndReceive方法。
	object：通常是一个存放了第一个参数buffer以及与远端进行通信的Socket对象的数据结构的对象实例。

（6）int  EndReceieve（IAsyncResult）
	返回收到的信息的byte数组长度。IAsyncResult作用不大。
	
（7）BeginSend(byte[],SocketFlag, AsyncCallBack, Object)
	byte[]：需要发送的数据。
	AsyncCallBack：回调函数，该回调函数中必须包含EndSend方法。
	object：通常为与远端进行通信的Socket实例。

（8）int EndSend（IAsyncResult）
	返回发送的信息的byte数组长度。IAsyncResult作用不大。

 
**服务端**：

	基本流程
	•     创建套接字
	•     绑定套接字的IP和端口号——Bind()
	•     使套接字处于监听状态等待客户端的连接请求——Listen()
	•     当请求到来后，使用BeginAccept()和EndAccept()方法接受请求，返回新的套接字
	•     使用BeginSend()/EndSend和BeginReceive()/EndReceive()两组方法与客户端进行收发通信
	•     关闭套接字

**客户端**：

	基本流程
	•     创建套接字并保证与服务器的端口一致
	•     使用BeginConnect()和EndConnect()这组方法向服务端发送连接请求
	•     使用BeginSend()/EndSend和BeginReceive()/EndReceive()两组方法与服务端进行收发通信
	•     关闭套接字


#### 2.1.1.2 程序示例
##### 2.1.1.2.1 服务端仅用来不断接收客户端信息
```
classStateObject
{
	public Socket workSocket = null;
	public const int BufferSize = 1024;
	public byte[] buffer = new byte[BufferSize];
}

classAsyncServer
{
	private IPEndPoint ipEndPoint;
	private Socket serverSocket;
	private Socket clientSocket;
	private List<Socket> clients;
	private ManualResetEvent AcceptDone;

	public AsyncServer()
	{
		clients = new List<Socket>();
		ipEndPoint = new IPEndPoint(IPAddress.Any,8001);
		serverSocket = newSocket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
		serverSocket.Bind(ipEndPoint);
		serverSocket.Listen(10);
		AcceptDone = new ManualResetEvent(false);
	}

	//启动函数
	public void Start()
	{
		Thread startT = new Thread(ListenThread);
		startT.Start();
	}

	public void ListenThread()
	{
		while (true)
		{
			AcceptDone.Reset();
			serverSocket.BeginAccept(newAsyncCallback(AcceptCallback), serverSocket);
			AcceptDone.WaitOne();
		}
	}

	public void AcceptCallback(IAsyncResult ar)
	{
		Socket temp =((Socket)ar.AsyncState).EndAccept(ar);
		StateObject state = new StateObject();
		state.workSocket = temp;
		temp.BeginReceive(state.buffer, 0, StateObject.BufferSize,0, new AsyncCallback(ReceiveCallback), state);
		AcceptDone.Set();
	}

	public void ReceiveCallback(IAsyncResult ar)
	{
		StateObject state =(StateObject)ar.AsyncState;
		Socket handler = state.workSocket;
		int i = handler.EndReceive(ar);
		if (i > 0)
		{
			string message =Encoding.UTF8.GetString(state.buffer, 0, i);
			Console.WriteLine(message);
		}
	}
}
```

##### 2.1.1.2.2 服务端与客户端双向通信
（该代码引用自他人）

异步通信：

客户端Client：

预定义结构体，用于异步委托之间的传递。用户根据自己需要定制即可

public class StateObject

{

    public Socket workSocket= null;

    public const int BufferSize= 256;

    public byte[] buffer= new byte[BufferSize];

    public StringBuilder sb = new StringBuilder();

}

正文：

public class AsynchronousClient

{

    private const int port =11000;

    private static ManualResetEvent connectDone= new ManualResetEvent(false);

    private static ManualResetEvent sendDone= new ManualResetEvent(false);

    private static ManualResetEvent receiveDone= new ManualResetEvent(false);

    private static String response= String.Empty;

 

    private static void StartClient(){

      try{

            IPHostEntry ipHostInfo= Dns.Resolve("host.contoso.com");

            IPAddress ipAddress= ipHostInfo.AddressList[0];

            IPEndPoint remoteEP= new IPEndPoint(ipAddress,port);

            Socket client= new Socket(AddressFamily.InterNetwork,

                SocketType.Stream, ProtocolType.Tcp);

 

            client.BeginConnect(remoteEP,new AsyncCallback(ConnectCallback), client);

            connectDone.WaitOne();

            Send(client, "This is atest<EOF>");

            sendDone.WaitOne();

 

            Receive(client);

            receiveDone.WaitOne();

            Console.WriteLine("Responsereceived : {0}", response);

            client.Shutdown(SocketShutdown.Both);

            client.Close();

}

        catch (Exception e){

            Console.WriteLine(e.ToString());}

    }

 

    private static void ConnectCallback(IAsyncResult ar)

    {

        try{

            Socket client = (Socket)ar.AsyncState;

            client.EndConnect(ar);

            Console.WriteLine("Socketconnected to {0}",

             client.RemoteEndPoint.ToString());

            connectDone.Set();

        }

        catch (Exception e){

            Console.WriteLine(e.ToString());}

    }

 

    private static void Receive(Socket client)

    {

        try{

            StateObject state= new StateObject();

            state.workSocket= client;

            client.BeginReceive(state.buffer,0, StateObject.BufferSize, 0,

                new AsyncCallback(ReceiveCallback),state);

        }

        catch (Exception e){

            Console.WriteLine(e.ToString());}

    }

 

    private static void ReceiveCallback(IAsyncResult ar)

    {

        try{

            StateObject state = (StateObject)ar.AsyncState;

            Socket client =state.workSocket;

            int bytesRead= client.EndReceive(ar);

            if (bytesRead> 0){

                state.sb.Append(Encoding.ASCII.GetString(state.buffer,0, bytesRead));

                client.BeginReceive(state.buffer,0, StateObject.BufferSize, 0,

                    new AsyncCallback(ReceiveCallback),state);

            }

            else{

                 if (state.sb.Length> 1)

                {

                    response= state.sb.ToString();

                }

                receiveDone.Set();

            }

        }

        catch (Exception e){

            Console.WriteLine(e.ToString());}

    }

 

    private static void Send(Socket client, String data)

    {

         byte[] byteData= Encoding.ASCII.GetBytes(data);

        client.BeginSend(byteData,0, byteData.Length, 0,

            new AsyncCallback(SendCallback),client);

    }

 

    private static void SendCallback(IAsyncResult ar)

    {

        try{

             Socket client = (Socket)ar.AsyncState;

            int bytesSent= client.EndSend(ar);

            Console.WriteLine("Sent {0}bytes to server.", bytesSent);

            sendDone.Set();

}

        catch (Exception e){

            Console.WriteLine(e.ToString());}

    }

 

    public static int Main(String[] args)

    {

        StartClient();

        return 0;

    }

}

服务器端Server：

预定义结构体，用于异步委托之间的传递。同客户端的一致。不再赘述

正文：

public class AsynchronousSocketListener

{

    public static ManualResetEvent allDone= new ManualResetEvent(false);

    public AsynchronousSocketListener(){}

    public static void StartListening()

    {

        byte[] bytes = new Byte[1024];

         IPHostEntry ipHostInfo= Dns.Resolve("127.0.0.1");

        IPAddress ipAddress= ipHostInfo.AddressList[0];

        IPEndPoint localEndPoint= new IPEndPoint(ipAddress,11000);

         Socket listener= new Socket(AddressFamily.InterNetwork,SocketType.Stream,ProtocolType.Tcp);

         try{

            listener.Bind(localEndPoint);

            listener.Listen(100);

            while (true){

                allDone.Reset();

                Console.WriteLine("Waitingfor a connection...");

                listener.BeginAccept(new AsyncCallback(AcceptCallback),listener);

                allDone.WaitOne();

            }

        }

        catch (Exception e){

            Console.WriteLine(e.ToString());}

        Console.WriteLine("\nPressENTER to continue...");

        Console.Read();

    }

    public static void AcceptCallback(IAsyncResult ar)

    {

         Socket listener =(Socket)ar.AsyncState;

        Socket handler =listener.EndAccept(ar);

        StateObject state= new StateObject();

        state.workSocket= handler;

        handler.BeginReceive(state.buffer,0, StateObject.BufferSize, 0,newAsyncCallback(ReadCallback),state);

       allDone.Set();

    }

    public static void ReadCallback(IAsyncResult ar)

    {

        String content= String.Empty;

        StateObject state = (StateObject)ar.AsyncState;

        Socket handler =state.workSocket;

        int bytesRead= handler.EndReceive(ar);

        if (bytesRead> 0)

        {

             state.sb.Append(Encoding.ASCII.GetString(state.buffer,0, bytesRead));

            content= state.sb.ToString();

            if (content.IndexOf("<EOF>") > -1){

                Console.WriteLine("Read {0}bytes from socket. \n Data : {1}",

                content.Length,content);

                Send(handler, "Serverreturn :" + content);

            }

            else{

                handler.BeginReceive(state.buffer,0, StateObject.BufferSize, 0,

                new AsyncCallback(ReadCallback),state);

            }

        }

    }

    private static void Send(Socket handler, String data){

         byte[] byteData= Encoding.ASCII.GetBytes(data);

        handler.BeginSend(byteData,0, byteData.Length, 0,

        new AsyncCallback(SendCallback),handler);

    }

    private static void SendCallback(IAsyncResult ar)

    {

        try{

             Socket handler =(Socket)ar.AsyncState;

            int bytesSent= handler.EndSend(ar);

            Console.WriteLine("Sent {0}bytes to client.", bytesSent);

            handler.Shutdown(SocketShutdown.Both);

            handler.Close();

        }

        catch (Exception e){

            Console.WriteLine(e.ToString());

        }

    }

    public static int Main(String[] args)

    {

        StartListening();

        return 0;

    }

}

 

## 2.2 采用TCPListener、TCPClient实现TCP
### 2.2.1 同步通信
#### 2.2.1.1 简介
**TCPListener介绍**

1、构造函数

	（1）TCPListener（IPEndPoint）

	（2）TCPListener（Int32）

	（3）TCPListener（IPAddress，Int32）

	这三种构造函数都是指定服务器本身监听的地址与端口。

2、开启监听

	（1）Start（）

	（2）Start（Int32）

	设置了最大挂起连接数，即最大的可以等待在Accept上的请求。

3、接受客户端连接

	（1）Socket AcceptSocket ()
		阻塞方法。返回Socket与客户端进行通信

	（2）TcpClient AcceptTcpClient ()
		阻塞方法。返回TcpClient与客户端进行通信

4、关闭监听器

	void Stop（）

**TCPClient介绍**

1、构造函数

	（1）TCPClient（）
		当使用这种不带任何参数的构造函数时，将使用本机默认的ip地址并将使用默认的通信端口号0。
		这样情况下，如果本机不止一个ip地址，将无法选择使用。

	（2）TCPClient（IPEndPoint）
		IPEndPoint用于指定在建立远程主机连接时所使用的本地网络接口（IP 地址）和端口号。
		使用上述两种构造函数，在与目标主机进行连接时，需要使用Connect方法进行连接。

	（3）TCPClient（String，Int32）
		连接到指定主机的，参数指定的是目标主句的地址，因此使用该方法后自动连接目标主机，无需Connect方法。

2、连接到目标主句

	（1）void Connect(IPEndPoint)
		使用指定的远程网络终结点将客户端连接到远程 TCP 主机。

	（2）Connect(IPAddress, int)
		使用指定的 IP 地址和端口号将客户端连接到 TCP 主机。

	（3）Connect(string, int)
		将客户端连接到指定主机上的指定端口

3、返回用于发送或接收数据的NetworkStream

	 NetworkStream GetStream ()

4、使用NetworkStream进行数据的接收与发送

	（1）int Read (byte[] buffer, int offset, int size)
		buffer：用于存储从 NetworkStream 读取的数据。
		offset：buffer 中开始将数据存储到的位置。
		size：要从 NetworkStream 中读取的字节数。
		返回值：从 NetworkStream 中读取的字节数。

	（2）void Write (byte[] buffer, int offset, int size)
		buffer：存放要写入 NetworkStream 的数据。
		offset：buffer 中开始写入数据的位置。
		size：要写入 NetworkStream 的字节数。

 

#### 2.2.1.2 程序示例
服务端：
```
class Server
{
	private TcpListener tcpListener;
	private Thread listenThread;
	public Server()
	{
		this.tcpListener = newTcpListener(IPAddress.Any, 3000);
		this.listenThread = new Thread(newThreadStart(ListenForClients));
		this.listenThread.Start();
	}

	private voidListenForClients()
	{
		this.tcpListener.Start();
		while (true)
		{
			try{
				TcpClient client =this.tcpListener.AcceptTcpClient();
				Thread clientThread = new Thread(newParameterizedThreadStart(HandleClientComm));
				clientThread.Start(client);
			}
			catch(Exception e)
			{
				break;
			}
		}
		this.tcpListener.Stop();
	}

	private voidHandleClientComm(object client)
	{
		TcpClient tcpClient = (TcpClient)client;
		NetworkStream clientStream =tcpClient.GetStream();
		byte[] message = new byte[4096];
		int bytesRead;
		while (true)
		{
			bytesRead = 0;
			try
			{
				bytesRead =clientStream.Read(message, 0, 4096);
			}
			catch
			{
				break;
			}

			if (bytesRead == 0)
				break;

			strings = Encoding.UTF8.GetString(message, 0, bytesRead);
			Console.WriteLine(s);
			byte[] SendMessage = Encoding.UTF8.GetBytes("收到信息");
			clientStream.Write(SendMessage,0, SendMessage.Length);
			clientStream.Flush();
		}

		clientStream.Close();
		tcpClient.Close();
	}

}
```


客户端：
```
	TcpClientclient = new TcpClient();
	IPEndPointserverEndPoint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 3000);
	client.Connect(serverEndPoint);
	NetworkStreamclientStream = client.GetStream();
	byte[] buffer= Encoding.UTF8.GetBytes("Hello Server!");
	clientStream.Write(buffer,0 , buffer.Length);
	clientStream.Flush(); 
	byte[] message= new byte[4096];
	int bytesRead= clientStream.Read(message, 0, 4096);
	string s =Encoding.UTF8.GetString(message, 0, bytesRead);
	Console.WriteLine(s);
	clientStream.Close();
	client.Close();
```


#### 简介
* （1）创建对象
		绑定指定的端口进行数据的接收与发送：
		UdpClient  udpClient= new UdpClient(11000);

		使用随机的端口进行数据的接收与发送：
		UdpClient  udpClient= new UdpClient(0);
		使用随机端口进行发送，每次发出去的信息端口号码都是不同的，这样接收方要把信息发回来，就没办法了。

		绑定指定终结点，进行数据的发送：
		UdpClient  udpClient= new UdpClient(IPEndPoint);


* （2）发送数据
	发送的时候创建一个结点，来指定接收方的地址
		Send(byte[],Int32,IpEndPoint)
		Send(byte[],Int32,String,Int32)
		Send(byte[],Int32)
		在使用第三个send方法时由于没有指定远端地址，需要在之前使用Connect方法来指定远端地址。


* （3）接收信息
		byte[] Receive(IpEndPoint)
		该方法是一个阻塞方法，如果没有收到数据，会一直在该方法上等待。


#### 程序示例
##### 实现广播功能

* 发送端：
其实UDP广播就是向255.255.255.255发送数据，接收端只需绑定UDP广播的端口号即可。发送端，发送的地址，255.255.255.255：Port，即，IPAddress.Broadcast：Port。

```cpp
UdpClient UDPsend = new UdpClient(newIPEndPoint(IPAddress.Any, 0));
//其实IPAddress.Broadcast 就是 255.255.255.255
IPEndPoint endpoint = newIPEndPoint(IPAddress.Broadcast, 8080);
byte[] buf = Encoding.Default.GetBytes("This is UDPbroadcast");
UDPsend.Send(buf, buf.Length, endpoint);
``` 

* 接收端：
接收端仅需绑定发送端发送时指定的端口，便可以接收广播信息

```cpp
IPEndPoint LocalIp = new IPEndPoint(IPAddress.Any, 8080);
//任意地址，用于Receive中的ref参数，返回发送端的地址。
IPEndPoint Remote = new IPEndPoint(IPAddress.Any, 0);
UdpClient udpClient = new UdpClient(LocalIp);
//Receive方法返回byte数组。
byte[] d = udpClient.Receive(ref Remote);
string s = Encoding.UTF8.GetString(d, 0, d.Length);
```

##### 实现一对一通信
* 发送端：

```cpp
//定义接收端的地址
IPEndPoint targetIp = newIPEndPoint(IPAddress.Parse("192.168.2.119"), 8001);
UdpClient udpClient= new UdpClient(0);
byte[] data = Encoding.UTF8.GetBytes(“Hello”);
//发送信息，第二个参数为发送的字节数
udpClient.Send(data, data.Length, targetIp);
``` 

* 接收端：

```cpp
//定义本地地址，用于不断监听该地址收到的信息。该地址与发送端的发送地址//一致。
IPEndPoint LocalIp = new IPEndPoint(IPAddress.Any,8001);
//任意地址，用于Receive中的ref参数，返回发送端的地址。
IPEndPoint Remote = new IPEndPoint(IPAddress.Any, 0);
UdpClient udpClient = new UdpClient(LocalIp);
//Receive方法返回byte数组。
byte[] d = udpClient.Receive(ref Remote);
string s = Encoding.UTF8.GetString(d, 0, d.Length);
```
 

##### 实现一对多通信（组播）
IP组播通信需要一个特殊的组播地址，IP组播地址是一组D类IP地址，范围从224.0.0.0 到239.255.255.255。其中还有很多地址是为特殊的目的保留的。
224.0.0.0到224.0.0.255的地址最好不要用，因为他们大多是为了特殊的目的保持的（比如IGMP协议）。
因此如果设定的IP不符合D类IP规则，则程序报错。

* 发送端：

```cpp
//定义组播地址，注意一定要是D类广播
IPEndPoint targetIp = new IPEndPoint(IPAddress.Parse("226.1.1.2"),8001);
UdpClient udpClient= new UdpClient(0);
byte[] data = Encoding.UTF8.GetBytes(“Hello”);
//发送信息，第二个参数为发送的字节数
udpClient.Send(data, data.Length, targetIp);
```

* 接收端：

```cpp
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
```

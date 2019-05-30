## 采用TCPListener、TCPClient实现TCP
### 同步通信

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

 

#### 程序示例
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


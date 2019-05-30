##### 简介
* 服务端：

	服务端的主要职责是处理各个客户端发送来的数据，因此在客户端的Socket编程中需要使用两个线程来循环处理客户端的请求，一个线程用于监听客户端的连接情况，一个线程用于监听客户端的消息发送。

	基本流程
	* 创建套接字
	* 绑定套接字的IP和端口号——Bind(EndPoint)
	* 将套接字处于监听状态等待客户端的连接请求——Listen(int) 监听端口，其中parameters表示最大监听数。
	* 阻塞方法，当请求到来后，接受请求并返回本次会话的套接字——Accept()接受客户端链接，并返回一个新的链接，用于处理同客户端的通信问题。
	* 使用返回的套接字和客户端通信——Send()/Receive()
			Send(byte[]) 简单发送数据。
			Send(byte[],SocketFlag)   使用指定的SocketFlag发送数据。
			Send(byte[],int, SocketFlag)   使用指定的SocketFlag发送指定长度数据。
			Send(byte[], int, int,SocketFlag)  使用指定的SocketFlag，将指定字节数的数据发送到已连接的socket(从指定偏移量开始)。

			Receive(byte[])  简单接受数据。
			Receive(byte[],SocketFlag)    使用指定的SocketFlag接受数据。
			Receive(byte[], int, SocketFlag)    使用指定的SocketFlag接受指定长度数据。
			Receive (byte[], int,int, SocketFlag)    使用指定的SocketFlag，从绑定的套接字接收指定字节数的数据，并存到指定偏移量位置的缓冲区。
	* 返回，再次等待新的连接请求
	* 关闭套接字

* 客户端：

	客户端相对于服务端来说任务要轻许多，因为客户端仅仅需要和服务端通信即可，可是因为在和服务器通信的过程中，需要时刻保持连接通畅，因此同样需要两个线程来分别处理连接情况的监听和消息发送的监听。

	基本流程
	* 创建套接字保证与服务器的端口一致
	* 向服务器发出连接请求——Connect(EndPoint)连接远程服务器，如果没有相应的远程等待连接，直接返回错误。
	* 和服务器端进行通信——Send()/Receive()
	* 关闭套接字

##### 程序示例
* **简单的单次通信**

服务端接收客户端信息并向客户端返回信息

服务端：
```cpp
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
```cpp
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

* **多次通信**

```cpp
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



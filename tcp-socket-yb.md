##### 简介
1. BeginAccept(AsynscCallBack,object)
	* 开始接收一个异步连接尝试。
	* object：通常传入的是服务Socket对象实例。
	* AsynscCallBack：回调函数，该回调函数中必须包含EndAccept方法。
	* 应用程序调用BegineAccept方法后，系统会使用单独的线程执行指定的回调方法并在EndAccept上一直处于阻塞状态，直至监测到挂起的链接。EndAccept会返回新的socket对象。供你来同远程主机数据交互。调用BeginAccept当希望原始线程阻塞的时候，请调用WaitHandle.WaitOne方法。当需要原始线程继续执行时请在回调方法中使用ManualResetEvent的set方法。


2. Socket EndAccept（IAsyncResult）
	* 重点是返回一个Socket与客户端进行通信。IAsyncResult作用不大。


3. BeginConnect（EndPoint,AsyncCallBack, Object）
	* EndPoint：指定想要连接的服务端地址。
	* AsynscCallBack：回调函数，该回调函数中必须包含EndConnect方法。
	* object：通常传入的是客户端Socket对象实例。


4. void EndConnect（IAsyncResult）
	* 无返回值，在为连接成功前一直在该方法上等待。


5. BeginReceive(byte[] buffer,int offset, int size, SocketFlags, AsyncCallbac, object)
	* buffer:用于存放接收信息。
	* offset：指定buffer的其实存储位置。
	* size：指定接收的信息byte长度。
	* SocketFlags：socket标志位。
	* AsynscCallBack：回调函数，该回调函数中必须包含EndReceive方法。
	* object：通常是一个存放了第一个参数buffer以及与远端进行通信的Socket对象的数据结构的对象实例。


6. int  EndReceieve（IAsyncResult）
	* 返回收到的信息的byte数组长度。IAsyncResult作用不大。
	

7. BeginSend(byte[],SocketFlag, AsyncCallBack, Object)
	* byte[]：需要发送的数据。
	* AsyncCallBack：回调函数，该回调函数中必须包含EndSend方法。
	* object：通常为与远端进行通信的Socket实例。


8. int EndSend（IAsyncResult）
	* 返回发送的信息的byte数组长度。IAsyncResult作用不大。

 
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


##### 程序示例
* **服务端仅用来不断接收客户端信息**

```cpp
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
```

* **服务端与客户端双向通信**
（该代码引用自他人）

*	* 客户端Client：


```cpp
//预定义结构体，用于异步委托之间的传递。用户根据自己需要定制即可
public class StateObject
{
	public Socket workSocket= null;
	public const int BufferSize= 256;
	public byte[] buffer= new byte[BufferSize];
	public StringBuilder sb = new StringBuilder();
}

public class AsynchronousClient
{
    private const int port =11000;
    private static ManualResetEvent connectDone= new ManualResetEvent(false);
    private static ManualResetEvent sendDone= new ManualResetEvent(false);
    private static ManualResetEvent receiveDone= new ManualResetEvent(false);
    private static String response= String.Empty;

    private static void StartClient()
	{
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
			Console.WriteLine(e.ToString());
		}
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
            Console.WriteLine(e.ToString());
		}
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
            Console.WriteLine(e.ToString());
		}
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
            Console.WriteLine(e.ToString());
		}
    }

    public static int Main(String[] args)
    {
        StartClient();
        return 0;
    }
}
```

* 	* 服务器端Server：

```cpp
// 预定义结构体，用于异步委托之间的传递。同客户端的一致。不再赘述
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
            Console.WriteLine(e.ToString());
		}
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
```
 
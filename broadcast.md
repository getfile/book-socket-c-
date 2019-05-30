#####  单播
* 概念:
	* 主机之间一对一的通讯模式，网络中的交换机和路由器对数据只进行转发不进行复制。每个以太网帧仅发往单个目的主机,目的地址指明单个接收接口，任意两个主机的通信不会干扰网内其他主机。可用TCP或者UDP实现。


* 优点：
	* 服务器及时响应客户机的请求
	* 服务器针对每个客户普通的请求发送普通的数据，容易实现个性化服务。


* 缺点：
	* 服务器针对每个客户机发送数据流，服务器流量＝客户机数量×客户机流量；在客户数量大、每个客户机流量大的流媒体应用中服务器不堪重负。
	* 现有的网络带宽是金字塔结构，城际省际主干带宽仅仅相当于其所有用户带宽之和的5％。如果全部使用单播协议，将造成网络主干不堪重负。现在的P2P应用就已经使主干经常阻塞。而将主干扩展20倍几乎是不可能。


##### 广播
* 概念:
	* “广播”在网络中的应用较多，主机之间一对全体的通讯模式，网络对其中每一台主机发出的信号都进行无条件复制并转发，所有主机都可以接收到所有信息（不管你是否需要）。可用UDP实现。
	* 同单播和多播相比，广播几乎占用了子网内网络的所有带宽。
	* 在网络中不能长时间出现大量的广播包，否则就会出现所谓的“广播风暴”。如在会场上只能有一个人用麦克发言，想象一下如果所有的人同时都用麦克风发言，那会场上就会乱成一锅粥。
	* 广播风暴就是网络长时间被大量的广播数据包所占用，正常的点对点通信无法正常进行，外在表现为网络速度奇慢无比。出现广播风暴的原因有很多，一块有故障的网卡，就可能长时间向网络上发送广播包而导致广播风暴。


* 优点：
	* 网络设备简单，维护简单，布网成本低廉
	* 由于服务器不用向每个客户机单独发送数据，所以服务器流量负载极低。


* 缺点：
	* 无法针对每个客户的要求和时间及时提供个性化服务。
	* 网络允许服务器提供数据的带宽有限，客户端的最大带宽＝服务总带宽。例如有线电视的客户端的线路支持100个频道（如果采用数字压缩技术，理论上可以提供500个频道），即使服务商有更大的财力配置更多的发送设备、改成光纤主干，也无法超过此极限。也就是说无法向众多客户提供更多样化、更加个性化的服务。
	* 广播禁止允许在Internet宽带网上传输。


##### 多播
* 概念:
	* “多播”也可以称为“组播”，主机之间一对一组的通讯模式，也就是加入了同一个组的主机可以接受到此组内的所有数据，网络中的交换机和路由器只向有需求者复制并转发其所需数据。主机可以向路由器请求加入或退出某个组。这样既能一次将数据传输给多个有需要（加入组）的主机，又能保证不影响其他不需要（未加入组）的主机的其他通讯。可用UDP实现（使用TCP则需要逐个建立连接，比较耗费资源）。
	* 网上视频会议、网上视频点播特别适合采用多播方式。因为如果采用单播方式，逐个节点传输，有多少个目标节点，就会有多少次传送过程，这种方式显然效率极低，是不可取的；如果采用不区分目标、全部发送的广播方式，虽然一次可以传送完数据，但是显然达不到区分特定数据接收对象的目的。采用多播方式，既可以实现一次传送所有目标节点的数据，也可以达到只对特定对象传送数据的目的。


* 优点：
	* 需要相同数据流的客户端加入相同的组共享一条数据流，节省了服务器的负载。具备广播所具备的优点。
	* 由于组播协议是根据接受者的需要对数据流进行复制转发，所以服务端的服务总带宽不受客户接入端带宽的限制。IP协议允许有2亿6千多万个组播，所以其提供的服务可以非常丰富。
	* 此协议和单播协议一样允许在Internet宽带网上传输。


* 缺点：
	* 与单播协议相比没有纠错机制，发生丢包错包后难以弥补，但可以通过一定的容错机制和QOS加以弥补。
	* 现行网络虽然都支持组播的传输，但在客户认证、QOS等方面还需要完善，这些缺点在理论上都有成熟的解决方案，只是需要逐步推广应用到现存网络当中
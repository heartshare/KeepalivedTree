# KeepalivedTree
keepalived技术研究


<pre>
keepalived软件起初是专门为LVS负载均衡软件设计的，用来管理并监控LVS集群系统中各个服务节点
的状态，后来又加入了高可用的VRRP功能，因此，keepalived除了能够管理LVS软件外，还可以作为其
它服务如 Nginx, Haproxy, Mysql等的高可用解决方案软件。
</pre>

<pre>
Keepalived服务的三个重要功能
    1）管理LVS负载均衡软件
    2）实现LVS集群节点的健康检查中
　　3) 作为系统网络服务的高可用性（failover）
</pre>

![](https://i.imgur.com/5ixGdwC.png)

<pre>
keepalived原理

    Keepalived高可用服务对之间的故障切换转移，是通过 VRRP (Virtual Router 
    Redundancy Protocol ,虚拟路由器冗余协议）来实现的。

　　在 Keepalived服务正常工作时，主 Master节点会不断地向备节点发送（多播的方式）心跳消
    息，用以告诉备Backup节点自己还活看，当主 Master节点发生故障时，就无法发送心跳消息，
    备节点也就因此无法继续检测到来自主 Master节点的心跳了，于是调用自身的接管程序，接管
    主Master节点的 IP资源及服务。而当主 Master节点恢复时，备Backup节点又会释放主节点故
    障时自身接管的IP资源及服务，恢复到原来的备用角色。


   Keepalived的工作原理：

　　Keepalived高可用对之间是通过VRRP通信的，因此，我们从 VRRP开始了解起：

　　　　1) VRRP,全称 Virtual Router Redundancy Protocol,中文名为虚拟路由冗余协
          议，VRRP的出现是为了解决静态路由的单点故障。

　　　　2) VRRP是通过一种竟选协议机制来将路由任务交给某台 VRRP路由器的。

　　　　3) VRRP用 IP多播的方式（默认多播地址（224.0_0.18))实现高可用对之间通信。

　　　　4) 工作时主节点发包，备节点接包，当备节点接收不到主节点发的数据包的时候，就启动接
       管程序接管主节点的开源。备节点可以有多个，通过优先级竞选，但一般 Keepalived系统运
       维工作中都是一对。

　　　　5) VRRP使用了加密协议加密数据，但Keepalived官方目前还是推荐用明文的方式配置认证
       类型和密码。

　　Keepalived服务的工作原理：

　　   Keepalived高可用对之间是通过 VRRP进行通信的， VRRP是遑过竞选机制来确定主备的，主
      的优先级高于备，因此，工作时主会优先获得所有的资源，备节点处于等待状态，当主挂了的
      时候，备节点就会接管主节点的资源，然后顶替主节点对外提供服务。

　　  在 Keepalived服务对之间，只有作为主的服务器会一直发送 VRRP广播包,告诉备它还活着，
     此时备不会枪占主，当主不可用时，即备监听不到主发送的广播包时，就会启动相关服务接管
     资源，保证业务的连续性.接管速度最快可以小于1秒。
</pre>
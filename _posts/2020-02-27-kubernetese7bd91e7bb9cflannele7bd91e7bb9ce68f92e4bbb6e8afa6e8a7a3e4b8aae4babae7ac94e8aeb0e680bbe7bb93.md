---
ID: 32
post_title: >
  Kubernetes网络flannel网络插件详解个人笔记总结
author: Toki
post_excerpt: ""
layout: post
permalink: 'https://lazytoki.cn/2020/02/27/kubernetes%e7%bd%91%e7%bb%9cflannel%e7%bd%91%e7%bb%9c%e6%8f%92%e4%bb%b6%e8%af%a6%e8%a7%a3%e4%b8%aa%e4%ba%ba%e7%ac%94%e8%ae%b0%e6%80%bb%e7%bb%93/'
published: true
post_date: 2020-02-27 18:36:17
---
<h2>title: Kubernetes网络-flannel网络插件详解 个人笔记总结
date: '2019-09-16 20:44:39'
updated: '2019-09-16 20:49:04'
tags: [flannel, kubernetes, 个人笔记]
permalink: /articles/2019/09/16/1568637879400.html</h2>

<img src="https://img.hacpai.com/bing/20190720.jpg?imageView2/1/w/960/h/540/interlace/1/q/100" alt="" /> 

<h2>前言：</h2>

因为 Kubernetes的网络可以使用第三方网络插件，所以给我们提供了多样化的网络解决方案，让我们可以根据自身情况选择自己需要的网络方案。

<h3>CNM & CNI阵营：</h3>

• 容器网络发展到现在，形成了两大阵营，就是Docker的CNM和Google、CoreOS、Kuberenetes主导的CNI。首先明确一点，CNM和CNI并不是网络实现，他们是网络规范和网络体系，从研发的角度他们就是一堆接口，你底层是用Flannel也好、用Calico也好，他们并不关心，CNM和CNI关心的是网络管理的问题。

• CNM (Docker LibnetworkContainer Network Model)

• CNI（Container Network Interface）


<strong>首先我们得先了解k8s网络的设计原则，然后才能更好的理解flannel网络的应用</strong>

<h3>Kubernetes网络设计模型：</h3>

•在Kubernetes网络中存在两种IP（Pod IP和Service Cluster IP），Pod IP 地址是实际存在于某个网卡(可以是虚拟设备)上的，Service Cluster IP它是一个虚拟IP，是由kube-proxy使用Iptables规则重新定向到其本地端口，再均衡到后端Pod的。

基本原则：
• 每个Pod都拥有一个独立的IP地址（IPper Pod），而且假定所有的pod都在一个可以直接连通的、扁平的网络空间中。

设计原因：
• 用户不需要额外考虑如何建立Pod之间的连接，也不需要考虑将容器端口映射到主机端口等问题。

网络要求：
• 所有的容器都可以在不用NAT的方式下同别的容器通讯；所有节点都可在不用NAT的方式下同所有容器通讯；容器的地址和别人看到的地址是同一个地址。

<h3>K8S网络主要解决以下网络通信问题:</h3>

• <strong>同一pod下容器与容器的通信；</strong>

• <strong>同一节点下不同的pod之间的容器间通信；</strong>

• <strong>不同节点下容器之间的通信；</strong>

• <strong>集群外部与内部组件的通信；</strong>

• <strong>pod与service之间的通信；</strong>

<h3>1、容器间通信：</h3>

同一个<a href="https://www.kubernetes.org.cn/tags/pod">Pod</a>的容器共享同一个网络命名空间，它们之间的访问可以用  

localhost地址 + 容器端口就可以访问。

<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t1-eae44b28.jpg" />

<h3>2、同一Node中Pod间通信：</h3>

同一Node中Pod的默认路由都是docker0的地址，由于它们关联在同一个docker0（cni0）网桥上，地址网段相同，所有它们之间应当是能直接通信的。

<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t2-0540cce4.jpg" />

<h3>3、不同Node中Pod间通信：</h3>

不同Node中Pod间通信要满足2个条件：  

<strong>**Pod的IP不能冲突</strong>；<strong> </strong>将Pod的IP和所在的Node的IP关联起来，通过这个关联让Pod可以互相访问**。所以就要用到flannel或者calico的网络解决方案。  

对于此场景，情况现对比较复杂一些，这就需要解决Pod间的通信问题。在Kubernetes通过flannel、calico等网络插件解决Pod间的通信问题。以flannel为例说明在Kubernetes中网络模型，flannel是kubernetes默认提供网络插件。Flannel是由CoreOs团队开发社交的网络工具，CoreOS团队采用L3 Overlay模式设计flannel， 规定宿主机下各个Pod属于同一个子网，不同宿主机下的Pod属于不同的子网。

<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t3-29411050.jpg" />

<h3>容器网络方案：</h3>

<strong>隧道方案（ Overlay Networking ）</strong>

隧道方案在IaaS层的网络中应用也比较多，大家共识是随着节点规模的增长复杂度会提升，而且出了网络问题跟踪起来比较麻烦，大规模集群情况下这是需要考虑的一个点。

• Weave：UDP广播，本机建立新的BR，通过PCAP互通

• Open vSwitch（OVS）：基于VxLan和GRE协议，但是性能方面损失比较严重

• <strong>Flannel：UDP广播，VxLan</strong>

• Racher：IPsec

<h3>CNI Plugin – Flannel插件解决方案：</h3>

• Flannel是CoreOS团队针对Kubernetes设计的一个网络规划服务，简单来说，它的功能是让集群中的不同节点主机创建的Docker容器都具有全集群唯一的虚拟IP地址。

• 在默认的Docker配置中，每个节点上的Docker服务会分别负责所在节点容器的IP分配。这样导致的一个问题是，不同节点上容器可能获得相同的内外IP地址。并使这些容器之间能够之间通过IP地址相互找到，也就是相互ping通。

• Flannel的设计目的就是为集群中的所有节点重新规划IP地址的使用规则，从而使得不同节点上的容器能够获得“同属一个内网”且”不重复的”IP地址，并让属于不同节点上的容器能够直接通过内网IP通信。

• Flannel实质上是一种“覆盖网络(overlaynetwork)”，也就是将TCP数据包装在另一种网络包里面进行路由转发和通信，<strong>目前已经支持udp、vxlan、host-gw、aws-vpc、gce和alloc路由等数据转发方式，默认的节点间数据通信方式是UDP转发。</strong>

例如通过使用后端（backend）为vxlan的转发方式的flannel架构图:

<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t4-93aac0c9.jpg" />

简而言之就是Flannel之所以可以搭建kubernets依赖的底层网络，是因为它可以实现以下两点：

• 它给每个node上的pod分配相互不想冲突的IP地址；

• 它能给这些IP地址之间建立一个覆盖网络，同过覆盖网络，将数据包原封不动的传递到目标容器内。

下图是通过vxlan的方式实现节点间的通信：
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t5-f35761a2.jpg" />

<h3>flannel数据包转发方式(backend原理解析)</h3>

<h4>1.hostgw</h4>

• hostgw是最简单的backend，它的原理非常简单，直接添加路由，将目的主机当做网关，直接路由原始封包。

例如下图，我们从etcd中监听到一个EventAdded事件subnet为10.1.15.0/24被分配给主机Public IP 192.168.0.100，hostgw要做的工作就是在本主机上添加一条目的地址为10.1.15.0/24，网关地址为192.168.0.100，输出设备为上文中选择的集群间交互的网卡即可。对于EventRemoved事件，只需删除对应的路由。

<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t6-1a7fbaa2.jpg" />

但是hostgw有个不足的地方就是，我们知道当backend为hostgw时，主机之间传输的就是原始的容器网络封包，封包中的源IP地址和目的IP地址都为容器所有。这种方法有一定的限制，就是要求所有的主机都在一个子网内，即二层可达，否则就无法将目的主机当做网关，直接路由。

<h4>2.udp</h4>

• 而udp类型backend的基本思想是：既然主机之间是可以相互通信的（并不要求主机在一个子网中），那么我们就可以直接将容器的网络封包作为负载数据在集群的主机之间进行传输呢？这就是所谓的overlay。具体过程如图所示：
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t7-20bb5ebe.jpg" />

当容器10.1.15.2/24要和容器10.1.20.2/24通信时，因为该封包的目的地不在本主机subnet内，因此封包会首先通过网桥转发到主机中。最终在主机上经过路由匹配，进入网卡flannel0。需要注意的是flannel0是一个tun设备，它是一种工作在三层的虚拟网络设备，而flanneld是一个proxy，它会监听flannel0并转发流量。当封包进入flannel0时，flanneld就可以从flannel0中将封包读出，由于flannel0是三层设备，所以读出的封包仅仅包含IP层的报头及其负载。最后flanneld会将获取的封包作为负载数据，通过udp socket发往目的主机。同时，在目的主机的flanneld会监听Public IP所在的设备，从中读取udp封包的负载，并将其放入flannel0设备内。由此，容器网络封包到达目的主机，之后就可以通过网桥转发到目的容器了。

最后和hostgw不同的是，udp backend并不会将从etcd中监听到的事件里所包含的lease信息作为路由写入主机中。每当收到一个EventAdded事件，flanneld都会将其中的subnet和Public IP保存在一个数组中，用于转发封包时进行查询，找到目的主机的Public IP作为udp封包的目的地址。

<h4>3.vxlan</h4>

• 首先，我们对vxlan的基本原理进行简单的叙述。从下图所示的封包结构来看，vxlan和上文提到的udp backend的封包结构是非常类似的，不同之处是多了一个vxlan header，以及原始报文中多了个二层的报头。

<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t8-a5a38075.jpg" />

上图所示，当主机B加入flannel网络时，和其他所有backend一样，它会将自己的subnet 10.1.16.0/24和Public IP 192.168.0.101写入etcd中，和其他backend不一样的是，它还会将vtep设备flannel.1的mac地址也写入etcd中。

之后，主机A会得到EventAdded事件，并从中获取上文中B添加至etcd的各种信息。这个时候，它会在本机上添加三条信息：

<strong>1.路由信息：所有通往目的地址10.1.16.0/24的封包都通过vtep设备flannel.1设备发出，发往的网关地址为10.1.16.0，即主机B中的flannel.1设备。</strong>

<strong>2.fdb信息：MAC地址为MAC B的封包，都将通过vxlan首先发往目的地址192.168.0.101，即主机B</strong>

<strong>3.arp信息：网关地址10.1.16.0的地址为MAC B</strong>

现在有一个容器网络封包要从A发往容器B，和其他backend中的场景一样，封包首先通过网桥转发到主机A中。此时通过，查找路由表，该封包应当通过设备flannel.1发往网关10.1.16.0。通过进一步查找arp表，我们知道目的地址10.1.16.0的mac地址为MAC B。到现在为止，vxlan负载部分的数据已经封装完成。由于flannel.1是vtep设备，会对通过它发出的数据进行vxlan封装（这一步是由内核完成的，相当于udp backend中的proxy），那么该vxlan封包外层的目的地址IP地址该如何获取呢？事实上，对于目的mac地址为MAC B的封包，通过查询fdb，我们就能知道目的主机的IP地址为192.168.0.101。

最后，封包到达主机B的eth0，通过内核的vxlan模块解包，容器数据封包将到达vxlan设备flannel.1，封包的目的以太网地址和flannel.1的以太网地址相等，三层封包最终将进入主机B并通过路由转发达到目的容器。

<h4>查看flannel配置文件：</h4>

可以通过<code>cat /var/run/flannel/subnet.env</code>的命令查看flannel的子网分配情况

• flannel在每个Node上启动了一个flanneld的服务，在flanneld启动后，将从etcd中读取配置信息，并请求获取子网的租约。所有Node上的flanneld都依赖<a href="https://github.com/coreos/etcd/">etcd</a> <a href="https://github.com/coreos/etcd/">cluster</a>来做集中配置服务，etcd保证了所有node上flanned所看到的配置是一致的。同时每个node上的flanned监听etcd上的数据变化，实时感知集群中node的变化。flanneld一旦获取子网租约、配置后端后，会将一些信息写入/run/flannel/subnet.env文件。

<h3>demo演示过程（待更新）</h3>
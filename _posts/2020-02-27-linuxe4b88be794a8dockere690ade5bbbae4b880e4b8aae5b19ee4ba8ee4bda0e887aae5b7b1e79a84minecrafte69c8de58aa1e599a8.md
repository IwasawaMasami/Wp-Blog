---
ID: 33
post_title: >
  Linux下用Docker搭建一个属于你自己的Minecraft服务器
author: Toki
post_excerpt: ""
layout: post
permalink: 'https://lazytoki.cn/2020/02/27/linux%e4%b8%8b%e7%94%a8docker%e6%90%ad%e5%bb%ba%e4%b8%80%e4%b8%aa%e5%b1%9e%e4%ba%8e%e4%bd%a0%e8%87%aa%e5%b7%b1%e7%9a%84minecraft%e6%9c%8d%e5%8a%a1%e5%99%a8/'
published: true
post_date: 2020-02-27 18:36:07
---
<h2>title: Linux下用Docker搭建一个属于你自己的Minecraft服务器
date: '2019-09-04 15:28:20'
updated: '2019-10-24 13:07:00'
tags: [Docker, Linux, Minecraft]
permalink: /articles/2019/09/04/1567582100387.html</h2>

<img src="https://img.hacpai.com/bing/20190109.jpg?imageView2/1/w/960/h/540/interlace/1/q/100" alt="" />

<strong>可能需要有一定的Linux基础（不过不怎么影响，大家遇到不会的linux命令可以找度娘问问，基本都有的），比如文件的修改，移动，复制。目录的创建，删除等操作。</strong>

<h3>开始前的准备工作：</h3>

一台内存2G以上 带宽1m以上的Linux操作系统的VPS 或者 虚拟机（Vmware或者Virtual Box）中2G内存以上的Linux操作系统都可以。

个人推荐：ubuntu16.04LTS 或者 Centos 7.X

<h3>安装Docker：</h3>

以Centos 7为例子，安装命令为：

<blockquote><strong>yum -y install docker</strong></blockquote>

ubuntu 安装命令为：

<blockquote><strong>apt-get install -y docker</strong></blockquote>

如果遇到 unable to find......类似的错误，请使用以下命令更换源：

Centos操作系统：
<code>wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo</code>
Ubuntu操作系统请参考链接：<a href="https://blog.csdn.net/wu_cai_/article/details/78826208">修改Ubuntu的apt-get源为网易源</a>

<h3>简单了解docker</h3>

参考链接为：<a href="https://www.runoob.com/docker/docker-tutorial.html">Docker的介绍和优点</a>

Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从Apache2.0协议开源。
Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

从下面这张图片就可以明白
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t1-0e3d6cc9.jpg" />

Infrastructure：云平台为你创建的实例也就是你的计算资源
Host Operating system：在实例中安装的操作系统（Linux或者Windows）
Docker：以服务的方式运行在操作系统里
图中最上面所示的APP A B C E F就是运行在docker上的容器，这些容器里装的就是我们Minecraft游戏服务端

参考图片链接为：<a href="https://www.docker.com/resources/what-container">Docker官网</a>

<h3>使用docker部署Minecraft游戏服务端</h3>

首先使用命令启动docker服务
systemctl start docker 或者 service docker start

搭建一个spigot核心的1.11.2版本的游戏服务端：

<blockquote>docker run -d -v /root/on/host:/data
-e TYPE=SPIGOT -e VERSION=1.11.2
-p 25565:25565 -e EULA=TRUE --name mc-spigot --privileged=true tookizhang/minecraft-spigot:1.11.2</blockquote>

<strong>每个参数的含义：-d 就是把这个容器挂起来，让它能够持续运行
-v 通过映射的方式将物理主机的目录对应到容器里的目录
-e 指定环境变量，意思是可以指定服务端的版本和类型
-p 端口映射，因为容器是沙箱性质的，所以需要把容器端口映射到物理主机端口上去
--name 自定义你的容器名字</strong>

<blockquote>--privileged=true 当你遇到下面这个错误的时候，请在命令里加上这个。
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t2-9436f0a2.jpg" /></blockquote>

其实当你运行完这个命令以后，再等1-2分钟 你就可以直通过客户端直接登录这个服务端了。

<strong>运行测试效果：</strong>

请务必保持全程联网：
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t3-8ff344f2.jpg" />

正在运行的服务端
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t4-4fe48ff8.jpg" />

登录游戏：
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t5-bd8b1aea.jpg" />

默认游戏是开启了正版验证，如要更改请根据-v对应的目录进行更改
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t6-2c20461d.jpg" />

搭建一个游戏服务端就是只需要一个命令，就这么简单！

参考链接为：<a href="https://github.com/itzg/docker-minecraft-server">Github</a>
这只是一个简单快速的搭建过程，如果大家对docker细节感兴趣的话，可以访问上面的链接，源码作者有详细介绍如何搭建不同的MC服务端。

<h3>docker查看和管理Minecraft游戏服务端</h3>

那么命令是运行完了，我们如何去查看和管理我们刚刚搭建的服务端喃？

搭建完后查看和管理要用到命令：

<blockquote>docker images 【查看已有镜像】
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t7-65f7f2b9.jpg" />

docker ps 【查看容器进程】
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t8-6864d830.jpg" />

docker logs -f 容器名字 【查看容器内的日志】
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t9-f9f7b590.jpg" />

docker exec -i 容器名字 rcon-cli 【进入服务端consle界面】
<img src="http://lazytoki.cn/wp-content/uploads/2020/04/t10-ac04ba7c.jpg" /></blockquote>

后面三个就不截图了

<blockquote>docker start 容器名字 【开始运行容器】
docker stop 【停止正在运行容器】
docker restart 【重启容器】</blockquote>

（可选）使用docker build命令构建属于自己的镜像
请观看以下视频链接
<a href="https://www.bilibili.com/video/av65076015/">哔哩哔哩 1.13秒开始</a>
或者直接观看视频：

<iframe src="//player.bilibili.com/player.html?aid=65076015&cid=112940531&page=1" width="550" height="350" frameborder="no" scrolling="no" allowfullscreen="allowfullscreen"></iframe>

<h4>总结</h4>

<strong>第一次发帖，如果大家发现我还没有讲到的欢迎在留言区补充
大家互相学习，多多交流。</strong>
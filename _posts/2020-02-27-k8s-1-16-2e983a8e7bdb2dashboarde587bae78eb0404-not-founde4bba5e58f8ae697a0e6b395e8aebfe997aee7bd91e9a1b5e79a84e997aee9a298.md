---
ID: 34
post_title: >
  K8S-1.16.2部署dashboard出现404 NOT
  FOUND以及无法访问网页的问题
author: Toki
post_excerpt: ""
layout: post
permalink: 'https://lazytoki.cn/2020/02/27/k8s-1-16-2%e9%83%a8%e7%bd%b2dashboard%e5%87%ba%e7%8e%b0404-not-found%e4%bb%a5%e5%8f%8a%e6%97%a0%e6%b3%95%e8%ae%bf%e9%97%ae%e7%bd%91%e9%a1%b5%e7%9a%84%e9%97%ae%e9%a2%98/'
published: true
post_date: 2020-02-27 18:35:53
---
<h2>title: K8S-1.16.2部署dashboard出现404 NOT FOUND以及无法访问网页的问题
date: '2019-10-22 17:54:46'
updated: '2019-10-24 11:15:47'
tags: [Troubleshooting, Kubernetes]
permalink: /articles/2019/10/22/1571738086097.html</h2>

<img src="https://img.hacpai.com/bing/20181108.jpg?imageView2/1/w/960/h/540/interlace/1/q/100" alt="" />

<h3>问题解决思路</h3>

最近由于工作需要我又重新部署了一遍K8S,这次我使用了最新版本1.16.2来部署。
刚开始都顺风顺水，最后节点起来了，我想把仪表盘给装了，结果就出问题了。
首先是发现创建完用户登录进去后出现 404 NOT FOUND,请求的资源尚未找到，错误截图如下：
<img src="http://lazytoki.cn/wp-content/uploads/2020/02/1-5f9af1d3.jpg" />
于是我去官方的<a href="https://github.com/kubernetes/dashboard/issues">dashboard的issue</a>寻找答案，我在官方issue里发现有一些人遇到和我一样的问题，是因为现在官方的dashboard的yaml文件已经升级到了2.X版本，很多人拿着1.X的版本部署dashboard导致和1.16.2版本的K8S出现不兼容现象（ps: 官方主页更新真慢）。

于是我根据官方的给出的<a href="https://github.com/kubernetes/dashboard">教程</a>，我部署了官方最新的<a href="[https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml](https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml)">dashboard yaml文件</a>，并且pod和service都是处于running状态。
接着我跟着以下步骤：

<strong>IMPORTANT:</strong> Read the <a href="https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/README.md">Access Control</a> guide before performing any further steps. The default Dashboard deployment contains a minimal set of RBAC privileges needed to run.

$ kubectl proxy

Now access Dashboard at:
<a href="http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/"><code>http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/</code></a>.

<strong>然而不幸的是还是无法访问dashboard界面，甚至连登录界面都进不了了。</strong>
issue里有许多人都和我一样的情况，然而其他人给了好多意见，但是还是无法有效解决，而且官方匆匆把这些issue给关闭了（摊手）。

最后我通过在群里询问，有一位大佬给出意见叫我把dashboard通过<strong>nodeport</strong>的方式进行访问才解决问题。
具体如下：
记得填添加admin用户：

<pre><code>kubectl create -f admin.yaml </code></pre>

<pre><code>apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding 
metadata: 
  name: admin-user
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system</code></pre>

已经创建好的admin.yaml:
<a href="https://img.hacpai.com/file/2019/10/admin-6c006f36.zip">admin.zip</a>

首先修改官方的2.X版本的yaml，在这个地方修改加粗部分，添加nodeport部分：

<pre><code>kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30000
  selector:
    k8s-app: kubernetes-dashboard
      type: NodePort
---</code></pre>

这是一份改好的danshboard yaml文件：
<a href="https://img.hacpai.com/file/2019/10/recommended-0e51340b.zip">recommended.zip</a>

然后我们重新部署dashboard，接着重新访问，发现可以成功登录且能够查看pod：
<img src="http://lazytoki.cn/wp-content/uploads/2020/02/2-098fb1d5.jpg" />

哎 就这么坑爹:(
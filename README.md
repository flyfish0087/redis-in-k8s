#Redis in K8s


<img src="https://github.com/marscqy/redis-in-k8s/blob/master/k8s-logo.png" width="100px" style="float:left" /><img src="https://github.com/marscqy/redis-in-k8s/blob/master/redis-logo.jpg" width="100px" style="margin-left:70px;float:left"/>


-----

   
这是一个帮助你在Kubernetes(K8S)环境中搭建redis集群和哨兵模式的样例。  

这里有三个文件夹目录和若干yaml配置文件，他们都是来帮助搭建redis环境的。(如果需要使用statefulset，请将你的k8s版本提升至1.5以上~)

images 文件夹中包含了一个Dockerfile，你可以使用一下命令来创建镜像。语法请参考搜索Docker。redis环境启动规则在run_new.sh 脚本中。
```
docker build -t $YOUR_TAG .
```

k8s_installer 是一个在单节点上安装kubernetes的脚本。使用这个脚本你首先得能连网，因为我没有把其中的rpm包全部下载下来。


redis_cluster_installer 是一个在CentOS 7 下搭建redis集群的脚本，后续我会优化。

-----


***目前发现2个问题： 1.K8S 集群外如何访问Redis，仅仅添加一个NodePort Service 远远不够   2.Cluster 模式情况下 可扩展性不够，增删节点做的不完善~***

问题1解决:方法1.自己写网络组件啦~,这个要求比较高,看方法2.
         方法2.修改sf-redis-cluster.yaml,思路是使用宿主机网络.为pod添加以下两个属性.但是这里有局限性,就是你如果想搭建集群,就必须要有3个以上的K8S从节点 (redis 集群搭建条件:必须有3个或者三个以上的主节点.),因为使用了hostnetwork 之后,一个node 上只有一个redis-cluster 的pod.
      ```
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      ```

redis 在K8S中的性能损耗:
在我本地环境,每台虚拟机条件一样,4core 的cpu,7G的内存,在k8s中会损失24%的性能,主要性能损耗在网络组件上,所以网络组建推荐是calico.

环境
---
k8s 高于 **1.5** 版本 因为要用statefulset 嘛

1.5 环境下删除
每个statefulset 下面的
```
        securityContext:
          capabilities: {}
          privileged: true
```
Dockfile 
---
基于alpine3.6  redis的版本为4.0.1 
修改时区为东八区

shell 脚本 ^M 错误?
---
在打镜像之前先格式化下shell脚本
step1:vi or vim
step2: set ff=unix
step3: 保存 


这么多yaml?
---
sf 表示statefulset
svc 表示service

- sentinel 所需: 
    - sf-redis-master.yaml
    - sf-redis-slave.yaml
    - sf-redis-sentinel.yaml
    - svc-redis-master.yaml
    - svc-redis-slave.yaml
    - svc-redis-sentinel.yaml

- cluster 所需:
    - sf-redis-cluster.yaml
    - sf-redis-cc.yaml
    - svc-redis-cluster.yaml
    - svc-redis-cc.yaml
    
    
如有疑问,请联系我:  
email:cqyisbug@163.com  
qq:377141708  
wechat:antscqy  

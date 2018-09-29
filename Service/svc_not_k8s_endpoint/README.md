### 没有selector 的Service
Service 抽象了该如何访问Kubernetes Pod， 但也能够抽象其它类型的backend，例如：
* 希望在生产环境中使用外部的数据库集群，但测试环境使用自己的数据库。
* 希望服务指向另一个Namespace中或其它集群中的服务
* 正在将工作负载转移到Kubernetes集群，和运行在Kubernetes集群之外的backend
  
  在任何这些场景中，都能够定义没有selector的Service.

  #### Example httpd service
  在其它节点创建httpd服务，服务中制作一个http 首页服务，k8s平台上创建service, endpoint, endpoint与service挂钩。endpoint中的subenetes address下的ip地址配置httpd 服务的ip地址。

  参考地址：https://kubernetes.io/cn/docs/concepts/services-networking/service/
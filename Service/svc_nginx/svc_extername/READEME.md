k8s service 支持使用外部dns解析的方式访问外部服务。 用户访问K8S里面的svc域名时，返回一个外部的dns域名，外部的dns域名解析到最终服务。

### K8S发布服务
* ClusterIP: 通过机器的内部IP暴露服务，选择该值，服务只能够在集群内部访问，这也是默认的ServiceType
* NodePort: 通过Node上的IP和静态端口(NodePort) 暴露服务。NodePort服务会到路由到ClusterIP服务，这个ClusterIP服务会自动创建。通过请求<NodeIP>:<NodePort>，可以从集群的外部访问一个NodePort服务
* LoadBalancer: 使用云提供商的负载均衡器，可以向外保留服务。外部的负载均衡器可以路由到NodePort服务和ClusterIP服务
* ExternalName: 通过返回CNAME和它的值，可以将服务映射到externalName字段的内容(例如: foo.bar.example.com)


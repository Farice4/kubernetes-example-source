Service 可以通过yaml文件编辑好，通过kubectl create -f xxx.yaml来创建，也可以基于已经创建好的Pod/Deployment来创建。

通过已创建好的Pod/Deployment来创建，命令：
kubectl expose deployment/my-nginx   （deployment/my-nginx是已创建好了的deployment my-nginx)
kubectl expose pod/my-nginx           (pod/my-nginx是已创建好了的pod my-nginx)
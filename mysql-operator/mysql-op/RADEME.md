### Mysql Operator Install

#### Create deploy namespace
kubectl create namespace mysql-operator
kubectl create namespace database

#### Deploy mysql operator
helm install ./mysql-operator --namespace mysql-operator --name mysql-operator

#### Deploy mysql cluster
> Create root password
kubectl create secret generic mysql-root-password --from-literal=password="password"

> Create role
kubectl create -f role.yml

> Create mysql conf
kubectl create -f mysql-conf.yaml

> Create cluster
kubectl create -f cluster.yml

> Create mysql cluster svc
kubectl create -f mariadb-svc.yml

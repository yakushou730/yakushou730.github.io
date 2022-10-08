---
title: "Local development setting"
authors: [yakushou730]
date: 2022-05-08T18:33:11+08:00
description: "local development setting"
tags: ["local"]
draft: false
---

整理

| service    | local | k8s  | username | password|
|------------|-------|------|---|---|
| mysql      | 30000 | 3306 | root | secret |
| postgresql | 30001 | 5432 | root | secret |
 | redis      | 30002 | 6379 |   |   |
 | etcd | 30003 | 2379, 2380 | root | |

## 本機開發安裝項目

#### zsh
1. 安裝 powerlevel10k 主題
   - [site](https://github.com/romkatv/powerlevel10k#oh-my-zsh)

#### golang
1. 先至 golang 官網安裝一版新的 golang (因為要有 golang 才可以 build golang)
   - [website install](https://go.dev/dl/)
2. 安裝 GVM 來管理 golang 套件
   - [site](https://github.com/moovweb/gvm)
3. GVM 安裝後就可以安裝各種對應版本的 golang
   - intel 是 amd64
   - M1 (apple silicon) 是 arm64
4. 安裝 Goland
   - [website install](https://www.jetbrains.com/go/)

Optional:

1. 安裝 [expvarmon](https://github.com/divan/expvarmon)
   - 用來看 pprof 的效能
2. 安裝 [hey](https://github.com/rakyll/hey)
   - 用來做壓測的工具
3. 安裝 [staticcheck](https://staticcheck.io/)
   - 一種 golang 的 linter 分析工具

#### git
1. 安裝 cz-conventional 
   - [site](https://github.com/commitizen/cz-conventional-changelog)
   - 這樣可以用 `git cz` 來做到統一的 commit
```shell
npm install -g commitizen cz-conventional-changelog
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
```

#### K8S
1. 安裝 docker
   - [website install](https://www.docker.com/get-started/)
2. 安裝 kubectl
   - [site](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/)
3. 安裝 helm
   - [site](https://helm.sh/docs/intro/install/)
4. 安裝 k3d
   - [site](https://k3d.io/v5.4.1/)
5. 下指令建出 local 端的 cluster
```shell
k3d cluster create local-cluster --registry-create local-registry -p "30000-30200:30000-30200@server:0"
```

如果要把 local image 載入到 k3d 的話，指令為
```shell
k3d image import -c local-cluster service-amd64:1.0
```

6. 安裝 kustomiize
   - [site](https://kubectl.docs.kubernetes.io/installation/kustomize/)

#### 其他服務
1. 透過 helm 安裝 mysql
   - [site](https://artifacthub.io/packages/helm/bitnami/mysql) 
```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install local-mysql \
  --set auth.rootPassword=secret,auth.database=db_dev \
    bitnami/mysql
```
Result
```shell
NAME: local-mysql
LAST DEPLOYED: Sun May  8 19:04:49 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mysql
CHART VERSION: 8.8.26
APP VERSION: 8.0.28

** Please be patient while the chart is being deployed **

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace default

Services:

  echo Primary: local-mysql.default.svc.cluster.local:3306

Execute the following to get the administrator credentials:

  echo Username: root
  MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default local-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run local-mysql-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mysql:8.0.28-debian-10-r23 --namespace default --command -- bash

  2. To connect to primary service (read/write):

      mysql -h local-mysql.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"



To upgrade this helm chart:

  1. Obtain the password as described on the 'Administrator credentials' section and set the 'root.password' parameter as shown below:

      ROOT_PASSWORD=$(kubectl get secret --namespace default local-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)
      helm upgrade --namespace default local-mysql bitnami/mysql --set auth.rootPassword=$ROOT_PASSWORD
```
建立一個 node port 來通 mysql

local-mysql-svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: local-mysql-svc
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: local-mysql
    app.kubernetes.io/managed-by: Helm
  ports:
    - port: 3306
      nodePort: 30000
      targetPort: 3306
      protocol: TCP
```
這樣可以透過 localhost:30000 來連接到 k3d 內的 3306

2. 透過 helm 安裝 postgresql
   - [site](https://artifacthub.io/packages/helm/bitnami/postgresql)
```shell
helm install my-release bitnami/postgresql
helm install local-postgresql \
  --set global.postgresql.auth.username=root,global.postgresql.auth.password=secret,global.postgresql.auth.database=db_dev \
    bitnami/postgresql
```
Result
```shell
NAME: local-postgresql
LAST DEPLOYED: Sun May  8 22:05:30 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 11.1.6
APP VERSION: 14.2.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    local-postgresql.default.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_ADMIN_PASSWORD=$(kubectl get secret --namespace default local-postgresql -o jsonpath="{.data.postgres-password}" | base64 --decode)

To get the password for "root" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default local-postgresql -o jsonpath="{.data.password}" | base64 --decode)

To connect to your database run the following command:

    kubectl run local-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:14.2.0-debian-10-r25 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host local-postgresql -U root -d db_dev -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/local-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U root -d db_dev -p 5432
```

建立一個 node port 來通 postgresql

local-postgresql-svc

```yaml
apiVersion: v1
kind: Service
metadata:
   name: local-postgresql-svc
spec:
   type: NodePort
   selector:
      app.kubernetes.io/name: postgresql
      app.kubernetes.io/instance: local-postgresql
      app.kubernetes.io/managed-by: Helm
   ports:
      - port: 5432
        nodePort: 30001
        targetPort: 5432
        protocol: TCP
```
這樣可以透過 localhost:30001 來連接到 k3d 內的 5432

3. 透過 helm 安裝 redis 
   - [site](https://artifacthub.io/packages/helm/bitnami/redis)

```shell
helm install local-redis bitnami/redis
```
Result
```shell
NAME: local-redis
LAST DEPLOYED: Sun May  8 22:13:31 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: redis
CHART VERSION: 16.5.2
APP VERSION: 6.2.6

** Please be patient while the chart is being deployed **

Redis&trade; can be accessed on the following DNS names from within your cluster:

    local-redis-master.default.svc.cluster.local for read/write operations (port 6379)
    local-redis-replicas.default.svc.cluster.local for read-only operations (port 6379)



To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default local-redis -o jsonpath="{.data.redis-password}" | base64 --decode)

To connect to your Redis&trade; server:

1. Run a Redis&trade; pod that you can use as a client:

   kubectl run --namespace default redis-client --restart='Never'  --env REDIS_PASSWORD=$REDIS_PASSWORD  --image docker.io/bitnami/redis:6.2.6-debian-10-r146 --command -- sleep infinity

   Use the following command to attach to the pod:

   kubectl exec --tty -i redis-client \
   --namespace default -- bash

2. Connect using the Redis&trade; CLI:
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h local-redis-master
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h local-redis-replicas

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/local-redis-master : &
    REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h 127.0.0.1 -p
```
建立一個 node port 來通 redis

local-redis-svc

```yaml
apiVersion: v1
kind: Service
metadata:
   name: local-redis-svc
spec:
   type: NodePort
   selector:
      app.kubernetes.io/name: redis
      app.kubernetes.io/instance: local-redis
      app.kubernetes.io/managed-by: Helm
   ports:
      - port: 6379
        nodePort: 30002
        targetPort: 6379
        protocol: TCP

```
這樣可以透過 localhost:30002 來連接到 k3d 內的 6379

4. 透過 helm 安裝 etcd
   - [site](https://artifacthub.io/packages/helm/bitnami/etcd)

```shell
helm install local-etcd bitnami/etcd --set auth.rbac.create=false
```
Result
```shell
NAME: local-etcd
LAST DEPLOYED: Sun Oct  9 02:07:38 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: etcd
CHART VERSION: 6.13.5
APP VERSION: 3.5.2

** Please be patient while the chart is being deployed **

etcd can be accessed via port 2379 on the following DNS name from within your cluster:

    local-etcd.default.svc.cluster.local

To create a pod that you can use as a etcd client run the following command:

    kubectl run local-etcd-client --restart='Never' --image docker.io/bitnami/etcd:3.5.2-debian-10-r23 --env ETCDCTL_ENDPOINTS="local-etcd.default.svc.cluster.local:2379" --namespace default --command -- sleep infinity

Then, you can set/get a key using the commands below:

    kubectl exec --namespace default -it local-etcd-client -- bash
    etcdctl  put /message Hello
    etcdctl  get /message

To connect to your etcd server from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/local-etcd 2379:2379 &
    echo "etcd URL: http://127.0.0.1:2379"
```
建立一個 node port 來通 etcd

local-etcd-svc

```yaml
apiVersion: v1
kind: Service
metadata:
   name: local-etcd-svc
spec:
   type: NodePort
   selector:
      app.kubernetes.io/name: etcd
      app.kubernetes.io/instance: local-etcd
      app.kubernetes.io/managed-by: Helm
   ports:
      - port: 2379
        nodePort: 30003
        targetPort: 2379
        protocol: TCP

```
這樣可以透過 localhost:30003 來連接到 k3d 內的 2379

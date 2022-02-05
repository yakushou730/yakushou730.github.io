---
title: "Certified K8s Administrator"
authors: [yakushou730]
date: 2022-01-09T21:07:35+08:00
description: "Certified K8s Administrator"
tags: ["programming","k8s"]
draft: false
---

## Core Concepts

> k8s 的目的是要以自動化的形式管理 container 形式的 app
> 
> 讓使用者可以簡單的部署，並讓各服務的溝通可以簡單化

解說:

假設現在有兩艘船，cargo ship 和 control ship

- Cargo Ship
  - 負責實際運載 container 的工作
  - 每艘船的船長，負責管理船上的 activities，負責接收當 master 需要裝載 container 時的資訊，
    並回報給 master 這艘船的狀態和 container 的狀態
- Control Ship
  - 負責監測和管理 cargo ships
  - 透過 control plane 元件負責管理 cargo ship 的大小事
  - ETCD: 每天都有很多 container 要被載到 cargo ships 上，所以需要紀錄不同的船上有哪個 container，包含載入的時間
    而這些是以 key-value 的形式被保存的
  - kube-scheduler: 當 cargo ship 來的時候，要負責判斷並把對應的 container 奘載給他
  - Controller manager:
    - 指派特殊工作，例: operation team 要處理交通控制，或是 container 壞掉的時候，怎麼樣建立一個可以用的 container
    - service office 用來讓各部門溝通，或是讓不同的船之間溝通
- 最高部門: kube-apiserver，各部門之間管理的最高層級，負責 Cluster 的工作協調

對應到 k8s 的名詞
- Worker Nodes: host Application as Containers
  - Container Runtime Engine: 讓 node 可以處理 container
  - kubelet: 船長的角色，是一個裝在 node 上的 agent，接收來自 kube-apiserver 的指令，在 node 上 deploy 或 destroy container
  - kube-proxy: 讓 nodes 之間可以知道要溝通的服務在哪邊 (web container 對應 db container)
- Master: Manage, Plan, Schedule, Monitor Nodes
  - ETCD Cluster: 一個 key-value 的資料庫
  - kube-scheduler: 負責把 container 起在對應的 node 上
    - 可能包含的條件有 resource requirements for nodes capacity, policies, ...etc
  - Controller manager
    - Node-Controller: 處理新增/刪除 node，以及處理出問題的 node
    - Replication-Controller: 確保期望的 container 數量都是對的
  - kube-apiserver: 管理 cluster 的最高層級，提供 k8s API 來讓使用者操作
    定時監控 kubelet 上 node 和 container 的狀態報告 

## ETCD

> ETCD is a distributed reliable key-value store that is simple, secure & fast

default port: 2379

```shell
# 安裝完後執行 binary 檔案
./etcd
# 設定 key-value
./etcdctl set key1 value1
# 取得 value
./etcdctl get key1 # 得到 value1
# help
./etcdctl
```

所有 k8s 上要紀錄的資料都會更新在 etcd datastore 上，然後才會在 cluster 上生效

不同的安裝方式
1. 自己獨立安裝的，把 binary 檔案設定成 service 並執行
   - 需要注意的欄位 `--advertise-client-urls https://${INTERNAL_IP}:2379`，這是要設定在 apiserver 上的 url
2. 透過 kubeadm 安裝的
   - 會把 etcd 做成 pod 安裝在 cluster 上 kube-system 的 namespace 中
   - 所以可以進到 pod 裡面並操作 key-value store
   - 多 cluster 的情況，要注意的欄位是 `--initial-cluster controller-0=https://${CONTROLLER0_IP}:2380,controller-1=https://${CONTROLLER1_IP}:2380`
```shell
kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```

## Kube API Server
當執行 `kubectl` 的時候，就是在發 API 給 kube-apiserver

1. 以 `kubectl get pods` 為例

kube-apiserver 會先做 authenticate 和 validate，再去 ETCD 拿 pods 資料回給 使用者

2. 以建立 pods 為例

kube-apiserver 做完 authenticate 和 validate 以後，會先建立一個 pod (還不在任何 node 上)

先更新到 ETCD 上，再回應 使用者 `Pod created!`

接著 scheduler 持續監控 apiserver，發現了這個沒有在 node 上的 pod

scheduler 識別出正確的 node 來放這個新的 pod，並通知 apiserver

apiserver 再把這資訊寫進 ETCD 並且把資訊傳給對應 worker node 的 kubelet

收到資訊的 node 就建立這個 pod 出來 (透過 Container Runtime Enginer 部署 image)

完成後 kubelet 把資訊回傳給 kube-apiserver，apiserver 再把資訊寫進 ETCD

> 可以注意到幾乎全部的溝通都要通過 apiserver

整合剛剛的步驟
1. Authenticate User
2. Validate Request
3. Retrieve data
4. Update ETCD
5. Scheduler
6. Kubelet

## Kube Controller Manager
Controller manager 管理了許多 k8s 不同的 controller

Controller 可以負責的任務
- 當有新船抵達或是有船離開，監控並做出必要的動作
- 管理船上的 container

簡化:
1. 狀態監控 (Watch status)
2. 情況補救 (Remediate situation)

> k8s 中，controller 是一個程序 會持續監控系統中的 component 狀態
> 
> 並做出處置讓整個系統可以處在 desired state

Node Controller
- 監控 node 狀態，以及做出處置讓 application 可以持續運作
- 中間要透過 apiserver
- node controller 每 5秒 確認一次 node 的狀態
- 如果 node controller 超過 40秒 讀不到 node 的 heartbeat，那該 node 會被標為 unreachable
- 如果標為 unreachable 超過 5分鐘，就註銷那個 node 上面的 pod，並移去其他健康的 node (如果那個 pod 是在 replicaset 之中的話)

Replication Controller
- 監控 replicaset 狀態，並確保預期的 pod 數量正常的，如果有 pod 掛了就建立新的

> 一堆 controller 組合起來，就像 k8s 的大腦，整個包起來就是 kube-controller-manager

## kube scheduler
負責決定 哪個 pod 要去哪個 node，但不是 scheduler 操作把 pod 放去哪 (這是 kubelet 的工作)

> kubelet 會負責建立 pod

為什麼需要 scheduler ?
- 因為每個 container 的大小或需要的環境都不太一樣，所以需要特別處理來讓生產效益最高

scheduler 的決定流程
1. 先看 container 的環境需求，首先 filter nodes，讓不滿條件的 node 先排除
   - 比如說 CUP, memory 不滿足...等等
2. 再做 rank nodes，會有一個 priority function 來對每個 node 做評分

## Kubelet
kubelet 是 worker node 上面的船長，負責和 apiserver 溝通

負責 container 的裝載，並回報給 master

負責
1. Register Node (to cluster)
2. Create Pod
3. Monitor Nodes & Pods

## Kube Proxy
Pod Network 讓 cluster 內的 pod 可以互相溝通

先前學過的 相互溝通可以透過 Service object

network 不是實體，所以不會建立 pod，而是紀錄在 memory 裡面

kube-proxy 是一個 process，在每個 node 上執行

每當有新的 service 被發現，就會在每個 node 上建立對應的 rules 來導流量給這些 service 到 pod
- IP table

## Namespace
k8s 常用的 namespace
- default
- kube-system
- kube-public

namespace 是可以有 policy 權限控管的

也可以對 namespace 做資源上的限制

舉例:
namespace: default
  - web-pod
  - db-service
  - web-deployment
namespace: dev
  - db-service
  - web-pod
 
如果 default 內的 web-pod 要對同 namespace 的 db-service 存取，使用 `mysql.connect("db-service")`

如果 default 內的 web-pod 要對 dev namespace 的 db-service 存取，使用 `mysql.connect("db-service.dev.svc.cluster.local")`
- `cluster.local` 是 default domain name for k8s
- `svc` 是 subdomain for service
- `dev` 是 namespace
- `db-service` 是 service name

```shell
# 指令建立 namespace 名稱 dev-ns
kubectl create ns dev-ns
# 列出 namespace default 的 pods
kubectl get pods
# 列出指定 namespace 的 pods
kubectl get pods --namespace=kube-system
# 建立 pod 到指定的 namespace
kubectl create -f pod-definition.yaml --namespace=dev
```

在 pod-definition.yaml 裡面指定 namespace
```yaml
# 假設指定 namespace dev
metadata:
  namespace: dev
```

切換 namespace
```shell
# context 是 cluster
kubectl config set-context $(kubectl config current-context) --namespace=<target namspace>
# 當前 cluster
kubectl config set-context --current --namespace=<target namespace>
```

```shell
# 列出所有 namespace 下的 pod
kubectl get pods --all-namespaces
```

```shell
# 常用指令
# --no-headers 是指不要印出 header 行

# 其他組合
kubectl get ns --no-headers | wc -l
kubectl get po -n research --no-headers | wc -l
```

> 練習中有一個範例是在 pod 顯示頁面輸入 host 和 port 來連線到 redis
> 
> host 要是 能連到 db pod 的 service name，而不是 db 的 pod name

## Imperative vs Declarative
以搭計程車為例:
- Imperative: 告訴司機先往左，再往右，再直走... 等等等步驟
  - 重點是 how to do
- Declarative: 直接告訴司機終點地址
  - 重點是 最後結果 (what to do)

以 Infrastructure as Code 為例

Imperative
```
1. Provision a VM by the name 'web-server'
2. Install NGINX Software on it
3. Edit configuration file to use port 8080
4. Edit configuration file to web path '/var/www/nginx'
5. Load web pages to '/var/www/nginx' from GIT Repo - X
6. Start NGINX server
```

Declarative
```yaml
VM NAME: web-server
Package: nginx
Port: 8080
Path: /var/www/nginx
Code: GIT Repo - X
```

以 Kubernetes 為例

Imperative
- 透過數個 `kubectl create` 指令來建立 pod / deployment ....等等等
- 要自己手動下指令調整各種設定
- 不需要 configuration file
- 很難 track，換個人就不知道怎麼操作了

Declarative
- 只要透過 `kubectl apply` 使用定義好的 configuration file，其他邏輯都不用管

## Kubectl apply command
系統會比對設定檔的差異

`Local file` <-> 比對 k8s `Live object configuration` (in memory)

如果不一樣的話就會更新，並把最新的版本更新到 `Last applied Configuration`
- 這資料是存在 k8s Live Object Configuration 的 `metadata.annotations.kubectl.kubernetes.io/last-applied-configuration`
- 要透過 `kubectl apply` 才會生成，`kubectl create` 或 `kubectl replace` 是不會更新這資料的

## Manual Scheduling
Pod 在 k8s 上的 live object configuration 上面會被賦予 `nodeName`

而 scheduler 是專門查 pod 上誰沒有 nodeName，發現的話就用自己的演算法賦予 node 上去 

> k8s 不給修改已經賦予 nodeName 的 pod 上面的 nodeName

pod-bind-definition.yaml
```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node2
```

再把上面的 yaml 轉成 json 後 post 給 k8s 做處理

```shell
# 檢查 scheduler 是否在 k8s 上
kubectl get pods --namespace kube-system
```

## Labels and Selectors
 用來分群的

```shell
# 透過 selector 來 filter 特定 label 的資料
kubectl get pods --selector app=App1
# 多個 labels 用逗號隔開
kubectl get pods --selector env=prod,bu=finance,tier=frontend
```

**Annotations**

用來儲存其他資訊

## Taints and Tolerations
解釋如何決定並把 pod 放到對應 node 上的機制 (限制哪些 pod 可以被 scheduled)

如果要避免把 pod 放到某個 node，就對那個 node 做 taint (又叫污點)

再來是把想要有抵抗力的 pod 做 toleration，這樣就可以被放到那個 node 裡面

所以
- taint 是設定在 node 上
- toleration 是設定在 pod 上

```shell
# 語法
# taint-effect 是指 如果 POD 無法 tolerate 這個 taint 的話會怎麼樣
# 其中 taint-effect 可以為 NoSchedule | PreferNoSchedule | NoExecute
# NoSchedule: 不會被 schedule 到這個 node
# PreferNoSchedule: 盡量不要把 pod 放到這個 node
# NoExecute: 不會被 schedule 到這個 node 而且已經在上面的 pod 也會被趕走
kubectl taint nodes node-name key-value:taint-effect
# 範例
kubectl taint nodes node1 app=blue:NoExecute
```

對應在 pod spec 上的設定

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx-controller
      image: nginx
  tolerations: # 注意要用 double quotes
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

> master node 天生就被給予了 taint，pod 不會被放上去

```shell
# 查看 taint
kubectl describe node kubemaster | grep Taint
# 移除 taint [原本的 taint 是 node-role.kubernetes.io/master:NoSchedule]
# 看起來是最後面加個減號可以拿掉
kubectl taint nodes controlplane node-role.kubernetes.io/master:NoSchedule-
```

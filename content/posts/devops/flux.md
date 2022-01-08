---
title: "flux"
authors: [yakushou730]
date: 2022-01-02T21:49:17+08:00
description: "flux"
tags: ["programming","devops"]
draft: false
---

目前看起來像是 K8S 的 CD 工具

## Flux 是什麼?
Flux 是一組用於 k8s 的 delivery solution
- Flux 提供 GitOps for apps 和 infrastructure
  - Flux 和 Flagger 提供 canaries/feature flags/AB rollouts 來部署 app
  - 可用來管理 k8s 資源
- 推 git 以後，剩下的 Flux 會處理
  - 可套用 CD (by Flagger) 和 PD (progressive delivery)
  - container image 發生更新後，Flux 可以自動推 git
- Flux 相容現有的工具
  - 含 github/gitlab/bitbucket/s3...，以及全部有提供 cli 的 provider
- Flux 可用於任何 k8s 和所有常見的 k8s 工具
  - Kustomize/Helm/RBAC...等等
- Flux 處理 multi-everything
  - 可以處理多數個 cluster
- Flux 有告警和提示
  - 只要 git push，可以得到通知
- 使用者信任 Flux
  - Flux 是 CNCF 孵化出的專案
- Flux 有友善的社群
  - 任何人都可以貢獻


Flux 是一組 k8s 用的工具，可以 sync 資源設定 (像 git repository)

且如果有新 code deploy 的時候會自動更新 configuration
- Declarative
  - 在 git 有描述系統的 desired state，包含 apps/configuration/dashboard/monitoring/everything else
- Automated
  - 強制用 YAML 來宣告系統，不再需要透過 kubectl (所有的更動都會自動 sync) 
- Auditable
  - 所有的事情都透過 PR，透過 git history 可以隨時回復狀態 as snapshot
- Designed for k8s
- Out-of-the-box-integration
  - 幾乎支援全部的平台 kustomize/helm/github/gitlab/harbor/custom webhooks/notifications to most team communication platforms
- Extensible
  - 可簡單建立 CD solution

## Get Started

準備事項 (記得切換到要用的 k8s cluster)
```shell
export GITHUB_TOKEN=<github token with repo permission>
export GITHUB_USER=<github username>
# flux 的前置檢查
flux check --pre
```

官方建議在本機用 kind 安裝 k8s cluster 後操作

再來是在 cluster 上安裝 flux
```shell
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fleet-infra \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal
```

安裝了什麼可以用指令看一遍
```shell
kubectl get all -n flux-system
# result
NAME                                           READY   STATUS    RESTARTS   AGE
pod/helm-controller-779b58df6b-pwhkv           1/1     Running   8          30h
pod/kustomize-controller-5db6bfc56d-6px6n      1/1     Running   8          30h
pod/notification-controller-7ccfbfbb98-htjc6   1/1     Running   8          30h
pod/source-controller-565f8fbbff-9wd77         1/1     Running   9          30h

NAME                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/notification-controller   ClusterIP   10.96.21.107    <none>        80/TCP    30h
service/source-controller         ClusterIP   10.96.53.181    <none>        80/TCP    30h
service/webhook-receiver          ClusterIP   10.96.219.192   <none>        80/TCP    30h

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helm-controller           1/1     1            1           30h
deployment.apps/kustomize-controller      1/1     1            1           30h
deployment.apps/notification-controller   1/1     1            1           30h
deployment.apps/source-controller         1/1     1            1           30h

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/helm-controller-779b58df6b           1         1         1       30h
replicaset.apps/kustomize-controller-5db6bfc56d      1         1         1       30h
replicaset.apps/notification-controller-7ccfbfbb98   1         1         1       30h
replicaset.apps/source-controller-565f8fbbff         1         1         1       30h
```

```shell
kubectl api-resources | grep flux
# result
helmreleases                      hr           helm.toolkit.fluxcd.io/v2beta1           true         HelmRelease
kustomizations                    ks           kustomize.toolkit.fluxcd.io/v1beta2      true         Kustomization
alerts                                         notification.toolkit.fluxcd.io/v1beta1   true         Alert
providers                                      notification.toolkit.fluxcd.io/v1beta1   true         Provider
receivers                                      notification.toolkit.fluxcd.io/v1beta1   true         Receiver
buckets                                        source.toolkit.fluxcd.io/v1beta1         true         Bucket
gitrepositories                   gitrepo      source.toolkit.fluxcd.io/v1beta1         true         GitRepository
helmcharts                        hc           source.toolkit.fluxcd.io/v1beta1         true         HelmChart
helmrepositories                  helmrepo     source.toolkit.fluxcd.io/v1beta1         true         HelmRepository
```

```shell
kubectl get gitrepositories.source.toolkit.fluxcd.io -n flux-system
# result
NAME          URL                                            READY   STATUS                                                              AGE
flux-system   ssh://git@github.com/yakushou730/fleet-infra   True    Fetched revision: main/f03ba1bebe1ed2d251aaa189efbb6ad3c84a28d9     30h


kubectl get kustomizations.kustomize.toolkit.fluxcd.io -n flux-system
# result
NAME          READY   STATUS                                                              AGE
flux-system   True    Applied revision: main/f03ba1bebe1ed2d251aaa189efbb6ad3c84a28d9     30h
```

執行後的流程
1. 在個人的 github 上生成一個 fleet-infra 的 repo
2. flux 生成需要用到的 flux component 的 manifest，並推到 repo 內 (repo/clusters/my-cluster) 做追蹤
3. 依照 manifest 部署 Flux Components 到對應的 k8s cluster，讓 cluster 會去對這個 repo 做 sync

再來的動作
1. 把該 repo 拉回 local 要操作用的
2. 用 flux 指令建立一個 GitRepository manifest，指向 podinfo repository 的 master branch
  - 範例是用 [github.com/stefanprodan/podinfo](github.com/stefanprodan/podinfo)
```shell
flux create source git podinfo \
  --url=https://github.com/stefanprodan/podinfo \
  --branch=master \
  --interval=30s \
  --export > ./clusters/my-cluster/podinfo-source.yaml
```

建立出 podinfo-srouce.yaml
```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  name: podinfo
  namespace: flux-system
spec:
  interval: 30s
  ref:
    branch: master
  url: https://github.com/stefanprodan/podinfo
```
3. commit fleet-infra repo 以後推到 github
```shell
# 確認 sources git
flux get sources git
# result
NAME       	READY	MESSAGE                                                          	REVISION                                       	SUSPENDED
flux-system	True 	Fetched revision: main/f03ba1bebe1ed2d251aaa189efbb6ad3c84a28d9  	main/f03ba1bebe1ed2d251aaa189efbb6ad3c84a28d9  	False
podinfo    	True 	Fetched revision: master/132f4e719209eb10b9485302f8593fc0e680f4fc	master/132f4e719209eb10b9485302f8593fc0e680f4fc	False
```



4. 部署 podinfo application，用 flux create 建立 `kustomization` 作為部署
```shell
flux create kustomization podinfo \
  --target-namespace=default \
  --source=podinfo \
  --path="./kustomize" \
  --prune=true \
  --interval=5m \
  --export > ./clusters/my-cluster/podinfo-kustomization.yaml
```

podinfo-kustomization.yaml
```yaml
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: podinfo
  namespace: flux-system
spec:
  interval: 5m0s
  path: ./kustomize
  prune: true
  sourceRef:
    kind: GitRepository
    name: podinfo
  targetNamespace: default
```
5. commit fleet-infra repo 以後推到 github

![flux-get-started.jpg](/k8s/flux-get-started.jpg)

## 核心概念
**Sources**
Source 在 repo 定義了系統和需求的 desired state

Source 會生產出 artifact 給 Flux 組建操作，像是套用給 cluster

Source 的改動會被識別出來，並生產出新的 artifact

所有的 Source 會被 K8s 稱作 Custom Resource

如
- GitRepository
- HelmRepository
- Bucket

> 管資源來源
> 
> 建立 GitRepository source 只會讓 flux 從 git repository 追蹤並拉最新的版本下來

**Reconciliation**
Reconciliation 是用來確保更新 cluster 上面是 desired state

- `HelmRelease` reconciliation
  - 確保 Helm release 的狀態匹配 
- `Bucket` reconciliation
  - 下載封存 Bucket 的內容，紀錄觀察到的版本
- `Kustomization` reconciliation
  - 確保 application 部署到 cluster 上的狀態和 git repository (或 S3) 匹配

**Kustomization**
是一個 custom resource，代表 k8s 資源的 local set

用來讓 Flux 對 Cluster 做 reconcile

即透過 Kustomization 讓 reconcile 可以運作

透過修改 `.spec.interval` 可以調整 reconcile 的間隔

> 管部署
> 
> 要有 kustomization 才會真正部署拉下來的 source

**Bootstrap**
安裝 Flux 元件的程序 (以 GitOps 的方式)

套用 manifest 到 cluster，建立 GitRepository 和 Kustomization 到 Flux 元件

再把 manifest 推到 git repo 

## 特殊名詞
**RBAC (Role-Base Access Control)**

角色的訪問控制，決定能不能訪問 k8s API

**CRD (Custom Resource Define)**
自定義資源，讓開發者可以創建自定義的資源對象

## Kustomization
Kustomization API 定義了一組 pipeline 用來 fetching, decrypting, building, validating 和 applying k8s manifest

**規格**

kustomization 物件定義了 k8s manifest 的 source (透過 source controller)

> `spec.sourceRef` 就是 reference 到 source controller 管理的一個物件
> 
> 當 source 的 revision 改變了，會觸發 event 來讓 kustomize build 和 apply
> 
> 支援的 Source type: GitRepository, Bucket

source 裡會包含 kustomization file 的路徑

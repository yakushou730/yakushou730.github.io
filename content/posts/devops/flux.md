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

準備事項
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

執行後的流程
1. 在個人的 github 上生成一個 fleet-infra 的 repo
2. 在 repo 內加入了 flux component manifest
3. 部署 Flux Components 到對應的 k8s cluster
4. 設定 flux components 組態到 repo/clusters/my-cluster 做追蹤

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


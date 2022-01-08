---
title: "helm"
authors: [yakushou730]
date: 2022-01-06T15:51:08+08:00
description: "helm"
tags: ["programming","k8s"]
draft: false
---

## What is Helm

一般來說，要 deploy 一個服務，即使是簡單的 app

也要建立一整套完整的 k8s 元件

ex: Secret, Deployment, Service, PersistentVolumeClaim... 等等，需要 apply 一堆 yaml 檔案

Helm 被稱為 k8s 的 package manager

> 想像一個遊戲由千百個檔案組成
> 
> 我們不需要自己處理那摩多檔案，透過安裝程式的方式
> 
> 做到一次安裝一個完整的遊戲，對應的檔案也都被安裝到正確的位置
> 
> 而 Helm 就像這樣，一次處理一堆 YAML 檔案和 K8s 物件，來設定 application

Helm 讓我們操作 k8s 的元件就像操作 app 一樣，而不是一堆檔案物件

```shell
# 透過 helm 安裝 wordpress
helm install wordpress
# 修改完 yaml 檔案套用
helm upgrade wordpress
# 可倒退版本
helm rollback wordpress
# 移除 wordpress
helm uninstall wordpress
# debug 資訊 
helm --debug
```

## Helm2 vs Helm3 
Helm3 多了
- RBAC (Role Based Access Control)
- CRD (Custom Resource Definition)
- 提供 snapshot (revision)

Helm3 移除了 Helm2 用的 Tiller

## Helm Components
Charts
- 一包 files
- 包含 Helm 需要的所有 instruction 在 k8s cluster 上建立物件
- 套用 Chart 到 cluster 上的時候，會建立 release (每個 release 都代表一次 chart 的安裝)
- 像 docker 有 docker hub 一樣，helm 也有對應的 online chart repository
  - ArtifaceHUB
- 為了追蹤 helm 對 k8s cluster 做了什麼變更，Helm 會儲存這些資訊 (as metadata)
  - Helm 把這資訊以 k8s secret 的方式存在 k8s cluster 
- yaml file 裡面可以像填空一樣放 template
  - 用來填空的資訊放在 values.yaml
- 我們可以下載到各式各樣的 Helm charts，之後的工作就是透過 values.yaml 填空把 configuration 設定好

```shell
# 安裝 chart
helm install [release name] [chart-name]
```

## Helm charts
一般的 helm chart 都還會包含的 files
- values.yaml: 用來放要嵌入 template 的資訊
- chart.yaml: 標明這份 chart 本身的資訊
  - 包含 apiVersion, appVersion, name...等等

chart.yaml
- apiVersion: 套用的 api 版本 (Helm3 用 v2)
- appVersion: 是指這個 app 要用哪個版本 deploy (ex: wordpress)
- version: 這份 chart 的版本
- name: chart 的名稱
- type:
  - application (default): 指是建立作為 deploying application
  - library: 指提供 utilities 來幫助 building charts
- dependencies: 有相依的套件，同樣以 helm 的方式載入
- keywords: 有點像 tag 的概念，可以被搜尋
- maintainers: 維護者資訊
- home: homepage url
- icon: icon url

![helm-structure.png](/k8s/helm-structure.png)

## Working with Helm: basics

```shell
# help
helm --help
# repo help (支援 sub command 的 help)
helm repo --help
# 在 Artifact Hub 上搜尋 wordpress
helm search hub wordpress
# 在 local repo 搜尋已加入的 wordpress
helm serach repo wordpress
```

`helm reop` 指令是對應 chart repository 的操作
- 包含 add, remove, list...
  - `helm repo list` 列出加入的 repo
  - `helm repo update` 更新加入的 repo 資訊

範例: 安裝 wordpress
[wordpress in Artiface HUB](https://artifacthub.io/packages/helm/bitnami/wordpress)
```shell
# 查看 wordpress 的 readme 
# 把 wordpress chart 路徑加入，這樣之後要抓的話就知道路徑
helm repo add bitnami https://chars/bitnami.com/bitnami
# 安裝 release 名為 my-release 的版本
helm install my-release bitnami/wordpress
# 查看 release
helm list
# 移除 release
helm uninstall my-release
```

> repo 不代表 application
> 
> application 是在該 repo 下面的應用

## Customizing chart parameters
安裝 chart 的時候如果沒指定參數的話是套用全部的 default value

```shell
# 指定參數
# 會 overwrite values.yaml
helm install --set wordpressBlogName="Helm Tutorials" my-release binami/wordpress
             --set wordpressEmail="yakushou730@gmail.com"
# 或是準備好自定義的 custom-values.yaml
helm install --values custom-values.yaml my-release bitnami/wordpress

# 兩段式 release
# 1. 先拉 charts 然後更改內容
# 2. 改完後 release
helm pull bitnami/wordpress
# or
helm pull --untar bitnami/wordpress
# 拉下來後更改 values.yaml，之後release
helm install my-release ./wordpress

# 安裝 bitami repo 下的 apache application，release 名為 amaze-surf
helm install amaze-surf bitnami/apache
```

## Lifecycle management with Helm
```shell
# 指定安裝版本
helm install nginx-release bitnami/nginx --version 7.1.0

# 更新 release 版本
helm upgrade nginx-release bitnami/nginx
helm upgrade dazzling-web bitnami/nginx --version 9
# 查看 release 紀錄
helm history nginx-release
# 退回指定的 release 版本 (by 指定 revision)
helm rollback nginx-release 1
```

## Writing a Helm chart
把寫 helm chart 想像成是寫安裝精靈

也可以做到 在更新某個版本之前 先備份 database (hook)

```shell
# 建立一個新的 charts 框架
helm create nginx-chart
# 如果 templates 裡面的內容用不到的話就拿掉，放上要用的 k8s yaml 檔案
```

```yaml
# 為了讓 chart 可以複用，metadata 的地方名稱可以使用嵌入的
# helm install [ReleaseName] ./chartRepo
name: {{ .Release.Name }}-nginx

# 可組合
name: {{ .Values.image.repository }}:{{ .Values.image.tag }}
```

如果需要用到 Chart.yaml 作為嵌入變數的話，可以使用
- Chart.Name
- Chart.ApiVersion
- Chart.Version
- ... 等等等

或是用到 k8s cluster 本身的資源
- Capabilities.KubeVersion
- Capabilities.ApiVersions
- ... 等等等

或是用到 Values.yaml 作為嵌入變數
- Values.[自定義欄位 必須是小寫開頭]

> 變數名稱的重點
> 
> Release / Chart / Capabilities 後面接大寫開頭的欄位 (因為是 builtin 欄位)
> 
> Values 的自定義欄位必須是小寫開頭

完整的 helm chart = template + release details + char details + values.yaml

## Making sure chart is working as intended
安裝 chart 之前的 3 種驗証方式
- lint: 驗證 yaml 格式正確
- template: 驗證 template 格式正確
- dry run: 驗證 charts 對 k8s 是正確的

```shell
# helm lint
helm lint ./nginx-chart
# helm template 可以看組出來的 chart 長怎樣
helm template ./nginx-chart
# 可以補上 release name，如果 template 有用到的話才有資訊
helm template hello-world-1 ./nginx-chart
# 可以補上 debug flag，印出的錯誤訊息比較完整
helm template ./nginx-chart --debug
# --dry-run 可以檢查如果真的要安裝到 k8s 的結果
helm install hello-world-1 ./nginx-chart --dry-run
```

## Functions
functions 可以操作資料且不影響 values.yaml 內的資料

```yaml
# 假設 .Values.image.repository 是 nginx
# upper 是把字串改成大寫
{{ upper .Values.image.repository }} # image: NGINX
# quota 是在字串外多加上雙引號
{{ quote .Values.image.repository }} # image: "nginx"
# replace 可以用來取代字元
{{ replace "x" "y" .Values.image.repository }} # image: nginy
# default 會以第一個參數為預設值，如果第二個參數不存在的話
{{ default "nginx" .Values.image.repository }}
```

functions 的種類繁多，如 string, regex, file path, k8s, ...

## Pipelines
用 `|` 來做到管線式的參數轉換 

```yaml
{{ upper .Values.image.repository }} # image: NGINX
# 同
{{ .Values.image.repository | upper }} # image: NGINX
# 也可以一直串下去
{{ .Values.image.repository | upper | quote }} # image: "NGINX"
# 套用 default: 如果 .Values.image.tag 不存在的話，套用 default .Chart.AppVersion
{{ .Values.image.tag | default .Chart.AppVersion }}"
```

## Conditions
根據條件決定要不要在 yaml 檔刪除或新增對應的 key-value pair

```yaml
# 下面 - 的意思是轉換的時候不要空下這兩行
# eq 根據後面的兩個參數決定是 true 或 false
metadata:
  {{- if .Values.orgLabel }}
  labels:
    org: {{ .Values.orgLabel }}
  {{- else if eq .Values.orgLabel "hr" }}
  labels:
    org: human resources
  {{- else }}
  {{- end }}
```

|Function|Purpose|
|---|---|
|eq|equal|
|ne|not equal|
|lt|less than|
|le|less than or equal to|
|gt|greater than|
ge|greater than or equal to|
not|negative|
|empty|value is empty|

## With Blocks
重複的 namespace 可以用 with 包成 block

`$` 代表 root namespace
```yaml
data:
{{- with .Values.app }}
  {{- with .ui }}
  background: {{ .bg }}
  foreground: {{ .fg }}
  {{- end }}
  {{- with .db }}
  database: {{ .name }}
  connection: {{ .conn }}
  {{- end }}
  release: {{ $.Release.Name }}
{{- end }}
```

## Ranges
可以用來設定 array 資料

```yaml
# values.yaml
regions:
  - ohio
  - newyork
  - otario
  - london
  - sigapore
  
# configmap.yaml
data:
  regions:
  {{- range .Values.regions }}
  - {{ . | guote }} # 支援 pipeline
  {{- end }}
```

## Named Templates
重複使用相同的 key-value pair

可以提出為 `_helpers.tpl`

其中 `_` 的開頭代表告訴 helm 忽略這個檔案，不要當成 manifest 來用

```yaml
# _helpers.tpl
{{- define "labels" }}
  app.kubernetes.io/name: {{ .Release.Name }}
  app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

# 在要嵌入的地方放
# 後面的小點點很重要，代表 labels 內要參考的 scope 是 current scope
{{- template "labels" . }}
```

> 要注意 _helpers.tpl 內的縮排
> 
> 要和目標放入的地方縮排一致
> 
> 但可能要放入的地方縮排不一致
> 
> 所以要另外透過 (pipeline) indent [num] 處理

```yaml
# 原本的呼叫方式
{{- template "labels" . }}
# 多 縮排 2 格
{{- include "labels" . | indent 2 }}
# 多 縮排 4 格
{{- include "labels" . | indent 4 }}
```

> template 是 action
> 
> include 是 function，才可以接 pipeline

## Chart Hooks
可以設定在套用新的 helm 設定前，觸發 hook

**典型的 workflow**

`$ helm upgrade` -> `verify` -> `render` -> `upgrade`

加入 pre-upgrade 和 post-upgrade 後

`$ helm upgrade` -> `verify` -> `render` -> `pre-upgrade` -> `upgrade` -> `post-upgrade`

其他的如

`$ helm install` -> `verify` -> `render` -> `pre-install` -> `upgrade` -> `post-install`

`$ helm delete` -> `verify` -> `render` -> `pre-delete` -> `upgrade` -> `post-delete`

`$ helm rollback` -> `verify` -> `render` -> `pre-rollback` -> `upgrade` -> `post-rollback`

重點是可以起一個 pod 或 job 來處理 script 之類在 hook 要做的程序

其中透過 `annotations` 標記來讓 helm 忽略這份 manifest 檔案 (不要當成一般的 k8s manifest)

```yaml
# templates/backup-job.yaml
kind: Job
metadata:
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "5" # 設定 pre-upgrade job 的優先權，要是字串格式 
    "helm.sh/hook-delete-policy": hook-succeeded # 如果成功執行的話就砍掉這個 job
```

hood-delete-policy 有三種
- hook-succeeded: 如果執行成功的話就刪除
- hook-failed: 如果執行失敗的話就刪除
- before-hook-creation: default，在新的 hook 開始的時候，刪掉舊的 resource

![helm-hooks-sample.png](/k8s/helm-hooks-sample.png)

## Packaging and Signing Charts
打包上傳 helm charts

```shell
# 以上傳 nginx-chart 為例
helm package ./nginx-chart
```

整包 nginx-chart 會被打包成 nginx-chart-0.1.0.tgz (透過 Chart.yaml 內的版本)

> tgz 是指 tar archive compressed with gzip

上傳時要簽名，讓其他使用者知道這包 helm 是合法的

```shell
# 產生簽名
gpg --quick-generate-key "Yakushou"
# 如果是 production 要用的 key，盡量使用完整資訊
gpg --full-generate-key "YakuShou"
# 產生舊版格式的金鑰檔案
gpg --export-secret-keys >~./.gnupg/secring.gpg

# 打包 helm 並附上簽名
helm package --sign --key "Yakushou" --keyring ~/.gnupg/secring.gpg ./nginx-chart
# 列出 gpg key list
gpg --list-keys
```

## Uploading Charts
上傳 helm charts 需要的要件
- 打包好的 package (如: nginx-chart-0.1.0.tgz)
- index.yaml
- provenance (如: nginx-chart-0.1.0.tgz.prov)

![helm-uploading-charts.png](/k8s/helm-uploading-charts.png)

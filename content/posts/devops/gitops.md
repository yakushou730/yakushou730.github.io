---
title: "gitops"
authors: [yakushou730]
date: 2022-01-02T22:41:12+08:00
description: "gitops"
tags: ["programming","devops"]
draft: false
---

## What is GitOps
為了避免手動 deploy 而提出的解決方案

Infrastructure as Code 提倡後，大家會把 infrastructure 寫成 code 以後管控

但是針對變更 infrastructure 仍然後是在本機安裝工具後手動變更 (ex: ansible / terraform...)

GitOps 提倡一切 (infrastructure/network/policy/configuration/security...) 都以 YAML 的形式放在 git 上，並且透過 PR 自動更新資源

## GitOps flow
1. Create Pull/Merge Request
   - 任何人都可以發 PR，大家可以 review
2. Run CI pipeline
   - 驗證 configuration files
   - 跑自動化測試
3. Approve Changes
   - 到這步代表 code 是經過測試以及 well reviewed
4. Run CD Pipeline
   - merge 回 main 以後觸發 CD pipeline
   - 部署到 aws/k8s...等等

優點:
- 自動程序
- 整個環節是透明的
- 良好品質

## Push vs Pull model
Push Deployment
- 傳統方式，在 CI/CD 過了以後主動去做 deploy
- **push** to environment

Pull Deployment
- 在 environment 上面安裝 Agent (ex: k8s cluster)
- Agent 會從 git repository 拉下來做 deploy
- Agent 會比對當前的 actual state 和 git repository 上的 desired state



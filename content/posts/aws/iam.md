---
title: "iam"
authors: [yakushou730]
date: 2022-02-07T11:51:44+08:00
description: "iam"
tags: ["aws","devops"]
draft: false
---

## 設定帳戶密碼失敗
可能是碰到需要 MFA 操作，但是在剛建立使用者的時候還沒有 MFA 的關係

[AWS: Allows MFA-authenticated IAM users to manage their own credentials on the My Security Credentials page](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_aws_my-sec-creds-self-manage.html)

[AWS IAM won't let my users change their passwords](https://serverfault.com/questions/992027/aws-iam-wont-let-my-users-change-their-passwords)

做法有兩種
1. 改 policy 的規則，讓變更密碼這件事不需要 MFA (官方說不推薦這做法)
2. 砍掉帳號重新建立一個，然後先不要勾 `Require password reset`，等他登入後設定完 MFA，這樣之後就可以改密碼了


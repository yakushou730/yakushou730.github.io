---
title: "Hugo 的基本建立教學"
authors: [yakushou730]
date: 2021-11-10T10:48:27+08:00
description: "Hugo 的基本建立教學"
tags: ["hugo"]
draft: false
---
## 流程

以下為使用 [Hugo](https://gohugo.io/) 架部落格，並架在 [github page](https://pages.github.com/) 上

1. 我的電腦是使用 macOS，會需要透過 [homebrew](https://brew.sh/index_zh-tw) 來安裝需要用到的套件
   - git: 檔案版本控管
   - gvm: golang 的版本控管，用來安裝 golang
   - hugo: hugo cli 用來下對應的 hugo 指令
2. 使用 hugo 指令初始化一個新的網站 hugo-blog

   並且初始化 git

```
$ hugo new site hugo-blog
$ cd hugo-blog
$ git init
```
3. 搭配 [DoIt](https://themes.gohugo.io/themes/doit/) 主題作為顯示，以 submodule 加入至主題
```
$ git submodule add https://github.com/HEIGE-PCloud/DoIt.git themes/DoIt
```
4. 再來是設置 `config.toml` 以下是 [範例](https://hugodoit.pages.dev/theme-documentation-basics/#basic-configuration )，參考用
```
baseURL = "http://example.org/"
# [en, zh-cn, fr, ...] determines default content language
defaultContentLanguage = "en"
# language code
languageCode = "en"
title = "My New Hugo Site"

# Change the default theme to be use when building the site with Hugo
theme = "DoIt"

[params]
  # DoIt theme version
  version = "0.2.X"

[menu]
  [[menu.main]]
    identifier = "posts"
    # you can add extra information before the name (HTML format is supported), such as icons
    pre = ""
    # you can add extra information after the name (HTML format is supported), such as icons
    post = ""
    name = "Posts"
    url = "/posts/"
    # title will be shown when you hover on this menu link
    title = ""
    weight = 1
  [[menu.main]]
    identifier = "tags"
    pre = ""
    post = ""
    name = "Tags"
    url = "/tags/"
    title = ""
    weight = 2
  [[menu.main]]
    identifier = "categories"
    pre = ""
    post = ""
    name = "Categories"
    url = "/categories/"
    title = ""
    weight = 3

# Markup related configuration in Hugo
[markup]
  # Syntax Highlighting (https://gohugo.io/content-management/syntax-highlighting)
  [markup.highlight]
    # false is a necessary configuration (https://github.com/dillonzq/LoveIt/issues/158)
    noClasses = false
```
5. 都設定好後 可以用 hugo 指令起看看伺服器

   可以在 [本機端](http://localhost:1313/) 看到網頁

   到目前為止本機端就先告一段落了
```
$ hugo serve
```
6. 因為要使用 github pages 的關係，先建立一個個人的 repository

   名稱需要是固定格式，而我的是 yakushou730.github.io

7. 建立的 repo 用法
   
   branch main 是用作主要推 code 的分支，透過設定 github action 把網頁內容推到 gh-pages 分支上
```
$ git add .
$ git commit -m "init blog (main)"
$ git remote add origin https://github.com/yakushou730/yakushou730.github.io.git
$ git push -u origin main

$ git checkout -b gh-pages
$ git rm -rf .
$ echo "gh-pages" > "README.md"
$ git add .
$ git commit -m "init gh-pages branch"
$ git push -u origin gh-pages

git checkout main
```

8. 上面的指令打完後，這個專案就有了 main 和 gh-pages 兩支分支

   接著要設定 github actions 做到自動化部署

9. 到 [個人存取token頁面](https://github.com/settings/tokens/new) 建立新的 access token
   
   要給 github action 存取專案用的

   把 repo 和 workflow 打勾以後點擊 generate token

   token 等等要用到，可以先複製起來

10. 點擊專案的 Settings > Secretes > New repository secret
   
    用剛剛的 token 建立一組 secret，name 命名為 `HUGO_DEPLOY_TOKEN`

11. 再來是設定 github actions，點擊 actions 的 tab

    參考 [actions-hugo](https://github.com/peaceiris/actions-hugo)

    去設定要跑的flow

```
name: github-deploy-flow

on:
  push:
    branches:
      - main  

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  
          fetch-depth: 0   

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.89.2'
          extended: true  

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.HUGO_DEPLOY_TOKEN }}
          PUBLISH_BRANCH: gh-pages 
          PUBLISH_DIR: ./public    
          commit_message: ${{ github.event.head_commit.message }}
``` 
   接著按下 Start commit 儲存 workflow 的設定文件

   可以等待部署看看，若成功的話會打綠勾勾

12. 到 Repository > Settings 的 github pages 區塊

    把 branch 換成 gh-pages 就可以了


## 參考網站:
- [如何將Hugo部落格部署到Github上?](https://yurepo.tw/2021/03/%E5%A6%82%E4%BD%95%E5%B0%87hugo%E9%83%A8%E8%90%BD%E6%A0%BC%E9%83%A8%E7%BD%B2%E5%88%B0github%E4%B8%8A/)
- [DoIt Site](https://hugodoit.pages.dev/theme-documentation-basics/)

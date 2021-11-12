---
title: "Markdown 基本教學"
authors: [yakushou730]
date: 2021-11-10T12:34:12+08:00
description: "Markdown 基本教學"
tags: ["markdown"]
draft: false
---

## 斜體字
用底線 (underscore) 把要轉斜體字的字串夾起來 `_TARGET_`

結果: _TARGET_

## 粗體字
用兩個星星符號 (asterisk) 把要轉斜體字的字串夾起來 `**TARGET**`

結果: **TARGET**

## 標題

標題分成六種大小 在標題文字前面加上井字號 (hash mark)

井字號越少代表越大

```
# TARGET
## TARGET
### TARGET
#### TARGET
##### TARGET
###### TARGET
```
# TARGET
## TARGET
### TARGET
#### TARGET
##### TARGET
###### TARGET

或是加上 - 或 = 也可以表示標題
```
TARGET
======

TARGET
------
```
TARGET
======

TARGET
------

## 連結

連結的格式為中括號內放文字，小括號內放連結
`[TARGET DESCRIPTION](TARGET LINK)`

結果: [TARGET DESCRIPTION](https://example.com)

## 無序條列

利用 + 或 - 或 * 來做無序條列
```
- TARGET1
- TARGET2

  second line
- TARGET3
```

- TARGET1
- TARGET2

  second line
- TARGET3

## 有序條列

利用數字和點來做有序條列

```
1. TARGET1
2. TARGET2
3. TARGET3
```

1. TARGET1
2. TARGET2
3. TARGET3

## 參考類型的連結

利用中括號來完成

```
This is [ref1][1] and [ref2][ref-2]

[1]: http://google.com/        "Google"
[ref-2]: http://search.yahoo.com/  "Yahoo Search"
```

This is [ref1][1] and [ref2][ref-2]

[1]: http://google.com/        "Google"
[ref-2]: http://search.yahoo.com/  "Yahoo Search"

## 圖片

`![圖片替代文字](/path/to/img.jpg "Title文字")`

## 引用

用 小於 符號可以作為引用顯示

`> TARGET`

結果:
> TARGET

## 區塊程式碼

用三個反引號 (backtick) ` 符號包起來

```
TARGET
```

## 分隔線

可以用三個連續符號表示 - 或 * 或 _

```
***
```

***

---
title: 變更Hexo的主題
tags: 'hexo, Github, blog, theme'
date: 2017-05-04 09:41:12
---

繼上一篇說明如何在Github上用hexo建立blog，這次要介紹怎麼樣變更blog的主題(theme)。


## 找到你喜歡的主題(Theme)
Hexo[官方網站](https://hexo.io/themes/)有提供一些主題可以給你使用，這邊用到的主題也是官方提供的喔!

![Offical themes](/images/change-blog-theme-official-themes.png)

## 把原始碼完整的抓下來
在選擇主題後，試著找到他們的Github repository連結。

![Find out theme's repository url](/images/change-blog-theme-find-repository-url.png)

接著把HTTPS的git連結複製下來。

![Copy git repository url](/images/change-blog-theme-copy-repository-url.png)

用`git clone`指令把它抓下來，並放在themes/cactus-dark資料夾下。
``` bash
C:\Users\anson\blog\jsgao0> git clone https://github.com/probberechts/cactus-dark.git themes/cactus-dark
Cloning into 'themes/cactus-dark'...
remote: Counting objects: 1571, done.
remote: Compressing objects: 100% (34/34), done.
remote: Total 1571 (delta 10), reused 0 (delta 0), pack-reused 1526Receiving objects:  99% (1556/1571), 5.29 MiB | 542.00 KiB/s
Receiving objects: 100% (1571/1571), 5.41 MiB | 539.00 KiB/s, done.
Resolving deltas: 100% (671/671), done.
Checking connectivity... done.

```

## 在Hexo設定檔切換主題名稱
Hexo的設定檔就是`_config.yml`這個檔案。 裡頭的theme值預設為`landscape`，我們要將其修改成`cactus-dark`。

![Change theme name to cactus-dark in _config.yml](/images/change-blog-theme-modify-config.png)

## 結論與心得
單純地替換主題並不難，如果要做些客製化或是增加一些插件，就需要花更多時間了。
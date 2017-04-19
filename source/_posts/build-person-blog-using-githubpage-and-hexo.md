---
title: 用Github Page和Hexo建立個人部落格
---
因為常常看到許多大大用[Jekyll](https://jekyllrb.com/)或[Hexo](https://hexo.io/)在[Github Page](https://pages.github.com/)上建個人部落格，所以也想自己試著用Hexo做個人部落格。
這篇會記錄從沒有任何工具一直到建立好個人部落格。


## Git
[Git](https://git-scm.com/)是用來管理程式碼的工具，它最主要的特性就是分散式管理(分享程式碼)。

### 安裝Git
到[Git](https://git-scm.com/)官方網站根據你的OS(作業系統)版本下載安裝程式，然後安裝它。

![Click download button](/images/build-person-blog-using-githubpage-and-hexo-git-download.png)

### 測試Git
搜尋一下cmd.exe，並且打開它。
![Open command line window](/images/build-person-blog-using-githubpage-and-hexo-open-command-line-window.png)

接著在命令視窗裡面輸入下面的指令，來檢查git程式有沒有安裝成功。
``` bash
C:\Users\anson> git --version
git version 2.7.1.windows.2

```

## Github
[Github](https://github.com/)是提供Git的一個網路平台，我們可以將自己電腦上的程式碼透過Git建立版本後，發佈到Github上。[這邊有一篇](http://www.ithome.com.tw/news/95283)保哥寫的較詳細的Github基礎文章，想知道比較詳細內容的話，可以讀讀看。

### 申請Github帳號

### 建立給部落格用的Repository

### SSH Keys


## NodeJS & NPM
[NodeJS](https://nodejs.org/en/)是基於JavaScript的伺服器端程式，[NPM](https://www.npmjs.com/)則是用來管理NodeJS模組與套件的工具。

### 安裝NodeJS & NPM
不同以前NodeJS和NPM要分別安裝，現在只需要安裝NodeJS，就會順便幫你裝好NPM。到[官方網站](https://nodejs.org/en/download/)下載對應OS的版本並安裝。

### 測試NodeJS & NPM
測試NodeJS。
``` bash
C:\Users\anson> node -v
v6.10.2

```
測試NPM。
``` bash
C:\Users\anson> npm -v
3.10.5

```
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
[Github](https://github.com/)是提供Git的一個網路平台，我們可以將自己電腦上的程式碼透過Git建立版本後，發佈到Github上。[這邊有一篇](http://www.ithome.com.tw/news/95283)保哥寫的Github基礎文章，想知道較詳細內容的話，可以讀讀看。

### 申請Github帳號
Github的[首頁](https://github.com/)就是註冊頁，填完資料、驗證完email就可以囉!

![Github home page](/images/build-person-blog-using-githubpage-and-hexo-register-github.png)

### 建立給部落格用的Repository
申請之後，用帳號密碼登入。接著可以看到右上角有個＋，點一下，會有個 **New Repository** 的選項，點下去。

![Click add button then click new repository button](/images/build-person-blog-using-githubpage-and-hexo-create-new-github-repository.png)

接著你的瀏覽器會跳到下面這頁。把 **Repository Name** 這欄填上*name.github.io*，其中name是你的帳號，例如:我的帳號是jsgao0，就填上*jsgao0.github.io*。 

![Create repository page](/images/build-person-blog-using-githubpage-and-hexo-create-new-github-repository-page.png)

### SSH Keys
嗯... 這個事幹嘛的? 跟Github有啥關係?

<blockquote>
  SSH是目前較可靠，專為遠端登入對談和其他網路服務提供安全性的協定。
  <footer style="text-align: right;">https://zh.wikipedia.org/wiki/Secure_Shell</footer>
</blockquote>

在發佈到Github的時候，就是透過這個協定在傳輸的。然而，這個協定可以用兩種方式做安全驗證，密碼、金鑰。我們接下來要用金鑰的驗證方式，所以需要有一組公開金鑰+私有金鑰。 公開金鑰會被放在Github上面，而私有金鑰會放在你自己的電腦裡。

說這麼多，要怎麼做? 先到[Putty官網](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)下載金鑰產生器吧!

![Download puttygen](/images/build-person-blog-using-githubpage-and-hexo-download-puttygen.png)

下載完之後，請執行它。接著你會看到 **Generate** 的按鈕，按下去就可以開始了!

![Start puttygen](/images/build-person-blog-using-githubpage-and-hexo-start-puttygen.png)

產生的過程挺有趣的，你必須讓滑鼠指標一直在程式視窗的空白處移動。

![Generating puttygen](/images/build-person-blog-using-githubpage-and-hexo-generating-puttygen.png)

產生完，我們直接將公開金鑰(public key)和私有金鑰(private key)分別存成id_rsa.pub和id_rsa，並且放到 *C:\Users\windows_user\\.ssh* 的資料夾下面。

![Finish puttygen](/images/build-person-blog-using-githubpage-and-hexo-finish-puttygen.png)

再來是最後步驟囉! 回到剛剛Github建立的repository頁面，點右上角的 **Settings**。

![Go to Github settings page](/images/build-person-blog-using-githubpage-and-hexo-github-settings-page.png)

1. 在Settings頁面，選擇左邊的 **Deploy keys** 選項。
2. 然後點右上角的 **Add deploy key**。
3. 再來就填入key的 **title** 和公開金鑰(把id_rsa.pub用記事本打開貼上)，要注意貼在Key這個欄位的公開金鑰，必須要以ssh-rsa開頭並空一格，再將public key的內容貼上。

![Setup deploy key](/images/build-person-blog-using-githubpage-and-hexo-github-setup-deploy-key.png)

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

## 開始建立Blog!
一堆前置作業終於做完了，來開始建立Blog吧!

### 安裝hexo
這行指令是用來安裝hexo，要注意*-g*這個參數，它代表global，意思是裝到系統環境變數中，之後直接在命令提示字元視窗就能直接使用hexo的指令。

``` bash
C:\Users\anson> npm install hexo -g

```

### 用hexo建立部落格
``` bash
C:\Users\anson\> mkdir blog // 建立資料夾
C:\Users\anson\> cd blog // 切換到blog資料夾裡面
C:\Users\anson\blog> hexo init jsgao0 // 建立新的blog，名為jsgao0
... // 一堆正在安裝的資訊
INFO  Start blogging with Hexo!

```

### 安裝hexo的佈署套件
``` bash
C:\Users\anson\blog> cd jsgao0
C:\Users\anson\blog\jsgao0> npm install hexo-deployer-git --save

```

### 在本地端測試有沒有建立成功
hexo有提供簡易的web server讓你在本地端就能查看你準備發布的blog。

``` bash
C:\Users\anson\blog\jsgao0> hexo s
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.

```

執行之後，它會提示你網站的進入點是*http://localhost:4000/*。不要懷疑，打開你的瀏覽器，輸入它!

![hexo hello world!](/images/build-person-blog-using-githubpage-and-hexo-hello-world.png)

## 將你的部落格發佈到github
在本地測試完，就要發佈上去囉! 發佈之前，要先設定一些東西...

### 把你的blog在git history裡初始化

``` bash
C:\Users\anson\blog\jsgao0> git init
Initialized empty Git repository in C:/Users/anson/blog/jsgao0.git/

```

### 把原始碼加到Git history裡
在git工具裡，是以commit為單位來記錄程式碼版本。每個commit裡面可以有很多個檔案的修改內容(新增、修改、刪除)。這邊我們要建第一個commit，這樣才能建立第一個分支(master)。

``` bash
C:\Users\anson\blog\jsgao0> git add *
C:\Users\anson\blog\jsgao0> git commit -m "init"
 86 files changed, 5567 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 _config.yml
 create mode 100644 package.json
 create mode 100644 scaffolds/draft.md
 create mode 100644 scaffolds/page.md
 ...

```

### 設定SSH url
到你剛剛開的Github repository首頁，點綠綠的按鈕(Clone or download)，再點右邊的**Use SSH**，然後把下面的連結複製起來貼到下面指令的後面。 這個步驟是要設定之後在發佈程式碼的時候的目的地(Github)。

![Clone or download button](/images/build-person-blog-using-githubpage-and-hexo-ssh-url.png)

``` bash
C:\Users\anson\blog\jsgao0> git remote add origin ssh_url <--連結貼在這裡
C:\Users\anson\blog\jsgao0> git remote -v // 查看有沒有設定成功
origin  https://github.com/jsgao0/jsgao0.github.io.git (fetch)
origin  https://github.com/jsgao0/jsgao0.github.io.git (push)

```

### 把剛才新增在master分支上的commit上傳到Github
提醒一下，這只是把程式碼傳到Github，並不是**發佈**部落格哦!

``` bash
C:\Users\anson\blog\jsgao0> git push origin master
Counting objects: 107, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (97/97), done.
Writing objects: 100% (107/107), 520.86 KiB | 0 bytes/s, done.
Total 107 (delta 0), reused 0 (delta 0)
To https://github.com/jsgao0/jsgao0.github.io.git
 * [new branch]      master -> master

```

### 發佈前的小小修改
記得剛剛我們放到Github的分支是master嗎?通常我們用git管理的時候，master是用來存放原始碼。而且，在使用name.github.io這類網址的時候，Github會自動編譯master上面的檔案，製作成網頁。

在發佈到Github之前，我們必須要做些修改，讓hexo能夠產出靜態網頁並且發佈到Github上面。其實不困難，只要把剛剛的ssh url加到_config.yml就可以囉!

記住: type必須為git而非github(hexo 3.0之後無效了)

![Add ssh url and set branch as master](/images/build-person-blog-using-githubpage-and-hexo-setup-master-branch.png)

### 發佈!
萬事俱備了，發佈吧!

``` bash
C:\Users\anson\blog\jsgao0> hexo d

```

## 結論與心得
因為平常就有在使用npm、git和Github，所以一開始看了一些文件，花了一個多小時就搞定了。會需要花一個小時，主要是因為Windows的private key放的位置不正確，爬了許多文章才知道有固定位置。另外還花了一些時間在替換這個theme(預設的真的很陽春XD)，這部分有空再來分享怎麼替換:p。
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
---
title: Hexo, devcontainer, Github Page 架設教學
tags:
    - Hexo
    - Blog
    - devcontainer
    - docker
categories:
    - Technical
---
###### 須具備知識: `Markdown`, `NPM`, `Git`, `Travis CI`

---

Hi 大家好，我是 Mike。
我喜歡在閒暇時間的時間研究新奇的東西，所以想啟個 blog 紀錄一下研究的東西，讓自己之後可以複習相關內容。所以我也研究了幾個常見的部落格框架，例如 [JekyII](https://jekyllrb.com/)、[Hugo](https://gohugo.io/)、[Hexo](https://hexo.io/)。
而最終我選擇了 **Hexo** 作為我部落格生成靜態檔的工具，並利用 github page 的免費資源來部屬，並利用 Travis CI 做 CD。

## 各單元介紹

### Hexo

首先介紹一下 Hexo 有哪些特點:

1. **不須了解網頁開發的技術**: Hexo 可將 [Markdown](https://markdown.tw/) 語法的文件轉換成網頁靜態檔，因此只需要學 Markdown 語法就好。
2. **套用主題方便**: 有開發者開發相關主題，讓使用者非常簡單就可以套用美美的主題。
3. **安裝套件方便**: 底層基於 Node.js，因此套件都是包圍在 npm 社群中

### devcontainer

利用 Docker 啟一個 vscode server container，讓環境分離，以防污染到 local 環境。以下也介紹幾個 devcontainer 的特色:

1. **開發環境分離**: 可以依照每個 project 分離每個環境，尤其當程式語言不同版本需要重新安裝時，使用 devcontainer 尤其方便
2. **快速建置開發環境**: 有了 devcontainer 的設定後，只需要安裝 vscode 和 Remote container extension 後便可以投入開發，減少環境建置的問題，也可防止因為設備 OS 不同，需要重新撰寫環境建置腳本的問題。

### Github Page

Github 提供每位使用者一個免費的 domain (`https://<username>.github.io`) 可以做靜態網頁的部屬，但我也沒找到 github 有提到 repo 的大小上限，只有建議大小低於 100 MB。

### Travis CI

Travis CI 是一個用來做 CI / CD 建置的工具。

- CI (*Continuously Integration*) 意思是持續整合，透過測試的方式，讓新作的改動不會影響到已被測試的部分，讓開發者再推送新變更時更有信心。但這次我們不需要這部分。

- CD (*Continuously Delivery / Deploy*) 意思是持續部屬，透過腳本設定將通過 CI 的內容部屬到你定義的地方，就這次教學來說，就是推到 github 上。

## 教學開始

首先提提為什麼我會特別需要介紹 devcontainer，主要是因為最近我離職後手邊目前沒有已經建置好開發環境的電腦，只剩家裡的 Windows 桌機，一方面不想汙染桌機環境，另一方面也懶得研究 Windows 系統的開發環境建置，因此便選擇 devcontainer 作為我拿來寫 blog 的工具。之後有空我會發一篇教學 devcontainer 的使用教學~~

接下來就廢話少說，進入正題。

### [Optional] devcontainer 設定

> 如果你不介意自己安裝 npm 和 hexo 套件的話，可以省略這段!!

devcontainer 主要是基於 `.devcontainer/devcontainer.json` 這個檔案設定和 vscode 相關的設定，在安裝 [Remote-Container extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) 後，可以開啟 Command Palette(`Ctrl/Cmd + Shift + P`) 後輸入 `Remote-Containers: Add Development Container Configuration Files` 透過互動式介面生成 `devcontainer.json`，或是直接參考我下面的範例。

```json
// devcontainer.json

// For format details, see https://aka.ms/devcontainer.json. For config options,
// see the README at:
// https://github.com/microsoft/vscode-dev-containers/tree/v0.177.0/containers/markdown
{
    "name": "Hexo blog Editing",
    "dockerFile": "Dockerfile",
    // Set *default* container specific settings.json values on container create.
    "settings": {
        "terminal.integrated.shell.linux": "/bin/bash"
    },
    // Add the IDs of extensions you want installed when the container is created.
    "extensions": [
        "yzhang.markdown-all-in-one",
        "streetsidesoftware.code-spell-checker",
        "DavidAnson.vscode-markdownlint",
        "shd101wyy.markdown-preview-enhanced",
        "bierner.github-markdown-preview"
    ],
    // Use 'forwardPorts' to make a list of ports inside the container available locally.
    // "forwardPorts": [],
    // Use 'postCreateCommand' to run commands after the container is created.
    
    // need to add this line because ownership is always root user and root group
    "postCreateCommand": "sudo chown -R node . && sudo chgrp -R node .",
    // Comment out connect as root instead.
    // More info: https://aka.ms/vscode-remote/containers/non-root.
    "remoteUser": "node",
    "containerEnv": {
        // Because commit time of git would use local timezone in container,
        // if we don't set timezone to taipei, timezone of commit time would be UTC
        "TZ": "Asia/Taipei"
    }
}
```

接著除了 vscode 設定外，你也可以自定義 Container 內的設定，例如安裝一些系統性套件，或是設定一些環境變數 ... 等。而 Dockerfile 位於 `.devcontainer/Dockerfile`。範例如下。

```dockerfile
# See here for image contents:
# https://github.com/microsoft/vscode-dev-containers/tree/v0.177.0/containers/javascript-node/.devcontainer/base.Dockerfile

# [Choice] Node.js version: 16, 14, 12
ARG VARIANT="16-buster"
FROM mcr.microsoft.com/vscode/devcontainers/javascript-node:0-${VARIANT}

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# [Optional] Uncomment if you want to install an additional version of node using nvm
# ARG EXTRA_NODE_VERSION=10
# RUN su node -c "source /usr/local/share/nvm/nvm.sh && nvm install ${EXTRA_NODE_VERSION}"

# [Optional] Uncomment if you want to install more global node modules
RUN su node -c "npm install -g hexo-cli"
```

### Hexo 設定

如果你沒有使用 devcontainer 的話，首先需要做的事情是先安裝 hexo 的互動化工具 command。

```bash
npm install -g hexo-cli
```

跑完這個 command 後，會在全域的 npm 安裝 hexo 工具，之後不管是要生成新的文章，或是跑一個測試網站，甚至之後的部屬都需要和這個套件互動，因此這個套件必裝不可!

接著，執行以下指令，請 hexo 幫我們初始化整個資料夾

```bash
cd <project_folder>
hexo init .
```

執行結束後，會在此目錄生成以下結構的專案。

```text
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

檔案結構說明如下:

1. `_config.yml`: 設定 hexo 的地方
2. `package.json`: npm 專案的資訊檔，**原則上不需改動**
3. `scaffolds`: 裡面定義了範例檔，hexo 將利用這些檔案生成我們要的貼文、草稿、頁面
4. `source/_drafts`: 裡面為每篇**草稿**存放的地方
5. `source/_posts`: 裡面為每篇**文章**存放的地方
6. `themes`: 裡面儲存了各式可套用的主題

### 主題套用

主題可從 [官方主題](https://hexo.io/themes/) 中挑選。而我選擇的是 [NexT](https://theme-next.js.org/) 主題，以下將針對 NexT 主題套用做教學。

首先先執行以下指令安裝主題到專案中

```bash
git submodule add https://github.com/theme-next/hexo-theme-next themes/next
```

至於為什麼是用 submodule 而不是按照官方建議直接 clone 的原因是直接 clone 的話，之後部屬時會無法正確生成對應的靜態檔。

再來修改設定檔中的 `theme` 屬性來套用新主題。

```yml
# _config.yml
...
theme: next
...
```

接下來如果需要修改 NexT 的設定的話，也要注意不要直接修改 `themes/next/_config.yml` 而是去修改 `_config.yml` 中的 `theme_config` 屬性，直接把你要修改的屬性結構複製過來修改即可，如下所示。

```yml
# _config.yml
...
theme: next
theme_config:
    # 修改編排方式
    scheme: Mist
    # 開啟黑暗模式
    darkmode: true
    # 要開啟快捷 link，並設定路徑和 icon
    menu:
        home: / || fa fa-home
        about: /about/ || fa fa-user
...
```

### 部屬到 Github Page

首先我們要做的是到你的 github 創立一個名為 `<username>.github.io` 的 repository。

![創立 repo]()

接著到 [Travis CI 官網](https://travis-ci.com/) 用 Github 帳號登入，登入後新增剛剛創立的 repo，讓他 Travis CI 認得它。

![新增 Travis CI 支援]()

接著回到 Github [創建 token](https://github.com/settings/tokens)，當 Github 看到這個 token 會認為是你在進行操作，藉此取代帳號密碼。新增完成後，複製 token 資料。

複製完後切換到 Travis CI 中的 `<username>.github.io` 專案，並在設定頁面中的 Environment Variables 新增一個名為 GH_TOKEN 的 variable，並在 Value 欄位中貼上剛剛複製的內容。

最後在我們的專案中新增一個名為 `.travis.yml` 的檔案，此檔案是用來針對 Travis CI 進行設定，讓他們按照我們的指示做對應的事情，範例如下。

```yml
# use node js language to build
language: node_js
# use linux as base os
os: linux
# use Ubuntu Xenial
dist: xenial
# define node js version
node_js:
    - 16
# cache dependencies
cache: npm
# run build only on main branch was changed
branches:
    only:
        - main
script:
    # generate static HTML files
    - hexo generate
deploy:
    # use github page deploy process
    provider: pages
    # Does git push over HTTPS
    strategy: git
    # keep builded pages
    skip_cleanup: true
    # token created from github,
    # and read from environment variable defined in Travis CI config, named GH_TOKEN
    token: $GH_TOKEN
    # keep old build/files from previous deployments
    keep_history: true
    # define deploy provider
    on:
        branch: main
    # define which dir would be pushed to github as static website
    local_dir: public
```

依照這樣設定完後，原則上只要 `main` branch 有變更，就會自動觸發 Travis CI 進行網站更新，而 Travis CI 做完後會把靜態網站檔推到 `gh-pages` branch

最後我們需要把 Github Page 的靜態網站來源 branch 改成 `gh-pages`。首先，點選 Github `<username>.github.io` 的設定 > Pages > Source，選擇 branch 選單中的 `gh-pages` 並儲存，如此一來便完成所有設定了。

接著 access `https://<username>.github.io/` 便可看到你的內容囉。

## 結語

過程有些複雜，但設定完後每次上文章都只到 push to main 就可以休息開網站等它更新了，在摸索過程也撞蠻多坑的，希望大家看完可以少踩一些坑，最後如果有問題歡迎詢問，如果有錯誤的地方，也請不吝指教。

## 參考來源

1. [Hexo 文件](https://hexo.io/)
2. [Travis CI 文件](https://docs.travis-ci.com/)

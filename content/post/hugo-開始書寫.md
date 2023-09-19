---
title: "Hugo 開始書寫"
date: 2019-08-03T14:29:59+08:00
draft: false
tags: [Hugo, Github, Wordpress]
---

wordpress.com 由於限制頗多，而且對於撰寫程式碼不是那麼友善（許多特殊符號需要自行轉譯處理）。參考官網提供的工具參考 https://gohugo.io/tools/migrations/ ，選用 https://github.com/palaniraja/blog2md 將匯出的文章轉換成 Hugo 格式。隨後參考 https://gohugo.io/hosting-and-deployment/hosting-on-github/ 把網站發布到 Github 上面。

按照範例說明，會在 Github 建立兩個 Reposiotry，一個儲存靜態發布前的相關檔案，也就是 Hugo 的目錄結構本身，一個放置靜態網站，也就是 Hugo 產生的靜態網站。每次更新，都需要一連串的操作跟 git 提交。因為有點囉唆所以說明有提供 `deploy.sh` 協助簡化這些動作。目前也寫了類似的發佈檔 `deploy.bat` 提供在 Windows 底下使用。附上 `deploy.bat`

{{< highlight Batchfile />}}

@ECHO OFF

ECHO Deploying updates to GitHub...

REM Build the project.
hugo
REM Go To Public folder
cd public
REM Add changes to git.
git add .
REM Commit changes.
SET msg="rebuilding site %DATE%"
if NOT "%1"=="" (
    SET msg=%1
)
git commit -m %msg%
REM Push source and build repos.
git push origin master
cd ..
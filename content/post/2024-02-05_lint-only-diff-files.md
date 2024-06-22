---
title: Lint only diff files
date: 2024-02-05T13:51:30+08:00
keywords: ["lint", "phpcs", "github action"]
---

過去我們會在提交的時候運行代碼風格檢查，但隨著檔案越來越多，檢查的時間也越來越長。但其實每次提交改動的檔案數量不多，所以可以根據改動的檔案做檢查即可。目前透過 Github action 嘗試進行設定，建立 Pull Request 的時候執行。

## PHP

首先根目錄有 `phpcs.xml` 的 PHP CodeSniffer 設定檔，並且只在 php 檔案有變更或者設定檔變更的情況下觸發。先透過 curl 下載 PHP CodeSniffer，然後利用 `git diff` 跟目標分支比對出此次修改的檔案名稱跟路徑，並且利用 `--diff-filter=dr` 過濾掉刪除的檔案跟只是更名的檔案，最後透過 `egrep` 過濾出 php 結尾的檔案。

```shell
git diff --name-only --diff-filter=dr \
${{ github.event.pull_request.base.sha }} |egrep '\.php$'
```

`github.event.pull_request.base.sha` 這個變數是目標分支的 SHA 值，通常是 master 分支的 SHA。但因為使用官方的 `actions/checkout` checkout 分支，預設只會包含目前 Pull Request 這支分支，所以要加上 `fetch-depth: 0` 這個參數，才能夠取得其他分支，這樣一來才能取得 master 的 SHA。

綜合上述的說明，設計下方的 workflow：


```yaml
name: Run lint

on:
  pull_request:
    # 只在底下檔案有變動的情況觸發
    paths:
      - "**.php"
      - "phpcs.xml"
      - ".github/workflows/php-lint.yaml"

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - run: |
          curl -OL https://squizlabs.github.io/PHP_CodeSniffer/phpcs.phar

      - run: |
          php phpcs.phar --standard=./phpcs.xml $(git diff --diff-filter=dr --name-only ${{ github.event.pull_request.base.sha }} |egrep '\.php$')
```

網路上透過 `github action phpcs` 可以找到許多現成的 github action，功能更完善，包括整合機器人帳號可以自動提示等等。目前的訴求還很簡單，先簡單進行。


## JavaScript

有 PHP 的例子之後，JS、TS 的部分就簡單多了。 `.eslintrc.js` 設定檔放置在根目錄，利用 npx 的方式執行 eslint，只檢查 ts, js 檔案的變更。 `package.json` 要先包含 `eslint` 的安裝。

```yaml
name: Run lint

on:
  pull_request:
    paths:
      - "**.ts"
      - "**.js"
      - ".eslintrc.js"
      - ".github/workflows/js-lint.yaml"

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - run: |
          npm install
          npx eslint -c .eslintrc.js $(git diff --diff-filter=dr --name-only ${{ github.event.pull_request.base.sha }} |egrep '\.[tj]s$')
```

## CSS

CSS 的部分使用 `stylelint` 依法泡製，只檢查 css 檔案的變更。`package.json` 要先包含 `stylelint` 的安裝。

```yaml
name: Run lint

on:
  pull_request:
    paths:
      - "**.css"
      - ".stylelintrc.json"
      - ".github/workflows/css-lint.yaml"

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - run: |
          npm install
          npx stylelint  -c .stylelintrc.json $(git diff --diff-filter=dr --name-only ${{ github.event.pull_request.base.sha }} |egrep '\.css$')
```

## 無差異情況的修正

由於 `egrep` 在沒有檔案差異的情況下，會回傳非零的代碼，這會導致執行中斷。所以需要在後面加上 `|| true`，並且判斷只有在檔案內容有的時候才進行檢查。

```shell
git diff --name-only ${{ github.event.pull_request.base.sha }} |egrep '\.php$' > .list.txt || true

# 有檔案內容才進行檢查
[ -s ./.list.txt ] && phpcs --standard=./phpcs.xml --file-list=./.list.txt
```


## 話說

話說上面的這三個工具其實都支援 cache 的使用，所以本地端檢查即是檢查全部，只要啟用快取速度上都挺快的。主要是 CI/CD 的環境沒有特別去維護快取內容所以預設都是重新檢查。因此這樣的改動才發揮它的價值。
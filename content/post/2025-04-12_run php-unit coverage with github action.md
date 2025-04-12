---
title: Run PHPUnit coverage with Github action
date: 2025-04-12T10:52:57+08:00
draft: false
keywords: ["Github Action", "PHPUnit", "awk", "sed"]
---

由於中途導入單元測試，為了確保測試覆蓋率能穩定的成長，每個月我們會審視當月份覆蓋率是否往目標成長。而過去的做法就是每個月執行一次，然後將數據收集起來。為了將這個覆蓋率的行為自動化，花了點時間整合到 Github Action。

## Coverage Report as Comment (Clover)

從 Github marketplace 搜尋 `Coverage` 其實[可以找到不少現成的方案](https://github.com/marketplace?query=coverage&type=actions)。有些甚至跟單元測試工具無縫結合，可以省去自己整合的需求。

首先我挑選了 [Coverage Report as Comment (Clover)](https://github.com/marketplace/actions/coverage-report-as-comment-clover) ，他能夠將覆蓋率報告作為備註顯示在 comment 裡頭。具體結果類似如下：

```
Coverage report for commit: 0688430
File: coverage.xml

Cover ┌─────────────────────────┐ Freq.
   0% │ ██░░░░░░░░░░░░░░░░░░░░░ │  5.6%
  10% │ ░░░░░░░░░░░░░░░░░░░░░░░ │  0.0%
  20% │ ░░░░░░░░░░░░░░░░░░░░░░░ │  0.0%
  30% │ ░░░░░░░░░░░░░░░░░░░░░░░ │  0.0%
  40% │ █░░░░░░░░░░░░░░░░░░░░░░ │  0.8%
  50% │ █░░░░░░░░░░░░░░░░░░░░░░ │  2.4%
  60% │ █░░░░░░░░░░░░░░░░░░░░░░ │  0.8%
  70% │ █░░░░░░░░░░░░░░░░░░░░░░ │  1.6%
  80% │ █░░░░░░░░░░░░░░░░░░░░░░ │  0.8%
  90% │ ████░░░░░░░░░░░░░░░░░░░ │ 11.3%
 100% │ ███████████████████████ │ 76.6%
      └─────────────────────────┘
 *Legend:* █ = Current Distribution

Summary - Lines: 90.93% | Methods: 75.71%

🤖 comment via lucassabreu/comment-coverage-clover
```

由於他官方範例就是利用 PHPUnit 產生 clover 格式的覆蓋率報告，所以很容易的就整合到現在的單元測試流程。

```yaml
- name: Execute tests
  run: src/vendor/bin/phpunit -c src/phpunit.xml
  run: src/vendor/bin/phpunit -c src/phpunit.xml --coverage-clover=coverage.xml
  env:
    XDEBUG_MODE: coverage

- name: Coverage Report as Comment (Clover)
  uses: lucassabreu/comment-coverage-clover@main
  with:
    file: coverage.xml
```

## Code Coverage Summary

上面的結果看起來還不錯，但想試試看其他的套件結果如何，這次我改用 [Code Coverage Summary](https://github.com/irongut/CodeCoverageSummary)。雖然他的範例不是針對 PHPUnit，但他接受 cobertura 格式的覆蓋率檔案。PHPUnit 一樣支持。

```yaml
- name: Execute tests
  run: src/vendor/bin/phpunit -c src/phpunit.xml --coverage-cobertura=coverage.xml
  env:
    XDEBUG_MODE: coverage

- name: Code Coverage Report
  uses: irongut/CodeCoverageSummary@v1.3.0
  with:
    filename: coverage.xml
    badge: true
    fail_below_min: false
    format: markdown
    hide_branch_rate: false
    hide_complexity: true
    indicators: true
    output: both
    thresholds: '50 75'

- name: Add Coverage PR Comment
  uses: marocchino/sticky-pull-request-comment@v2
  if: github.event_name == 'pull_request'
  with:
    file: coverage.xml
    recreate: true
    path: code-coverage-results.md
```

生成的結果類似如下：

![Code Coverage](https://img.shields.io/badge/Code%20Coverage-91%25-success?style=flat)

Package | Line Rate | Health
-------- | --------- | ------
Console/Kernel.php | 67% | ➖
Exceptions/Handler.php | 50% | ➖
Http/Kernel.php | 0% | ❌
Http/Middleware/Authenticate.php | 100% | ✔
....
**Summary** | **91%** (2070 / 2272) | ✔

<!-- Sticky Pull Request Comment -->

## 顯示在 README.md

結果都對了，但新一輪的問題是，這些結果預設是顯示在 Pull requests 裡頭作為 Comment。我不希望每次查看覆蓋率都得去翻閱之前合併請求的備註內容。這樣很難顯眼的看到當前專案的覆蓋率。

為了將內容顯示在隨時可以存取的地方，我找到一篇將結果放在 Wiki 的文章
- [How to setup PHPUnit Coverage Report and PHPStan Report using GitHub Actions and GitHub Wiki Pages](https://medium.com/@vlreshet/how-to-setup-phpunit-coverage-report-and-phpstan-report-using-github-actions-and-github-wiki-pages-86effdfd8c53)

但隨即詢問同事才知道免費的 Github team 不支援 Wiki。既然如此不如就放在 README.md 吧！完整的需求如下：

- 只在 main 分支進行覆蓋率測試
- 其他 pull requests 一樣執行單元測試
- 將測試結果的文字顯示在 README.md，並且提交該變更。

我將上述的需求詢問 GPT 大神之後，得到一個粗略的版本，然後在幾次翻修後得到了我想要的版本：

```yaml

name: Run tests

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  php-tests:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: 8.3
          tools: composer:v2
          coverage: xdebug

      - name: Run composer install
        run: composer install -d`pwd`/src -n --prefer-dist --no-interaction --no-plugins

      - name: Execute tests without coverage
        if: github.ref != 'refs/heads/main'
        run: src/vendor/bin/phpunit -c src/phpunit.xml





      ### Code Coverage
      - name: Execute tests with coverage
        if: github.ref == 'refs/heads/main'
        run: src/vendor/bin/phpunit -c src/phpunit.xml --coverage-text --only-summary-for-coverage-text | tee coverage.txt
        env:
          XDEBUG_MODE: coverage

      - name: Extract Coverage Summary
        if: github.ref == 'refs/heads/main'
        run: |
          CONTENT=$(sed -n '/^  Classes:/,/^```/p' coverage.txt)
          START="## Code Coverage Report Summary"
          NEW_CONTENT=$(printf "%s\n\n\`\`\`sql\n%s\n\`\`\`" "$START" "$CONTENT")
          awk -v new_content="$NEW_CONTENT" '
          /## Code Coverage Report Summary/ {
              in_block = 1;
          }
          in_block {
              if (/```$/) {
                  in_block = 0;
                  print new_content; # 輸出新內容
              }
              next
          }
          { print }
          ' README.md > README.tmp
          mv README.tmp README.md

      - name: Commit and Push Changes
        if: github.ref == 'refs/heads/main'
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
          git add README.md
          git commit -m "Update code coverage report summary" || echo "No changes to commit"
          git push origin ${{ github.head_ref }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

為了配合上述的內容，我需要在 README.md 裡頭添加一段覆蓋率報告文字

```
## Code Coverage Report Summary

```sql
  Classes: XX.XX%
  Methods: XX.XX%
  Lines:   XX.XX%
` ` `
```

單元測試後會產生類似內容，我將內容擷取出來後，透過 `awk` 取代 README.md 相同的內容，改為最新的覆蓋率報告。最後結果呈現如下：


# My Project

## Code Coverage Report Summary

```sql
  Classes: 57.89% (55/95)
  Methods: 76.92% (190/247)
  Lines:   91.11% (2070/2272)
```

## 踩坑

在開發的過程中踩了幾個坑：

###  awk: newline in string $## Code Coverage Re... at source line 1

首先希望先在本機端執行一次 awk 的內容取代。但出現了如標題的錯誤，原因是因為 awk 指令在 linux 跟 MacOS 版本有差異。尤其在處理換行字元的部分。但本地端只是測試，所以我將輸出的內容去掉換行字元。先在本地端能正確模擬就好。

### 覆蓋率的內容不見了

將這個 action 放到另外一個專案，跑完之後內容卻只剩下如下標題：

```
## Code Coverage Report Summary

```

這是由於 README.md 的換行字元所導致，該檔案是 `CRLF` 儲存在 Windows 的環境，但 action 執行在 UNIX 上面採用的是 `LF`。這似乎會導致 awk 執行結果不正確。重新儲存檔案為 `LF` 字元後解決。


### fatal: You are not currently on a branch.

由於開發的時候，我先在 pull request 所在的分支執行，在最終 `git commit` 的環節出錯，該問題可以透過添加 `ref: ${{ github.head_ref }}` 解決。

```yaml
- name: Checkout the correct branch
  uses: actions/checkout@v4
  with:
    ref: ${{ github.head_ref }}
```
---
title: "SublimeLinter-phpcs 使用自訂規則檔"
date: 2019-08-03T16:57:39+08:00
draft: false
---

為了建立一個樣板專案，同時整合代碼風格檢查，今天莫名其妙踩了一些坑。

全編輯器設定 SublimeLinter Settings - User
{{< highlight javascript >}}
{
    "linters": {
        "phpcs": {
            "args": "--standard='${folder}/phpcs.xml'"
        }
    }
}

{{< /highlight >}}

個別專案設定，設定在 *.sublime-project
{{< highlight javascript >}}
{
    "settings": {
        "SublimeLinter.linters.phpcs.args": "--standard='${folder}/phpcs.xml'"
    }
}
{{< /highlight >}}

由於設定路徑成功或失敗，都**不會產生錯誤**。另外即便規則檔有誤也**不會有錯誤說明**。所以檢查的方式是透過終端機執行 `phpcs` 先確定規則檔是否可以正確套用。

先故意製造一個錯誤，例如以 PSR-2 規則而言，類別名稱必須以大寫開頭。

{{< highlight bash >}}
$ phpcs --standard=D:/Develop/www/project-template/phpcs.xml
............................................E..


FILE: ...\Develop\www\project-template\site\tests\Feature\ExampleTest.php
----------------------------------------------------------------------
FOUND 1 ERROR AFFECTING 1 LINE
----------------------------------------------------------------------
 8 | ERROR | Class name "axampleTest" is not in camel caps format
----------------------------------------------------------------------

Time: 1.32 secs; Memory: 8Mb{{< /highlight >}}

確定找得到問題之後，回到編輯器，看到底下畫面表示設定正確了。

![editor](/SublimeLinter-phpcs.jpg)
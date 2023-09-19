---
title: "PHPUnit TearDown & register_shutdown_function"
date: 2023-03-25T19:40:00+08:00
draft: false
tags: [PHPUnit TearDown Laravel]
---

## WHAT

近期在開發 Laravel Log 相關套件。於是利用 `orchestra/testbench` 作為開發套件的工具。但是當所有測試都正常執行之後，結尾卻意外發生錯誤。
```php
PHP Fatal error:  Uncaught ReflectionException: Class config does not exist
```

原本這個問題以為只在 Linux 環境發生(由於代碼提交上去 CI/CD 主機才發現)。所以反覆地在 CI/CD 主機進行代碼的測試。

但隔天才意識到 Mac 端雖然正常執行，但終端機的 return code 卻是 `255`
這表示 Mac 也一樣報錯，只是沒有顯示出來！後續修改 `php.ini` 的 `error_log` 將錯誤 log 搜集起來顯示。果然看到一樣的問題。


## WHY

功能需求安裝 [illuminated/console-logger](https://packagist.org/packages/illuminated/console-logger) 。這是一個 Command Log 套件，能夠紀錄 Command 執行的開始、結束時間，以及獨立出 Log 檔案。開發時引用了這個套件。並且發現只有在使用這個套件的測試結束時會報錯。

於是追蹤到他自定義的 [ExceptionHandler](https://github.com/dmitry-ivanov/laravel-console-logger/blob/master/src/Exceptions/ExceptionHandler.php) 發現他註冊了 `register_shutdown_function`。該方法會在 Command 結束的時候自動調用。作者是為了自動記錄下指令的結束執行時間，預期會產生如下內容到 Log。

```ini
Execution time: {$executionTime} sec.
Memory peak usage: {$memoryPeak}.
%separator%
```

套件原本是透過 [Monolog](https://github.com/Seldaek/monolog) 的 Handler 將記錄寫入到檔案。於是我自定義了 `Laravel Eloquent` 的 Handler 要將記錄寫入到資料庫。於是錯誤就這樣產生了...

細細爬梳之後發現， `PHPUnit` 結束時會調用 `TearDown` 方法，`orchestra/testbench` 的 `TearDown` 方法在結束時會將 `Laravel Container` 清空。

而 `register_shutdown_function` 在測試結束的時候，自動被調用。此時 `Laravel Container` 被清空，所以 `Laravel Eloquent` 也就無法透過 `config` 這個 Container 裡頭的 alias 完成工作。於是就報錯了。

簡單整理一下流程：
1. 測試某個指令執行
1. 指令呼叫 illuminated/console-logger 提供的方法，註冊了 register_shutdown_function。
1. 測試結束，orchestra/testbench 呼叫了 TearDown 清空 Laravel Container。
2. 由於執行結束 register_shutdown_function 所註冊的方法被呼叫。
3. 呼叫的內容試圖透過 Laravel Container 裡頭的方法執行，結果因為 Container 是空的，所以就報錯了。

## HOW

為了改善這個只在測試才發生的問題（套件安裝起來使用沒有問題，主要大部分時候 Laravel Container 不會隨意被清空）。目前是覆寫套件自帶 ExceptionHandler 。主要是在執行 shutdown 相關的方法之前，先檢查一次 Container 是否有所需的服務。
```php!
public function onShutdown(): void
    if (app()->bound('config')) {
        parent::onShutdown();
    }
}
```



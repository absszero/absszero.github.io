---
title: Upgrade to Laravel 11 from Laravel 10
date: 2025-02-15T17:09:20+08:00
draft: false
keywords: ["Laravel", "artisan", "casts"]
---

去年五月將一個舊專案改寫成 Laravel。礙於當時的 PHP 版本只能維持在 8.1，所以最終沒能直接使用 Laravel 11。而今年在一番討論之後，決定各自升級手邊系統，來到當前官方還在維護的版本。利用上週六需要補班的時間嘗試將專案升級到 Laravel 11。

## Eloquent Model casts Method

由於本地環境已經使用 Laravel Herd 進行開發，透過 GUI 很容易的將 PHP 版本切換到 8.3。檢視過 8.1 到 8.3 之間的相容性問題之後，回來開始 Laravel 11 版本升級。官方提供的[升級說明](https://laravel.com/docs/11.x/upgrade)所提到相容性衝擊的部分，對去年才改寫的專案衝擊不大。我主要修改的部分只有 [Eloquent Model casts Method](https://laravel.com/docs/11.x/upgrade#eloquent-model-casts-method)，把 Model 的 `$casts` 屬性，改用 `casts` 方法替換。

```php
    // 原本....
    protected $casts = [
        'create_order' => 'json',
        'order_result' => 'json',
        'payment_result' => 'json',
        'invoice_data' => 'json',
    ];


    // 改寫成....
    protected function casts(): array
    {
        return [
            'create_order' => 'json',
            'order_result' => 'json',
            'payment_result' => 'json',
            'invoice_data' => 'json',
        ];
    }
```

不改動應該也是不影響的，11 版的[原始碼](https://github.com/laravel/framework/blob/11.x/src/Illuminate/Database/Eloquent/Concerns/HasAttributes.php#L76)還是留著對 `$casts` 屬性的支持。

## Application Structure

官方升級手冊提到[應用程式結構有變更](https://laravel.com/docs/11.x/upgrade#application-structure)，但不建議 Laravel 10 的用戶升級到到 Laravel 11。但官方文件沒說明 `如何避免升級到新版結構` 。直到我踩了這個坑之後，總算曉得怎麼升級上去了。

首先每次升級有過習慣，就是升級套件之外，整個應用程式其實有許多零碎的小修改，可以透過 [GitHub 比較工具](https://github.com/laravel/laravel/compare/10.x...11.x)找出來。我會借機同步最新內容以利下次升級。

可以看到幾個重要檔案或目錄被刪除。包括：

- app/Console/Kernel.php
- app/Exceptions/Handler.php
- app/Http/Kernel.php
- app/Http/Middleware 整個目錄
- app/Providers 整個目錄

再來幾個進入點檔案修改，一個是 `artisan` ，還有 `bootstrap/app.php` 以及 `public/index.php` 。另外新增了 `bootstrap/providers.php` ，而自動載入 provider 的 `config/app.php` 移除了 `providers` 的部分。

如果你按照這個勢態修改進入點檔案。其實整個應用程式的結構就會變成使用 11 版的結構。所以如果要保有 10 的結構，請保留底下檔案不要同步 11 版：

- artisan
- bootstrap/app.php
- public/index.php

另外請保留 config/app.php 裡頭的 `providers` ，不要改用 `bootstrap/providers.php` 代替。

## 最後

由於該專案是個重新改寫的專案，再改寫的過程中有機會補上相關測試進行升級。所以 9x% 的代碼覆蓋率對於這次升級應該有潛在的幫助。目前仍然有些舊專案由於中途才開始增加測試，目前覆蓋率可能落於 50% 以下。如果今年計劃要升級到維護中的版本，估計會增加不少不確定性。
---
title: Test non-existent SQL functions
date: 2024-04-20T16:19:18+08:00
draft: false
keywords: ["SQLite", "Laravel", "Test", "PostgreSQL"]
---

Laravel 測試預設配置 SQLite 做為資料庫，並使用 In memory 的方式執行測試。但產品資料庫會使用的 SQL Function 不一定能在 SQLite 上使用，目前在嘗試幾個做法來解決測試無法通過特定資料庫的 SQL Function 問題。目前在 PostgreSQL 上使用 `TO_CHAR` 進行日期格式化。但 SQLite 並沒有相同的函式，但有相似的函式 `strftime`。

## Mock DB:raw()

第一個想到做法是模擬對象 `DB:raw`，當查詢不複雜，這個方式方便有效。但如果整個查詢會用到很多 DB Facade 的時候，每一個用到的地方都需要適當的模擬。也因此。 `DB::connection()->enableQueryLog()` 跟 `DB::connection()->getQueryLog()` 也無法使用，如此一來就失去查看 DB Query 的作用。

```php
// Mock DB::raw
DB::shouldReceive('raw')->andReturn("strftime('%Y-%m-%d', created_at)");

Post::select(DB::raw("TO_CHAR(created_at, 'YYYY-MM-DD')"));
```

## PDO::sqliteCreateFunction()

再來是透過 `PDO::sqliteCreateFunction()` 可以在查詢使用到特定的 SQL Function 的時候，利用 PHP 的回調函式作為 SQLite 的 User Defined Function。[官方的手冊顯示這個方法還在實驗性階段](https://www.php.net/manual/zh/pdo.sqlitecreatefunction.php)，得斟酌使用。
第一次學到這個方法是在 [Laravel 為SQLite 增加自定 Function](https://medium.com/@reccatsai/laravel-%E7%82%BAsqlite-%E5%A2%9E%E5%8A%A0%E8%87%AA%E5%AE%9A-function-5ca116423f32) 這邊文章提到。但實際使用過中主要問題是參數的不對稱。TO_CAHR 使用 YYYY-MM-DD 作為 20XX-XX-XX 顯示，但在 PHP 方面卻是 Y-m-d。為此會需要花額外功夫對應參數。

```php
$connection = \DB::connection();
$connection->getPdo()->sqliteCreateFunction('TO_CHAR', fn($field, $format) => date('Y-m-d', strtotime($field)));

Post::select(DB::raw("TO_CHAR(created_at, 'YYYY-MM-DD')"));
```

## if database connection is sqlite

最後一個方式在程式碼裡頭做判斷，可以說是為了測試而寫程式。缺點就是得在程式碼中埋入相對應的測試用程式碼。但同時也避免的上述方法的缺點。

```php
$dateFormat = "TO_CHAR(created_at, 'YYYY-MM-DD')";
if ('sqlite' === config('database.default')) {
    $dateFormat = "strftime('%Y-%m-%d', created_at)";
}

Post::select(DB::raw($dateFormat));
```

## 總結

以使用順序而言， `PDO::sqliteCreateFunction()` 可以優先考量，如果沒有對稱性問題的方法，可以統一在 TestCase 定義。滿足後續測試使用到相同 SQL Function 的問題。
再者是 `Mock DB:raw`，在不複雜的情境可以模式對象解決。最後不行再進到程式碼編寫相對應的測試用程式碼。
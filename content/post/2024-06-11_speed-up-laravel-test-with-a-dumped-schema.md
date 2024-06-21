---
title: Speed up Laravel test
date: 2024-06-11T21:19:58+08:00
keywords: ["Laravel", "Test", "Schema", "PostgresQL"]
---

## In Memory

最近在進行一個既有專案的開發，為了能夠在上面做單元測試，著手進行準備工作。首先嘗試用 SQlite InMemory 的方式。然後在 TestCase 繼承了 `RefreshDatabase` 處理資料庫的初始化。

```php
namespace Tests;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    use CreatesApplication;
    use RefreshDatabase;
}
```

由於當前的專案超過 120 個表，並且包含 260 以上的 migration 需要執行，任何一個測試初始化大約都需要十秒以上的時間。仔細查看 `RefreshDatabase` 的原始碼，會發現其實在 InMemory 的實作，每一次都是重新執行一次 migration。

```php
    /**
     * Refresh the in-memory database.
     *
     * @return void
     */
    protected function refreshInMemoryDatabase()
    {
        $this->artisan('migrate', $this->migrateUsing());

        $this->app[Kernel::class]->setArtisan(null);
    }
```

## Real Server

畢竟這是記憶體的特性，不會保留上一次測試的執行的資料庫環境。但為了盡可能加速資料庫，開始考慮使用真實資料庫的情況。有幾個好處跟壞處：

好處：
- SQL 語法跟產品環境完全相容。不需要針對 SQL 語法不同的地方改寫測試。
- 上一次測試的 schema 會持續保留到後續的測試。

壞處：
- 磁碟的速度遠慢於記憶體，擔心換到真實資料庫測試，也許速度不會有太大改善。
- 實際上到 CI/CD 的主機執行測試，也得準備測試用的資料庫。

由於現行也想不到其他更好的辦法，直接開了一個新的 DB Server 作為測試資料庫用。最後測試結果很理想，實際上單一測試的初始化時間，縮短了 `10` 秒左右。

## Dumped Schema

為了讓測試速度更近一步地縮短，接著打算利用 Laravel 8 `schema:dump` 指令，先將當前的測試資料庫包含相關已執行的 migration 建立一個 SQL 備份。

```shell
./artisan schema:dump --database=testing_pgsql
```

執行之後多一份 `database/schema/testing_pgsql-schema.dump` 檔案，由於[官方採用的備份指令](https://github.com/laravel/framework/blob/1ca3acff94e4ff409451370fd03638670d30b5cb/src/Illuminate/Database/Schema/PostgresSchemaState.php#L61)沒有包含備份的格式，預設會使用 `pg_dump` 預設的格式匯出資料庫。

```shell
pg_dump --no-owner --no-acl --host="${:LARAVEL_LOAD_HOST}" \
--port="${:LARAVEL_LOAD_PORT}" \
--username="${:LARAVEL_LOAD_USER}" \
--dbname="${:LARAVEL_LOAD_DATABASE}"
```

我則是另外用指令加上 `Fp` 匯出純文字的 SQL 檔案，方便後續可以直接編輯。

```shell
pg_dump --no-owner --no-comments --no-acl -Fp \
--host="${:LARAVEL_LOAD_HOST}" \
--port="${:LARAVEL_LOAD_PORT}" \
--username="${:LARAVEL_LOAD_USER}" \
--dbname="${:LARAVEL_LOAD_DATABASE}"
```

實際測試的結果，大約縮短 `1.5` 秒左右。


## Ram Disk

然後在翻找資料的過程，有人提到他利用 Ram Disk 將容器掛載到該磁碟。於是我也找來 [TmpDisk](https://github.com/imothee/tmpdisk) 建立一個記憶體磁碟，大約配置 256 MB 的記憶體就足夠了。接著把資料庫初始化在該磁碟上。然後手動建立一個空的資料庫。

實測的結果速度又縮短了 `0.5` 秒左右。但有一個明顯的缺點是，每一次重新開機，就得重新再做一次資料庫初始化，頗為麻煩，最後還是直接使用真實磁碟作為測試資料庫。


## CI/CD

最終就是整合到 CI/CD 的環境，幸好 GitHub Action 已經有可利用的 [Action](https://github.com/marketplace/actions/setup-postgresql-with-postgresql-extensions-and-unprivileged-user)。輕鬆透過容器化建立一個測試資料庫，用於自動化單元測試。


```yaml
    - name: Setup Postgres database
    uses: Daniel-Marynicz/postgresql-action@master
    with:
        postgres_user: postgres
        postgres_password: postgres
        postgres_db: my_database
        exposed_postgres_port: 5432
```

## 結論

過往總是使用 SQLite In Memory 的特性建立測試，所以很自然的選擇用此當做測試的資料庫。但也由於不同資料庫之間的語法各異，許多資料庫原生的方法調用就會有所顧忌。現在有真實資料庫的加持，總算能夠在更貼近產品環境的範圍下，進行測試的建構與驗證。也從當中發現許多只有在用真實資料庫才會遇到的問題。

像是使用 Factory 建構測試資料的時候，就遇到許多隨機產生的內容，由於超出欄位長度而產生錯誤，但在 SQLite 由於使用動態欄位長度的緣故，沒有產生錯誤。另外在確認 SQL
語法的時候，所產生的語法也會貼近真實資料的內容，而非 SQLite 的內容。這樣更容易複製再透過資料庫查詢工具，檢查實際的情況。
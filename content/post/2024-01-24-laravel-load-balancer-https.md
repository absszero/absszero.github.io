---
title: Laravel x Load Balancer x HTTPS
date: 2024-01-24T12:35:27+08:00
draft: false
keywords: ["Laravel", "HTTPS", "X-Forwarded"]
---

由於 Laravel 生成網址的功能是根據一些特定的 Headers，特別是生成 https 開頭網址。當產品環境單純直接使用 Web Server 掛載 SSL 憑證不會有類似問題。但如果搭配 Load Balancer 而相關的 Headers 沒有正確的透過 X-Forwarded Headers 轉送過來，結果就是沒能正確產生 HTTPS 連結。

## 判斷依據

精華是 `symfony/http-foundation/Request.php` 的 `isSecure` 方法。

```php
public function isSecure()
{
    if ($this->isFromTrustedProxy() && $proto = $this->getTrustedValues(self::HEADER_X_FORWARDED_PROTO)) {
        return \in_array(strtolower($proto[0]), ['https', 'on', 'ssl', '1'], true);
    }

    $https = $this->server->get('HTTPS');

    return !empty($https) && 'off' !== strtolower($https);
}
```

- 如果是信任的 Proxy ，根據 `X-Forwarded-Proto` 的值是否是 `'https', 'on', 'ssl', '1'` 其中一個來決定是否使用 HTTPS。

- 如果不是信任的 Proxy，根據伺服器變數的 `HTTPS` 只要不是 `off` 就視為使用 HTTPS。

## 設定信任的 Proxies

Laravel 官網在 [Configuring Trusted Proxies](https://laravel.com/docs/10.x/requests#configuring-trusted-proxies) 說明的很清楚。只要在 `App\Http\Middleware\TrustProxies` 把 Load Balancer 的 IP 填入到 `protected $proxies` 即可。 但如果你不確定 IP 來源，也可以帶入一個 `protected $proxies = '*';`。系統會根據伺服器變數 `REMOTE_ADDR` 自動填入。

## 如果都不行

如果上述設定 Proxy 的方式還是不行。還有兩個方法：

- 設定伺服器變數的 `HTTPS` 為 `on`
- 在 `AppServiceProvider` 的 `boot()` 方法中設置，強制 HTTPS（寫死）：

    ```php
    if($this->app->environment('production')) {
        \URL::forceScheme('https');
    }
    ```


目前隨著部署環境多元，開始遇到 `X-Forwarded-Proto` 轉送過來卻是 `http` 的問題，只能先請 SRE 工程師代為協助了。
---
title: Laravel dynamic facade
date: 2024-04-11T18:42:53+08:00
draft: false
keywords: ["Laravel", "Facade"]
---

Laravel 在 Facade 的使用很普遍，內建就提供了多種 Facade。並且在過去的使用情境也透過繼承 `Illuminate\Support\Facades\Facade` 建立自己的 Facade。唯獨 `Dynamic Facade` 我今天才發現它的好用！

# 使用 app 容器的缺點

首先在過往還未使用 Dynamic Facade 的情境，最常使用的方法是 app 容器的方式來實現測試的 Mock。

```php
// Test.php
// 在測試中先使用 Mockery::mock
$mock = Mockery::mock(MyClass::class);
$mock->shouldReceive('someMethod')->andReturn();

// app 容器中綁定這個 mock 物件
$this->app->instance(MyClass::class, $mock);

// Source.php
// 透過容器取得 MyClass，實現物件 mock
app(MyClass::class)->someMethod();
```

上面的方法在過去的實現中有幾個明顯的缺點：

- Mock 物件需要綁定到 app 容器。
- 需要透過 app 容器才能取得物件。
- IDE 工具上，原生無法透過點擊 `someMethod` 轉跳到該方法的定義。(至少 VScode 不行，但在 Sublime Text 上可以。)

# 改用 Dynamic Facade 解決依賴注入缺點

上面的問題在使用 Dynamic Facade 改寫之後：

```php
// Test.php
use Facades\MyClass;

MyClass::shouldReceive('someMethod')->andReturn();

// Source.php
use Facades\MyClass;

MyClass::someMethod();
```

已經沒有上述的問題，而且在類別方法的呼叫方面更貼近原生 PHP 的寫法。當然相信 Dynamic Facade 也不是完全沒有缺點，等後續的使用中慢慢體會囉～


---
title: "Laravel Command Class Name"
date: 2023-10-31T11:31:24+08:00
lastmod: 2023-10-31T11:31:24+08:00
draft: false
keywords: []
description: ""
tags: [Laravel, Plugin Development, Class name]
categories: []
author: ""
---

## Why

隨著 Laravel 開發的指令越多，要記得每一個指令的檔案位置也相對困難。由於從指令列執行 `artisn list` 只顯示每個指令所設定的 `signature` 屬性，但不曉得對應的檔案名稱。只期望 signature 跟檔案名稱具有相似性，可以順藤摸瓜。也因此才有了開發相關套件的念頭。

## How

之前在 [Laravel Goto](https://marketplace.visualstudio.com/items?itemName=absszero.vscode-laravel-goto) 的 VSCode 套件開發上，是透過檢索每一個指令的 `signature` 屬性，建構出每一個指令與相對應檔案的關聯。這次希望透過 `artisan` 指令的執行，能夠將指令的類別名稱與路徑名稱顯示出來。

最初的構想是透過添加一個全域型的 `--classname` 參數顯示指令列表的類別名稱，類似如下：

```shell
$php ./artisan --classname
Available commands:
  clear-compiled        Illuminate\Foundation\Console\ClearCompiledCommand
  completion            Symfony\Component\Console\Command\DumpCompletionCommand
  ...
```

不過翻開原始碼，`artisn list` 指令藉由 `Symfony\Component\Console\Command\ListCommand` 顯示其內容。

`ListCommand` 藉由 `Symfony\Component\Console\Helper\DescriptorHelper` 挑選出合適的 Descriptor 類別。預設會使用 `Symfony\Component\Console\Descriptor\TextDescriptor`，其中 `describeApplication` 方法描述如何渲染 `artisn list` 指令的顯示內容。

如果要達到上述的功能，勢必須要修改 TextDescriptor 類別。要動到 `Symfony` 的套件難度太大。後續決定以獨立指令的方式來實現。

## Now

後續建立了一個全新的 Laravel Package，設計一個名為 `classname` 的指令。並且藉由繼承 `TextDescriptor` 並覆寫 `describeApplication` 來實現。由於原本的實作只需少許修改就能滿足需求，也因此幾乎照搬 describeApplication 的程式碼。由於希望該套件能能支援 Laravel 5.6 ~ 10.x 之間的版本，相依的 `symfony/console` 套件也橫跨了 4.x ~ 6.x 之間的版本。其中差異的部分是對於字元寬度的計算邏輯，透過簡單的邏輯判斷解決了該問題。

另外為了能夠顯示指令的路徑，增加 `--path` 選項，並利用 `ReflectionClass` 反射取得檔案位置：

```php
$reflector = new ReflectionClass($command);
$commandDescription = $reflector->getFileName();
```

最後增加 `command-name` 參數，可以針對特定的指令顯示其指令類別

```shell
# classname [options] [--] [<command-name>]
$php ./artisan classname clear-compiled

 Available commands:
  clear-compiled        Illuminate\Foundation\Console\ClearCompiledCommand
```

完整的專案位置：[Laravel Command Classname](https://github.com/absszero/laravel-command-classname)

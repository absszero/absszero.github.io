---
title: "透過 PHP 解析包含 BOM 的 CSV 文件"
date: 2019-08-26T20:12:35+08:00
draft: false
tags: [BOM, CSV]
---

之前拿到的 csv 檔大部分都很乾淨，使用 PHP 內建的 `fgetcsv` 逐行解析不會有什麼問題，但今天解開一個欄位就發現它的字串長度跟預期的不同，並且無法用 `ltrim` 函式移除左側不要的字元，印出來看起來又正常。於是透過 `ord` 函式把前幾個字元印出來瞧瞧。

```php
<?php

$handle = fopen("test.csv", "r"));
$data = fgetcsv($handle));

// 239
echo ord($data[0][0]);
// 187 
echo ord($data[0][1]);
// 191
echo ord($data[0][2]);
```



按照過去的經驗， 開頭的不可見字元應該是 BOM，查了 Wiki 也確定這三個數字是 UTF-8 的 BOM 沒錯。由於第一行是欄位說明，所以直接略過取第二行。
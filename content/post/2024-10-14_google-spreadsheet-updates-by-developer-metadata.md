---
title: Google spreadsheet updates by developer metadata
date: 2024-10-14T20:23:43+08:00
draft: false
keywords: ["spreadsheet", "metadata"]
---

透過 Google Sheets API 可以很方便的寫入資料到 Google 試算表。而我最近的一個挑戰是要更新後續新增的的資料。主要的難處是檢索，由於 API 不支援資料搜尋，所以無法透過查找特定內容取得資料位置。這個問題也同樣困擾其他使用者。

我在 [Google Sheets API: How to find a row by value and update it's content](https://stackoverflow.com/questions/49161249/google-sheets-api-how-to-find-a-row-by-value-and-update-its-content) 找到想要的答案。方法是透過將特定的資料打上 `developer metadata` ，後續就可以針對打上的 metadata 進行檢索。

然而我能找到的範例有點零碎，甚至是用 JavaScript 寫的簡單範例，很難完整的套用到 PHP 上面。為此折騰一番之後，總算裡出自己的心得，在此做一個紀錄，給未來的自己。


## 初始化 Google Sheet Service

初始化之前，當然你需要安裝 [google-api-php-client](https://github.com/googleapis/google-api-php-client)。


```php
$client = new Google\Client();
$client->setApplicationName('Google Sheets');
// 設定權限
$client->setScopes([Google\Service\Sheets::SPREADSHEETS]);
$client->setAccessType('offline');
// 引入金鑰
$client->setAuthConfig(__GOOGLE_SHEET_KEY_FILE);
$sheets = new Google\Service\Sheets($client);
```

## 設定 Metadata

google-api-php-client 有獨立的類別可以設定底下的 payload，但對我而最簡便是直接用 array 設定。所以底下直接使用 array 設定請求內容。

```php
$payload = [
    [
        "createDeveloperMetadata" => [
            "developerMetadata" => [
                "location" => [
                    "dimensionRange" => [
                        "sheetId" => 0, // 通常可以在網址列的 gid=xxxxx 找到
                        "dimension" => 'ROWS', // metadata 會套用在整個 row 上
                        "startIndex" => 0, // 第一列索引值是 0
                        "endIndex" => 1 // 結束列，但不包含該列
                    ]
                ],
                "visibility" => "DOCUMENT",
                "metadataKey" => 'MY_KEY',
                "metadataValue" => 'MY_VALUE'
            ]
        ]
    ]
];
$request = new Google\Service\Sheets\BatchGetValuesByDataFilterRequest();
$request->setRequests($payload);

// 成功之後，內容會包含 metadataId
$result = $sheets->spreadsheets->batchUpdate($spreadsheetId, $request);
```

## 根據 Metadata 查找

查找透過 `metadataKey` 與 `metadataValue`，也可以透過 `metadataId` 查找。


```php
$payload = [
    "developerMetadataLookup" => [
        "metadataKey" => 'MY_KEY',
        "metadataValue" => 'MY_VALUE',
    ]
];
$request = new Google\Service\Sheets\BatchGetValuesByDataFilterRequest();
$request->setMajorDimension('ROWS');
$request->setDataFilters([$payload]);

$result = $sheets->spreadsheets_values->batchGetByDataFilter($spreadsheetId, $request);
```

## 結語

透過 metadata 的好處是當標記的資料移動位置之後，你仍然能正確的存取到該資料，而不會因為使用者異動試算表而失效。麻煩的點是每次新增資料都要打上 metadata。

所以如果需要完整的 CRUD，還是使用資料庫會容易一些。只是這次的實作，主要是為了方便使用者使用試算表檢索資料及操作，所以將結果統整在上面。
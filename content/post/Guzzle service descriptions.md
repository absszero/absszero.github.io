---
title: "Guzzle service descriptions - 使用描述檔建立 API Client"
date: 2019-10-06T15:01:03+08:00
draft: false
tags: [Guzzle API]
---


## 何謂 Service Descriptions

最近在查找關於 Fixer.io 的用戶端套件。發現平常慣用的 Guzzle  HTTP Client 還有另外一種開發方式 - `service descriptions`。該方式借鏡 Swagger 利用定義 API 操作的方式，藉由特定描述的 PHP Array 建立 API Client 操作。簡單的說，只要產生一個 PHP Array 裡面描述每一種 API 的格式說明，就可以呼叫 API 囉！



## 定義 Service Descriptions

基本上只要能把 PHP Array 生成的格式都可以用來定義 Service Descriptions，目前最方便的方法是透過 PHP  Array 或者 JSON。

底下以 Open Weather Map 提供的當前天氣 API 為例：

```php
// description.php
<?php return [
    // API Base URL
    'baseUri' => 'http://api.openweathermap.org/data/2.5/',
    // 各式 API 操作描述
    'operations' => [
        // 調用方法，之後透過 $client->current() 調用該 API
        'current' => [
            'httpMethod' => 'GET',
            // 會跟上方的 baseUri 組合成完整的 API 網址
            'uri' => 'weather',
            // 回傳內容，跟底下的 Model 相對應
            'responseModel' => 'getResponse',
            // 參數
            'parameters' => [
                "APPID" => [
                    // 參數放置: query, uri 或者 json 等等
                    "location" => "query", 
                    "type" => "string",
                    "required" => true, // 必填欄位
                ],
                'city' => [
                    'type' => 'string',
                    'location' => 'query',
                    'default' => 'London,uk', // 預設值
                    'sentAs' => 'q', // 傳送時的欄位名稱，會將 city 改為 q 
                    "required" => true,
                    // 傳送前用調用指定的方法做預處理
                    "filters" => [
                    	"strtolower",
                    ]
                ]
            ],
        ]
    ],
    'models' => [
        'getResponse' => [
            'type' => 'object',
            'additionalProperties' => [
                'location' => 'json'
            ]
        ]
    ]
];

```



## 調用 API

```php
<?php
use GuzzleHttp\Client;
use GuzzleHttp\Command\Guzzle\GuzzleClient;
use GuzzleHttp\Command\Guzzle\Description;

$client = new Client();

// 引入描述檔
$desc = require 'description.php';
$description = new Description($desc);

// 提供每一個 API 通用的 query 
$config = ['defaults' => ['APPID' => '']];
$guzzleClient = new GuzzleClient($client, $description, null, null, null, $config);

$result = $guzzleClient->current(['city' => 'LONDON,UK']);

echo $result['main']['temp'];

```



## 其他功能

- responseClass ，將回傳的內容用特定的類別包裹
- errorResponses，將錯誤訊息用特定的類別包裹並拋出



## 參考

* https://github.com/guzzle/guzzle-services
* https://guzzle3.readthedocs.io/webservice-client/guzzle-service-descriptions.html
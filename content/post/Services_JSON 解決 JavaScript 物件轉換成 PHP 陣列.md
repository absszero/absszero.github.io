---
title: 'Services_JSON 解決 JavaScript 物件轉換成 PHP 陣列'
date: Wed, 08 Apr 2015 15:21:40 +0000
draft: false
tags: [IT 筆記, JavaScript, JSON, object, PHP, services_json]
---

[Services_JSON](https://pear.php.net/package/Services_JSON) 是 json_encode/json_decode 的實做，雖然 PHP 5.2 已經可以直接使用，不過格式比須是嚴謹的 JSON 格式(RFC 4627)。 但 JavaScript 的物件結構本身跟 JSON 格式相仿，這時候可以透過 Services_JSON 提供的 **SERVICES_JSON_LOOSE_TYPE** 把物件轉換成 PHP 陣列。 Service_JSON 提供的檔案很輕巧，只有一支 JSON.php
{{< highlight php >}}
<?php
$json_text = '{abc:123}';

// 正確的 JSON 格式應為 {"abc":123}，鍵值的部份用雙引號包裹 $json = new Services_JSON(SERVICES_JSON_LOOSE_TYPE);

$data = $json->decode($json_text);

/*array(1) { 'abc' => int(123) }*/
{{< /highlight >}}

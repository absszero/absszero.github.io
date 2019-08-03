---
title: 'Javascript 關鍵字使用 in 判斷物件是否有該屬性成員'
date: Wed, 22 Jul 2015 14:15:09 +0000
draft: false
tags: [IT 筆記, JavaScript]
---

今天在別人的 js 範例當中看到一個關鍵字 in，
{{< highlight javascript >}}
if ('items' in obj) {
    return obj.items;
}
{{< /highlight >}}
 過去用過的 in 是 for..in，做為疊代物件屬性
{{< highlight javascript >}}
    for (var prop in obj) { ... }
{{< /highlight >}}
 從 stack overflow 看到這篇說明 [JavaScript object detection: dot syntax versus 'in' keyword](http://stackoverflow.com/questions/7174748/javascript-object-detection-dot-syntax-versus-in-keyword) 裡面的範例解釋很貼切，這表示這是用來判斷屬性是否存在，而不是屬性的內容。
{{< highlight javascript >}}
    var object = {property: 0};
    if(object.property) { ... } // will not run if('property' in object) { ... } // will run
{{< /highlight >}}
 如果考量到物件原型(prototype)有一樣名稱的情況，更精確的使用如下， 如此可以限制屬性只限定在目前物件當中。
{{< highlight javascript >}}
    if(object.hasOwnProperty('property')) {...}
{{< /highlight >}}

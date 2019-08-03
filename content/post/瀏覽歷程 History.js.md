---
title: '瀏覽歷程 History.js'
date: Tue, 07 Apr 2015 14:16:45 +0000
draft: false
tags: [API, history.js, html5, IT 筆記, JavaScript, pushstate]
---

AJAX 載入會使得瀏覽器的瀏覽歷程沒辦法正確運作。 HTML5 的 History API 滿足了方面的需求。 而 History.js 把 HTML5 跟 HTML4 以及各個瀏覽器之間的差異統一了。 {{< highlight javascript >}}
History.Adapter.bind(window,'statechange',function(){
    var State = History.getState();
});

History.pushState({state:1}, "YOUR_TITLE", "?URL_PARAMETER=1");
{{< /highlight >}}
 目前在 AJAX 載入分頁上面使用，可以回到上一頁而不是每次都返會第一頁。 透過 pushState 的操作同時還可以變更頁面的標題，告知使用者之前頁面位置。 而 url 參數的部份也可以跟著變動，方便在伺服器端根據不同的參數，載入不同的分頁位置。
---
title: 'VisSense.js 與延遲載入'
date: Sat, 28 Mar 2015 05:14:33 +0000
draft: false
tags: [IT 筆記, lazyload, VisSense]
---

[VisSense.js](http://vissense.github.io/vissense) 可以提供某個 HTML 元件目前在使用者的畫面出現的情況。例如是否完全出現(fullyvisible)、隱藏(hidden)，甚至出現出現在畫面上的比例是多少。也可以對元件的出現狀況進行監控，了解出現時間的長短。看 [Demo](https://vissense.github.io/vissense-demo/#/demos/demo-track-visibility) 有了這樣的功能，便可以選擇性的在使用者移動到某個畫面的時候，才載入該畫面的元件。例如圖片、社群元件、廣告等等... 圖片
{{< highlight html >}}
<img class="lazyload" alt="" width="624" height="480" data-src="http://i.imgur.com/XjFZG2Q.jpg" />
<script> $('img.lazyload').each(function(){
    var element = $(this);
    var visibility = VisSense(this, { fullyvisible: 0.05 });
    visibility.monitor({ fullyvisible: function(monitor) {
        element.attr('src', element.data('src'));
        monitor.stop();
    } }).start();
 });
</script>
{{< /highlight >}}
 Facebook Like button
 {{< highlight html >}} <div class="fb_like_button"> <div class="fb-like" data-href="http://yor_http_link/" data-layout="button_count" data-action="like" data-show-faces="false" data-share="false"></div> </div>
<script> //載入 Facebook SDK
$.getScript('//connect.facebook.net/en_UK/all.js', function(){ FB.init({ appId: 'YOUR_APP_ID', }); });

$('div.fb_like_button').each(function(){
    var element = this;
    var visibility = VisSense(element, { fullyvisible: 0.05 });
    visibility.monitor({ fullyvisible: function(monitor) {
        FB.XFBML.parse(element);
        monitor.stop();
    } }).start();
});

</script>
{{< /highlight >}}

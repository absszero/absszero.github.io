---
title: '[ Web ] Accelerate scripts running multiple tasks in parallel using asynchronous programming: Unusual Site Speedup Techniques: Part 3 - PHP Classes blog'
date: Mon, 15 Nov 2010 12:41:05 +0000
draft: false
tags: [asynchronous programming, Chrome, 非同步程式設計, 輪詢, Gearman, Google, IT 筆記, JavaScript, Node.js, PHP]
---

http://www.phpclasses.org/blog/post/133-Accelerate-scripts-running-multiple-tasks-in-parallel-using-asynchronous-programming-Unusual-Site-Speedup-Techniques-Part-3.html

Introduction
------------

許多伺服器上執行程序會花這麼長的時間，主要在於程序花了很多時間在等待其他程式或者硬體訊息或其他遠端電腦的回應。 那能不能在回應等待的這個時間內，同時進行一些事情呢？ 答案是可以的，但前提示是要採用 asynchronous programming ( 非同步程式設計 )。

What is asynchronous programming?
---------------------------------

就像在準備晚餐，你打算準備一碗湯當作前菜，然後主餐是牛排，最後甜點是蛋糕。 你當然可以煮好一個之後再煮另外一個，但當某一道菜在火爐上亨煮的同時， 你其實可以開始準備另外一道菜，減少整體調理的時間。 asynchronous programming 就類似這種方式。假設現在有一個情形是一個網頁。 網頁許多部份由資料庫不同的查詢結果組成，通常你進行一個查詢之後，將查詢結果放到一個變數當中， 接著進行另外一個查詢，然後再放到其他的變數，反覆上述的動作，然後將結果組合起來，產生最後的 HTML。 如果現在你在等待前一個查詢結果的同時，馬上又發起了其他部份的查詢，結果就是你可節省了大量的時間， 就好煮晚餐的時候，同時進行多項料理的處理。

Implementing asynchronous programming in practice
-------------------------------------------------

在低階程式設計語言當中，你需要跟底層的作業系統溝通，好了解哪些程式正在等待而哪些已經完成了。 如此你才能在正確的時候觸發相關的後續動作。為此可能需要週期性的輪詢各種系統資源以確認發生的情形。 就好像做晚餐，要時常查看在火爐上亨煮的食材，等他好了之後才作下一個動作。 而在高階的程式設計語言當中通常有許多簡單的方式可以實現 asynchronous programming。 通常你可以透過某些函式設定回呼函式( callback function )。 回呼函式可以在某些事件發生之後被自動的呼叫，甚至可以設定多個回呼函式好對應不同的情形。 事實上他們也是在暗地裡週期性的檢查事件的發生情形，不過你不需要控制這些部份，系統會幫你做好這個部份。

Asynchronous programming in JavaScript
--------------------------------------

在高階語言當中具有內建 asynchronous programming 的例子之一就是 JavaScript。 在瀏覽器上就經常針對不同的事件定義相關的程式碼。 你透過使用像是 onclick、onmouseover 等等屬性指派相關的動作，例如下列的例子：`Click me!`
另外你也可以透過給特定頁面元素，註冊事件回呼含是的方式，像是使用 addEventListener。 {{< highlight html >}} <div id="mydiv">Click me!</div>
{{< /highlight >}}


Server side asynchronous programming in JavaScript with Node.js
---------------------------------------------------------------

上述的例子是描述在瀏覽器上的運作情形。想想看有沒有可能在伺服端實現 asynchronous programming？ 經過許多年的努力，現在有一些伺服端的解決方案，可以讓你在伺服端使用 JavaScript 達到類似的功能。 其中一個就是 [Node.js](http://nodejs.org/)。 他是根據 [Google V8](http://code.google.com/p/v8/)的 JavaScript 執行引擎所打造的 framework。 也就是 Google Chrome 所採用的 JavaScript 執行引擎。 他執行快速的秘訣在於他可以把 JavaScript 轉換成與 CPU 相關的原生機械碼。 Node.js 包含了 V8，並且更廣大的開發許多模組，使他可以執行像是檔案存取、資料庫處理以及網路通訊的功能。 Node.js 跟 PHP 最大的差別在於 Node.js 在預設的情況，所有的一切都是以非同步的方式執行。 它代表每一個在 Node.js 的函式在執行之後會直接返回，而只有當真正的工作結束之後， 並且呼叫了設定的回呼函式才會得到結果。 舉個例子，如果你想要列出目前目錄底下檔案，你就得設定一個函式給它。 {{< highlight javascript >}}
var fs = require('fs');

fs.readdir(".", function(error, files) {
    if(error) throw error;

    for(var f in files)
        console.log("Path: " + files\[f\]);
    }
);
{{< /highlight >}}


Asynchronous programming in PHP
-------------------------------

不像 Node.js，PHP 當中每個動作都是同步執行。 這表示當你呼叫了一個函式，只有當這個函式執行完成並返回結果之後才會繼續向下執行。 目前 PHP 當中並沒有內建任何可以利用回呼函式方式達成 asynchronous programming 的方法。 不過仍然有一些函式可以達到某些 asynchronous programming 的效果，例如 [stream_select](http://tw2.php.net/stream_select) 他的功能是透過輪詢的方式檢查開啟的檔案或者網路連線，確認在設定的時間內或資源結束之前，有沒有任何資料讀取或寫入。 另外一個實現 asynchronous programming 的方法是使用 Gearman， 它是一種中介程式並且以獨立伺服器方式運作。用以呼叫進行特定程式，稱為 worker 的程式。 而這些 worker 程式可以用 PHP 來撰寫，所以你可以撰寫任何所需的 worker 程式。 而 woker 除了執行相同的主機上的，甚至可以是遠端的主機也行。 你可以透過 César Rodas 所寫的 [how to use Gearman with PHP](http://www.phpclasses.org/blog/post/108-Distributing-PHP-processing-with-Gearman.html)了解更多關於 Gerarman 的資訊。 其中一個有助於 asynchronous programming 地方在於他能夠以非同步的方式呼叫工作進行。 這表示當他呼叫某個工作進行，呼叫的程式不需要等待該工作完成。 負責取得資料的非同步工作會自行完成，不過這當中還是有一些問題，所以這並非完全的解決方案。 其他解決方法還有使用 PHP 的 [pcntl](http://www.php.net/pcntl) 擴充套件， 藉由產生新的程序來達成同步執行 PHP。不過該方法對於 CPU 跟記憶體的消耗比較大。

Conclusions
-----------

在 PHP 使用 asynchronous programming 是有可能的，只是目前的方式並不是那麼直接好用。 PHP 5.3 之後新增了 closure 的支援，closure 基本上就是匿名函式， 他讓你在呼叫函式的地方，直接使用定義函式的方式設定一個函式。 而這一步對於在 PHP 上開發 asynchronous programming 變得更加友善。 而不用像以前一樣，得先定義一個有具體名稱的函式然後才能使用。 另外一方面，asynchronous programming 也改變了程式執行的先後順序， 進而改變我們的思考方式。例如哪些地方該使用同步而哪些地方該使非同步操作？ 該問題已經超過了目前這個主題，你可以從 dealing with control flow challenges of asynchronous programming 得到更多資訊。
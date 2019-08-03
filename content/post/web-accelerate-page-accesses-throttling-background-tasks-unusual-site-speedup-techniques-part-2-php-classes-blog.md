---
title: '[ Web ] Accelerate Page Accesses Throttling Background Tasks: Unusual Site Speedup Techniques: Part 2 - PHP Classes blog'
date: Mon, 15 Nov 2010 12:31:49 +0000
draft: false
tags: [CPU, IT 筆記, Loading, PHP]
---

http://www.phpclasses.org/blog/post/132-Accelerate-Page-Accesses-Throttling-Background-Tasks-Unusual-Site-Speedup-Techniques-Part-2.html

Introduction
------------

通常網站速度決定了一個網站的成功與否。 而這方面是從使用者的觀點，觀察頁面是否能夠夠快的被處理呈現。 前一部分討論緩慢的原因是因為外部內容的影響，像是廣告或這其他外掛工具。 本篇會討論伺服器環境所造成的影響。

Server side background processes
--------------------------------

在一部網站伺服器並非所有的程序都是用來產生網頁內容的，先把這些非相關的稱為 **background processes**。 而這類的程序仍然執行與網站相關的事情，像是發送電子報、備份、爬找網站頁變並且更新內部搜尋引擎索引... 而這些眾多的其中程序，必須要確保不會因為作業上佔用系統資源而影響到網站的運作。

CPU Throttling of background processes
--------------------------------------

要避免 background processes 所可能造成的影響，最明顯的方式就是把這些工作分散到其他的主機上。 不過該方式也增加了管理其他主機的需求，並且在這個例子當中，我們不把他當作主要的解決方法。 假設我們有一個電子報發送的工作，發送的時候會在短時間內佔用大量的系統資源。 但電子報的發送工作就算暫緩也不會造成太大的影響，所以我們可以利用放慢該作業速度的方式減緩對系統資源的快速耗用。 這個方式是在需要的時候，透過控制程式的執行速度來達成，這種方式稱為 **CPU throttling**。 CPU throttling in PHP 要達成該目的的其中一個方法是在程式當中加入重複的迴圈，停止程式向下執行。 在 PHP 當中我們可以使用 sleep() 指定等待的秒數之後來達到該功能。 現在另外一個重點是了解何時是使用的時機？關於這點就得了解一下作業系統的運作方式。 先進的作業系統多半是先占式多工( preemptive multi-tasking )，表示程序會排在一個佇列當中，輪流使用 CPU 資源。 為此 CPU 會分割出許多時段，讓程序輪流使用，一旦一個時段的使用時間結束，就輪流到另外一個程序使用。 當執行太多繁重的程序時，就會增加其他程序的等待時間。而目標就是降低等待中的程序數量。 在 Linux 上可以透過 /proc/loadavg 得知等待的程序數量。 並且可以透過 PHP 的 fopen() 以及 fread() 或者 file_get_contents() 取得其中的內容。

The system monitor class
------------------------

調節 CPU 的過程相當簡單，我們查看 loadavg 檔案內容確定目前負載是否太高， 如果過高就呼叫 sleep() 進入等待，然後過一陣子之後在檢查負載是否依然過高， 如果是就持續等待，直到負載降到我們期望的數值。 為了簡化這個過程，原作者發展了一個名為 [system monitor](http://www.phpclasses.org/system-monitor) 的類別，並且發布在 PHPClasses。 該類別當中有一個名為 Throttle 的方法，它會返回一個數值告知是否超過的預設的負載， 你可透過類別的 cpu_load_limit 變數控制預設負載。 所以現在的問題就是要如何來決定何時 CPU 的負載是屬於過高的，而何時 CPU 的負載已經降當可接受的程度？ 原作者根據 CPU 的數量與核心數量來計算。 假設有 1 顆含有雙核心的 CPU，則該 CPU 的負載極限為 1 x 2，然後再乘於 2，也就是 2 x 2 = 4。 也就是說當有兩個或超過兩個程序進入等待，那 background processes 就應該進入睡眠狀態。 另外一個你可能想知道的因素還有進入睡眠的時間應該多久？以及多久應該檢查目前負載的高低？ Linux 的 loadvag 提供了三個的統計數值，分別代表 最近 1 分鐘、最近 5 分鐘以及最近 15 分鐘的負載情形。 預設 system monitor 會使用最近 1 分鐘的數值，但你可以決定使用哪一個。 根據這點，你可以設定每 60 秒才進行一次負載查詢。但你可以更頻繁的查詢，好處理急促的負載變動。 另外程序的睡眠時間也可以合理的設定為 60 秒，但你仍然可以減少該時間以確保 background processes 不會延遲太多。 你的處理程式應該會看起來像下列這樣：
{{< highlight php >}}
$monitor = new system_monitor_class;
$monitor->cpu_load_limit = 4;

// Work loop starts here
for($i = $start;

$i Throttle($cause))
    die('Fatal error: '.$monitor->error);
    if($cause == THROTTLE_CAUSE_NONE)
        break;

sleep(60);
    }
}

{{< /highlight >}}


Throttling external programs
----------------------------

透過上述的說明要控制 PHP 程式變得非常容易。 但是對於非 PHP 程式要如何調節則是另一個問題。 PHPClasses 一開始就遇到過這個問題， 由於內部使用的搜尋引擎使用的是一個名為 HtDig 的外部程式。 他是一個古老的開放原始碼程式，經由 C++ 撰寫而成。 他主要用來爬找整個網站的索引。由於爬找的頁面超過 50,000 頁所以執行時間相當長。 也許應該重新設計一個使用 PHP 的搜尋引擎，這樣就比較容易調節使用。 不過就目前而言這並不是有能立即解決的辦法，另外也得因此而改變 HiDig 跟 PHPClasses 的整合。 幸好在 Linux 上面有一個簡單的方法可以讓目前的程序進入睡眠狀態， 只要你執行的身份跟程序相同或者以 root 執行即可。 這個方式主要的內容是根據傳遞一個 SIGSTOP 訊號給正在執行的 background process。 它會讓程序進入睡眠狀態，直到他收接到一個 SIGCONT。 在 PHP 你可以透過 proc_terminate() 傳遞一個訊號給正在執行的程序。 如果你希望程序能夠透過 PHP 開啟，並且能夠加以控制， 你可以透過 proc_open() 執行，並且讓他在背景執行。 原作者最近改良了 system monitor 類別，讓他能夠透過 ThrottleExecute() 方法執行外部程式。 程式部份看起來應該會類似於底下這樣：
{{< highlight php >}}
$monitor = new system_monitor_class;
$monitor->cpu_load_limit = 4;
$command = 'path_to_external_command command parameters';
$parameters = array( 'Command' => $command, 'CaptureOutput' => 'string');
if(!$monitor->ThrottleExecute($parameters)) {
    die('Fatal error: '.$monitor->error);
}
if($parameters\['Status'\] != 0) {
    die('the command failed for some reason');
}

do_something_with_output($parameters\['Output'\]);
{{< /highlight >}}

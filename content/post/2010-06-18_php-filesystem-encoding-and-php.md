---
title: '[ PHP ] Filesystem encoding and PHP'
date: Sat, 19 Jun 2010 07:26:16 +0800
draft: false
tags: [IT 筆記, Linux, Mac, PHP, UTF-8]
---

許多 PHP 應用程式會將檔案儲存本地端的檔案系統當中。 對於大部分讀者而言在多數的情況下，妳很可能只使用 US-ASCII 編碼的方式儲存檔案， 可能是因為妳的檔名是基於資料庫欄位 ( 這也是大部分情形妳應該使用的方式 )， 或者是因為妳的使用者不需要使用非英文以外的字元。 當妳需要處理不同字元，很重要的就是去瞭解不同作業系統之間對字元的處理方式。 而令人不感到意外的是，他們之間的確使用不同的方式處理。 為了描述這些差異，我將透過 Ubuntu，OS/X 10.6.3 以及 Windows XP 和 7 做相同的測試。

Linux
-----

在 Linux 檔案名稱是以二進制的方式存在。 Linux 不在乎妳使用何種編碼方式，因此它允許任何字元除了 0x00。 這代表檔案名稱可以包含換行字元 (n)，跳格字元 (t) 甚至鈴響符號( ascii code 07 )。 為了描述這一點，我打算透過 PHP 撰寫一個簡短的程式：
{{< highlight php >}}
<?php file_put_contents("saved by the x07.txt","contents");

?>
{{< /highlight >}}
 當程式執行之後，我透過 'ls' 查看的時候得到一個問號， 但是當我使用自動補齊的時候，它自動補齊為 ^G ( 這就是鈴響符號 )。 在 Nautilus 當中，它顯示 [如此](http://www.rooftopsolutions.nl/blog/user/files/posts/fsencoding_gnome.png)。 如果妳執行底下的程式碼：
{{< highlight php >}}
<?php print_r(glob('saved\*'));

?>
{{< /highlight >}}
 結果顯示它遺失了我設定的鈴響字元，並且我聽到一聲簡短的 "嗶" 聲。 我不認為這樣的結果是個好情形。即使底層的檔案系統允許這種情況， 應用程式在列出檔案名稱的時候，仍然會試著編碼該字元然後顯示給使用者。 但妳無法在任何 PHP 的頁面上顯示該字元，甚至當妳在網址列中使用該字元可能遭受防火牆阻擋。 這個情況同樣適用於 linux 主機上的應用程式。 大部分的應用程式像是 Gnome Terminal 或者 Nautilus，預設是使用 UTF-8。 但對於 PuTTY 它仍然是是使用最古老的 ISO-8859-1 (latin1)。 使用 PuTTY 跟 Nautilus 明顯的差異在於閱讀任何非 ASCII 字元的時候。 其他我想在 linux 上面測試的部份，是關於使用 filemanager 建立包含特殊字元的檔名的情形。 為了測試這個部份，我使用 ü ，因為這個字元在使用 unicode 轉換時有諸多個方法而顯得有些混亂， 並且該字元也存在於 ISO-8859-1。 回到測試。我現在要從 Nautilus 介面底下產生一個新的檔案，並且從 PHP 觀察它會有什麼反應。 我建立一個檔案名稱為 test_ü.txt 的檔案，並且使用底下的程式將它列出：
{{< highlight php >}}
<?php list($file) = glob('test_\*');

echo urlencode($file) . "n";

?>
{{< /highlight >}}
 **輸出結果：**test_%C3%BC.txt %C3%BC 是編碼位置( codepoint )U+00FC 的 UTF-8 編碼，針對 ü 最普遍的編碼方式，非常好！ 接接下來的測試是要使用 ISO-8859-1/latin1 編碼， latin1 中 ü 所代表的是 0xFC。程式部份是這樣：
{{< highlight php >}}
<?php file_put_contents("uumlaut_xFC.txt","contents");

?>
{{< /highlight >}}
 Linux 以嚴謹的字節順序( byte sequence )儲存該檔案。'ls' 顯示結果又是問號， 並且在 gnome 的型態顯示上我所得到的是 '不正確編碼' 的問號。

OS/X
----

在 OS/X 上面，所有的檔名以 UTF-16 的方式儲存。妳不需要特別去瞭解這個， 因為 PHP 的 API 所使用的是 UTF-8，並且會自動幫妳轉換的動作。 我們會從響鈴符號開測試。結果跟 linux 上的相同。響鈴字元被表示成 ?。 使用查找器搜尋的時候，這個字元則是完全消失了。當使用下列程式顯示的時候它卻存在：
{{< highlight php >}}
<?php list($filename) = glob('saved\*');

echo urlencode($filename) . "n";

?>
{{< /highlight >}}
 **輸出結果：**saved+by+the+%07.txt 接著，我們將進行 ü 的測試，我將使用 latin-1 進行編碼，在這個 UTF-8 的檔案系統上，它應該會顯示不正確的結果。
{{< highlight php >}}
<?php file_put_contents("uumlaut_xFC.txt","contents");

?>
{{< /highlight >}}
 但有點古怪是，如果我現在使用 'ls' 我得到的結果會是： {{< highlight bash >}} drwxr-xr-x 10 evert2 staff 340 16 Apr 17:08 . drwxr-xr-x 32 evert2 staff 1088 16 Apr 16:53 .. -rw-r--r-- 1 evert2 staff 8 16 Apr 16:54 saved by the ?.txt -rw-r--r-- 1 evert2 staff 121 16 Apr 16:54 test1.php -rw-r--r-- 1 evert2 staff 8 16 Apr 16:54 test2.php -rw-r--r-- 1 evert2 staff 101 16 Apr 17:07 test3.php -rw-r--r-- 1 evert2 staff 57 16 Apr 17:08 test4.php -rw-r--r-- 1 evert2 staff 8 16 Apr 17:08 uumlaut_%FC.txt
{{< /highlight >}}
 OS/X 沒有按照字面上內容處理，而是將它做 urlencode，並且用其結果進行儲存。 這個過程是自動的；但是當妳的使用者想要使用 latin1 的檔案名稱時可能會造成混亂。 最後測試要再做一次變音符號( umlaut )的儲存，但這次使用的是正確的 UTF-8 編碼：
{{< highlight php >}}
<?php file_put_contents("uumlaut2_xC3xBC.txt","contents");

?>
{{< /highlight >}}
 起初認為應該會如預測般儲存，但奇怪的是當我們檢查實際儲存的情形：
{{< highlight php >}}
<?php list($file) = glob('uumlaut2_\*');

echo urlencode($file) . "n";

?>
{{< /highlight >}}
 **輸出結果：**uumlaut2_u%CC%88.txt OS/X 用 u0xCC88 取代了 0xC3BC。注意到這個 u 可是打錯的。 OS/X 使用不同的方式儲存 ü。我們所使用的編碼是 unicode 編碼位置的 U+00FC，它是 ü 的所在。 OS/X 首先儲存 u，接著儲存兩個逗點，它使用 3 個位元來儲存而不是 2 個位元。 這個過程稱為正規化( normalization )。 Unicode 定義了少數幾個正規化模型說明相互組合的字元該如何儲存。 所以即使他們擁有不同的字節順序跟編碼位置，他們所代表的意義還是相同的。 [PHP intel extention](http://kr2.php.net/intl) 包含某種類別，允許妳自訂 unicode 的正規化方式。 也就是 [normalizer](http://kr2.php.net/manual/en/class.normalizer.php) 這個類別。[文件](http://kr2.php.net/manual/en/class.normalizer.php)中簡短的說明四種正規化方法的出處。 OS/X 使用稍微修改過的正規劃形式 D 標準( 的確，即使有標準的規劃，但沒有人會完全按標準設計。 ) 這裡說明如何使用自訂的方法轉換：
{{< highlight php >}}
<?php $before = "xC3xBC";

$after = Normalizer::normalize($before, Normalizer::FORM_D);

echo 'Before: ', urlencode($before), "n";

echo 'After: ', urlencode($after), "n";

?>
{{< /highlight >}}
 **輸出結果：** Before: %C3%BC After: u%CC%88 在 OS/X 上系統會自動進行正規化處理。 就算妳嘗試開啟一個非正規化標準的檔案，OS/X 也會在開啟之前直接根據 形式 D 套用。

Windows
-------

Windows 同樣使用 UTF-16 儲存檔案名稱( 透過 NTFS )。 跟 OS/X 相同，在PHP使用檔案系統的 API 時，轉換會自動進行。 我們將從響鈴測試開始：
{{< highlight php >}}
<?php file_put_contents("saved by the x07.txt","contents");

?>
{{< /highlight >}}
 **輸出結果：**Warning: file_put_contents(saved by the .txt): failed to open stream: Invalid argument in C:Documents and SettingsAdministratortesttest.php on line 2 事實上， Windows 不允許使用像是響鈴字元這類的控制字元。 接下來我們要嘗試 latin-1 編碼 ü：
{{< highlight php >}}
<?php file_put_contents("uumlaut_xFC.txt","contents");

list($file) = glob('uumlaut_\*');

echo urlencode($file) . "n";

?>
{{< /highlight >}}
 **輸出結果：**uumlaut_%FC.txt windows 不只在編碼工作上可以正確執行，cmd.exe 跟 windows explorer 都顯示了正確的結果。 這表示 windows 跟 PHP 確實從 ISO-8859-1/latin1 轉換到 UTF-8 了。 我們可以以 UTF-8 進行編碼來證實這一點。
{{< highlight php >}}
<?php file_put_contents("uumlaut2_xC3xBC.txt","contents");

list($file) = glob('uumlaut2_\*');

echo urlencode($file) . "n";

?>
{{< /highlight >}}
 **輸出結果：**uumlaut2_%C3%BC.txt 當 windows 正確儲存的時候，檔名在 cmd.exe 跟 windows explorer 底下會產生一些混亂的現象。 這裡看起來像是 Ã¼。這看起來挺糟的。我知道 Windows 並不支援 UTF-8， 所以我不禁想知道如果用 windows explorer 產生一個包含非 ascii 字元的檔案名稱， 然後從 php 讀出該檔案名稱會發生什麼事情。 結果很有意思，我用了 ü 跟 한글，她指的是韓國的書寫方式，韓文。 針對這個目錄下的底下這兩個檔案，我做了如下操作：
{{< highlight php >}}
<?php $files = glob('\*');

foreach($files as $file) { echo urlencode($file), "n";

} echo "total: " . count($files) . "n";

?>
{{< /highlight >}}
 **輸出結果：** test.php uumlaut%FC.txt total: 2 我的韓文檔案完全消失了。為了確認我用 scandir 再做一次：
{{< highlight php >}}
<?php $files = scandir('.');

foreach($files as $file) { echo urlencode($file), "n";

} echo "total: " . count($files) . "n";

?>
{{< /highlight >}}
 **輸出結果：** . .. hangul_%3F%3F.txt test.php uumlaut%FC.txt total: 5 說來奇怪它居然在這裡顯示的出來。儘管這時候韓文字元被%3F取代了：它代表問號字元。 我們之前也發生過字元被用問號取代的情形，但這是第一次發生在純字串字元上。

Conclusion
----------

在檔案名稱當中使用非拉丁字元會產生難以理解的結果。 如果不是在 Windows 下使用，其結果都可能一樣。 Windows 擁有完整的 API 處理各國的檔案名稱，但我猜 PHP 可能不完全支援。 我確信在 PHP6 已經針對該問題在規劃中，但目前卻有這方面的問題。 我希望在整個語言都 unicode 之前，檔案系統的 api 可以被替換掉。 至於 Linux( 使用二進制除儲存每一個東西，並允許任何 0x00 以外字元 )可能是最簡潔的方法， 最終的檔名能滿足人們讀寫上的需求，這也就表示檔名會經由編碼處理。 在這些案例當中最好的系統確實是 OS/X，不但使用 UTF-8 處理每一個部份， 並且將不正確的字元序列處理的很好，並且將相同意思但不同形式的字元以相同的方式儲存( 採用正規化 )。 這裡我是建議的方法： 如果妳希望在不同的作業系統上使用共通的作法處理所有的字元， 除了使用預先編碼( intermediate encoding )沒有其他的方式。 舉個例子妳可以在寫入磁碟之前簡單的透過 urlencode 處理所有的檔名。 使用 url 編碼不代表妳就不需要考慮編碼的問題。 url 編碼代表妳用不同的方式儲存這些位元，但這些字元仍然保持一致的意義。 所以妳必須確保妳的檔名保持有效的 UTF-8 字元序列。 UTF-8 已是今日編碼的首要選擇。 如果妳肯定妳只會用到 ISO-8859-1/latin-1 字集底下的字元， 那下列的表格既適用於這種情形：

Windows

使用 ISO-8859-1 編碼

Linux

使用 UTF-8 編碼 (允許其他的編碼形式，但不推薦).

OS/X

使用 UTF-8 編碼。會自動根據 D 形式的正規化進行轉換

底下表格順序說明各種作業系統的表現情況：

url 編碼的檔案名稱

描述

Linux

OS/X

Windows

%07

響鈴

磁碟上顯示為 %07

磁碟上顯示為 %07

產生錯誤並且無法儲存該檔名

%FC

ISO-8859-1 當中的 ü

在磁碟上顯示 %FC，視窗介面下顯示問號

在磁碟上顯示 %25FC ( %25 = %，所以在磁碟上表示為 %FC 的字串 )

在磁碟上顯示 %FC，視窗介面下也顯示正確的結果

%C3%BC

UTF-8 C 形式正規化中的 ü

在磁碟上顯示 %C3%BC，視窗介面下也顯示正確的結果

在磁碟上顯示 u%CC%88，視窗介面下也顯示正確的結果

在磁碟上顯示 %C3%BC，視窗介面下也顯示 Ã¼

u%CC%88

UTF-8 D 形式正規化中的 ü

在磁碟上顯示 u%CC%88，視窗介面下也顯示正確的結果

在磁碟上顯示 u%CC%88，視窗介面下也顯示正確的結果

未測試，但猜測跟上一個測試結果類似



Configuration list
------------------

最後，我所使用的相關軟體列表：

*   Windows
    *   同時在 XP SP3 跟 7 上測試
    *   由 [windows.php.net](http://windows.php.net) 所建置的 PHP 5.3.2 VC9 x86
    *   NTFS 檔案系統
*   Linux
    *   Ubuntu 9.10
    *   由 ubuntu package repository 取得的 PHP 5.2.10
    *   ext3 檔案系統
*   OS/X
    *   v10.6.3
    *   OS/X 上運作的 PHP 5.3.1
    *   HFS+ 檔案系統
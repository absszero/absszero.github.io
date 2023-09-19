---
title: '[ PHP ] Unpredictable hashes for humans'
date: Sun, 11 Jul 2010 12:41:50 +0000
draft: false
tags: [hash, IT 筆記, PHP, 加密]
---

Unpredictable hashes for humans http://seld.be/notes/unpredictable-hashes-for-humans 對網站開發人員產生隨機的 ID 或者雜湊值是很常見的事情， 舉個例子，大型的專案或者開發框架可能需要實現自己的 PHP session 處理器，或者在 API 當中把這個部份抽象化處理。 或是使用 [session_set_save_handler()](http://php.net/session_set_save_handler) 覆蓋 PHP 內部所使用的 API。 如果你要這麼做，除非妳透過 PHP 的核心來達到這件事情， 否則妳應該特別注意所產生的 session id 作為 cookie 傳送到使用者的時候，是否能夠長時間的當作 seesion 儲存。 其他常見的使用方式像是產生 CSRF token 的單一雜湊值，用來取代表單或網址列內容， 並且在最後透過驗證 token 檢查 email 或諸此之類的東西。

Common misconceptions and human errors
--------------------------------------

程式設計人員最常見的錯誤是由於人類對於判斷隨機性的程度非常糟糕。 特別是當妳利用某一個雜湊演算法的時候，會讓妳很難從雜湊值當中看出它是否具備唯一性跟足夠的隨機性。 為了描述這一點，底下是兩個 md5( rand(1,5) ) 所產生輸出： `eccbc87e4b5ce2fe28308fd9f2a7baf3 a87ff679a2f3e71d9181a67b7542122c` 如果妳只看表面這些數值，會覺得結果夠隨機， 當如果把它當作妳的 session id，基本上當有五個人同時使用的時候就會產生 id 衝突， 即使少於這個數目，也有可能在連續使用的時候產生兩個相同的數值。 大部分的使用者可以發現這個問題，所以不會犯下這麼明顯的錯誤。 但另一個常見的錯誤："讓我們用時間當亂數"， 除非妳的網站運行在多台高流量的伺服器上面，否則事實上時間是一個線性亂數， 而當你用 [microtime(true)](http://php.net/microtime) 時妳應該可以得到一個相當單獨的 id，並且只有很小的機率會發生衝突。 但是如果有人知道妳的伺服器大約的系統時間，這可能會變點非常不安全， 一個攻擊者可能在每個毫秒之間產生一組雜湊值，藉此挾持 session。 我不在此描述詳細的攻擊細節，因為在此我們比較在乎的是關於網站安全方面而非 seesion 挾持。 另外一個常見的方式是使用 [rand()](http://php.net/rand), [mt_rand()](http://php.net/mt_rand), 或者 [uniqid()](http://php.net/uniqid)。 前兩個隨機程度稍微好一點，但屬於弱加密( cryptographically weak )，而某些攻擊者也會利用他們，所以依賴這個方式變得相當危險。 而最後一個也是根據時間而定，所以比起 [microtime()](http://php.net/microtime) 也沒有太大的用處。

Randomness explained
--------------------

首要瞭解的部份是隨機數值是如何產生的。 電腦是一種有限狀態機，他們被製造來產生可靠而且持續的內容， 像是提供特定的輸入就能得到相同的輸出。 因此要求他們要無中生有是相當困難的。為了避免這種情形， 作業系統有許多方法可以得到真正的隨機值， 一種幾乎不太可能被猜中的數值。這些經由主機的環境使用無序狀態( 像是雜訊或者混亂情形 )所產生的數值， 透過各種輸入的總和，像是硬體裝置的存取或者風扇轉速之類的訊息。 在大部分 UNIX 主機上有兩個變數儲存在檔案當中提供妳讀取隨機來源， /dev/random 是一個加密鎖定的亂數產生器，這代表當妳從來源存取之後它會被清空， 並且進行鎖定直到再次填入新的數值之後才可以再存取。 對於高負載的伺服器而言這個用意很好，但是當它淨空的時候也相對的危險， 當新的使用者等待獲得 seesion id 時可能會需要等待速秒或者完全的逾時， 妳可不希望他們再第一次到訪的時候就發生這種事情。 另一個替代方式是 /dev/urandom，一個_非鎖定_的變數。 它會提供一些紀錄資料並且給妳即時的大量的資料。 因為他不是加密保護的，所以它可能會給妳兩個類似的輸出， 但如果妳沒辦法依賴 /dev/random 夠快得到資料這會是妳最好的方式， 而且它比使用 PHP 的 [mt_rand()](http://php.net/mt_rand) 來得好。 很抱歉的 Windows 當中並沒有相對應的檔案可以作為存取，不過可以透過 OS API 達到。 在 PHP 上面妳可以透過 [COM](http://php.net/COM) extension。雖然它也沒有經過安全加密， 而且有時候根據 windows 的情況會回傳失敗的結果，所以我不建議採用這個方式。 同樣在 PHP 5.3.0，[openssl_random_pseudo_bytes](http://php.net/openssl_random_pseudo_bytes) 提供你一個良好的計算方式， 透過 libssl 回傳偽隨機( pseudo-random )的位元組。 當你使用它檢查數值的時候請特別注意 $strong 參數， 因為當它設定為 true 時，表示演算法不會使用最佳的方式來處理當前的狀況，所以你最好自己手動處理。 上述的方式引起了我的注意，因為在它執行的時候， unix 系統上首先會將內容送入 /dev/random 然後在幾個毫秒後將資料設定為過期。 所以當你非常講求速度的時候( 舉例你要為每一個請求產生一個 id 的時候 )，你可能不會想用這個方式。 如果你執行 PHP 5.3.0 或以上版本另外一個選擇是 [mcrypt_create_iv](http://php.net/mcrypt_create_iv) 。 在之前的版本中如果是在 windows 上面執行，結果很不安全。 5.3.0 之前的版本沒有真正安全的作法可以在 windows 上取得一個無序的內容， 所以你最好升級到新的版本。 現在，透過 MCRYPT_DEV_RANDOM 這個旗標設定，妳可以取得比較好的無序內容。

Hash algorithms
---------------

取得隨機之後你需要做的事情是將資料正規劃處理，透過這個方式可以獲得兩方面的好處： 隱藏產生單一 id 所使用的方法，以及讓資料可以被加密以及儲存成為 cookie。 傳給使用者一個不夠隨機的資料可能會給攻擊者一些產生方法提示，所以不建議這麼做。 而使用包含不正確字元資料的 cookie 可能會造成表頭錯誤或者在某些 PHP 產生頁面產生警告， 使得除錯更為困難。 因為這個理由，PHP 提供 [hash](http://php.net/hash) extension，並且在 PHP 5.1.2 設定為預設啟用， 所以大部分的主機應該都有支援該功能。透過執行 [hash_algos()](http://php.net/hash_algos) 或者妳的 [phpinfo](http://php.net/phpinfo)， 妳可以看妳的版本支援哪些加密演算法。有許多演算法可以從這邊做選擇，而品質之間則有很大的差異。 某些並非設計來作為加密用途，所以他們很容易受到碰撞攻擊，有些過去可能表現的很好，不過後期已經證實可以破解了。 另外一個選擇的依據是跟他們的執行速度，比較慢的加密演算法代表了攻擊者需要使用更多、更強大的處理能量才能夠暴力破解， 所以你不會希望使用一個運算速度短又快的。 在 [hash()](http://docs.php.net/manual/en/function.hash.php) function 的說明頁上，有使用者所提供的效能評測結果。 這些結果當中，在目前這篇撰寫時 Whirlpool 是當中執行時間最長的，而它也是這當中加密品質最好的一個因此也是我所推薦的。 儘管如此，請隨時注意安全警告。加密演算法持續的遭受攻擊破解，並且不能保證從現在開始 5 年內 Whirlpool 仍然是安全的。 不過這並不是這篇文章的主旨，請注意就算是熱門的 MD5 有不建議用來當作密碼的加密使用， 因為由於他的熱門性，許多人已經為它建立了相關的 rainbow table。

The code
--------

總結以上的內容，這是提供一個可以在任何機器的基本例子，並且應用上所有可用的技術期望有良好結果。 顯然這代表這當中多了一些東西，你可能不想要每一個情況都全部用上，但我仍希望可以作為一個良好的參考。
{{< highlight php >}}
<?php
/**
 * Tries a bunch of methods to get entropy in order
 * of preference and returns as soon as it has something
 *
 * @return string
 */
function generateEntropy() {
    // use mcrypt with urandom if we're on 5.3+
    if (version_compare(PHP_VERSION, '5.3.0', '&gt;=')) {
        return mcrypt_create_iv(64, MCRYPT_DEV_URANDOM);
    }

    // otherwise try ssl (beware - it may slow down
    // your app by a few milliseconds)
    if (function_exists('openssl_random_pseudo_bytes')) {
        $entropy = openssl_random_pseudo_bytes(64, $strong);
        // skip ssl since it wasn't using the strong algo
        if ($strong) {
            return $entropy;
        }
    }

    // try to read from the unix RNG
    if (is_readable('/dev/urandom') &amp;&amp; ($handle = fopen('/dev/urandom', 'rb'))) {
        $entropy = fread($handle, 64);
        fclose($handle);
        return $entropy;
    }

    // Warning !
    // from here on, the entropy is considered weak
    // so you may want to consider just throwing
    // an exception to realize that your code is running
    // in an insecure way

    // try to read from the windows RNG
    if (class_exists('COM')) {
        try {
            $com = new COM('CAPICOM.Utilities.1');
            $entropy = base64_decode($com-&gt;GetRandom(64, 0));
            return $entropy;
        } catch (Exception $ex) {
        }
    }

    // last solution.. barely better than nothing
    return uniqid(mt_rand(), true);
}

/**
 * Grabs entropy and hashes it to normalize the output
 *
 * @param string $algo hash algorithm to use, defaults to whirlpool
 * @return string
 */
function getUniqueHash($algo = 'whirlpool') {
    $entropy = generateEntropy();
    return hash($algo, $entropy);
}

echo getUniqueHash(); // 0532448b807325ae01661d46d7a534c852b6fc00ac14964b69bd0bb6f37f6c7c69ab4435c36dc57190c2a1cb4bd209a9f6a09e52c1ca25ef9ca20f04f34e643c
echo generateEntropy(); // wçGœè’‰Ù&lt;Vxîiíè y lTS7SŠ‚ìÊ‡‡èäìûòn5ïO‚¼ˆ¦¢P„Íg¤'_k�yXE‘ëg
{{< /highlight >}}
 這個程式會試著透過最好的方式取得適當的無序內容，並且盡快的回傳某些訊息。 最後預設會使用 Whirlpool 演算法回傳他的雜湊值。 Whirlpool 會回傳相當長的雜湊值，因為他有 128 的字元長， 所以如果你打算作為 URL 的 token 可能太長了一點。 你可能會想根據需求改用其他的演算法，像是 sha256( 64的字元 )或者 md5( 32字元 )， 如果你真的不需要衝突攻擊保護，作為這樣的用途已經足夠了， 另外 md5 真的不應該用來作為密碼的雜湊加密，而這就是另一個複雜的話題了。
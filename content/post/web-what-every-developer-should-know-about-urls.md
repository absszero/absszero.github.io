---
title: '[ Web ] What Every Developer Should Know About URLs'
date: Sun, 13 Jun 2010 09:48:42 +0000
draft: false
tags: [IT 筆記]
---

What Every Developer Should Know About URLs http://www.skorks.com/2010/05/what-every-developer-should-know-about-urls/ 我最近寫了一篇關於[軟體開發基礎的價值](http://www.skorks.com/2010/04/on-the-value-of-fundamentals-in-software-development/)。 而且我相信妳也需要穩固妳的基礎，如果妳想成為一個好的開發人員的話。 許多人針對該篇文章發表了相當有益的看法，事實上要了解何謂基礎是有點困難的( 想在更深入進行了解的話 )。 所以我覺得發表一系列相關的文章是一個不錯的方法，而這篇則是一系列的第一節。 **在現在要成為一名開發人員，幾乎無可避免去做一些網路相關的工作**。 這代表妳一定有機會處理 URL。我們都知道 URL 是什麼，但是使用者所知道跟開發人員應該知道還是有所區別。 身為一名網站開發人員，妳真的沒有理由不知道 URL，其實它本身沒有太多東西。 但是我發現即使是富有經驗的開發人員還是對 URL 的知識有所缺漏。 所以我想我會針對每一個開發人員所需的 URL 做一個全面的導覽。 繫好妳的安全帶 – 這不會花太多時間的 :)。

The Structure Of A URL
----------------------

這還不簡單，就是以 HTTP 開頭然後以 .com 結果，對吧？ 大部分的 URL 都有一套通用的語法，根據下列九個部份構成： `<scheme>://<username>:<password>@<host>:<port>/<path>;<parameters>?<query>#<fragment>` 大部分的 URL 不會包含全部的語法。最常用到的部份，不用懷疑肯定是：scheme、host 和 path 。 依序看一下每一個部份：

*   **scheme** - 指定 URL 所要存取的資源協定( _像是 http , ftp_ )。[這裡有相當多的不同的 scheme](http://en.wikipedia.org/wiki/URI_scheme)。 如果某一個 scheme 是經由官方發布的，它會在 [IANA](http://en.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority) 登記註冊( _像是 http , ftp_ )， 但有許多非官方的 scheme( _未註冊的_ )依然被普遍使用( _像是 sftp 或者 svn_ )。 scheme 必須以英文字母開頭，而透過第一個 : (_冒號_ )跟剩餘的 URL 區隔。 沒錯，// 並不是分隔符號的一部分，但確實作為下半部份 URL 的開端。
*   **username** - 它與 password 連用，表示 URL 所使用的 host 跟 port 根據什麼樣的權限認證。 某些 scheme 需要認證資訊用以存取某個資源，這部份是使用者名稱的認證資訊。 username 跟 password 在 ftp 形式的 URL 上很普遍，而在 http 的 URL 就比較少見，不過妳還是很常用到。
*   **password** - URL 認證資訊的另一部份，它跟 username 之間以另外一個 : ( _冒號_ ) 字元區隔。 username 跟 password 會透過 @(_at_) 跟 host 做區隔。妳可以只提供 username 或者同時提供 username 跟 password。 舉例： `ftp://some_user@blah.com/ ftp://some_user:some_path@blah.com/` 如果妳沒有提供 username 跟 password，妳所使用的程式( _例如瀏覽器_ )會自動套用預設內容存取妳想要存取的 URL。
*   **host** - 就像我剛剛提到的，它是組成認證功能的一部分。
*   host 可以為網域名稱或者 IP，而我們應該知道網域名稱會被轉換成 IP 位址( _透過 DNS 查詢_ )， 然後找出我們所要存取的主機。
*   **port** - 認證功能的最後一個部份。它基本上告訴我們特定服務在該主機上所監聽的網路埠號為何。 我們都知道 HTTP 預設使用埠號 80，如果一個 http 的 URL 省略了 port 的部份，它就會假設是 80。
*   **path** - 在 URL 上它會透過一個 / (_斜線_)字元區隔。path 是一連串用 / 切開的區段。 path 基本上告訴我們資源存在於伺服器哪裡。 每一個 path 的區段可以包含使用 ;(_分號_)字元分隔的參數，舉例： `http://www.blah.com/some;param1=foo/crazy;param2=bar/path.html`上面是完全有效的 URL，雖然這種包含參數的區段幾乎沒看過(_我個人是從來沒見過_)。
*   **parameters** - 談談 parameters，它可以出現在 path 之後而在 query 字串之前， 一樣是透過 ;

字元分割，舉例： `http://www.blah.com/some/crazy/path.html;param1=foo;param2=bar`就像我說得，這個不常看到。
*   **query** - 這個部份網站開發人員都知道它非常常見。 這是大部分人偏愛方式，用以傳遞參數到伺服器。 它以 _**key=value**_ 鍵值對的方式，並且把剩下 URL 用一個 ?(_問號_)字元分隔開來。 而它通常會將每一個 query 用 & (_and 符號_)字元分隔。 妳可能不知道事實上用 ;(_分號_)字元分隔也一樣有效。下列的 URL 是完全相同的意思： `http://www.blah.com/some/crazy/path.html?param1=foo&param2=bar http://www.blah.com/some/crazy/path.html?param1=foo;param2=bar`
*   **fragment** - 這是 URL 的額外的選項，用於指出一個資源的特定部份。 我們通常看到它用在 html 文件當中連結到某一個特定的章節。在 URL 中 fragment 根據一個 # (_井號_)字元分隔開來。 當我們透過 URL 從伺服器請求一個資源的時候， 用戶端( _也就是瀏覽器_ )通常不會把 fragment 傳送到伺服器( _至少在 HTTP 不會_ )。 當用戶端取得資源後，它才會使用 fragment 指向相關的部份。

以上是所有妳需要知道的 URL 結構部份。 現在當妳使用 fragment 時妳不應該有任何藉口不知道 - "井號連結這個東西會導引到該 html檔案的特定部份"。

Special Characters In URLs
--------------------------

就像 URL 應該怎麼正確的編碼一樣，關於哪些字元可以使用在 URL 而哪些不行有很多的疑問。 開發人員通常會根據常識來斷定( _也就是像 / 還有 : 字元很明顯的應該被編碼，因為他們在 URL 裡頭有特殊的意義_ )。 其實不需這樣，妳應該很清楚的了解這個東西 - 它很簡單，真相就在這裡。 某些字元出現在 URL 的時候妳需要特別注意， 首先這種字元在 URL 當中有特別的意義，也就是所謂的 _**保留字元**_，他們是： `";" | "/" | "?" | ":" | "@" | "&" | "=" | "+" | "$" | ","` 這代表在些字元通常在 URL 當中使用有特別的意思( _就像是把每一個部份分隔開來_ )。 如果某一部份的 URL( _像是 query 的參數_ )可能包含這些字元，那它應該在被放到該 URL 之前先進行脫逸的動作。 我在之前有談過 [URL 編碼](http://www.skorks.com/2009/08/different-types-of-encoding-schemes-a-primer/)的部份，看一下，我們會短暫的回顧這部份。 第二種得注意的字元是所謂的_非保留字元_。它根據下列的字元所組成： `"-" | "_" | "." | "!" | "~" | "*" | "'" | "(" | ")"` 這些字元可以直接放在 URL 的任何地方( _注意到 URL 的某些部份可能不允許出現該字元_ )。 這代表當妳要把它當作 URL 的一部分的時候，不需要經過編碼或脫逸這些字元。 **妳依然可以對他們進行脫逸而不會改變他們在 URL 原本所代表的意思，但不建議這個做。** 第三種值得注意的是所謂 _**'避用字元'**_，也就是不建議在 URL 當中使用的字元， 它包括下列這些字元： ``"{" | "}" | "|" | "\" | "^" | "[" | "]" | "`"`` 這些字元會被列為避免使用的原因是因為，閘道器有時會變動這些字元，或這把他們當作分隔符號。 但不代表閘道器一定為變動這些字元，不過可能會發生。 所以如果妳在 URL 使用這些字元而且沒有進行脫逸的話，可能會有一定的風險。 我這裡要說的是**如果妳的 URL 可能包含這些字元( _像是 query 的參數_ )，**請務必脫逸這些字元。 最後一種字元是例外字集。 它由所有的 ASCII 控制字元組成，包含空白字元以及底下的字元( 稱為分隔符號 )： `"<" | ">" | "#" | "%" | '"'` 控制字元是無法顯示的 US-ASCII 字元( _就像是十六進制的 00-1F 以及 7F_ )。 就像 # 還有 % 這些在 URL 當中有特殊意義的字元， 這些字元如果包含在 URL 當中一定要經過脫逸( _妳可以把他們當成保留字元_ )。 其他在這個字集的字元因為無法被顯示出來，所以脫逸是唯一可以表示他們的方式。 **<, > 還有 " 因為常常在內容當中被用來區隔 URL，所以這些也需要脫逸**。 要編碼或脫逸一個字元，我們通常將它的 ASCII 十六進制值加上一個 % 字元。 所以一個空白字元的 URL 編碼就是 _%20_ - 我們都見過這個。 ％ 字元他自己編碼的結果則是 _%25_。 以上就是妳該知道的關於各種特殊字元在 URL 的情形。 當然除了這些字元之外，英文跟數字可以直接使用而不需要經過編碼 :)。 有一些事情妳應該記住。URL 應該始終保持它的編碼形式。 只有當妳要開啟 URL 的時候才對它做解碼的動作。 URL 的每一個部份必須獨立編碼，這樣做的目的很明顯，妳不會希望編碼一個已經組合好的 URL， 因為這樣就沒辦法辨別哪些地方是保留字元( _這些地方不該被編碼_ )，而哪些地方不是( _這些地方應該被編碼_ )。 最後妳應該避免重複編碼或解碼一個 URL。 試想妳將某個 URL 做了一次編碼，但是進行了兩次解碼， 而這個 URL 的構成裡頭包含了 % 這個字元，如此妳會毀了這個 URL，舉例： `http://blah.com/yadda.html?param1=abc%613` 當它進行編碼之後看起來會像是： `http://blah.com/yadda.html?param1=abc%25613` 如果妳重複解碼之後妳會得到： 第一次解碼`：http://blah.com/yadda.html?param1=abc%613` **正確的結果** 第二次解碼`：http://blah.com/yadda.html?param1=abca3` **多了一些東西** 順道一提，這些東西不是我自己漫天胡扯的，他們全都在 [RFC 2396](http://www.ietf.org/rfc/rfc2396.txt) 裡面清楚的定義， 如果妳希望的可以點過去看一下，不過它看起可能不是那麼有趣，而我的文章至少看起來不會那麼無聊。

Absolute vs Relative URLs
-------------------------

最後一件開發人員應該了解的事情是絕對與相對網址的差異，以及如何將相對網址轉會成絕對網址。 第一部份比較簡單，如果一個 URL 包含 scheme 的部份( _像是 http_ )， 那它就可以被視為絕對網址。但相對網址就有點複雜了。 相對網址永遠代表他是相對於另外一個網址( _就像他的名字 :)_ )， **其他的 URL 被稱為 base URL**。要轉換相對網址變成絕對網址的形式， 我們首先必須瞭解他的 base URL，接著把它跟相對網址的語法結合之後就得到絕對網址。 我們通常可以在 html 文件當中看到相對網址的存在。 在這種情況有兩個方法可以找到他的 base URL 所在。

1.  base URL 可能經由 HTML 的 **<base>** 標籤指出。
2.  如果沒有指定 base 標籤，找到該 html 文件的 URL 位置就會被視為他的 base URL

當我們有了 base URL，我們可以試著將我們的相對網址轉成絕對網址。 首先我們要將相對網址拆解成個別的結構( _也就是 scheme,authority(__host,post_),path,query,string,fragment )。 當我們拆解完成後，有幾個地方需要特別注意，他們可能都代表了我們的相對網址並不是完全的相對。

*   如果沒有 scheme、authority 或者 path，那麼這個相對網址則參考他的 base URL
*   如果他有 scheme，那麼這個相對網址事實上是絕對網址，所以用絕對網址的方式處理
*   如果沒有 scheme，但是有 authority(_host,post_)，那我們相對網址可能是網路位址， 那麼我們根據 base URL 的 scheme 並且把它跟相對網址用 :// 結合起來

**如果沒有上述的這些特殊情形，那我們得到的是一個完全的相對網址**。 現在我們需要針對下列的程序處理。

*   我們從 base URL 繼承他的 scheme 以及 authority(_host,post_)
*   如果我們的相對網址開頭是一個 /，那麼它就是一個絕對路徑， 我們從 base 繼承他的 scheme 以及 authority 並且用適當地分隔符號產生我們的絕對網址
*   如果相對網址的開頭不是一個 /，那我們會從 base URL 開始處理，把 base URL 中 / 以後的東西都移除
*   然後把我們相對網址加上去得到最後的路徑，現在我們根據前面的幾個字元對相對網址做一點小處理
*   如果最後的路徑包含 ./( _逗點加上斜線_ )，我們直接把它移除掉( _這代表我們的相對網址由 ./ 開始，就像 ./blah.html_ )
*   如果最後的路徑包含 ../( _逗點、逗點、斜線_ )，我們會移除它並且把路徑往上一個區段， 就好像把所有像這樣的路徑組合都移除掉 "**<segment>**/../"， 持續反覆這個動作，直到找不到任何的 ../ ( _這表示我們的相對路徑開頭是多個 ../ 像是 ../blah.html 或者 ../../blah.html_ )
*   如果路徑的結尾是 ..，那就移除它並且把路徑往上一個區段， 像是移除 "**<segment>**/../"( _這表示我們的相對路徑是 ..( 逗點、逗點 )_ )
*   如果路徑的結尾是 . ( 一個逗點 )，那就把它移除( _這表示我們的相對路徑很有可能是 .( 一個逗點 )_ )

根據這些規則我們簡單的把相對網址的 query 或者 fragment ， 利用合適的分隔符號包含到我們的 URL 當中，如此就將相對網址轉換成絕對網址。 這裡列出一些根據上述規則的使用範例：

1. base: http://www.blah.com/yadda1/yadda2/yadda3?param1=foo#bar relative: rel1 final absolute: http://www.blah.com/yadda1/yadda2/rel1
2. base: http://www.blah.com/yadda1/yadda2/yadda3?param1=foo#bar relative: /rel1 final absolute: http://www.blah.com/rel1
3. base: http://www.blah.com/yadda1/yadda2/yadda3?param1=foo#bar relative: ../rel1 final absolute: http://www.blah.com/yadda1/rel1
4. base: http://www.blah.com/yadda1/yadda2/yadda3?param1=foo#bar relative: ./rel1?param2=baz#bar2 final absolute: http://www.blah.com/yadda1/yadda2/rel1?param2=baz#bar2
5. base: http://www.blah.com/yadda1/yadda2/yadda3?param1=foo#bar relative: .. final absolute: http://www.blah.com/yadda1/

現在妳應該能夠把相對網址轉換成絕對網址了，並且知道各種不同形式的相對網址會造成什麼樣的影響。 對我來說這已經在我的網站開發上應用了，而且還持續的使用中。 不論妳在什麼時候都需要知道關於 URL 的知識，這些都相當簡單，所以下次不要有藉口說妳不太清楚這些東西。 下一次我們要談的東西，是關於如果 URL 是包含在本文裡頭的情形。 所以下次我會告訴妳如何[透過正規表示式使用他們](http://feeds.feedburner.com/softwaretechandmore)( _以及如何從本文中把 URL 拉出來_ )。 而當我們了解了他的結構跟特殊字元之後，產生一個合適的正規表示式應該就容易了。敬請期待。 ![](//dictionarytip/skin/dtipIconHover.png)
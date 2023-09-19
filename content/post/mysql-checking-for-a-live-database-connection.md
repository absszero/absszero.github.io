---
title: '[ MySQL ] Checking for a live database connection considered harmful'
date: Sat, 19 Jun 2010 07:39:37 +0000
draft: false
tags: [IT 筆記, MySQL]
---

Checking for a live database connection considered harmful http://www.mysqlperformanceblog.com/2010/05/05/checking-for-a-live-database-connection-considered-harmful/ 對我來說觀察客戶的資料庫是很稀鬆平常的事情， 而我注意到很多資源耗用，浪費在傳送查詢之前的資料庫連線狀態檢查。 底下是根據設計模式所撰寫的虛擬碼： {{< highlight sql >}} function query_database(connection, sql) if !connection.is_alive() and !connection.reconnect() then throw exception end return connection.execute(sql) end</pre>
{{< /highlight >}}
 許多熱門的開發平台都做了類似的檢查。但在程式當中有兩件事情並不正確： 1) 他的結果並不正確 2) 他會產生大量的效能耗用

It Does Not Work
----------------

程式無法正確運作的原因是因為競態條件( race condition )， 如果一個連線在檢查的時候是有效的，這不保證在執行 connection.execute(sql) 他仍然是有效的。 而如果他不是有效的，並且執行了重新連線。這仍舊不能保證連線有效。 檢查後才進行查詢這種方式是沒有用的。 取而代之的，程式碼應該被改寫成下列形式： {{< highlight sql >}} function query_database(connection, sql, retries=1) while true try result=connection.execute(sql) return result catch InactiveConnectionException e if retries&gt;

0 then retries = retries - 1 connection.reconnect() else throw e end end end end
{{< /highlight >}}
 注意到我已經把 is_active() 的呼叫完全移除了。 如果這個連線是有效的，查詢結果成功； 如果不是，顯示失敗並且重新連線，然後再試一次。 這個程式碼提供一個良好的參數，如果有這個需求可以根據等待時間或無法連線的情況控制嘗試的次數。 不過根據我的經驗，在許多應用程式上這個功能用處可能不大。 大部分的應用程式應該能根據情形進行重新連線，但實際情況這方面並沒有處理的很好。

Performance Overhead
--------------------

檢查連線是否有效的典型作法包括透過 MyQL 協定階層呼叫 'ping' 或者 'statistic' 指令， 但這樣會在 SHOW GLOBAL STATUS 的時候增加 Com_admin_commands 的次數。 或進行一些像是 'SELECT 1' 這類沒有任何作用的查詢，但這些都很難獲得正確的連線狀態。 這些方式對應用程式而所需的執行成本過高。 主要的花費成本在於兩個地方：網路應用程式的往返時間增加了查詢時間，以及造成資料庫系統的額外負擔。 增加在資料庫系統的成本可能非常高。 前些日子我看到一個 Ruby on Rails 的程式，使用 'Administrator command: statics' 指令， 結果使用了超過 40% 的伺服器查詢時間。 避免不必要的連線查詢可以節省將近一半的資料庫系統負載。這可是一點都不誇張！ 當應用程式不斷進行查詢的時候，額外查詢所造成的損耗常常被忽略。 但是在高流量的應用程式上，卻會大大影響查詢時間， 而一些高效能的應用程式甚至會擔心他們查詢時間會長於一個毫秒或更久。 當你每秒對資料庫系統執行 2 萬個查詢，額外 2 萬個檢查連線是否有效就會造成很大的影響。 而 'statistics' 或者 'ping' 這類的查詢事實上對於應用程式查詢的代價則更高。 而這些只是資料庫系統的負載。在應用程式方面，你基本上增使用了用了兩倍查詢次數。 每次你要進行一個查詢，你的應用程式發出一個檢查連線的網路請求到資料庫並返回結果， 接著另外一個網路請求針對要執行的查詢。如此產生了更多的問題。 這個問題是根據上述不良的虛擬碼，在最常見的狀態下遭受不常見狀態的連累。 連線在大部分的時候都是有效的，你不需要透過 ping 或者重新連線確認。 一個好的方式是使用相同的程式並且修正競態條件的問題。 再來，當某一個連線失效的時候，你可以在嘗試進行查詢之後發現這個問題。 在那之前，假設每一件事情都正確無誤，所以盡管執行查詢就對了。 我希望那些提供程式庫的上游維護廠商可以發現這個問題並且修正它， 因為這對於應用程式逐漸發展起來會有很大的幫助。 這也是我們目前實驗室運作的很好的地方，而即使在這個領域， 除非效能真的出現了問題，大家才會真正的發現它的存在。 這裡是另外一個範例描述，這些可笑的查詢所造成的衝擊：

| Rank | Rank Query ID | Response time | Calls | R/Call | Item |
|------|---------------|---------------|-------|--------|------|
|1 | 0x5E796D5A4A7D1CA9 | 10651.0708 | 73.1% | 120487 | 0.0884 | ADMIN STATISTICS |
|2 | 0x85FFF5AA78E5FF6A |  1090.0772 |  7.5% |  23621 | 0.0461 | BEGIN |
|3 | 0x6E85B9A9C9FF813E |   868.0335 |  6.0% |   6923 | 0.1254 | UPDATE scores |
|4 | 0xA3A0423749EC0E37 |   851.0152 |  5.8% |   6020 | 0.1414 | UPDATE user_datas |
|5 | 0x813031B8BBC3B329 |   822.0041 |  5.6% |  23299 | 0.0353 | COMMIT |
|6 | 0xA873BBC4583C4C85 |   278.4533 |  1.9% |   6985 | 0.0399 | SELECT users user_devices |


沒錯，伺服器 73% 的負載被消耗在檢查連線是否存在。
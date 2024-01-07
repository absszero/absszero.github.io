---
title: 'Monitoring JBoss logfile with Chainsaw v2'
date: Sun, 15 Apr 2012 02:54:42 +0800
draft: false
tags: [apache chainsaw jboss log4j, IT 筆記]
---

Chainsaw 可以用來監控 JBoss 所使用的 Log4j，透過以下的設定，讓開啟 Chainsaw 的時候自動監聽。

設定 JBoss
--------

首先找到 {{< highlight bash >}} {JBOSS_HOME}server{YOUR_SERVER_FOLDER}confjboss-log4j.xml
{{< /highlight >}}
 開啟之後在 {{< highlight xml >}} <log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/" debug="false"> ... </log4j:configuration>
{{< /highlight >}}
 之間增加一個 appender {{< highlight xml >}} <appender name="SOCKET" class="org.apache.log4j.net.SocketHubAppender"> <param name="Port" value="8888"/> </appender>
{{< /highlight >}}
 最後底下的 {{< highlight xml >}} <root> <appender-ref ref="CONSOLE"/> <appender-ref ref="FILE"/> </root>
{{< /highlight >}}
 變更為 {{< highlight xml >}} <root> <appender-ref ref="CONSOLE"/> <appender-ref ref="FILE"/> <strong><appender-ref ref="SOCKET"/></strong> </root>
{{< /highlight >}}
 儲存之後重新啟動 JBoss。 接著透過 netstat -na ，你可以觀察到 TCP 0.0.0.0:8888 已經處於 LISTENING 的狀態。

設定 Chainsaw v2
--------------

假設 Chainsaw 目錄位置是 D:Downloadschainsaw-bundle，在所在目錄新增一個檔案 chainsaw-receiver.xml。 內容如下： {{< highlight xml >}} <?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE log4j:configuration > <log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/" debug="true"> <plugin name="JBoss" class="org.apache.log4j.net.SocketHubReceiver"> <param name="Port" value="8888"/> <param name="Host" value="127.0.0.1"/> </plugin> </log4j:configuration>
{{< /highlight >}}
 然後執行 Chainsaw，透過選單 **View** -> **Show Application-wide Preferences** [![](https://absszero.files.wordpress.com/2012/04/2012-04-15_103116.png "Chainsaw View-show application wide preferences menu")](https://absszero.files.wordpress.com/2012/04/2012-04-15_103116.png)

 接著在 **Automatic Configuration URL** 的部份輸入： {{< highlight bash >}} file:///d:/Downloads/chainsaw-bundle/chainsaw-receiver.xml
{{< /highlight >}}
 按下 OK 之後重新啟動 Chainsaw，視窗右列的部份就會自動使用設定的 Receiver 監聽 JBoss 的 log4j [![](https://absszero.files.wordpress.com/2012/04/2012-04-15_104455.png "Chainsaw receiver list")](https://absszero.files.wordpress.com/2012/04/2012-04-15_104455.png)

參考
--

[ChainsawAndSocketHubWalkthrough](http://wiki.apache.org/logging-log4j/ChainsawAndSocketHubWalkthrough)
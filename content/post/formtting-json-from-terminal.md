---
title: '在命令提示字元底下格式化顯示 JSON 內容'
date: Sat, 12 May 2012 10:56:17 +0000
draft: false
tags: [CMD, 縮排, IT 筆記, JSON, python, 命令提示字元, 格式化]
---

當我們需要在命令提示字元下顯示 JSON 字串的時候，通常得到的是一串沒有縮排的內文。 不過利用 **python -mjson.tool** 的協助，可以將內容格式化後顯示，看起來漂亮多了！

安裝 Python
---------

建立 json.cmd
-----------

為了能夠快速的呼叫 **python -mjson.tool **而不用輸入這一長串指令，開啟記事本，並且輸入 {{< highlight bash >}} @ECHO OFF python -mjson.tool
{{< /highlight >}}
 然後輸入檔名 **C:windowsjson.cmd**，並儲存。

呼叫 json.cmd
-----------

透過底下的方式呼叫 json.cmd {{< highlight bash >}} command | json.cmd
{{< /highlight >}}
 例如：curl localhost:9200/_cluster/nodes/_local | json.cmd 得到的結果，就會是漂亮 JSON 格式化後內容輸出。 {{< highlight javascript >}} { "cluster_name": "elasticsearch", "nodes": { "KFQG-7ADT5WqJnju-e4RzA": { "hostname": "GFive", "http_address": "inet\[/192.168.1.110:9200\]", "name": "Captain Britain", "transport_address": "inet\[/192.168.1.110:9300\]" } }, "ok": true }
{{< /highlight >}}


### 參考

[Formatting JSON From Terminal](http://benscheirman.com/2011/12/formatting-json-from-terminal "Formatting JSON From Terminal")
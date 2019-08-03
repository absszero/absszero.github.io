---
title: '[ Web ] Entities in modular XHTML'
date: Tue, 02 Nov 2010 13:04:41 +0000
draft: false
tags: [IT 筆記, UTF-8, XHTML, XML]
---

Entities in modular XHTML http://cweiske.de/tagebuch/xhtml-entities.htm 要在 HTML 裡頭應用某些特殊字元，可以把該字元寫成 "named entities"。 最常用的就是空白字元   但是當我們要用在未指定 DTD 的 XML 上面。   會被視為一個語法錯誤。 關於該問題有幾個解決辦法： 使用數字字元參照 {{< highlight html >}} <title>Hello & #160;</title>
{{< /highlight >}}
 針對特定字元進行定義 {{< highlight html >}} <?xml version="1.0" encoding="utf-8"?> <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd" \[ <!ENTITY nbsp "& #160;"> \]> <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
{{< /highlight >}}
 引入定義的所在網址
{{< highlight php >}}
<?xml version="1.0" encoding="utf-8"?> <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd" \[ <!ENTITY % xhtml-symbol PUBLIC "-//W3C//ENTITIES Symbols for XHTML//EN" "http://www.w3.org/MarkUp/DTD/xhtml-symbol.ent"> %xhtml-symbol;

\]> <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
{{< /highlight >}}
 由於我們現在的頁面多半使用 UTF-8 編碼，所以其實可以捨棄使用上述的方式， 而直接使用 UTF-8 編碼特定字元以避免上述問題。
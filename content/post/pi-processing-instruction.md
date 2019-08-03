---
title: '何謂PI(Processing Instruction)'
date: Sun, 07 Dec 2014 03:07:32 +0000
draft: false
tags: [IT 筆記, XML]
---

PI的英文全名是Processing Instruction，是【處理指令】的意思，在XML文件中PI是以『<?』為指令的開始符號，並以『?>』為指令的結束符號，與一般標籤的開始符號『<』、結束符號『<』不同，這就是要讓XML Parser易於分辨一般標籤與PI的不同，而這些PI是用來要求XML Parser執行一些特別的工作。像說XML文件的宣告指令，就必須位於XML文件的第一列，該列宣告中xml就是一個PI，如下：

 {{< highlight php >}}

<?xml version="1.0"?>
{{< /highlight >}}
 另外像在XML文件中引用排版樣本，也必須透過PI來告知XML Parser，底下是引用CSS排版樣本的指令設定，其中xml-stylesheet就是一個PI。<?xml-stylesheet href=URL type="text/css" ?>由上述兩個例子知道在PI中，還允許設定屬性。**停、看、聽**XML文件中的所有指令，在<?與指令名稱之間是不允許有空白格的，如果有空白格那就沒有滿足Well-Formed XML條件。 轉至 [XML問與答](http://cloudnfu.appspot.com/Lab/xml/faq/wellform/xmlqa07.htm)
---
title: '[ Web ] Accelerate Page Loading: Unusual Site Speedup Techniques: Part 1 - PHP Classes blog'
date: Mon, 15 Nov 2010 12:20:21 +0000
draft: false
tags: [IT 筆記, JavaScript]
---

http://www.phpclasses.org/blog/post/131-Accelerate-Page-Loading-Unusual-Site-Speedup-Techniques-Part-1.html

Introduction
------------

頁面上某些由其他網站提供的資訊，像是廣告、串連內容，可能會拖慢自身網站的速度。 本篇主要說明如何改善該問題。

Why remote content slows the load of Web pages?
-----------------------------------------------

載入外部內容主要透過兩個方式，一個是 inline HTML；另外一個是透過 JavaScript 動態產生 HTML。 主要會造成緩慢的問題是採用 JavaScript 動態產生 HTML 的內容。 因為大部分的瀏覽器必須等待 JavaScript 從遠端主機載入內容之後，才會將畫面內容呈現出來。

The solution: Delay All Remote JavaScript
-----------------------------------------

大部分網頁開發人員會知道廣告部份的尺寸以及他的正確位置。 如果我們先在那塊位置以 <div> 取代，然後再使用頁面的 onload 屬性替換該塊 div 的 innerHTML 屬性呢？ 但這個方法有一個潛在問題是許多廣告會使用 document.write 語法把 HTML 寫入。 如果使用 onload 載入，新寫入的 HTML 會出現在本文的最後頭。 要解決該問題的方式是，將外在寫入的內容包含在一個隱藏的 div 裡頭，然後用 DOM node 的方式移動該內容到正確的位置。

The Fast Page Content Loader object
-----------------------------------

為了更容易使用上述的方法，作者設計一了稱為 [Fast Page Content Loade](http://www.jsclasses.org/fast-content-loader) 的 JavaScript 物件。 並且針對該程式寫了一篇簡單的教學說明[Fast Page Loading Tutorial - Fast Page Content Loader package blog](http://www.jsclasses.org/blog/package/24/post/1-Fast-Page-Loading-Tutorial.html)

Browser issues
--------------

Fast Page Content Loader 目前在 Internet Explorer 跟 Opera 運作上有問題。 所以作者取消了這兩個瀏覽器的支援，這兩個瀏覽器使用該程式的時候，會直接在該位置顯示內容，而不具延後呈現的效果。
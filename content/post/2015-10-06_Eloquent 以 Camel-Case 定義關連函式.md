---
title: 'Eloquent 以 Camel-Case 定義關連函式'
date: Wed, 07 Oct 2015 04:06:05 +0800
draft: false
tags: [Eloquent, IT 筆記, laravel, PHP]
---

假設定義了一個書籍的 Model 並且每一本書有一個書皮封面 BookCover，所以定義以下的內容：
{{< highlight php >}}
<?php
class Book extends Eloquent {
    public function book_cover() {
        return $this->hasOne('bookCover');
    }
}
{{< /highlight >}}
 現在我們查詢書籍並且使用 Eager Loading 取得封面
{{< highlight php >}}
<?php
$book = Book::with('book_cover')->first();

$book_cover = $book->book_cover;
{{< /highlight >}}
 看起來用運作的很好，現在不使用 Eager Loading，然後試著查找某一筆書籍的書皮封面
{{< highlight php >}}
<?php
$book = Book::find(1);

$book_cover = $book->book_cover;

// $book_cover is null
{{< /highlight >}}
 結果顯示書皮封面 null，怎麼回事呢？ 原來 Laravel 內部實作取得關連的部份，需要使用 Camel-Case 來定義關連函式，所以：
{{< highlight php >}}
<?php
public function book_cover()
{{< /highlight >}}
 應該被定義成：
{{< highlight php >}}
<?php
public function bookCover()
{{< /highlight >}}
 Over~
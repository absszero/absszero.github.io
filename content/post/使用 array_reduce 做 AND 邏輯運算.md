---
title: '使用 array_reduce 做 AND 邏輯運算'
date: Tue, 12 Apr 2016 02:23:18 +0000
draft: false
tags: [IT 筆記]
---

在官方手冊的頁面上有提供做 AND 運算的範例
{{< highlight php >}}
<?php
function andFunc($a, $b) {
​	return $a && $b;
}

$foo = array(true, true, true);

var_dump(array_reduce($foo, "andFunc"));
{{< /highlight >}}
 而既然使用了 array_reduce，可以用來做更複雜的 AND 運用。例如要求一篇文章，同時存在多個關鍵字。
{{< highlight php >}}
<?php
$article = '......';

$co_exist = function($carry, $keyword) use ($article) {
​	if (false === strpos($article, $keyword)) {
​		$carry = false;
​	} else {
​		$carry = $carry && true;
​	}

​	return $carry;
};

$result = array_reduce($keywords, $co_exist, true);
{{< /highlight >}}

---
title: 'Baum, 在 Laravel 存取  Nested Set Model'
date: Thu, 25 Feb 2016 07:21:04 +0000
draft: false
tags: [IT 筆記]
---

過去曾經在 Laravel 上面實作 Nested Set Model 的應用，除了基本的建立、刪除、搜尋以及搬移節點，再來就是要驗證節點跟確保巢狀結構正確，必要時還要能夠重建整個結構。而上述這些功能， Baum（https://github.com/etrepat/baum）都具備了！底下記錄一下覺得蠻實用的功能：

### 建立節點


{{< highlight php >}}
<?php
$root = Category::create(['name' => 'Root category']);

//附加子節點 $child1 = $root->children()->create(['name' => 'Child 1']);

//從 array 建立多個節點 $categories = [ ['id' => 1, 'name' => 'TV & Home Theather'], ['id' => 2, 'name' => 'Tablets & E-Readers'], ['id' => 3, 'name' => 'Computers', 'children' => [ ['id' => 4, 'name' => 'Laptops', 'children' => [ ['id' => 5, 'name' => 'PC Laptops'], ['id' => 6, 'name' => 'Macbooks (Air/Pro)'] ]], ['id' => 7, 'name' => 'Desktops', 'children' => [ // These will be created ['name' => 'Towers Only'], ['name' => 'Desktop Packages'], ['name' => 'All-in-One Computers'], ['name' => 'Gaming Desktops'] ]] // This one, as it's not present, will be deleted // ['id' => 8, 'name' => 'Monitors'], ]], ['id' => 9, 'name' => 'Cell Phones'] ];

Category::buildTree($categories);

// => true
{{< /highlight >}}


### Scope

允許單一資料表裡頭包還多組巢狀結構，只要在 model 裡面定義
{{< highlight php >}}
<?php
class Category extends Baum\Node { ... protected $scoped = array('company_id');

... }
{{< /highlight >}}
 接著在查詢上層或下層節點的同時， scoped 定義的欄位就會被自動用於查詢。

### Without

有幾個函式可以把查詢結果進一步過濾：

*   withoutNode(node): 排除特定節點
*   withoutSelf(): 排除自身節點
*   withoutRoot(): 排除根節點

### 限制查詢階層數

蠻多時候需要取得子代節點，但不到包含孫代
{{< highlight php >}}
<?php
$node->descendants()->limitDepth(5)->get();
{{< /highlight >}}

---
title: '檢測大陸身份證字號 for PHP'
date: Mon, 23 Apr 2012 06:02:44 +0000
draft: false
tags: [IT 筆記, php, 身份證字號]
---

PHP 版本的檢測大陸身份證字號 {{< highlight php >}} 
<?php
$id = trim($id);

//長度檢查
$string_length = strlen($id);
if ( $string_length != 18) {
    return false;
}

//數字檢查
//前17碼必須為數字
for ($i = 0; $i &lt; $string_length - 1; $i++) {
    if ( ! is_numeric($id[$i]) ) {
        return false;
    }
}

//生日碼檢核
$date = substr( $id , 6 ,8 );
$date = strtotime( $date );
if ( ! $date ) return false;

//取得校驗碼
$factors = array(7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2); ////加權因子
$nValue = 0;
for ($i = 0; $i &lt; 17; $i++) {
    $nValue = $nValue + ($factors[$i] * $id[$i]);
}
$nValue %= 11;
$verify_ref = array("1", "0", "X", "9", "8", "7", "6", "5", "4", "3", "2");
$last_verify_code = $verify_ref[$nValue];

return ( $last_verify_code == $id[17] );
{{< /highlight >}}
 參考了

*   [大陸公民身份號碼驗證規則](http://java2tw.blogspot.com/2010/01/blog-post.html)
*   [台湾 香港 中国 身份证规则](http://hi.baidu.com/aimu2000/blog/item/2656bcb7a40775e930add1f3.html)
---
title: 'Mocking View Facade in Laravel'
date: Mon, 07 Mar 2016 04:06:15 +0000
draft: false
tags: [IT 筆記]
---

在單元測試時，常會有需要 mock 元件的需求。按照 Laravel 4.2 官方文檔的說明，可以如下 mock 一個 Facade，https://laravel.com/docs/4.2/testing#mocking-facades
{{< highlight php >}}
<?php
//Event::fire('foo', array('name' => 'Dayle'));

Event::shouldReceive('fire')->once()->with('foo', array('name' => 'Dayle'));
{{< /highlight >}}
 那麼 View Facade 可以怎麼 mock 呢？ 先定義一個可以任意填入內容的 test.blade.php
{{< highlight php >}}
{{ $context }}
{{< /highlight >}}
 然後在單元測試的 setUp 部分定義這樣：
{{< highlight php >}}
<?php
$this->test_view = View::make('test');

View::shouldReceive('make')->andReturnUsing(function() {
    return $this->test_view;
});
{{< /highlight >}}
 如此一來就可以掌控 View 生成的內容，而不致於影響到測試。
{{< highlight php >}}
<?php
$this->test_view->with('ANY_CONTENT_YOU_WANT');
{{< /highlight >}}
 例如測試內文只有一個 A：
{{< highlight php >}}
<?php
class A {
    public view {
        return 'A' . View::make('name_of_view')->render();
    }
}

class ATest {
    public function setUp() {
        parent::setUp();
        $this->a = App::make('A');
        $this->test_view = View::make('test');
        View::shouldReceive('make')->andReturnUsing(function() {
            return $this->test_view;
        });
    }

    public function testView() {
        $this->test_view->with('WITHOUT_THXT_LETTER');
        $this->assertEquals(1, substr_count($this->a->view(), 'A'));
    }
}
{{< /highlight >}}

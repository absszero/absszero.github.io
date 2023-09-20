---
title: "開發踩坑, Laravel Goto for Sublime Text, Goto Middleware"
date: 2023-09-19T10:27:48+08:00
lastmod: 2023-09-19T10:27:48+08:00
draft: false
keywords: ["middleware", 'Laravel', 'mock', 'unit test']
description: ""
tags: []
categories: []
author: ""

---

[Laravel Goto for VSCode](https://marketplace.visualstudio.com/items?itemName=absszero.vscode-laravel-goto) 的 Middleware 轉跳功能完成後。隨後也逐步在 [Laravel Goto for Sublime Text](https://packagecontrol.io/packages/Laravel%20Goto) 把相同轉跳功能實做出來。正規表達式可以從 VSCode 研究結果套用，所以開發上還算快速。而且 Sublime Text Package 使用 Python 做同步執行，相較於 VSCode Extension 使用 JS 做非同步執行，開發單純的多。開發時間 VSCode 版本花了一週，這個版本則在一週內完成。但沒想到最困難的部分是單元測試，為此踩了三個坑。分別是：

- Mock not working
- sublime.active_window().folders() is empty
- Unit test panel has no stdout and stderr

## Mock not working

新增了 `workspace.py` 處理從當前專案目錄查找取得特定檔案內容。並且透過底下方式引入：

```python
# finder.py
from .workspace import get_file_content

def get_place:
    kernel_content = get_file_content(fxolder, 'app/Http/Kernel.php')
```

接著設計如下測試，並且嘗試 mock `get_file_content`：

```python
# finder_test.py
from . import unittest
from unittest.mock import patch

class TestFinder(unittest.ViewTestCase):
    @patch('LaravelGoto.lib.workspace.get_file_content')
    def test_middleware(self, mock_get_file_content):
        mock_get_file_content.return_value = 'Lorem ipsum dolor sit amet'
        place = get_place()
```

mock 所產生的物件是正確，但是測試的結果 finder.py 還是呼叫了原本的 `get_file_content` ，為什麼？

Python 在載入每個模組後會快取到 `sys.modules` 。 `@patch` 的原理是用一個 mock 的模組替換載入的模組。而當透過 `from ... import` 方式引入 `get_file_content` ，會將該方法從 `原本模組` 建立一個區域參照。所以即便已經 mock 了。實際上沒有透過 `sys.modules` 取得該模組。

```python
# 建立一個區域參照
from .workspace import get_file_content
```

一個解決辦法是不要建立區域參照，直接存取模組。

```python
# finder.py
from . import workspace

def get_place:
    kernel_content = workspace.get_file_content(fxolder, 'app/Http/Kernel.php')
```

## sublime.active_window().folders() is empty

Package 開發時，我會將 Package 的目錄開啟在 Sublime Text。由於這次開發需藉由解析 Laravel 專案的 `app/Http/Kernel.php` 取得已設置的 Middlewares。為了方便測試，刻意設計一個假的 Kernel 放在開發套件中。測試的時候 `sublime.active_window().folders()` 會返回當前的套件目錄，並且能順利找到 Kernel.php。

一切都很順利，直到發布上 Github 並且執行 Action。才發現 middleware 的測試無法通過。由於 Action logs 只顯示 unit test 執行過程的錯誤，透過 print 等方式無法顯示到 log (這又是另一個坑，後續說明)。所以問題排查遇到瓶頸。而 Action 需要提交才會觸發，每次除錯都要重新提交很沒效率。後續嘗試使用 [nektos/act](https://github.com/nektos/act) ，希望能在本地端除錯。但不知為何在本地端的 docker container 卻沒能順利執行（卡住了...）。後續終於找到方法將 print 的訊息顯示在 Action logs 之後，才意識到由於在 unit test 時，Sublime Text 運行測試過程中不會包含任何已開啟的目錄（本地測試的時候，因開發緣故會開啟套件的目錄），所以 `sublime.active_window().folders()` 始終回傳空值。最後將該方法 mock 之後，回傳當前的套件目錄位置解決。


## Unit test panel has no stdout and stderr

Sublime Text 官方提供了測試用的套件 [UnitTesting](https://github.com/SublimeText/UnitTesting) 。方便在本地端執行 Sublime Text 套件的相關測試。測試執行結束後，測試結果會顯示在 unit test panel 上。但測試過程中任何對 stdout 或 stder 的輸出都 `不會` 顯示在該 panel。而是反映在 `Consolep panel` 裡頭。本地開發的時候，可以任意地切換 panel 來查看所以沒什麼問題。但是上了 Github Action 執行的過程，卻只能查看留在 Action log 的 unit test panel 訊息。這對於要進一步除錯十分困難。

原本打算透過 ` assertEqual(first, second, msg=None)` 方法的第三個參數 `msg` 將額外訊息顯示在 log。但某些內容只在方法裡頭產生，不會傳遞到方法外面。如果要查看這些內容，必須設計新的測試滿足這方面。在腸枯思竭之際，想到先看一下測試套件的 Github issue 也許有相關的提問。果然在 [[Feature] have log on test report](https://github.com/SublimeText/UnitTesting/issues/135) 找到相關答案。方法也很簡單，只要在套件根目錄新增 `unittesting.json` 並且添加：

```json
{
    "capture_console": true
}
```

另外在查看 issue 的同時，也發現另外一個 issue。 [GitHub Actions macOS timeout installing Package Control](https://github.com/SublimeText/UnitTesting/issues/220)，剛好是困擾許久的問題。暫時先關閉 macos 的環境的測試。
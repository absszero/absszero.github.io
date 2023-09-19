---
title: "Laravel Goto for Sublime Text"
date: 2019-08-26T20:31:24+08:00		
draft: false
tags: [Sublime Text, Laravel, Plugin Development]
---





https://github.com/absszero/LaravelGoto

## Why

去年下定決心買一套 Sublime Text 之後（USD $75 的樣子），也算是認可 ST 在工作幫我省了不少時間。不過隨著 VS Code 功能逐步豐富，一直有想轉換編輯器的念頭。上個月某個週末夜晚想確認一下 VS Code 目前相關的  Extension 是否已經到位了。便逐步安裝 VS Code 以及相關套件。結果... 跟 ST 還是有段落差。`go to definition`  在 ST 上能夠針對字串直接轉跳，VS Code 則必須是類或方法的使用才能夠轉跳。

但 ST 也不是所有的文字都能正確轉跳，例如 Laravel Route 定義的部份，能夠轉跳到指定的 Controller，但是 Method 則不行；Blade 檔案的引用由於路徑目錄是透過 `.` 逗點來區隔（例如：my_dir`.`my_view），透過 `Goto anything` 也起不了作用。Package Control 上面能收尋到最接近的套件是 `Laravel OpenFile`。不過路徑必須事先選取起來，並且必須透過 `return view('your_view_path')` 叫用的檔案才能發揮作用。



## 功能需求

基於以上的原因，決定自己來做一個 ST 套件，順便練習一下 Python，於是有了 `LaravelGoto` 套件的實踐。目前最常用的功能包括：

- 開啟 Controller@Method。內建的 Goto 只能開啟 Controller 不能開啟後連帶轉跳到 Method。
- 開啟 View file。內建必須用 `/` 分隔目錄名稱，然後選取之後使用 `Goto anthing` 搜尋。
- 開啟靜態檔案（js, css...etc）。



## 找出字串範圍

建立一個 Plugin 並不困難，不過有些功能得設法實現。原本的 `Laravel OpenFile` 套件必須選取整個字串才能夠發揮作用，但大部分的情況，我希望可以像 `Goto definition` ，Ctrl + 滑鼠左鍵點擊該字串就可以直達指定的 View file 或者 Controller。藉此參考了 https://github.com/noahcoad/open-url/blob/master/open_url.py#L186	

該方法可以根據你目前游標位置，找出字串範圍，或者根據你所選的字串。

```python
    # selected 可以透過 self.view.sel()[0] 帶入，代表所選的第一個文字範圍
    def get_selection(self, selected):
        start = selected.begin()
        end = selected.end()
		
        # 當所選範圍開頭跟結果在同一個位置，表示只是指標位置而未做任何選取
        if start == end:
            # 找出當前位置指標所在的行範圍
            line = self.view.line(start)
            delimiters = "\"'"
            while start > line.a:
                # 透過逐字檢查，找到開頭距離游標最接近的單引號或雙引號
                if self.view.substr(start - 1) in delimiters:
                    break
                start -= 1

            while end < line.b:
                # 透過逐字檢查，找到結尾距離游標最接近的單引號或雙引號
                if self.view.substr(end) in delimiters:
                    break
                end += 1

		# 將開頭與結尾位置作為所選範圍
        return sublime.Region(start, end)
```



## 範圍轉換為路徑

有了選取範圍之後先轉為文字，然後判斷文字內容來決定路徑類型。

Controller 的部份比較困難的地方是當有兩個一樣的名稱的 Controller，需要透過附加 Namespace 來判定選取的是哪個 Controller（但找出 Namespace 同樣很困難！）

至於 View file，如果包含 `::`  表示啟用了 View namespace，所以只取雙冒號之後的路徑名稱

```python
    def get_path(self, selected):
        selection = self.get_selection(selected)
        # 轉換為字串
        path = self.substr(selection).strip()
        # 如果字串包含 @ 或 Controller 的字樣，判定為 Controller 
        if ("@" in path or "Controller" in path):
            namespace = self.get_namespace(selection)
            if namespace:
                path = namespace + '\\' + path
        else:
            # remove Blade Namespace
            path = path.split(':')[-1]
        return path
```



## 搜尋

找到部份檔案路徑之後，由於不同專案結構或使用方式（例如沒有使用 ST 的 Project 功能），很難找到完整的檔案路徑並且開啟。所以這邊是透過 `Goto anything` 功能檢索可能的檔案路徑。

首先判斷是否是靜態檔案，靜態檔案的特性是路徑副檔名可能是常用的 `js` ,`css` ，所以首先定義了網頁開發常用的靜態副檔名，作為檢索的依據。

如果不是靜態檔案，試著判斷是否是控制器路徑。如果是控制器則顯示 `Goto anything` 功能之後，呼叫 `insert` 指令逐字把控制器路徑輸入進去。不直接透過 `Goto anything` 查詢，是由於當我們使用 `insert` 指令的輸入完控制器檔名後，接著輸入 `@` 符號可以觸發 `Goto symbol` 的功能，如此後續輸入可以自動跳到我們所想要的 Method 位置。

最後，如果上述都不符合則判定是 View file，將路徑的 `.` 轉換成 `/` ，並且在結尾附加上 `.blade.php`，然後呼叫 `Goto anything` 功能搜尋檔案。



```python
    def search(self, path):
        args = {
            "overlay": "goto",
            "show_files": True,
            "text": path
        }

        # 是否是靜態檔案，已經預先定義了靜態檔案的副檔名範圍
        if (self.is_static_file(path)):
            self.window.run_command("show_overlay", args)
            return

        is_controller = self.is_controller(path)
        if is_controller:
            args["text"] = ''
        else:  # it's a view file
            args["text"] = args["text"].replace('.', '/') + '.blade.php'

        self.window.run_command("show_overlay", args)

        # if it 's Controller path, use insert command to trigger symbol search
        if is_controller:
            self.window.run_command("insert", {
                "characters": path.replace('@', '.php@')
            })
        return
```



## Namespace

由於控制器可能有檔名重複的情形，所以適當的加上 Namespace 可以更正確的檢索到合適的檔案。但由於複雜的 Namespace 在 Route 方面可能一層包一層，目前只能檢索到最外層的 Namespace。

根據 Laravel 跟 Lumen 各個版本的使用情況，目前有兩種設定 Namespace 的方式：

```python
patterns = [
    # Route::namespace('MyNamespace')
    re.compile(r"namespace\s*\(\s*(['\"])\s*([^'\"]+)\1"),
    # Route::group(['namespace' => 'MyNamespace'])
    re.compile(r"['\"]namespace['\"]\s*=>\s*(['\"])([^'\"]+)\1"),
]
```



接下來要透過 ST 語法解析內建的 selector 找出每一個 Route 函式包含的範圍，並且確保當前所選的控制器路由，是否包含在所選的函式範圍當中。如果是，則帶出可能的 Namespace。

```python
    def get_namespace(self, selected):
        # PHP 語法測試範例 https://github.com/sublimehq/Packages/blob/7c40526e68e07e0e4a6681bb2d04359115ec1331/PHP/syntax_test_php.php
        # 找出所有的函式範圍
        functions = self.view.find_by_selector('meta.function-call')
        for function in functions:
        	# 如果函式範圍不包含所選的位置則跳過
            if (not function.contains(selected)):
                continue
            # 一般函式的第一行應該就會包含 Namespace
            region = self.view.line(function)
            block = self.substr(region).strip()
            # 用正規表達式檢索批配出 Namespace
            for pattern in patterns:
                matched = pattern.search(block)
                if (matched):
                    return matched.group(2)
        return

```



## 總結

由於 Python 是執行在 Sublime Text 的內部，所有也得透過內部的 Console 查看程式編寫的情況。另外這年頭寫程式一定要來點單元測試，這部份也在後來添加了上去。完成作品已經提交 Package Control，不過要排到審查上線，大概還有許多時間。

後續開發套件的經驗也拿來修改 SublimeLinter-contrib-php-cs-fixer 套件，以符合新版 SublimeLinter 的改版需求。基本上還是解決工作的需求，不過 ST 仍然有些功能即使已經使用多年仍然一知半解。希望找個時間細部摸索一下，期望開發的時間能做更有效的應用。


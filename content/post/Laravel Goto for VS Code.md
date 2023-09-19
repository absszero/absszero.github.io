---
title: "Laravel Goto for VS Code"
date: 2019-09-09T20:16:24+08:00
draft: false
tags: [VS Code, Laravel, Plugin Development]
---





https://github.com/absszero/vscode-laravel-goto

## Why

在完成 [Laravel Goto for Sublime Text]({{< ref "Laravel Goto for Sublime Text.md" >}}) 之後，心想何不在 VS Code 上也試著實做一樣的套件？畢竟接觸到的套件多半只解決部份問題。其中一個未能解決的問題，就是 Namespace 的使用。

由於目前開發的專案愈趨複雜，路由的設定已經從成長快要上百條。沒有有效的透過 Namespace 區分 Controller。很容易就遇到撞名的問題。開發前先針對 VS Code Marketplace 上類似的專案搜尋一番：

- [Laravel goto view](https://marketplace.visualstudio.com/items?itemName=codingyu.laravel-goto-view) 視圖轉跳，能針對特定程式寫法下去搜尋，只能查找指定的視圖目錄。
- [laravel-goto-controller](https://marketplace.visualstudio.com/items?itemName=stef-k.laravel-goto-controller) Controller 轉跳，無法處理有兩個相同檔名的 Controller ，並且開啟之後不會自動轉跳到指定的 Method
- [goto-route-controller-laravel](https://marketplace.visualstudio.com/items?itemName=erlangparasu.goto-route-controller-laravel) Controller 轉跳或 Route 轉跳。蠻特別的一個套件，開發是使用 JS 而不是官方推薦的 TS。

上述的套件都是進行所謂的「精準轉跳」。亦即找到指定的檔案位置並開啟檔案。但也因此必須將檔案放置在限定的目錄位置才能發揮作用。如果專案結構跟 Laravel 官方預設位置有落差，便無法發揮作用。為了解決這個問題，嘗試開發「模糊轉跳」的方式。使用呼叫 `Go to File` 的方式，透過編輯器內建的模糊搜尋，找到指定的檔案。

上述的套件都是透過 HoverProvider 或者 LinkProvider 的方式，檢索出當前文件可以轉跳的位置，並且將它轉換成超連結。優點是可以看到可以點選的檔案位置，但缺點也是為了找出可點選的位置，必須逐一的爬找出文字範圍。開發人員多半了解 View 或者 Controller 所涵蓋的文字範圍，所以後續決定不透過 Provider 的方式顯示這些位置。



## 功能需求

參照之前開發套件的需求，所需要的功能包括：

- 開啟 Controller@Method。VS Code 可以開啟指定的 Controller，而開啟後要能夠跳到指定的 Method。
- 開啟 View file。



## 找出字串範圍

有了之前的開發，查找字串變得容易許多。編輯器文字範圍的觀念大致上差不多了。都有所謂的`點`，以及`範圍`。

```typescript
export function getSelection(editor: vscode.TextEditor, selected: vscode.Selection) : vscode.Range {
	let start = selected.start;
	let end = selected.end;

    // 如果有選擇範圍，返回選擇範圍
	if (!start.isEqual(end)) {
		return new vscode.Range(start, end);
	}

	const line = editor.document.lineAt(start);
	const DELIMITERS = "\"'"
    // 逐字查找目前游標所在的行，以雙引號或單引號當作邊際，找出文字邊際
	while (start.isAfter(line.range.start)) {
		let next = start.with({character: start.character - 1})
		let char = editor.document.getText(new vscode.Range(next, start));
		if (-1 !== DELIMITERS.indexOf(char)) {
			break;
		}
		start = next;
	}
	while (end.isBefore(line.range.end)) {
		let next = end.with({character: end.character + 1})
		let char = editor.document.getText(new vscode.Range(end, next));
		if (-1 !== DELIMITERS.indexOf(char)) {
			break;
		}
		end = next;
	}

	return new vscode.Range(start, end);
}
```



## 範圍轉換為路徑位置

有了選取範圍之後先轉為文字，然後判斷文字內容來決定路徑類型。針對控制器路徑需要處理使用 Namespace 的情形。

```typescript
export function getPlace(editor: vscode.TextEditor, selection: vscode.Range) : { path: string; method: string; }
{
	let method = "";
	let path = editor.document.getText(selection);
	if (isController(path)) {
        // 有包含 Method 的情況
		if (-1 !== path.indexOf('@')) {
			let splited = path.split('@');
			path = splited[0];
			method = splited[1];
		}
		// 需要查找 Namspace 的情況
		if ('\\' !== path[0]) {
			let namespace = (new Namespace).find(editor.document, selection);
			if (namespace) {
				path = namespace + '/' + path;
			}
		}

		path = path.replace(/\\/g, '/') + '.php';

	} else {
        // 其餘情況判定為 view file
		let splited = path.split(':');
		path = splited[splited.length - 1];
		path = path.replace(/\./g, '/') + '.blade.php';
	}
	return {path: path, method: method};
}
```



## Namespace 

從 route 檔案當中可以找到指定 Namespace。按照 Sublime Text 的方法，可以透過 TextMate grammer 查找出包含目前選定 Controller 所包含的程式碼範圍。VS Code 雖然在主題方面會利用 TextMate grammer 進行顏色渲染，但這部份並沒有提供給開發者。所以如果要經由該功能實現有幾個方法：

- [VSCode TextMate](https://github.com/microsoft/vscode-textmate) 安裝這個套件進行開發，但是它相依的套件安裝有點麻煩。
- [php-parser](https://www.npmjs.com/package/php-parser) 解析 PHP 的套件，利用語法解析當前的檔案，找出範圍。方法可行，但有點殺雞用牛刀。
- VS Code Language Server，例如外部呼叫 PHP 本身解析檔案，然後得知目前的範圍所選用的 Namespace 參數。

以上的方法都可行，但對於只要解析出目前的 Namespace，似乎花費太多運算成本。最後從 [PHP Symbols](https://marketplace.visualstudio.com/items?itemName=linyang95.php-symbols) 得到了靈感，直接本文逐步搜尋。查找的步驟主要是：

1. 根據正規表達式找出 Namespace 設定的字串位置，以之前開發所得到的結果有兩個規則：

    `/namespace\s*\(\s*(['"])\s*([^'"]+)\1/g`   
    
    `/['\"]namespace['"]\s*=>\s*(['"])([^'"]+)\1/g`
    
2. 找出花括號的範圍，上一個步驟找到字串，沿著字串往下搜尋最後勢必會看到帶著成對花括號的路由範圍。目前不考量包含註解或 PHP 語法有誤的情況，試著搜尋出結尾的花括號位置。

3. 有了成對的花括號位置範圍後，判斷選定的 Controller 是否在範圍內，成立則將該 Namespace 返回。



## 搜尋

VS Code 也內建了類似 Sublime Text `Goto anything`  ，名為 `workbench.action.quickOpen` 的指令可根據給定的路徑，列出最貼近的檔案位置。但 VS Code 本身並沒有內建解析 PHP Symbol 的能力。所以即便已經打開需要的檔案，也無法轉跳到選定的 Method 位置。這部份需要第三方的套件支持。不過對於 PHP 開發人員而言，應該早已安裝相關的套件支持了。

為了在開啟檔案之後轉跳到 Symbol，這邊藉由事件的綁定來實現。

```typescript
const place = getPlace(editor, selection);
if (place.method) {
    // 判斷切換到的編輯器視窗檔名是否是搜尋的檔名來判斷
    const event = vscode.window.onDidChangeActiveTextEditor(e => {
        if (undefined !== e) {
            // if opened document is selected document, go to symbol
            if (basename(place.path) === basename(e.document.uri.path)) {
                vscode.commands.executeCommand('workbench.action.quickOpen', '@' + place.method);
            }
        }
        event.dispose();
    });
}
```



## 總結

VS Code 的開發入門比起 Sublime Text 友善許多。並且在除錯以及測試整合方面都比 ST 更為完整。不過豐富的 API 也讓開發人員得小心爬梳才能找到合適的範例。TypeScript 方面頭一次接觸，對於編輯器能夠進行型別提示跟檢查感覺是十分實用，但對於首次接觸的我來說，還是有不小的難度（主要是 TS 的語法規範）

VS Code 發布流程也比 ST 更完整，目前沒有人工審查的要求，任何人在開發完成之後都可以藉由簡單的操作，立即將套件發布到 Marketplace，而後續的更新也有完整的指令支援。
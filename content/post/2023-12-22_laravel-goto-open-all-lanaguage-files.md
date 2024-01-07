---
title: "Laravl Goto 在新視窗開啟所有語系檔"
date: 2023-12-22T10:17:33+08:00
lastmod: 2023-12-23 20:23:37+08:00
draft: false
keywords: ["Laravel","Language"]
description: ""
tags: []
categories: []
author: ""

---

十月的時候我從 Github 收到一個功能請求，希望能夠在 Laravel Goto for Sublime Text 實現選取語系的 key ，然後開啟所有對應語系檔在新視窗上面。實作過程並不特別困難。原理是檢索語系目錄把其他語系目錄找出來。接著另開新視窗，將每一個語系檔打開。後續比較陌生的功能是檔案佈局。由於開啟多個檔案需要均勻編排每一個檔案頁籤的寬度。Sublime Text 的 `set_layout` API 提供這個功能。不過 `照舊` 官網文章寫得不夠詳細，看了[論壇上有的描述](https://forum.sublimetext.com/t/set-layout-reference/5713)才大概曉得怎麼邊排。後續選擇使用水平佈局的方式，畢竟用戶寬螢幕的可能性比較高。

接著我希望把相同功能實作在 VSCode 上面。前面查找語系目錄的實現同樣不是太困難。但是 VSCode 在開啟新視窗以及操作新視窗的方式跟 Sublime Text 是完全不一樣的。底下是 Sublime Text 開啟新視窗、設定佈局，以及開啟檔案的方式。使用 `run_command('new_window')` 開啟新視窗之後，透過 `active_window()` 能夠拿到這個新視窗，並且進行操作。

```python
active_window = sublime.active_window()
active_window.run_command('new_window')
new_window = sublime.active_window()
new_window.set_layout({
    "cols": cols,
    "rows": [0.0, 1.0],
    "cells": cells
})

for (idx, file) in enumerate(files):
    new_window.open_file(file)
    new_window.set_view_index(new_window.active_view(), idx, 0)
```

VSCode 可以透過 `vscode.openFolder` 指令開啟新視窗並且聚焦。但無法在原本的視窗中操作這個新視窗資源。所以後續開啟所有語系檔，必須在新的視窗中自動地被執行。（想到這點當時就想哭了...😢）

```js
await vscode.commands.executeCommand("vscode.openFolder", uri, {
	forceNewWindow: true,
});
```

### ExtensionContext.globalState

如何傳遞變數到另外一個視窗困擾我許久，其中甚至考慮將內容寫入本地端的磁碟上。新視窗開啟的時候檢查該檔案是否存在之類。進度停頓了一之後，所幸從相關的討論上查找到解決方面。套件啟動的時候會自動帶入一個 ExtensionContext，透過 `globalState` 屬性將資料暫存到裡頭。並且每個視窗中當套件啟動時能共享到這些資料。有了這個發現，終於能夠傳遞語系檔案到另外一個新視窗。

```js
// store all language files
content.globalState.update('open_all', args);
```

由於 ExtensionContext.globalState 在套件啟動（activate）的時候提供。為了取得 ExtensionContext，另外在 `activate` 註冊了一個新的指令將這個變數傳遞進來。

```js
const newWindowDispose = vscode.commands.registerCommand(
    'extension.vscode-laravel-goto.new-window',
    (args) => newWindow(context, args)
);
context.subscriptions.push(newWindowDispose);
```

接著需要呼叫的地方建立一個連結，點擊時呼叫該指令，進行寫入 globalState 以及開啟新視窗。

```js
    const files = [...];
    const command = 'extension.vscode-laravel-goto.new-window';
    const args = encodeURI(JSON.stringify([files]));
    const commentCommandUri = vscode.Uri.parse(`command:${command}?${args}`);
    return `\n\n[Open all files above in new window](${commentCommandUri})`;
```

### Open All Files

前面提到新視窗無法藉由舊視窗操作。所以新視窗必須主動檢查是否需要開啟檔案。現在已經透過 globalState 存入所有語系檔的位置。接著在新視窗啟動套件的同時，套件會檢查 globalState 是否有值，並開啟裡頭的檔案。

```js
	const args = content.globalState.get('open_all');
	content.globalState.update('open_all', null);

	// open all language files
	for (let index = 1; index < args.files.length; index++) {
		const uri = vscode.Uri.file(args.files[index]);
		const doc = await vscode.workspace.openTextDocument(uri);
		await vscode.window.showTextDocument(doc, vscode.ViewColumn.Beside);
		locateByLocation(vscode.window.activeTextEditor, args.location);
	}
```

寫到這邊主要功能已經完成，但由於開發使用 Extension Host 執行開發中的套件，所以當新視窗被開啟的時候，開發中的套件已經不在其掌控。所以必須不斷的打包 Extension，並且透過安裝/反安裝的方式載入開發中的套件，讓它可以在開新視窗的時候發揮作用，也是其中開發上比較麻煩的地方。

再來由於是藉由 macOS 開發，測試也是針對 macOS 磁碟環境設計。當我提交到 Github 進行 Windows 測試時遇到磁碟路徑問題。其中一個原因是透過 `Uri.parse` 處理路徑。但合理應該使用 `Uri.file`。[官方文件有明確的說明](https://code.visualstudio.com/api/references/vscode-api#Uri)

> The difference between Uri.parse and Uri.file is that the latter treats the argument as path, not as stringified-uri. E.g. Uri.file(path) is not the same as Uri.parse('file://' + path) because the path might contain characters that are interpreted (# and ?).

後續遇到相關測試環境配置問題，包括我的 @vscode/test-electron 套件太老舊，無法正確下載 Windows 版本的 VSCode。以及我參考新版的套件範例修改測試使用 @vscode/test 遇到問題。（後來還是回頭用原本的方法）。終於... 延遲已久的開啟多語系檔功能總算可以釋出一版。
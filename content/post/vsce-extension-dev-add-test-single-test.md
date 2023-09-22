---
title: "VSCode Extension Development, Run Single Test"
date: 2023-09-22T10:17:33+08:00
lastmod: 2023-09-22T10:17:33+08:00
draft: false
keywords: ["VSCode","Extension", "Unit Test"]
description: ""
tags: []
categories: []
author: ""

---

隨著 VSCode 開發 Extension 愈趨複雜，相關的單元測試也越來越多。每次增加一個方法以及相對測試，整個套件的測試都得重新執行一遍。為了更有效率的進行開發，目前要解決幾個痛點：

- 無法執行單一檔案的測試。
- 無法執行單一測試案例的測試。

針對以上兩點，逐步說明解決方法。

## 無法執行單一檔案的測試

當在 VSCode 裡頭透過按下 `F5` 執行 `Extension Tests` 的時候，會透過 `./vscode/launch.json` 發起另一個 VSCode 的主體並且執行 `test/suite/index.js` 的內容。而裡頭會將 `test/suite` 底下所有 `.test.js` 結尾的檔案，利用 Mocha 加入並執行測試。所以每次執行的時候才會重跑所有的測試。

```ts
// https://github.com/microsoft/vscode-extension-samples/blob/main/helloworld-test-sample/src/test/suite/index.ts

glob('**/**.test.js', { cwd: testsRoot }, (err, files) => {
    // Add files to the test suite
    files.forEach(f => mocha.addFile(path.resolve(testsRoot, f)));
```

為了執行單一檔案的測試，當前的解決思路：

- `./vscode/launch.json` 添加一組設定，並將當前檔案名稱帶給去。
- `test/suite/index.js` 根據當前檔案名稱，只把該檔案加到 Mocha 的測試。
- 紀錄這次檔案名稱到 `.vscode-test/.previous` 作為下次參考。
- 下次執行，如果檔名不是 `.test.ts` 結尾，透過 `.vscode-test/.previous` 取得上次的測試檔名，重複上次的測試（有時候會一邊修改 source ，一邊執行測試，方便不需要跳回測試檔案，按 F5 也能執行上次的測試）。

首先在 `./vscode/launch.json` 添加一組設定名稱為 `Test Current/Previous File` 。根據原本的 `Extension Tests` 複製一份，只是在 `env` 的欄位增加  `"TEST_FILE": "{$file}"` 。透過 env 環境變數將當前的檔名帶過去。

```json
{
    "name": "Test Current/Previous File",
    "type": "extensionHost",
    "request": "launch",
    "runtimeExecutable": "${execPath}",
    "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionTestsPath=${workspaceFolder}/out/test/suite/index"
    ],
    "outFiles": [
        "${workspaceFolder}/out/test/**/*.js"
    ],
    "env": {
        "EXTENSION_PATH": "${workspaceFolder}",
        "TEST_FILE": "${file}",
    },
    "preLaunchTask": "npm: watch"
}
```

接著在 `test/suite/index.ts` 添加相關判斷：

```ts
	return new Promise(async (c, e) => {
        // 保留預設的查找規則
		let filePattern = '**/**.test.js';

		if (process.env.TEST_FILE) {
			const previous = process.env.EXTENSION_PATH + '/.vscode-test/.previous';
            /**
             * 如果是 test.ts 結尾表示是測試檔，將它寫入到 previous 作為下次參考，
             * 反之 previous 存在則讀取出來，作為當前的測試檔案名稱。
             */
			if (process.env.TEST_FILE.endsWith('test.ts')) {
				filePattern = process.env.TEST_FILE;
				fs.writeFile(previous, filePattern, err => null);
			} else {
				if (fs.existsSync(previous) ) {
					filePattern = fs.readFileSync(previous, 'utf8');
				}
			}

            // 由於 TypeScript 編譯後的目錄跟副檔名有變化，需要進行調整。
			filePattern = filePattern.replace('src/test/', 'out/test');
			filePattern = filePattern.replace('.ts', '.js');
			filePattern = filePattern.replace(testsRoot, ''); // 下方會以 testsRoot 為當前目錄，所以須去掉
		}

        glob(filePattern, { cwd: testsRoot }, (err, files) => {
            // ......
        }
```

到這邊算是解決了。小遺憾是 `/.vscode-test/.previous` 這個臨時檔案，原本是希望 VSCode 關閉之後能自動刪除掉。但目前沒有具體做法，幸好影響也不大。


## 無法執行單一測試案例的測試

有了執行單一檔案測試的功能之後，接下來更近一步，希望能只執行單一測試案例。Mocha 本身支持使用 `grep` 參數進行過濾。所以接下來的問題是，找出該測試案例的名稱。

原本的想法是編輯器傳入行號，然後從行號上下檢索檔案所包含的測試案例區塊。但要識別區塊不容易。可能需要用到類似 AST(abstract syntax tree)。後來參考之前常用的套件  [Better PHPUnit](https://marketplace.visualstudio.com/items?itemName=calebporzio.better-phpunit&ssr=false#review-details) ，他也提供了類似的功能。他的解決思路很簡單，從目前的行號取得這一行的文字，然後用正規表達式檢查，符合的話回傳該方法名稱，否則往上一行查找，直到第一行。

```ts
// https://github.com/calebporzio/better-phpunit/blob/81447bd3c0478d6577d697b398578068f53c31d7/src/phpunit-command.js#L88
get method() {
    let line = vscode.window.activeTextEditor.selection.active.line;
    let method;

    while (line > 0) {
        const lineText = vscode.window.activeTextEditor.document.lineAt(line).text;
        const match = lineText.match(/^\s*(?:public|private|protected)?\s*function\s*(\w+)\s*\(.*$/);
        if (match) {
            method = match[1];
            break;
        }
        line = line - 1;
    }

    return method;
}
```

最後總結一下解決思路：

- `./vscode/launch.json` 添加一組設定，傳遞當前檔案名稱跟行號。
- `test/suite/index.js` 判斷是否有傳遞行號，根據它檢索出測試案例名稱。
- 將測試案例名稱加入到 Mocha 的 grep。
- 記錄行號到 previous，作為下次執行測試參考。

首先在 `./vscode/launch.json` 添加一組設定名稱為 `Test Current Case` 。根據 `Test Current/Previous File` 複製一份，只是在 `env` 的欄位增加  `"TEST_FILE_LINE": "{$lineNumber}"` 。透過 env 環境變數將當前的檔名帶過去。

```json
{
    "name": "Test Current Case",
    "type": "extensionHost",
    "request": "launch",
    "runtimeExecutable": "${execPath}",
    "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionTestsPath=${workspaceFolder}/out/test/suite/index"
    ],
    "outFiles": [
        "${workspaceFolder}/out/test/**/*.js"
    ],
    "env": {
        "EXTENSION_PATH": "${workspaceFolder}",
        "TEST_FILE": "${file}",
        "TEST_FILE_LINE": "${lineNumber}",
    },
    "preLaunchTask": "npm: watch"
}
```

`test/suite/index.ts` 增加方法檢索測試案例名稱：

```ts
async function getTestCase(filename: string, lineNumber: number) : Promise<string> {
	const document = await vscode.workspace.openTextDocument(filename);
	while (lineNumber > 0) {
		const lineText = document.lineAt(lineNumber).text;
		const match = lineText.match(/test\(\s*['"`]([^'"`]+)/);
		if (match) {
			return match[1];
		}
		--lineNumber;
	}

	return '';
}
```

接著添加到原本的執行單一測試檔案邏輯中：

```ts
	return new Promise(async (c, e) => {
		let filePattern = '**/**.test.js';

		if (process.env.TEST_FILE) {
			const previous = process.env.EXTENSION_PATH + '/.vscode-test/.previous';
			let lineNumber = '';
			if (process.env.TEST_FILE.endsWith('test.ts')) {
				filePattern = process.env.TEST_FILE;
				if (process.env.TEST_FILE_LINE) {
					lineNumber = process.env.TEST_FILE_LINE;
				}

                // 用 : 區隔出檔名跟行號
				fs.writeFile(previous, filePattern + ':' +lineNumber, err => null);
			} else {
				if (fs.existsSync(previous) ) {
                    // 根據 : 拆解出檔名跟行號
					const content = fs.readFileSync(previous, 'utf8').split(':');
					filePattern = content[0];
					lineNumber = content[1];
				}
			}

            // 檢索測試案例名稱，並加到 Mocha
			const testCase = await getTestCase(filePattern, +lineNumber);
			if (testCase) {
				mocha.grep(testCase);
			}
			filePattern = filePattern.replace('src/test/', 'out/test');
			filePattern = filePattern.replace('.ts', '.js');
			filePattern = filePattern.replace(testsRoot, '');
		}

		glob(filePattern, { cwd: testsRoot }, (err, files) => {
            // ....
        }
```

到此順利解決，完整檔案可參考：

- [.vscode/launch.json](https://github.com/absszero/vscode-laravel-goto/blob/master/.vscode/launch.json)
- [test/suite/index.ts](https://github.com/absszero/vscode-laravel-goto/blob/master/src/test/suite/index.ts)

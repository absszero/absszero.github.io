---
title: After extends the eslint rules
date: 2024-02-09T09:57:24+08:00
keywords: ["ESlint"]
---

最早接觸到 TypeScript 開發是透過 VSCode 的 [Your First Extension ](https://code.visualstudio.com/api/get-started/your-first-extension) 範例。教學透過 Yeoman 使用 VS Code Extension Generator 產生。預設已經自帶 ESlint 檢查。所以也沒特別在意就直接開發。在近期我回頭檢視相關的規則檔，才發現預設的規則其實非常寬鬆。

```js
{
	"rules": {
		"@typescript-eslint/member-delimiter-style": [
			"warn",
			{
				"multiline": {
					"delimiter": "semi",
					"requireLast": true
				},
				"singleline": {
					"delimiter": "semi",
					"requireLast": false
				}
			}
		],
		"@typescript-eslint/naming-convention": "warn",
		"@typescript-eslint/no-unused-expressions": "warn",
		"@typescript-eslint/semi": [
			"warn",
			"always"
		],
		"curly": "warn",
		"eqeqeq": [
			"warn",
			"always"
		],
		"no-redeclare": "warn",
		"no-throw-literal": "warn",
		"no-unused-expressions": "warn",
		"semi": "warn"
	}
}
```

所以以往開發很少注意到寫法上有潛在的問題。畢竟也初學 TypeScript，編譯能過就好。但最近回頭加入 ESlint 驗證的 CI/CD，才根據 ESLint 官方建議的設定加入底下的擴充。

```js
{
	extends: [
		'eslint:recommended',
		'plugin:@typescript-eslint/recommended-type-checked',
		'plugin:@typescript-eslint/stylistic-type-checked',
	],
}
```

想當然爾冒出了很多錯誤需要修正，這邊列出那些潛越的規則：

## eslint@typescript-eslint/no-floating-promises

使用 promise 需要被 await，以及合理的配置錯誤處理。

```js
// 錯誤, Promises must be awaited, end with a call to .catch, end with a call to .then with a rejection handler or be explicitly marked as ignored with the `void` operator.
vscode.window.showWarningMessage('Laravel Goto: unidentified string.');

// 正確
await vscode.window.showWarningMessage('Laravel Goto: unidentified string.');
```

## eslint@typescript-eslint/prefer-nullish-coalescing

建議用 `Nullish Coalescing Operator （??）` 取代 `||` 的寫法。

```js
// 錯誤, Prefer using nullish coalescing operator (`??`) instead of a logical or (`||`), as it is a safer operator.
path = await vscode.window.showQuickPick(items) || '';

// 正確
path = await vscode.window.showQuickPick(items) ?? '';
```

## eslint@typescript-eslint/no-inferrable-types

```js
// 錯誤, Type string trivially inferred from a string literal, remove type annotation.
consoleKernel: string = '';

// 正確
consoleKernel = '';
```

## eslint@no-useless-escape

取消多餘的跳脫字元。

```js
// 錯誤
const cmdFnPattern = /function commands\([^\)]*[^{]+([^}]+)/m;

// 正確
const cmdFnPattern = /function commands\([^)]*[^{]+([^}]+)/m;
```

## eslint@prefer-const

不會變動的變數，建議使用 const 定義。

```js
// 錯誤, 'commands' is never reassigned. Use 'const' instead.
let commands = new Map;

// 正確
const commands = new Map;
```

## eslint@typescript-eslint/no-misused-promises

```js
// 錯誤, Promise returned in function argument where a void return was expected.
const commandDispose = vscode.commands.registerTextEditorCommand('extension.vscode-laravel-goto', Command);

// 正確
const commandDispose = vscode.commands.registerTextEditorCommand('extension.vscode-laravel-goto', () => Command);
```

## eslint@typescript-eslint/no-unsafe-argument

any 型別需要適當的轉型處理。


```js
// 錯誤, Unsafe argument of type `any` assigned to a parameter of type `IOpenAllArgs`.
const newWindowDispose = vscode.commands.registerCommand(
	'extension.vscode-laravel-goto.new-window',
	(args) => newWindow(context, args)

// 正確
const newWindowDispose = vscode.commands.registerCommand(
	'extension.vscode-laravel-goto.new-window',
	(args) => newWindow(context, args as IOpenAllArgs)
```

## eslint@typescript-eslint/no-empty-function

```js
// 錯誤, Unexpected empty function 'deactivate'.
export function deactivate() {}

// 正確, 直接刪除未使用的方法
```



## eslint@typescript-eslint/unbound-method

預先綁定，避免在方法裡頭調用 `this` 產生非預期的問題。

```js
// 錯誤, Avoid referencing unbound methods which may cause unintentional scoping of `this`.
const places = [
  this.pathHelperPlace,
]

// 正確
const places = [
  this.pathHelperPlace.bind(this),
]
```

## eslint@typescript-eslint/prefer-for-of

簡單的 for loop 會建議改用 for-of 取代。

```js
// 錯誤, Expected a `for-of` loop instead of a `for` loop with this simple iteration
for (let i = 0; i < places.length; i++) {
	place = await places[i](this, place, this.document, this.selection);
	if (place.path) {
		return place;
	}
}

// 正確
for(const thePlace of places) {
	place = await thePlace(this, place, this.document, this.selection);
	if (place.path) {
		return place;
	}
}
```

## eslint@no-extra-boolean-cast

避免多餘的 Boolean 型別轉換。

```js
// 錯誤, Redundant Boolean call.
if ((Boolean)(match && match[3] === ctx.path))

// 正確
if ((match && match[3] === ctx.path))
```

## eslint@typescript-eslint/no-for-in-array

避免 for-in 可能出現的潛在問題。

```js
// 錯誤, For-in loops over arrays are forbidden. Use for-of or array.forEach instead.
for (const key in words) {

// 正確
words.forEach((word, key) => {
```

## eslint@typescript-eslint/array-type

```js
// 錯誤, Array type using 'Array<string>' is forbidden. Use 'string[]' instead.
let extensions: Array<string> = vscode.workspace.getConfiguration().get('laravelGoto.staticFileExtensions', []);

// 正確
let extensions: string[] = vscode.workspace.getConfiguration().get('laravelGoto.staticFileExtensions', []);
```

## eslint@typescript-eslint/restrict-template-expressions

使用樣板表達式，要確保樣板裡頭內容是字串或數字。

```js
// 錯誤, Invalid type "Uri" of template literal expression.
const commentCommandUri = vscode.Uri.parse(`command:${command}?${args}`);
return `[${path}](${commentCommandUri})`;

// 正確
const commentCommandUri = vscode.Uri.parse(`command:${command}?${args}`);
return `[${path}](${commentCommandUri.toString()})`;
```

## eslint@no-extra-semi

```js
// 錯誤, Unnecessary semicolon.
const files = await workspace.findFiles('**/resources/lang/**', 1);;

// 正確
const files = await workspace.findFiles('**/resources/lang/**', 1);
```

## eslint@typescript-eslint/no-unsafe-assignment

```js
// 錯誤, Unsafe assignment of an `any` value.
const routeRows: RouteRow[] = JSON.parse(raw.stdout);

// 正確
const routeRows: RouteRow[] = JSON.parse(raw.stdout) as RouteRow[];
```

## eslint@typescript-eslint/no-explicit-any

```js
// 錯誤, Unexpected any. Specify a different type.
export function spawnSync(command: string, args: string[]): SpawnSyncReturns<any>

// 正確
export function spawnSync(command: string, args: string[]): SpawnSyncReturns<Buffer>
```

## eslint@no-async-promise-executor

```js
// 錯誤, Promise executor functions should not be async.
return new Promise(async (c, e) => {
```

這條規則卡很久，因為裡頭有一個需要使用 `await` 的呼叫，但規則說明裡頭不應該有：

```js
const testCase = await getTestCase(filePattern, +lineNumber);
if (testCase) {
	mocha.grep(testCase);
}
```

接著我把它改成：

```js
getTestCase(filePattern, +lineNumber)
.then((testCase) => testCase && mocha.grep(testCase))
.catch(err => e(err)); // 如果不做 .catch 會違反另一個規則 eslint@typescript-eslint/no-floating-promises
```

最後總算可以移除 `async`：

```js
// 正確
return new Promise((c, e) => {
```

## 後話

添加三個擴充規則之後，錯誤其實蠻多的。原本還打算只添加一兩個擴充規則就好，但根據一個規則修改之後，又後續挑戰另外一個規則。最終才完成所有規則的修正。也多虧這些規則，讓我學到 TypeScript 應該注意的部分，包括一些型別如何適當轉換，以及 Promise 應用的相關知識。
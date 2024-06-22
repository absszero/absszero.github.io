---
title: Bootstrap and its breakpoint generation
date: 2024-06-22T08:11:35+08:00
keywords: ["Bootstrap", "breakpoint", "斷點", "Sass"]
---

## 自定義斷點

Bootstrap 在[官方文件](https://getbootstrap.com/docs/5.1/layout/breakpoints/)關於斷點(Breakpoint) 一開始就提到可以透過自訂變數設定斷點。最近在處理前端項目的時候，也注意到了前人已經設定該變數。支持開發上的斷點使用。

```scss
$grid-breakpoints: (
  xs: 0,
  sm: 576px,
  md: 768px,
  lg: 1024px,
  xl: 1200px,
  xxl: 1400px
);
```

另外文件也提到可以利用它所提供的 mixin 快速進行斷點利用。

```scss
@include media-breakpoint-up(sm) {
  .custom-class {
    display: block;
  }
}
```

## Undefined mixin

而當我開始在專案中使用斷點套用的 mixin 卻遇到 `Undefined mixin` 的問題。[搜索一番](https://stackoverflow.com/questions/71720120/scss-include-media-breakpoint-down-undefined-mixin-but-i-import-bootstrap-cla)原來需要在當前位置載入斷點相關的 scss。這是 Sass 的運作方式，採用模組化的方式執行。

```scss
@import 'node_modules/bootstrap/scss/functions';
@import 'node_modules/bootstrap/scss/variables';
@import 'node_modules/bootstrap/scss/mixins';
```

確實 `media-breakpoint-up` 可以正確的使用了，但我發現斷點的位置判斷卻不如預期。預設的斷點 `lg: 992px` 被我修改成 `lg: 1024px`。但最終生成出來的斷點仍然是 `lg: 992px`。為什麼呢？於是我又回頭查看文件，並且查看 BS(Bootstrap) 的原始碼。根據[官網的建議](https://getbootstrap.com/docs/5.1/customize/sass/)，客製化的變數放置的情況如下：


```scss
// custom.scss
// Option A: Include all of Bootstrap

// Include any default variable overrides here (though functions won't be available)
$grid-breakpoints: (
  xs: 0,
  sm: 576px,
  md: 768px,
  lg: 1024px,
  xl: 1200px,
  xxl: 1400px
);

@import "node_modules/bootstrap/scss/bootstrap";
```

我也在 `custom.scss` 文件裡頭透過 `@debug $grid-breakpoints` 看到斷點確實被設定正確了。然而我回到獨立的 Vue SFC 檔案，透過 import 相關的 BS mixins 之後，利用 `@debug $grid-breakpoints` 卻看到斷點又會重新設置成預設。幾番思考之後，意識到 Sass 作為一個 CSS 的預編輯器。所有的變數、mixin 只作用在引入它的文件中。所以正確的做法，是在每一個文件中載入 `custom.scss` 。

## 編譯緩慢

到此，確實自定義的斷點被正確的解釋，相關的 mixin 也可以使用。但有一個新的問題。 `custom.scss` 最後 import `node_modules/bootstrap/scss/bootstrap` 會開始將前面所定義的變數，編譯成相對應的 css。原本放置於 main.js 引入的 custom.scss，只在開頭的編譯一次 css。現在每一個文件都需要編譯一次，所以產生額外的編譯時間。

為了解決的這個問題而更進一步的拆分 custom.scss，將它分成兩個部分。只包含斷點相關的 `breakpoint.scss` 跟剩餘的 `custom.scss`。

```scss
/* breakpoint.scss */

$grid-breakpoints: (
  xs: 0,
  sm: 576px,
  md: 768px,
  lg: 1024px,
  xl: 1200px,
  xxl: 1400px
);

@import 'node_modules/bootstrap/scss/functions';
@import 'node_modules/bootstrap/scss/variables';
@import 'node_modules/bootstrap/scss/mixins';
```

```scss
/* custom.scss */

@import "./breakpoint.scss";
@import "node_modules/bootstrap/scss/bootstrap";
```

接著 main.js 依舊引入 custom.css，但在每一個會使用到斷點 mixin 的文件，獨立引入 `breakpoint.scss`。透過這方式達到單獨使用斷點相關的 mixin。最小程度減輕編譯 css 的問題。


## 後話

在解決的過程中，也有看到[有人建議透過 `vue.config.js` 全面載入 custom.scss 的做法](https://stackoverflow.com/questions/71600556/how-to-prepend-scss-files-in-vue3-dart-sass)。

```js
module.exports = {
  css: {
    loaderOptions: {
      scss: {
        additionalData: `@import "./src/scss/custom.scss";`,
      },
    },
  },
};
```

但實際測試之後，發現大大的增加編譯的時間。（也許在小型專案不覺得就是...）最後個別的針對需求載入相關的 scss 似乎是當前是最理想的方式。
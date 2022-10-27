---
title: "gulp 使用說明"
date: 2022-10-27T17:12:36+08:00
draft: false
tags: ["網頁切版直播課程"]
type: post
author: "Steve Lin"
description: ""
---

# gulp 使用說明

### [gulp官方文件](https://gulpjs.com/docs/en/getting-started/quick-start)

### gulpfile
- gulpfile就是gulpfile.js的簡稱，用gulp的指定的時後會自動載入
- 裡面會放相關api EX:src(), dest(), series(),parallel()
- 可用其他語言寫再轉譯
    -  TypeScript,命名gulpfile.ts跟安裝ts-node module.
    -  Babel,命名 gulpfile.babel.js跟安裝the @babel/register module.
    -  當檔案太大時可以寫在不同檔案，再import進來，資料夾名稱要命名成gulpfile.js(包含index.js)

### Gulp Task
> 非同步的js function
- public tasks:從gulpfile輸出，可以用gulp指令啟動
- private tasks:for內部使用，常用的方法有`series()`、`parallel()`
 外部使用者不能獨立使用
 ```javascript
 const { series } = require('gulp');

// The `clean` function is not exported so it can be considered a private task.
// It can still be used within the `series()` composition.
function clean(cb) {
  // body omitted
  cb();
}

// The `build` function is exported so it is public and can be run with the `gulp` command.
// It can also be used within the `series()` composition.
function build(cb) {
  // body omitted
  cb();
}

exports.build = build;
exports.default = series(clean, build);
 ```
 ![](https://gulpjs.com/img/docs-gulp-tasks-command.png)
 
 ### compose tasks
 `series()`跟`parallel()`可以讓個別tasks組合成一個，兩個都可以接受多個task方法，或是包成巢狀
 - series():讓tasks照順序執行
 - parallel():同時執行tasks
 ### 跟檔案互動
 > `src()`、`dest()`可以跟檔案系統互動
 - src:檔案來源位置
 - desc:檔案輸出位置
 - pipe:檔案串流方法
 ```javascript
 const { src, dest } = require('gulp');

exports.default = function() {
  return src('src/*.js')
    .pipe(dest('output/'));
}
```
### Globs
> glob 是由普通字符和 / 或通配字符組成的字符串，用于匹配文件路徑。可以利用一個或多個 glob 在文件系統中定位文件。
- 主要用來表示檔案路徑
- `src()`會用到至少一個glob去決定要用哪個檔案，或是可以傳globs array去匹配符合的(依照順序)
- 用`\`去分割字串，`\\`是溢出字元寫法
- 避免使用Node.js的`path.join`、`__dirname`、`__filename`、`process.cwd()` 因為會使用`\\`去組路徑
- 在單一字串中匹配任意數量 * :`index.js`但無法匹配到`scripts/index.js` 、`scripts/nested/index.js`
- 在多個字串中匹配任意數量 **:常用在匹配目錄名稱，`'scripts/**/*.js'`
- 負的匹配(取消) !:一定要接在non-negative glob後面，前面會先把匹配的檔案找出來，後面設定取消可以排除特定路徑的
`['scripts/**/*.js', '!scripts/vendor/**']`
- 同一個src()的globs如果有重複會自動去除，但不會處理不同的src()

### 使用套件
- npm搜尋`gulpplugin`、`gulpfriendly`
- [ plugin search page](https://gulpjs.com/plugins/)

### Watch files
- 會監控globs跟tasks，一旦有改變就會觸發
- 監控的不能是同步
- watcher會在tasks建立、改變或刪除的時候執行
- 也可以使用`events`來指定動作`add`, `addDir`, `change`, `unlink`, `unlinkDir`, `ready`, `error`
```javascript
const { watch } = require('gulp');

exports.default = function() {
  // All events will be watched
  watch('src/*.js', { events: 'all' }, function(cb) {
    // body omitted
    cb();
  });
};
```
- 一開始建立的時候不會執行直到有發生改變，或是可以用`ignoreInitial`指定第一次就要觸發
```javascript
const { watch } = require('gulp');

exports.default = function() {
  // The task will be executed upon startup
  watch('src/*.js', { ignoreInitial: false }, function(cb) {
    // body omitted
    cb();
  });
};
```
- 同時只能處理一個task，如果有另一個改變觸發，會被放到queue裡面(可以關掉)
- 延遲設定:task會過200ms才被觸發，可以調整秒數

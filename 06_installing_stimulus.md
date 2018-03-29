---
slug: /installing
---

# 安裝 Stimulus
# Installing Stimulus in Your Application
{:.no_toc}

為了在我們的應用程式安裝 Stimulus，請先加入 [`stimulus` npm package](https://www.npmjs.com/package/stimulus) 或使用 `<script>` 載入 [`stimulus.umd.js`](https://unpkg.com/stimulus/dist/stimulus.umd.js) 。

To install Stimulus in your application, add the [`stimulus` npm package](https://www.npmjs.com/package/stimulus) to your JavaScript bundle. Or, load [`stimulus.umd.js`](https://unpkg.com/stimulus/dist/stimulus.umd.js) in a `<script>` tag.

* TOC
{:toc}

## 使用 webpack
## Using webpack

Stimulus 整合了 [webpack](https://webpack.js.org/) 模組封裝工具（資源封裝）並使用它來達成自動載入我們專案中的 controller 檔案。

Stimulus integrates with the [webpack](https://webpack.js.org/) asset packager to automatically load controller files from a folder in your app.

具體來說是使用 webpack 的 [`require.context`](https://webpack.js.org/api/module-methods/#require-context) 設定載入 Stimulus controller 檔案所在的目錄，然後將 `definitionsFromContext` 處理 `context` 的結果傳入 `Application.load`：

> `require.context` 是 webpack 支援載入模組（載入欲編譯檔案）的一種方式，一般來說我們使用 `ES6`、`CommonJS`、`AMD` 語法讓 webpack 自動載入並編譯，除了上面這些寫在程式碼的語法外，webpack 也提供 `require.context` 這種方式讓我們加入模組。

Call webpack's [`require.context`](https://webpack.js.org/api/module-methods/#require-context) 
helper with the path to the folder containing your Stimulus controllers. Then, pass the resulting context to the `Application#load` method using the `definitionsFromContext` helper:

```js
// src/application.js
import { Application } from "stimulus"
import { definitionsFromContext } from "stimulus/webpack-helpers"

const application = Application.start()
const context = require.context("./controllers", true, /\.js$/)
application.load(definitionsFromContext(context))
```

### controller 檔案名稱與對應的識別名稱（identifier）
### Controller Filenames Map to Identifiers

遵循 controller 檔案的命名慣例 `[identifier]_controller.js` 便可以對應到 HTML 中的 `data-controller` 屬性。

Name your controller files `[identifier]_controller.js`, where `identifier` corresponds to each controller's `data-controller` identifier in your HTML.

Stimulus 的慣例是使用 `_` 底線來區分多個英文單字。controller 檔名的底線會在 `identifier` 轉換成 `-` 連字符號。例如：`content_loader_controller.js` 的識別名稱是 `content-loader`。

Stimulus conventionally separates multiple words in filenames using underscores. Each underscore in a controller's filename translates to a dash in its identifier.

我們也許想要使用子目錄作為命名空間，替 controller 分類。controller 路徑前面的 `/` 會變成 `--` 破折號。

You may also namespace your controllers using subfolders. Each forward slash in a namespaced controller file's path becomes two dashes in its identifier.

如果您偏好在檔名統一使用 `-` 而不是 `_` 。Stimulus 也支援，它們效果是一樣的。

If you prefer, you may use dashes instead of underscores anywhere in a controller's filename. Stimulus treats them identically.

檔名                               | 識別名稱
--------------------------------- | -----------------------
clipboard_controller.js           | clipboard
date_picker_controller.js         | date-picker
users/list_item_controller.js     | users\-\-list-item
local-time-controller.js          | local-time

## 其他建置工具
## Using Other Build Systems

Stimulus 也支援其他建置工具，但不支援自動載入 controller，您必須要自己手動載入並註冊 controller。

Stimulus works with other build systems, too, but without support for controller autoloading. Instead, you must explicitly load and register controller files with your application instance:

```js
// src/application.js
import { Application } from "stimulus"

import HelloController from "./controllers/hello_controller"
import ClipboardController from "./controllers/clipboard_controller"

const application = Application.start()
application.register("hello", HelloController)
application.register("clipboard", ClipboardController)
```
## 使用 Babel
## Using Babel

如果您有使用 [Babel](https://babeljs.io/)。注意，需要安裝 [transform-class-properties plugin](https://babeljs.io/docs/plugins/transform-class-properties/) 並加入設定：


If you're using [Babel](https://babeljs.io/) with your build system, you'll need to install the [transform-class-properties plugin](https://babeljs.io/docs/plugins/transform-class-properties/) and add it to your configuration:

```js
// .babelrc
{
  "presets": ["env"],
  "plugins": ["transform-class-properties"]
}
```

或者，使用已經包含 `transform-class-properties` 的 [stage-2 preset](https://babeljs.io/docs/plugins/preset-stage-2/)：


Or, use the [stage-2 preset](https://babeljs.io/docs/plugins/preset-stage-2/), which includes the transform-class-properties plugin and more:

```js
// .babelrc
{
  "presets": ["env", "stage-2"]
}
```

## 不使用建置工具
## Using Without a Build System

如果您並沒有使用任何建置工具，您也可以使用 `<script>` 直接載入 Stimulus，接著便可以在全域存取 `Stimulus` 物件（`window.Stimulus`）。

If you prefer not to use a build system, you can load Stimulus in a `<script>` tag and it will be globally available through the `window.Stimulus` object.

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <script src="https://unpkg.com/stimulus/dist/stimulus.umd.js"></script>
  <script>
    (function() {
      const application = Stimulus.Application.start()
      application.register("hello", class extends Stimulus.Controller {
        // ...
      })
    })()
  </script>
<head>
<body>
  <div data-controller="hello">...</div>
</body>
</html>
```

## 瀏覽器支援
## Browser Support

Stimulus 支援所有主流，最新版本，桌面和行動裝置上的瀏覽器。如果您的專案需要支援舊版的瀏覽器那麼請在載入 Stimulus `之前` 自行加入 `Array.from()`，`Element.closest()`、`Map`，`Object.assign()`，`Promise` 和 `Set` 的 `polyfills`。

Stimulus supports all evergreen, self-updating desktop and mobile browsers out of the box. If your application needs to support older browsers, include polyfills for `Array.from()`, `Element.closest()`, `Map`, `Object.assign()`, `Promise`, and `Set` before loading Stimulus.

為了支援 Internet Explorer 11+ 和 Safari 9+ 。Stimulus 也從 [core-js](https://www.npmjs.com/package/core-js) 和 [element-closest](https://www.npmjs.com/package/element-closest) 中使用了這些 [polyfills](https://github.com/stimulusjs/stimulus/blob/master/packages/%40stimulus/polyfills/index.js) 

Stimulus is known to support Internet Explorer 11+ and Safari 9+ using [these polyfills](https://github.com/stimulusjs/stimulus/blob/master/packages/%40stimulus/polyfills/index.js) from the [core-js](https://www.npmjs.com/package/core-js) and [element-closest](https://www.npmjs.com/package/element-closest) packages.

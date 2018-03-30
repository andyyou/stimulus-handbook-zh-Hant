# 建置實務範例

我們已經實作第一個 controller 也學習了關於 Stimulus 如何讓 HTML 和 JavaScript 繫結。現在讓我們參考 Basecamp 的一些功能來重新實作一個比較接近實際需求的 controller。

## DOM 剪貼簿 API

在 Basecamp 的使用介面上，佈局看起來像下圖：

<img src="assets/bc3-clipboard-ui.png" width="500" height="122" class="handbook__screenshot">

當您點擊上圖中的 `Copy` 按鈕時，Basecamp 會複製隔壁 Email 的文字到剪貼簿上。

其中 Web 有提供關於[存取作業系統剪貼簿的 API](https://www.w3.org/TR/clipboard-apis/)，不過並沒有 HTML 標籤可以直接完成這個功能因此我們需要透過按鈕等元素搭配 JavaScript 來實作。

## 實作複製按鈕

假設我們的 App 通過存取 PIN 碼來給使用者授權。如果在顯示 PIN 的介面旁邊提供一個複製的按鈕，那麼就可以方便使用者分享。

開啟 `public/index.html` 使用下面的程式碼取代 `<body>` 中的內容：

```html
<div>
  PIN: <input type="text" value="1234" readonly>
  <button>Copy to Clipboard</button>
</div>
```

## 建立 controller

接著，建立 `src/controllers/clipboard_controller.js` 並加入 `copy()` 方法：

```js
// src/controllers/clipboard_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  copy() {
  }
}
```

然後在我們 `index.html` 最外層的 `<div>` 加入 `data-controller="clipboard"`。任何時間點只要這個屬性出現在元素上，Stimulus 就會把 controller 的物件實例和元素連結起來。

```html
<div data-controller="clipboard">
```

## 定義目標元素

我們接著需要設定文字欄位 input 的參考，以便我們在使用 clipboard API 之前先取得其內容（我們的 PIN 碼）。讓我們加入 `data-target="clipboard.source"` 到 input 上：

```html
  PIN: <input data-target="clipboard.source" type="text" value="1234" readonly>
```

HTML 的屬性加好之後我們需要在 controller 中定義 target 如此以來我們就可以使用 `this.sourceTarget` 來取得元素：

```js
export default class extends Controller {
  static targets = [ "source" ]

  // ...
}
```

> ### `static targets` 這行程式是表示什麼意思？
>
> 流程是這樣的，當 Stimulus 載入我們的 controller class 的時候，它會去搜尋 targets 這個陣列裡面的字串， 這些字串就是 target 名稱。每一個在 `targets` 陣列中的 target 名稱，Stimulus 會在該 controller 中加入三個新的屬性，如上面的範例我們的 `"source"` target 名稱就會產生下面這些屬性：
>
> * `this.sourceTarget` 等於在 controller 影響的範圍內找到符合 `source` 的第一個元素，如果沒有任何元素符合，當我們存取屬性會產生錯誤。
> * `this.sourceTargets` 所有在 controller 範圍內符合 `source` 的元素
> * `this.hasSourceTarget` 判斷是否有符合 `source` 元素，如果有就是 `true` ，沒有則是 `false`

## 綁定 Action

現在我們可以來替複製功能的按鈕設定事件行為。

我們希望當按鈕被點擊的時候可以呼叫 controller 裡面的 `copy()` 方法，讓我們在 `<button>` 加上 `data-action="clipboard#copy"`：

```html
  <button data-action="clipboard#copy">Copy to Clipboard</button>
```

> ### 預設事件可省略
>
> 您可能注意到我們在 action 設定上省略了 `click->`，這是因為 Stimulus 定義了 `click` 是 `<button>` 的預設事件。
>
> 其他特定的元素也有預設事件，下列是完整的列表：
>
> 元素               | 預設事件
> ----------------- | -------------
> a                 | click
> button            | click
> form              | submit
> input             | change
> input type=submit | click
> select            | change
> textarea          | change

最後在我們的 `copy()` 方法，我們選取 input 的內容然後呼叫剪貼簿的 API：

```js
  copy() {
    this.sourceTarget.select()
    document.execCommand("copy")
  }
```

重新載入頁面然後點擊複製按鈕。接著切回我們的文字編輯器`貼上`。您應該可以看到貼上的內容是我們的 PIN 碼 `1234`。

## 設計彈性的使用者介面

雖然剪貼簿 API 在[主流瀏覽器的最新版本上支援度上已經完整了](https://caniuse.com/#feat=clipboard)，不過還是有些使用者使用的是舊版的瀏覽器。

我們也應該思考那些可能會產生的其他問題。例如：網路間歇中斷或 CDN 有設定存取權限導致 JavaScript 無法下載。

認為不應該浪費時間，直接停止支援舊版的瀏覽器，又或者像網路的連線問題直接不處理，認為應該讓使用者自己解決問題，都是開發者很容易採取的作法。但通常我們可以採取些比較彈性的作法來解決這些問題。

這種所謂彈性的作法即大家所知道的`漸進增強`，是一種 Web 界面的實踐，我們將基本的功能使用 HTML 和 CSS 完成，然後當一些新版瀏覽器的功能被支援時再透過 JavaScript 和 CSS 提升使用者經驗，即逐步的增加一些功能。

## 漸進加強 PIN 欄位

讓我們來看看能如何在 PIN 欄位的功能上使用漸進增強的作法，我們可以讓複製按鈕在不支援剪貼簿 API 的瀏覽器隱藏起來。如此我們就可以避免顯示一個不具備功能的按鈕。

我們從使用 CSS 隱藏按鈕開始。然後在 controller 檢查功能是否支援。如果 API 支援的話，我們在加入 class 到 controller 的元素來使按鈕出現。

首先我們在 `<button>` 加入 `class="clipboard-button"`

```html
  <button data-action="clipboard#copy" class="clipboard-button">Copy to Clipboard</button>
```

接著加入下面的 CSS 樣式到 `public/main.css`：

```css
.clipboard-button {
  display: none;
}

.clipboard--supported .clipboard-button {
  display: initial;
}
```

現在我們需要在 controller 實作 `connect()` 方法，在這個地方判斷是否要在 controller 元素加入 class

> `this.element` 即我們所謂的 controller 元素（controller 根元素）。


```js
  connect() {
    if (document.queryCommandSupported("copy")) {
      this.element.classList.add("clipboard--supported")
    }
  }
```

您可以關閉瀏覽器 JavaScript 的功能呢，重新載入頁面，注意到複製按鈕就不再出現了。

到這一步我們已經漸進增強了 PIN 欄位的部分：複製按鈕會依據瀏覽器是否支援功能出現或隱藏。

## Stimulus controller 重複使用性

到目前為止，我們已經看到了當頁面上有一個 controller 的物件實例時會發生什麼事。

而在同一個頁面上同時出現多個 controller 物件實例的狀況並不罕見。舉例來說我們可能想要顯示一個 PIN 的列表，每一個 PIN 欄位都有一個自己的複製按鈕。

controller 是可重複使用的：任何時候只要我們想要剪貼簿複製的功能，我們要做的就是在頁面上正確的繫結對應的 `data-*` 屬性。

讓我們增加另外的 PIN 功能到這個頁面上。複製 `<div>` 片段在下方貼上，然後修改其 input 中的 `value` 值

```html
<div data-controller="clipboard">
  PIN: <input data-target="clipboard.source" type="text" value="3737" readonly>
  <button data-action="clipboard#copy" class="clipboard-button">Copy to Clipboard</button>
</div>
```

重新載入頁面確認兩個按鈕都可以正常運作。

## Actions 和 Targets 可以套用在任何元素上

接下來我們增加更多的 PIN 欄位，這一次我們改用超連結 `<a>` 來取代按鈕：

```html
<div data-controller="clipboard">
  PIN: <input data-target="clipboard.source" type="text" value="3737" readonly>
  <a href="#" data-action="clipboard#copy" class="clipboard-button">Copy to Clipboard</a>
</div>
```

Stimulus 允許我們使用任意類型的元素只要 `data-action` 設定正確。如該元素支援設定的事件。

不過注意：點擊超連結瀏覽器預設行為會跳轉到超連結的 `href`。我們需要使用 `event.preventDefautl()` 取消預設行為。

```js
  copy(event) {
    event.preventDefault()
    this.sourceTarget.select()
    document.execCommand("copy")
  }
```

同樣的我們的 `source` 也不一定要是一個 `<input type="text">`。controller 只預期該元素有 `value` 屬性和 `select()` 方法。這意味著我們可以使用 `<textarea>` 來取代：

```html
  PIN: <textarea data-target="clipboard.source" readonly>3737</textarea>
```

## 總結與下一步

在這個章節我們看完了真實應用的範例。我們採用了漸進增強的作法讓我們的 controller 更具彈性可以支援舊版的瀏覽器抑或處理網路不佳的狀況。我們理解了複數 controller 物件實例的狀況，最後明白了 actions 和 targets 是如何和元素繫結以及限制。

下一章，我們將要學習 Stimulus 如何管理狀態。
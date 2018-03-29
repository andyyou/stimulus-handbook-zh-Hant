# 狀態管理

目前多數主流的前端框架主張我們把狀態全部交給 JavaScript 來處理。將 DOM 單純用來渲染輸出，由客戶端的樣板搭配伺服器端來的 JSON 去產生最後的結果。

Stimulus 採取不同的方式。一個 Stimulus 應用程式的狀態基本上會存在 DOM 的屬性上；controller 基本上不處理狀態。這樣的作法讓我們可以和各式各樣的 HTML 搭配，例如：一般初始化的頁面，Ajax 請求，Turbolinks 的 visit 機制，甚至其他的 JavaScript 函式庫。
套用的 controller 會自動建立而不需要其他初始化的步驟。

## 建立幻燈片

在上一章，我們已經看過了 Stimulus controller 如何在 HTML 中通過加入一個 class 樣式名稱來達成一個簡單的狀態管理。但如果我們需要的是儲存一個較為複雜的值，而不是簡單的註記？

我們將透過建置一個 slideshow controller 讓它能夠在 DOM 屬性上儲存當前選擇的 slide 的索引來探討這個問題。

一如往常我們從 HTML 開始著手：

```html
<div data-controller="slideshow">
  <button data-action="slideshow#previous">←</button>
  <button data-action="slideshow#next">→</button>

  <div data-target="slideshow.slide" class="slide">🐵</div>
  <div data-target="slideshow.slide" class="slide">🙈</div>
  <div data-target="slideshow.slide" class="slide">🙉</div>
  <div data-target="slideshow.slide" class="slide">🙊</div>
</div>
```

每一個 `slide` target 代表一系列投影片中的一頁。我們的 controller 將會負責處理一次只顯示一頁內容。

一開始我們可以使用 CSS 來把所有投影片隱藏起來，只有當 `slide--current` 套用在元素上的時候才顯示：

```css
.slide {
  display: none;
}

.slide.slide--current {
  display: block;
}
```

現在，我們可以開始來建立我們的 controller。開新檔案 `src/controllers/slideshow_controller.js`

```js
// src/controllers/slideshow_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "slide" ]

  initialize() {
    this.showSlide(0)
  }

  next() {
    this.showSlide(this.index + 1)
  }

  previous() {
    this.showSlide(this.index - 1)
  }

  showSlide(index) {
    this.index = index
    this.slideTargets.forEach((el, i) => {
      el.classList.toggle("slide--current", index == i)
    })
  }
}
```

我們在 controller 定義了一個方法 `showSlide()`，它會執行一次迴圈遍歷每一個投影片，如果切換的索引 `index` 和遍歷的索引 `i` 一致就把 `slide--current` class 樣式加上。

還有，我們在 controller 初始化的時候設定顯示第一個投影片，然後 `next()` 和 `previous()` action 則處理下一頁或上一頁的行為。

> ### 生命週期回調（callback）
>
> 在這個範例中我們使用 `initialize()` 是什麼？這個和我們之前提到的 `connect()` 有什麼不同呢？
>
> 在 Stimulus 中 controller 從初始化到解構有其生命週期，每個階段有對應的生命週期回調，這些方法通常可以協助我們設定狀態或解除繫結。
> 
> 回調        | 由 Stimulus 自動調用
> ---------- | --------------------
> initialize | 僅執行一次，當 controller 建立物件實例的時候
> connect    | 當 controller 和 DOM 完成繫結的時候
> disconnect | 當 controller 和 DOM 解除繫結的時候

重新載入頁面測試我們的上下頁按鈕可以正常運作。

## 從 DOM 讀取初始化狀態

注意到 controller 是如何使用 `this.index` 追蹤被選取顯示的投影片。

現在假設我們想要從第二則內容開始顯示，我們該如何在 HTML 提供起始投影片的索引設定呢？

其中一種載入起始索引的方式是在 HTML 使用 `data` 屬性。例如：我們可以加入 `data-slideshow-index` 屬性到 controller 的根元素上：

```html
<div data-controller="slideshow" data-slideshow-index="1">
```

然後在 `initialize()` 方法中，我們就可以讀取 DOM 的屬性，然後將值傳入 `showSlide()`：

```js
  initialize() {
    const index = parseInt(this.element.getAttribute("data-slideshow-index"))
    this.showSlide(index)
  }
```

在 controller 的根元素上使用 `data` 屬性來帶入參數是很常見的狀況，因此 Stimulus 提供了一個 API 方便我們讀取屬性值，我們可以使用 `this.data.get()`：

```js
  initialize() {
    const index = parseInt(this.data.get("index"))
    this.showSlide(index)
  }
```

> ### Data API 說明
>
> 每一個 Stimulus controller 都有一個 `this.data` 物件，其中有 `has()`，`get()`，`set()` 方法。這些方法提供一種較方便的方式讓我們存取 controller 根元素上的屬性並且需要搭配 controller 的識別名稱作為限制範圍。
>
> 例如我們上面的 controller：
> * `this.data.has('index')` 假如 controller 根元素有 `data-slideshow-index` 屬性的話，會回傳 `true`
> * `this.data.get('index')` 取得根元素上 `data-slideshow-index` 屬性值
> * `this.data.set('index', index)` 將 `index` 的值設定到根元素的 `data-slideshow-index` 屬性上
>
> 如果屬性的名稱超過一個單字，那麼在 JavaScript 中會使用 `camelCase` 格式，HTML 中則是 `attribute-name` 格式。例如：`data-slideshow-current-class-name` 屬性則是 `this.data.get('currentClassName')`

讓我們在 controller 根元素加入 `data-slideshow-index` 屬性，確認 controller 的程式碼也如上面更新了。然後重新載入頁面測試功能。

## 於 DOM 管理狀態的一致性

我們已經理解如何使用 `data` 屬性來帶入參數，調整 controller 的初始化行為。

不過當我們使用 slideshow 切換頁面的時候，屬性並沒有和 controller 中的 `index` 同步。假設我們要複製這個 controller 元素，這個副本將會恢復到初始化的狀態。

針對這個狀況我們可以在 controller 中透過為 `index` 定義 getter 和 setter 並搭配 Data API 解決這個問題：

```js
// src/controllers/slideshow_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "slide" ]

  initialize() {
    this.showCurrentSlide()
  }

  next() {
    this.index++
  }

  previous() {
    this.index--
  }

  showCurrentSlide() {
    this.slideTargets.forEach((el, i) => {
      el.classList.toggle("slide--current", this.index == i)
    })
  }

  get index() {
    return parseInt(this.data.get("index"))
  }

  set index(value) {
    this.data.set("index", value)
    this.showCurrentSlide()
  }
}
```

上面我們把 `showSlide()` 改成 `showCurrentSlide()` 然後不再帶參數，直接使用 `this.index`。因為 getter `get index()` 的關係，現在我們存取 `this.index` 會回傳 `data-slideshow-index` 屬性並轉換成 integer。
而 setter `set index()` 則會在我們設定 `this.index` 的時候同步更新頁面屬性。

現在我們的 controller 狀態也完整的保存在 DOM 裡了。

> 譯者註：如果您想試試 clone 的效果可以使用下面這段範例：

```js
// src/controllers/slideshow_controller.js
clone (e) {
  var el = document.querySelector('[data-controller=slideshow]').cloneNode(true)
  document.body.appendChild(el)
}
```

## 總結與下一步

在這一章我們理解了如何使用 Stimulus 的 Data API 來處理 index 狀態。

從實務應用的角度，我們的 controller 還不是很完整。
思考一下我們該怎麼修正 controller 來處理下面列出的問題：

* 當索引處在第一筆內容的時候，上一頁在內部會把索引值從 `0` 扣成 `-1`。我們能不能讓索引變成顯示最後一頁的值？（下一頁有類似的問題）
* 如果我們忘記設定 `data-slideshow-index` 那麼 `get index()` 裡面的 `parseInt()` 會傳回 `NaN`。我們是不是可以讓忘記設定的時候預設帶 `0`？

下一章我們將要看看如何在 controller 處理外部資源，如 HTTP request 和 timer 該如何處理。
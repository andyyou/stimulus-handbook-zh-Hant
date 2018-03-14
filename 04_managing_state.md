---
slug: /managing-state
---

# 狀態管理
# Managing State

目前多數主流的前端框架主張我們把狀態全部交給 JavaScript 來處理。將 DOM 單純用來渲染輸出，由客戶端的樣板搭配伺服器端來的 JSON 去產生最後的結果。

Most contemporary frameworks encourage you to keep state in JavaScript at all times. They treat the DOM as a write-only rendering target, reconciled by client-side templates consuming JSON from the server.

Stimulus 採取不同的方式。一個 Stimulus 應用程式的狀態基本上會存在 DOM 的屬性上；controller 基本上不處理狀態。這樣的作法讓我們可以和各式各樣的 HTML 搭配，例如：一般初始化的頁面，Ajax 請求，Turbolinks 的 visit 機制，甚至其他的 JavaScript 函式庫。
套用的 controller 會自動建立而不需要其他初始化的步驟。

Stimulus takes a different approach. A Stimulus application's state lives as attributes in the DOM; controllers themselves are largely stateless. This approach makes it possible to work with HTML from anywhere—the initial document, an Ajax request, a Turbolinks visit, or even another JavaScript library—and have associated controllers spring to life automatically without any explicit initialization step.

## 建立幻燈片
## Building a Slideshow

在上一章，我們已經看過了 Stimulus controller 如何在 HTML 中通過加入一個 class 樣式名稱來達成一個簡單的狀態管理。但如果我們需要的是儲存一個較為複雜的值，而不是簡單的註記？

In the last chapter, we learned how a Stimulus controller can maintain simple state in the document by adding a class name to an element. But what do we do when we need to store a value, not just a simple flag?

我們將透過建置一個 slideshow controller 讓它能夠在 DOM 屬性上儲存當前選擇的 slide 的索引來探討這個問題。

We'll investigate this question by building a slideshow controller which keeps its currently selected slide index in an attribute.

一如往常我們從 HTML 開始著手：

As usual, we'll begin with HTML:

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

Each `slide` target represents a single slide in the slideshow. Our controller will be responsible for making sure only one slide is visible at a time.

一開始我們可以使用 CSS 來把所有投影片隱藏起來，只有當 `slide--current` 套用在元素上的時候才顯示：

We can use CSS to hide all slides by default, only showing them when the `slide--current` class is applied:

```css
.slide {
  display: none;
}

.slide.slide--current {
  display: block;
}
```

現在，我們可以開始來建立我們的 controller。開新檔案 `src/controllers/slideshow_controller.js`

Now let's draft our controller. Create a new file, `src/controllers/slideshow_controller.js`, as follows:

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

Our controller defines a method, `showSlide()`, which loops over each slide target, toggling the `slide--current` class if its index matches.

還有，我們在 controller 初始化的時候設定顯示第一個投影片，然後 `next()` 和 `previous()` action 則處理下一頁或上一頁的行為。

We initialize the controller by showing the first slide, and the `next()` and `previous()` action methods advance and rewind the current slide.

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

> ### Lifecycle Callbacks Explained
>
> What does the `initialize()` method do? How is it different from the `connect()` method we've used before?
>
> These are Stimulus _lifecycle callback_ methods, and they're useful for setting up or tearing down associated state when your controller enters or leaves the document.
>
> Callback   | Invoked by Stimulus…
> ---------- | --------------------
> initialize | once, when the controller is first instantiated
> connect    | anytime the controller is connected to the DOM
> disconnect | anytime the controller is disconnected from the DOM

重新載入頁面測試我們的上下頁按鈕可以正常運作。

Reload the page and confirm that the Next button advances to the next slide.

## 從 DOM 讀取初始化狀態
## Reading Initial State from the DOM

注意到 controller 是如何使用 `this.index` 追蹤被選取顯示的投影片。

Notice how our controller tracks its state—the currently selected slide—in the `this.index` property.

現在假設我們想要從第二則內容開始顯示，我們該如何在 HTML 提供起始投影片的索引設定呢？

Now say we'd like to start one of our slideshows with the second slide visible instead of the first. How can we encode the start index in our markup?

其中一種載入起始索引的方式是在 HTML 使用 `data` 屬性。例如：我們可以加入 `data-slideshow-index` 屬性到 controller 的根元素上：

One way might be to load the initial index with an HTML `data` attribute. For example, we could add a `data-slideshow-index` attribute to the controller's element:

```html
<div data-controller="slideshow" data-slideshow-index="1">
```

然後
Then, in our `initialize()` method, we could read that attribute, convert it to an integer, and pass it to `showSlide()`:

```js
  initialize() {
    const index = parseInt(this.element.getAttribute("data-slideshow-index"))
    this.showSlide(index)
  }
```

在 controller 的根元素上使用 `data` 屬性來帶入參數是很常見的狀況，因此 Stimulus 提供了一個 API 方便我們讀取屬性值，我們可以使用 `this.data.get()`：

Working with `data` attributes on controller elements is common enough that Stimulus provides an API for it. Instead of reading the attribute value directly, we can use the more convenient `this.data.get()` method:

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

> ### The Data API Explained
>
> Each Stimulus controller has a `this.data` object with `has()`, `get()`, and `set()` methods. These methods provide convenient access to `data` attributes on the controller's element, scoped by the controller's identifier.
>
> For example, in our controller above:
> * `this.data.has("index")` returns `true` if the controller's element has a `data-slideshow-index` attribute
> * `this.data.get("index")` returns the string value of the element's `data-slideshow-index` attribute
> * `this.data.set("index", index)` sets the element's `data-slideshow-index` attribute to the string value of `index`
>
> If your attribute name consists of more than one word, reference it as `camelCase` in JavaScript and `attribute-case` in HTML. For example, you can read the `data-slideshow-current-class-name` attribute with `this.data.get("currentClassName")`.

讓我們在 controller 根元素加入 `data-slideshow-index` 屬性，確認 controller 的程式碼也如上面更新了。然後重新載入頁面測試功能。

Add the `data-slideshow-index` attribute to your controller's element, then reload the page to confirm the slideshow starts on the specified slide.

## 於 DOM 管理狀態的一致性
## Persisting State in the DOM

我們已經理解如何使用 `data` 屬性來帶入參數，調整 controller 的初始化行為。

We've seen how to bootstrap our slideshow controller's initial slide index by reading it from a `data` attribute.

不過當我們使用 slideshow 切換頁面的時候，屬性並沒有和 controller 中的 `index` 同步。假設我們要複製這個 controller 元素，這個副本將會恢復到初始化的狀態。

As we navigate through the slideshow, however, that attribute does not stay in sync with the controller's `index` property. If we were to clone the controller's element in the document, the clone's controller would revert back to its initial state.

針對這個狀況我們可以在 controller 中透過為 `index` 定義 getter 和 setter 並搭配 Data API 解決這個問題：

We can improve our controller by defining a getter and setter for the `index` property which delegates to the Data API:

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

Here we've renamed `showSlide()` to `showCurrentSlide()` and changed it to read from `this.index`. The `get index()` method returns the controller element's `data-slideshow-index` attribute as an integer. The `set index()` method sets that attribute and then refreshes the current slide.

現在我們的 controller 狀態也完整的保存在 DOM 裡了。

Now our controller's state lives entirely in the DOM.

> 譯者註：如果您想試試 clone 的效果可以使用下面這段範例：

```js
// src/controllers/slideshow_controller.js
clone (e) {
  var el = document.querySelector('[data-controller=slideshow]').cloneNode(true)
  document.body.appendChild(el)
}
```

## Wrap-Up and Next Steps

在這一章我們理解了如何使用 Stimulus 的 Data API 來處理 index 狀態。

In this chapter we've seen how to use the Stimulus Data API to load and persist the current index of a slideshow controller.

從實務應用的角度，我們的 controller 還不是很完整。
思考一下我們該怎麼修正 controller 來處理下面列出的問題：

From a usability perspective, our controller is incomplete. Consider how you might revise the controller to address the following issues:

* 當索引處在第一筆內容的時候，上一頁在內部會把索引值從 `0` 扣成 `-1`。我們能不能讓索引變成顯示最後一頁的值？（下一頁有類似的問題）
* 如果我們忘記設定 `data-slideshow-index` 那麼 `get index()` 裡面的 `parseInt()` 會傳回 `NaN`。我們是不是可以讓忘記設定的時候預設帶 `0`？

* The Previous button appears to do nothing when you are looking at the first slide. Internally, the `index` value decrements from `0` to `-1`. Could we make the value wrap around to the _last_ slide index instead? (There's a similar problem with the Next button.)
* If we forget to specify the `data-slideshow-index` attribute, the `parseInt()` call in our `get index()` method will return `NaN`. Could we fall back to a default value of `0` in this case?

下一章我們將要看看如何在 controller 處理外部資源，如 HTTP request 和 timer 該如何處理。

Next we'll look at how to keep track of external resources, such as timers and HTTP requests, in Stimulus controllers.

---
slug: /hello-stimulus
---

# 哈囉，Stimulus
# Hello, Stimulus

學習 Stimulus 最好的方式便是試著寫一個簡單的 controller，本章將會引導我們完成這個簡單的範例。

The best way to learn how Stimulus works is to build a simple controller. This chapter will show you how.

## 前置作業
## Prerequisites

為了能夠跟著教學操作，我們需要事先複製[`stimulus-starter`](https://github.com/stimulusjs/stimulus-starter)範例專案，這是一個針對學習 Stimulus 預先設定好的空白專案。

To follow along, you'll need a running copy of the [`stimulus-starter`](https://github.com/stimulusjs/stimulus-starter) project, which is a preconfigured blank slate for exploring Stimulus.

這邊我們推薦您使用 [Glitch `stimulus-starter` 專案來試試](https://glitch.com/edit/#!/import/github/stimulusjs/stimulus-starter) 這樣您就可以直接在瀏覽器中測試而不要安裝一堆東西。

We recommend [remixing `stimulus-starter` on Glitch](https://glitch.com/edit/#!/import/github/stimulusjs/stimulus-starter) so you can work entirely in your browser without installing anything:

[![Remix on Glitch](https://cdn.glitch.com/2703baf2-b643-4da7-ab91-7ee2a2d00b5b%2Fremix-button.svg)](https://glitch.com/edit/#!/import/github/stimulusjs/stimulus-starter)

如果您比較傾向使用自己習慣的編輯器，您可以使用 git 下載 `stimulus-starter`:

Or, if you'd prefer to work from the comfort of your own text editor, you'll need to clone and set up `stimulus-starter`:

```
$ git clone https://github.com/stimulusjs/stimulus-starter.git
$ cd stimulus-starter
$ yarn install
$ yarn start
```

接著在瀏覽器造訪 http://localhost:9000/

Then visit http://localhost:9000/ in your browser.

(注意到 `stimulus-starter` 專案使用 [Yarn](https://yarnpkg.com/) 管理相依套件，所以請確保您安裝所有相關套件。)

(Note that the `stimulus-starter` project uses the [Yarn package manager](https://yarnpkg.com/) for dependency management, so make sure you have that installed first.)

## 一切從 HTML 開始
## It All Starts With HTML

讓我們從一個簡單的練習開始：一個文字輸入欄位 input 和一個按鈕。當我們點擊按鈕時，在 console 顯示 input 裡面的值。

Let's begin with a simple exercise: a text field with a button. When you click the button, we'll display the value of the text field in the console.

每一個 Stimulus 專案都是從 HTML 開始的，這個專案也不例外。開啟 `public/index.html` 然後加入下面的 HTML

Every Stimulus project starts with HTML, and this project is no exception. Open `public/index.html` and add the following markup just after the opening `<body>` tag:

```html
<div>
  <input type="text">
  <button>Greet</button>
</div>
```

瀏覽器重新載入頁面您應該要看到 input 和按鈕。

Reload the page in your browser and you should see the text field and button.


## controller 為 HTML 帶來生命
## Controllers Bring HTML to Life

Stimulus 的核心目標就是自動繫結 DOM 元素和 JavaScript 物件。這個物件我們稱為 `controller`。

At its core, Stimulus' purpose is to automatically connect DOM elements to JavaScript objects. Those objects are called _controllers_.

讓我們繼承內建的 `Controller` class 來建立第一個 controller。在 `src/controllers` 目錄下建立新檔 - `hello_controller.js`。（範例專案已建立）

Let's create our first controller by extending the framework's built-in `Controller` class. Create a new file named `hello_controller.js` in the `src/controllers/` folder. Then place the following code inside:

```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
}
```

## 連結 controller 和 DOM 元素
## Identifiers Link Controllers With the DOM

下一步，我們需要告訴 Stimulus 這個 controller 該如何和 HTML 連結。我們需要透過在 `<div>` 的 `data-controller` 屬性中設定對應的識別名稱。

Next, we need to tell Stimulus how this controller should be connected to our HTML. We do this by placing an _identifier_ in the `data-controller` attribute on our `<div>`:

```html
<div data-controller="hello">
  <input type="text">
  <button>Greet</button>
</div>
```

識別名稱的存在就是為了連結元素和 controller。在這個範例識別名稱是 `hello` 接著 Stimulus 就知道要建立一個 `hello_controller.js` 的物件實例。如果想理解更多關於 controller 是如何被自動載入可以參考[安裝教學]({% link _handbook/06_installing_stimulus.md %})。

Identifiers serve as the link between elements and controllers. In this case, the identifier `hello` tells Stimulus to create an instance of the controller class in `hello_controller.js`. You can learn more about how automatic controller loading works in the [Installation Guide]({% link _handbook/06_installing_stimulus.md %}).

## 檢查是否正常運作？
## Is This Thing On?

重新載入頁面我們不會看到任何變化。那我們要怎麼知道 controller 有正確運作呢？

Reload the page in your browser and you'll see that nothing has changed. How do we know whether our controller is working or not?

一個簡單的方式就是在我們的 controller 裡面加入 `connect()` 方法，這個方法會在 Stimulus 完成 controller 和 DOM 綁定的時候被呼叫。

One way is to put a log statement in the `connect()` method, which Stimulus calls each time a controller is connected to the document.

在 `hello_controller.js` 中，如下實作 `connect()` 方法：

Implement the `connect()` method in `hello_controller.js` as follows:

```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  connect() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```

打開開發者工具，再次重新載入頁面會看到 `Hello, Stimulus!` 後面接著 controller 連結的 `<div>`。

Reload the page again and open the developer console. You should see `Hello, Stimulus!` followed by a representation of our `<div>`.

## action 回應 DOM 事件
## Actions Respond to DOM Events

現在我們來看看如何讓我們點擊 "Greet" 按鈕時在 console 顯示訊息。

Now let's see how to change the code so our log message appears when we click the "Greet" button instead.

我們直接把 `connect()` 改成 `greet()`：

Start by renaming `connect()` to `greet()`:

```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  greet() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```

現在，我們想要在點擊按鈕觸發 `click` 事件時呼叫 `greet()`。要先提到的是在 Stimulus 裡，controller 的方法我們叫做 `action`。

We want to call the `greet()` method when the button's `click` event is triggered. In Stimulus, controller methods which handle events are called _action methods_.

為了將按鈕的 `click` 事件和我們的 action 方法綁定在一起，讓我們開啟 `public/index.html` 在按鈕上加入 `data-action` 屬性：

To connect our action method to the button's `click` event, open `public/index.html` and add a magic `data-action` attribute to the button:

```html
<div data-controller="hello">
  <input type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

> ### action 設定格式說明
> `data-action` 的值 `click->hello#greet` 稱為動作描述子（action descriptor）。其實就是一個設定格式。下面是格式的說明：
> * `click` 為綁定的事件
> * `hello` 為 controller 識別名稱
> * `greet` 為調用的方法（action）

> ### Action Descriptors Explained
>
> The `data-action` value `click->hello#greet` is called an _action descriptor_. This particular descriptor says:
> * `click` is the event name
> * `hello` is the controller identifier
> * `greet` is the name of the method to invoke

在瀏覽器中開啟開發者工具然後重新載入頁面，當我們點擊按 Greet 鈕時應該可以在 console 看到訊息。

Load the page in your browser and open the developer console. You should see the log message appear when you click the "Greet" button.

## target 為重要元素建立 controller 的參考屬性
## Targets Map Important Elements To Controller Properties

我們接著要讓 action 可以對我們在 input 輸入的人名來打招呼，這也是我們練習的最後一步。

We'll finish the exercise by changing our action to say hello to whatever name we've typed in the text field.

為了要達到上面的需求，controller 裡需要取得 input 元素的參考，這樣我們才能透過 `value` 來取得 input 裡面的值。

In order to do that, first we need a reference to the input element inside our controller. Then we can read the `value` property to get its contents.

Stimulus 提供一種方式讓我們可以把關鍵或需要操作的元素加入為 `targets`，那樣我們就可以非常輕易的在 controller 裡面操作它們。讓我們打開 `public/index.html` 在 input 上加入 `data-target` 屬性。

Stimulus lets us mark important elements as _targets_ so we can easily reference them in the controller through corresponding properties. Open `public/index.html` and add a magic `data-target` attribute to the input element:

```html
<div data-controller="hello">
  <input data-target="hello.name" type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

> ### target 設定格式說明
>
> 觀察到 `data-target` 的值為 `hello.name` 又稱為目標元素描述子（target descriptor）。其實就是一個設定格式。下面是格式的說明：
> * `hello` 為 controller 是識別名稱
> * `name` 為 target 名稱

> ### Target Descriptors Explained
>
> The `data-target` value `hello.name` is called a _target descriptor_. This particular descriptor says:
> * `hello` is the controller identifier
> * `name` is the target name

接著當我們把 `name` 加入到 controller 目標元素的定義列表中，Stimulus 會自動建立一個 `this.nameTarget` 屬性，這個屬性會參考到符合條件的第一個目標元素。我們就可以直接使用這個屬性來讀取元素的值，用它來產生我們的問候句。

When we add `name` to our controller's list of target definitions, Stimulus automatically creates a `this.nameTarget` property which returns the first matching target element. We can use this property to read the element's `value` and build our greeting string.

開啟 `hello_controller.js` 如下更新程式：

Let's try it out. Open `hello_controller.js` and update it like so:

```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "name" ]

  greet() {
    const element = this.nameTarget
    const name = element.value
    console.log(`Hello, ${name}!`)
  }
}
```

然後在瀏覽器重新載入頁面，在 input 中輸入名字，點擊 Greet 按鈕。我們應該可以在 console 看到輸出結果。

Then reload the page in your browser and open the developer console. Enter your name in the input field and click the "Greet" button. Hello, world!

> 譯者註：Stimulus 會觀察頁面，搜尋 `data-*` 找到符合格式設定的 controller, action, target 自動繫結。

## 重構 controller
## Controllers Simplify Refactoring

我們已經知道一個 Stimulus 的 controller 就是 JavaScript class 的物件實例，而裡面的方法或我們稱為 action 就是用來對應一些事件的處理函式（event handlers）。

We've seen that Stimulus controllers are instances of JavaScript classes whose methods can act as event handlers.

依據這樣的脈絡慣例，意味著我們也有了一套重構的規範。舉例來說，我們可以把 `greet()` 裡面的 `name` 抽出來：

That means we have an arsenal of standard refactoring techniques at our disposal. For example, we can clean up our `greet()` method by extracting a `name` getter:

```js
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  static targets = [ "name" ]

  greet() {
    console.log(`Hello, ${this.name}!`)
  }

  get name() {
    return this.nameTarget.value
  }
}
```

## 總結與下一步
## Wrap-Up and Next Steps

恭喜！您已經完成了您的第一個 Stimulus controller！

Congratulations—you've just written your first Stimulus controller!

我們已經完整的概覽整個框架的核心概念：controller，識別名稱，`data-*` 的用法與關係，action 還有 target。下一篇我們將看到如何使用這些概念建立一個實務上的案例，它是從 Basecamp 專案中取出來的小功能。

We've covered the framework's core concepts: controllers, identifiers, actions, and targets. In the next chapter, we'll see how to put those together to build a real-life controller taken right from Basecamp.

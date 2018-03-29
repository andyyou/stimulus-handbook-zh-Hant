# 處理外部資源

在上一章，我們學會了如何從 DOM 中讀取資料並透過 Data API 使 controller 內部的狀態和 DOM 維持一致。

不過有時候我們的 controller 需要追蹤來自外部的資料，這裡我們說的`外部`指的是任何只要不是存在 DOM 或者 Stimulus 中的資料。舉例來說我們可能會需要發出一個 HTTP 請求並且在請求狀態發生變化時執行對應的處理。又或者我們想要使用 timer 持續變動狀態，但當 controller 不在繫結時要終止 timer。在這一章我們將學習如何處理這兩種狀況。

## 非同步載入 HTML

讓我們學習如何非同步的下載並填入局部的 HTML 片段。我們在 Basecamp 中使用了這種技術來讓載入頁面的速度加快，同時 views 盡可能不包含會根據使用者不同而有差異的內容，如此可以讓暫存更有效率。  

我們將會先建立一個通用的 `content-loader` controller ，它能夠將從遠端伺服器載回來的 HTML 放置到元素裡。然後我們將用它來實作載入一個未讀訊息的清單，就像我們在電子郵件的收件匣一樣。

同樣的，我們從 `public/index.html` 來開始這個收件匣的功能：

```html
<div data-controller="content-loader"
     data-content-loader-url="/messages.html"></div>
```

接著，新增檔案 `public/messages.html` 並加入我們訊息列表的 HTML 片段：

```html
<ol>
  <li>New Message: Stimulus Launch Party</li>
  <li>Overdue: Finish Stimulus 1.0</li>
</ol>
```

(在實務上我們通常會動態從伺服器取得 HTML，但這邊我們單純只是要示範如何加入 HTML 所以我們只用靜態檔案即可。)

現在我們可以開始實作 controller：

```js
// src/controllers/content_loader_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  connect() {
    this.load()
  }

  load() {
    fetch(this.data.get("url"))
      .then(response => response.text())
      .then(html => {
        this.element.innerHTML = html
      })
  }
}
```

當 controller 和 DOM 繫結 `connect()` 被調用，我們即開始使用 [Fetch] (https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) 讀取在 `data-content-loader-url` 屬性設定的 URL 。然後我們將讀取的 HTML 換到 根元素的 `innerHTML` 屬性中。

開啟瀏覽器開發者工具中的 `Network` 網路分頁，然後重新載入。我們會看到在 `index.html` 請求載入之後會有另外一個 `messages.html` 的請求跟在後面。

## 使用 Timer 自動更新

接著讓我改良我們的 controller 讓它可以定期更新。

我們將增加一個 `data-content-loader-refresh-interval` 的屬性用來設定更新的週期時間，單位是毫秒。

```html
<div data-controller="content-loader"
     data-content-loader-url="/messages.html"
     data-content-loader-refresh-interval="5000"></div>
```

然後修改 controller ，我們先確認是否有 `interval` 屬性的設定，如果有，就啟動一個 timer。

```js
  connect() {
    this.load()

    if (this.data.has("refreshInterval")) {
      this.startRefreshing()
    }
  }

  startRefreshing() {
    setInterval(() => {
      this.load()
    }, this.data.get("refreshInterval"))
  }
}
```

重新載入之後我們可以來觀察是否每 5 秒就會發出一個新的 request。我們可以修改 `public/messages.html` 看看是否有更新內容。

## 釋放資源

我們在 controller 連線之後啟動了 timer，但我們完全沒有停止它的機制。這意味著一旦我們 controller 掛載的元素被移除之後，HTTP request 依舊會在背景執行。

我們可以在 `startRefreshing()` 方法裡面紀錄一個 timer 的參考（變數），然後在 `disconnect()` 時取消 timer 。

```js
  disconnect() {
    this.stopRefreshing()
  }

  startRefreshing() {
    this.refreshTimer = setInterval(() => {
      this.load()
    }, this.data.get("refreshInterval"))
  }

  stopRefreshing() {
    if (this.refreshTimer) {
      clearInterval(this.refreshTimer)
    }
  }
}
```

現在我們可以確保 `content_loader_controller` 只會在繫結到元素上時才會發出請求。 

讓我們來看看最後完成的程式碼：

```js
// src/controllers/content_loader_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  connect() {
    this.load()

    if (this.data.has("refreshInterval")) {
      this.startRefreshing()
    }
  }

  disconnect() {
    this.stopRefreshing()
  }

  load() {
    fetch(this.data.get("url"))
      .then(response => response.text())
      .then(html => {
        this.element.innerHTML = html
      })
  }

  startRefreshing() {
    this.refreshTimer = setInterval(() => {
      this.load()
    }, this.data.get("refreshInterval"))
  }

  stopRefreshing() {
    if (this.refreshTimer) {
      clearInterval(this.refreshTimer)
    }
  }
}
```

## 總結與下一步

在這個章節我們我們學習了如何在 Stimulus 的生命週期中取得與釋放外部資源。

接著，我們將要來看看如何在我們的應用程式中設定 Stimulus。

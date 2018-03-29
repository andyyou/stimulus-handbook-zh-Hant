# Stimulus 源起

開發[Basecamp](https://basecamp.com)的時候我們使用了大量的   JavaScript，但我們不曾建立目前主流所謂的 `JavaScript 應用程式`。
我們所有的應用程式皆以伺服器端來渲染 HTML 為核心，然後透過 JavaScript 來點綴使其更加突出。

這就是所謂[Majestic Monolith](https://m.signalvnoise.com/the-majestic-monolith-29166d022228)的方式。Basecamp 可以在一半的平台上執行，其中包含原生的行動裝置應用程式，由 Ruby on Rails 所建置的集合，一系列 的controllers, views, models 。擁有單一的共享界面，可以在一個地方更新，這對於一個小團隊來說是非常關鍵的，即便我們支援了很多平台。

這使我們的效率依舊像過去一樣。回到一個程式開發者可以完成大量進度而不需要陷入層層間接或分散式系統的困惱。在所有人都認為將伺服器端程式限制在只負責產生 JSON ，然後基於 JavaScript 打造客戶端應用程式就是他們最好的選擇之前的那段時間。

這並不是說基於 JavaScript 和 JSON API 這種方式就沒有價值。只是一種通用於一般應用程式的作法，尤其是像 Basecamp 這類型的應用，這種方式讓整體回歸到簡單並具備生產力的狀態。

同時這也不表示單頁 JavaScript 應用程式沒有帶來任何益處。最主要的益處是從更新時需要完整頁面載入上解脫，得到更快的速度，更流暢的介面操作。

我們希望 Basecamp 也有那樣的感覺。就好像我們已經重寫了所有客戶端的程式或所有行動裝置的應用程式都用原生的方式。

這個願望引導我們找到一個二刀流的解決方案：[Turbolinks](https://github.com/turbolinks/turbolinks) 和 Stimulus。

### Turbolinks 負責全局，Stimulus 處理局部

在討論關於我們新的 JavaScript 框架 - Stimulus 之前，請容許我回顧一下 Turbolinks 的核心概念。

Turbolinks 是基於 Github 上一種稱為 [pjax](https://github.com/defunkt/jquery-pjax) 的方式開始的。基本概念維持不變。主要是因為全頁重新載入通常會覺得網頁變慢並不是因為瀏覽器必須要處理大量從伺服器傳來的 HTML。瀏覽器在處理 HTML 這方面實際上是非常快的。而且在大部分的情況下， HTML 資料比 JSON 資料大的這件事並不是主要的原因（尤其是有使用 gzip 的情況下）。真正造成緩慢的原因是 CSS 和 JavaScript 必須要重新初始化然後再套用到頁面。無論檔案是否已經被暫存了。如果您有大量的 CSS 和 JavaScript 這個處理過程可能會非常慢。

為了避免重新初始化這個步驟，Turbolinks 像單頁式應用程式一樣接管了整個程式操作，處理流程的狀態。只是大部分都是在背後處理。它攔截了所有的超連結然後使用 Ajax 取回新的頁面。伺服器依舊回傳完整的的 HTML。

單憑這個作法就可以讓大多數應用程式中的操作感到變快不少（如果搭配暫存，伺服器要在 100-200 毫秒回應是非常可能）。
以 Basecamp 來說，這個作法讓頁面切換之間提升了約 3 倍的速度。讓程式使用起來非常流暢，這也是單頁式應用程式最大的賣點。

不過 Turbolinks 只完成了故事的一半。處理全頁的部分。在單一頁面的局部更新方面還是不合格。例如顯示和隱藏元素，複製資料到剪貼簿，新增一個代辦事項到清單上以及其他互動操作。

在 Stimulus 之前，Basecamp 使用了些不同的方式來完成這些功能。有的使用 jQuery，有的使用原生的 JavaScript，還有採用物件導向風格建構的一套機制。但他們通常沒有明確的使用 `data-behavior` 屬性的方式來綁定事件。

雖然這樣很容易加入新的程式碼，但這並不是一個完整的解決方案，外加我們有太多共存的程式風格和設計模式。這讓我們很難去重複使用程式碼，也讓一些新進人員難以學習或遵循特定的方式開發。

### Stimulus 的三大核心概念

Stimulus 試圖將最佳的設計模式彙整成一個適中的小型框架，其核心主要圍繞著 3 個主要的概念： 控制器（controllers），執行動作（actions），還有元素對照（targets）。

> 註：為使讀者易於理解後續在 controller，action，target 的部分將使用英文。

採用漸進式增強的設計，當我們閱讀 HTML 時就可以明白一個樣版具備哪些行為。範例如下：

```html
<div data-controller="clipboard">
  PIN: <input data-target="clipboard.source" type="text" value="1234" readonly>
  <button data-action="clipboard#copy">Copy to Clipboard</button>
</div>
```

即便不懂 Stimulus 或者看過 controller 裡面的程式碼，我們還是可以輕易的理解。比起某個 HTML 片段搭配 JavaScript 的方式或許多主流的 JavaScript 框架，其仍保持著關注點分離的原則。

如你所見，Stimulus 並沒有改變建立 HTML 的過程。取而代之的，它是附加在原本的 HTML 之上。多數情況下，這些 HTML 是由伺服器端或透過 Ajax 請求處理而來的。

Stimulus 的重點在於操作既有的 HTML 文件。可以是添加用來隱藏或產生動畫的 CSS class，重新排列元素，又或者更新內容例如：轉換 UTC 時間為本地時間有可能被客戶端暫存，因此需要更新。

有些情況你可能希望 Stimulus 建立新的 DOM 元素，您完全可以這麼做。我們後續可能會針對這部分增加一些語法糖，讓開發起來變得容易些。不過目前來說重點是操作既有的元素而不是新增。

### Stimulus 與主流 JavaScript 框架的差異

上面闡述的設計導致 Stimulus 和目前主流的 JavaScript 框架產生蠻大的差異。大部分的主流框架主要都是利用 JSON 搭配某種樣板語言來產生元素。主要是使用 JSON 的資料帶入樣板中進而在一個空白頁面中渲染建立元素。

Stimulus 在處理狀態管理上也不太一樣。大部分的主流框架都支援狀態管理（使用 JavaScript 物件格式處理狀態資料），並且可以依據狀態去渲染 HTML。而 Stimulus 剛好相反。
狀態是被存在 HTML 之中，所以 controllers 會在頁面切換之間被刪除，但又像被暫存一樣使用 HTML 的暫存資料重新初始化。

這的確是個異常的典範。可以肯定許多用過主流框架的 JavaScript 資深開發者會嘲笑這樣的設計。沒關係，如果您對於使用像是 React + Redux 這類型的框架覺得滿意，那麼 Turbolinks + Stimulus 應該是不會吸引您。

另一方面，如果你正感覺到繁瑣，覺得增加的複雜度，分散式架構可能並不適用於你目前的工作。那麼你可能可以在我們的方法中找到解法。

### Stimulus 和相關的概念源自於仍未使用主流框架的專案

我們已經在 Basecamp 的幾個版本和其他應用程式中使用這種架構有一段時間了。Github 也使用類似的方式取得不錯的效果。這不僅僅是一種驗證過的替代解決方案，替代主流認知所謂的現代的 web 應用程式架構，還令人難以置信的是具備有力論證的一個。

事實上，這有點類似我們在 Basecamp 使用一些 [Ruby on Rails](https://rubyonrails.org/) 的技巧。
有種主流的方式並不是必要的錯覺，而且我們還可以完成更多，更快。

此外，您還不用做選擇。Stimulus 和 Turbolinks 可以和其他大型架構整合在一起。如果你專案中的 80% 並不是那麼複雜，可以考慮使用我們的二刀流。然後只針對那些需要的部分出動大型機具，如此才能真的受惠那些框架或函式庫。

在 Basecamp，我們確實也在某些需求上使用比較`重`的方式。例如日曆功能使用在客戶段渲染的方式。而我們的文字編輯器[Trix](https://trix-editor.org/)本身就具備完整的功能，這個時候其實就比較沒有使用 Stimulus 的必要。

這套框架主要在盡可能避免繁重的工作。為了能夠讓大量的介面互動維持在單純的 `請求-回應` 的模式，至於那些更複雜更頻繁的需求才使用才使用先進的工具。

更重要的是，這套工具主要是針對那些小團隊希望程式的操作流程可以更加流暢，穩定並逐步擴展團隊、導入主流的方式。

試試 Stimulus 吧。

---

David Heinemeier Hansson

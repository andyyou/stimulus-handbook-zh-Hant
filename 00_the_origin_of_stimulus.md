---
slug: /origin
---

# Stimulus 源起
# The Origin of Stimulus

在開發[Basecamp](https://basecamp.com)的時候我們使用了大量的 JavaScript，但我們不曾建立目前主流所謂的`JavaScript 應用程式`。
我們所有的應用程式皆以伺服器端來渲染產生 HTML 為核心，然後透過 JavaScript 來點綴使其更加突出。

We write a lot of JavaScript at [Basecamp](https://basecamp.com), but we don’t use it to create “JavaScript applications” in the contemporary sense. All our applications have server-side rendered HTML at their core, then add sprinkles of JavaScript to make them sparkle.

這就是所謂[Majestic Monolith](https://m.signalvnoise.com/the-majestic-monolith-29166d022228)的方式。Basecamp 可以在一半的平台上執行，其中包含原生的行動裝置應用程式，搭配一系列 controllers, views, models 由 Ruby on Rails 所建置的集合。擁有單一的共享界面，可以在一個地方更新，這對於一個小團隊來說是非常關鍵的，就算我們支援很多平台。

This is the way of the [majestic monolith](https://m.signalvnoise.com/the-majestic-monolith-29166d022228). Basecamp runs across half a dozen platforms, including native mobile apps, with a single set of controllers, views, and models created using Ruby on Rails. Having a single, shared interface that can be updated in a single place is key to being able to perform with a small team, despite the many platforms.

這使我們的效率依舊像過去一樣。回到一個程式開發者可以完成大量進度而不需要陷入層層間接或分散式系統的困惱。在所有人都認為將伺服器端程式限制在只負責產生 JSON ，然後基於 JavaScript 打造客戶端應用程式就是他們最好的選擇之前的那段時間。

It allows us to party with productivity like days of yore. A throwback to when a single programmer could make rapacious progress without getting stuck in layers of indirection or distributed systems. A time before everyone thought the holy grail was to confine their server-side application to producing JSON for a JavaScript-based client application.

這並不是說對於某些人或在某些時候，基於 JavaScript 和 JSON API 這種方式就沒有價值。只是一種通用於一般應用程式的作法，尤其是像 Basecamp 這類型的應用，這種方式讓整體回歸到簡單並具備生產力的狀態。

That’s not to say that there isn’t value in such an approach for some people, some of the time. Just that as a general approach to many applications, and certainly the likes of Basecamp, it’s a regression in overall simplicity and productivity.

同時這也不表示單頁 JavaScript 應用程式的流行沒有帶來任何益處。主要是從更新時需要完整頁面載入上解脫，得到更快的速度，更流暢的介面操作。

And it’s also not to say that the proliferation of single-page JavaScript applications hasn’t brought real benefits. Chief amongst which has been faster, more fluid interfaces set free from the full-page refresh.

我們希望 Basecamp 也有那樣的感覺。就好像我們已經重寫了所有客戶端的程式或所有行動裝置的應用程式都用原生的方式。

We wanted Basecamp to feel like that too. As though we had followed the herd and rewritten everything with client-side rendering or gone full-native on mobile.

這個願望引導我們找到一個二刀流的解決方案：[Turbolinks](https://github.com/turbolinks/turbolinks) 和 Stimulus。

This desire led us to a two-punch solution: [Turbolinks](https://github.com/turbolinks/turbolinks) and Stimulus.


### Turbolinks 負責全局，Stimulus 處理局部
### Turbolinks up high, Stimulus down low

在討論關於我們新的 JavaScript 框架 - Stimulus 之前，請容許我回顧一下 Turbolinks 的核心概念。

Before I get to Stimulus, our new modest JavaScript framework, allow me to recap the proposition of Turbolinks.

Turbolinks 是基於 Github 上一種稱為 [pjax](https://github.com/defunkt/jquery-pjax) 的方式開始的。基本概念維持不變。主要是因為全頁重新載入通常會覺得網頁變慢並不是因為瀏覽器必須要處理大量從伺服器傳來的 HTML。瀏覽器在處理 HTML 這方面實際上是非常快的。而且在大部分的情況下， HTML 資料比 JSON 資料大的這件事並不是主要的原因（尤其是有使用 gzip 的情況下）。真正造成緩慢的原因是 CSS 和 JavaScript 必須要重新初始化然後再套用到頁面。無論檔案是否已經被暫存了。如果您有大量的 CSS 和 JavaScript 這個處理過程可能會非常慢。

Turbolinks descends from an approach called [pjax](https://github.com/defunkt/jquery-pjax), developed at GitHub. The basic concept remains the same. The reason full-page refreshes often feel slow is not so much because the browser has to process a bunch of HTML sent from a server. Browsers are really good and really fast at that. And in most cases, the fact that an HTML payload tends to be larger than a JSON payload doesn’t matter either (especially with gzipping). No, the reason is that CSS and JavaScript has to be reinitialized and reapplied to the page again. Regardless of whether the files themselves are cached. This can be pretty slow if you have a fair amount of CSS and JavaScript.


為了避免重新初始化這個步驟，Turbolinks 像單頁式應用程式一樣接管了整個程式操作，處理流程的狀態。只是大部分都是在背後處理。它攔截了所有的超連結然後使用 Ajax 取回新的頁面。伺服器依舊回傳完整的的 HTML。

To get around this reinitialization, Turbolinks maintains a persistent process, just like single-page applications do. But largely an invisible one. It intercepts links and loads new pages via Ajax. The server still returns fully-formed HTML documents.

單憑這個作法就可以讓大多數應用程式中的操作感到變快不少（如果搭配暫存，伺服器要在 100-200 毫秒回應是非常可能）。
以 Basecamp 來說，這個作法讓頁面切換之間提升了約 3 倍的速度。讓程式使用起來非常流暢，這也是單頁式應用程式最大的買點。

This strategy alone can make most actions in most applications feel really fast (if they’re able to return server responses in 100-200ms, which is eminently possible with caching). For Basecamp, it sped up the page-to-page transition by ~3x. It gives the application that feel of responsiveness and fluidity that was a massive part of the appeal for single-page applications.


不過 Turbolinks 只完成了故事的一半。處理大範圍的部分。

But Turbolinks alone is only half the story. The coarsely grained one. Below the grade of a full page change lies all the fine-grained fidelity within a single page. The behavior that shows and hides elements, copies content to a clipboard, adds a new todo to a list, and all the other interactions we associate with a modern web application.

Prior to Stimulus, Basecamp used a smattering of different styles and patterns to apply these sprinkles. Some code was just a pinch of jQuery, some code was a similarly sized pinch of vanilla JavaScript, and some again was larger object-oriented subsystems. They all usually worked off explicit event handling hanging off a `data-behavior` attribute.

While it was easy to add new code like this, it wasn’t a comprehensive solution, and we had too many in-house styles and patterns coexisting. That made it hard to reuse code, and it made it hard for new developers to learn a consistent approach.

### The three core concepts in Stimulus

Stimulus rolls up the best of those patterns into a modest, small framework revolving around just three main concepts: Controllers, actions, and targets.

It’s designed to read as a progressive enhancement when you look at the HTML it’s addressing. Such that you can look at a single template and know which behavior is acting upon it. Here’s an example:

```html
<div data-controller="clipboard">
  PIN: <input data-target="clipboard.source" type="text" value="1234" readonly>
  <button data-action="clipboard#copy">Copy to Clipboard</button>
</div>
```

You can read that and have a pretty good idea of what’s going on. Even without knowing anything about Stimulus or looking at the controller code itself. It’s almost like pseudocode. That’s very different from reading a slice of HTML that has an external JavaScript file apply event handlers to it. It also maintains the separation of concerns that has been lost in many contemporary JavaScript frameworks.

As you can see, Stimulus doesn’t bother itself with creating the HTML. Rather, it attaches itself to an existing HTML document. The HTML is, in the majority of cases, rendered on the server either on the page load (first hit or via Turbolinks) or via an Ajax request that changes the DOM.

Stimulus is concerned with manipulating this existing HTML document. Sometimes that means adding a CSS class that hides an element or animates it or highlights it. Sometimes it means rearranging elements in groupings. Sometimes it means manipulating the content of an element, like when we transform UTC times that can be cached into local times that can be displayed.

There are cases where you’d want Stimulus to create new DOM elements, and you’re definitely free to do that. We might even add some sugar to make it easier in the future. But it’s the minority use case. The focus is on manipulating, not creating elements.


### How Stimulus differs from mainstream JavaScript frameworks

This makes Stimulus very different from the majority of contemporary JavaScript frameworks. Almost all are focused on turning JSON into DOM elements via a template language of some sort. Many use these frameworks to birth an empty page, which is then filled exclusively with elements created through this JSON-to-template rendering.

Stimulus also differs on the question of state. Most frameworks have ways of maintaining state within JavaScript objects, and then render HTML based on that state. Stimulus is the exact opposite. State is stored in the HTML, so that controllers can be discarded between page changes, but still reinitialize as they were when the cached HTML appears again.

It really is a remarkably different paradigm. One that I’m sure many veteran JavaScript developers who’ve been used to work with contemporary frameworks will scoff at. And hey, scoff away. If you’re happy with the complexity and effort it takes to maintain an application within the maelstrom of, say, React + Redux, then Turbolinks + Stimulus will not appeal to you.

If, on the other hand, you have nagging sense that what you’re working on does not warrant the intense complexity and application separation such contemporary techniques imply, then you’re likely to find refuge in our approach.


### Stimulus and related ideas were extracted from the wild

At Basecamp we’ve used this architecture across several different versions of Basecamp and other applications for years. GitHub has used a similar approach to great effect. This is not only a valid alternative to the mainstream understanding of what a “modern” web application looks like, it’s an incredibly compelling one.

In fact, it feels like the same kind of secret sauce we had at Basecamp when we developed [Ruby on Rails](https://rubyonrails.org/). The sense that contemporary mainstream approaches are needlessly convoluted, and that we can do more, faster, with far less.

Furthermore, you don’t even have to choose. Stimulus and Turbolinks work great in conjunction with other, heavier approaches. If 80% of your application does not warrant the big rig, consider using our two-pack punch for that. Then roll out the heavy machinery for the part of your application that can really benefit from it.

At Basecamp, we have and do use several heavier-duty approaches when the occasion calls for it. Our calendars tend to use client-side rendering. Our text editor is [Trix](https://trix-editor.org/), a fully formed text processor that wouldn’t make sense as a set of Stimulus controllers.

This set of alternative frameworks is about avoiding the heavy lifting as much as possible. To stay within the request-response paradigm for all the many, many interactions that work well with that simple model. Then reaching for the expensive tooling when there’s a call for peak fidelity.

Above all, it’s a toolkit for small teams who want to compete on fidelity and reach with much larger teams using more laborious, mainstream approaches.

Give it a go.

---

David Heinemeier Hansson
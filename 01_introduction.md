---
slug: /introduction
---

# 簡介
# Introduction

Stimulus 是一個有`適度`目標的 JavaScript 框架。跟其他框架不同的是 Stimulus 並不打算處理整個程式前端的部分。相反，它主要透過自動繫結 JavaScript 和元素的方式來增強 HTML 的功能。

Stimulus is a JavaScript framework with _modest_ ambitions. Unlike other frameworks, Stimulus doesn't take over your application's entire front-end. Rather, it's designed to augment your HTML by connecting elements to JavaScript objects automatically.

## 繫結 HTML 至 JavaScript
## Connecting HTML to JavaScript

Stimulus 運作的方式是透過持續監控頁面並等待 `data-controller` 屬性出現才套用動作。類似 `class` 樣式屬性，您可以在裡面設定多個值。不過我們在這不是套用 class 樣式名稱，`data-controller` 主要用來設定繫結或停止繫結 Stimulus 的控制器（controllers）。

> 譯者註：您可以同時套用多個 controller。例如： `data-controller="clipboard members"`

Stimulus works by continuously monitoring the page, waiting for the magic `data-controller` attribute to appear. Like the `class` attribute, you can put more than one value inside it. But instead of applying or removing CSS class names, `data-controller` values connect and disconnect Stimulus _controllers_.

這就跟我們把 CSS 的樣式設定到 HTML 元素上一樣，`data-controller` 則是把 HTML 和 JavaScript 作關聯。

Think of it like this: in the same way that `class` is a bridge connecting HTML to CSS, `data-controller` is a bridge from HTML to JavaScript.

在這個基礎的概念上，Stimulus 加入 `data-action` 屬性用來設定事件（events），即元素該觸發 controller 裡的哪個方法（action）。然後 `data-target` 屬性則是協助我們在 controller 影響的範圍內方便的操作特定元素。

On top of this foundation, Stimulus adds the magic `data-action` attribute, which describes how events on the page should trigger controller methods, and the magic `data-target` attribute, which gives you a handle for finding elements in the controller's scope.

## 關注點分離
## Separation of Concerns

Stimulus 提供的神奇屬性讓我們可以簡潔的分離內容和行為，就跟 CSS 將樣式和內容分離是相同的概念。此外，Stimulus 的慣例自然的讓我們把相關的程式碼透過名稱組織在一起。

Stimulus' magic attributes let you cleanly separate content from behavior in the same way you already separate content from presentation with CSS. Plus, Stimulus' conventions naturally encourage you to group related code by name.

這樣的安排協助我們建立易重複使用，各自具備專屬任務的 controllers，適當的結構以避免我們的程式變成 JavaScript 大雜燴。

This arrangement helps you build reusable, trait-like controllers, giving you just enough structure to keep your code from devolving into "JavaScript soup."

## HTML 具備可讀性
## A Readable Document

當 JavaScript 的行為對照到上述的屬性時，我們就可以透過 HTML 明白它是在做什麼的。這在當我們離開這個專案一陣子並忘記許多細節的狀況下特別有幫助，我們只需要大概閱讀 HTML 就可以抓出脈絡。

When your JavaScript behavior is mapped out in magic attributes, you can _read_ a fragment of HTML and know what's going on. That's a welcome relief when you return to a template six months later and don't recall exactly how things fit together.

可讀性的標記同時也表示我們團隊中的其他成員可以輕易的理解前端樣板、HTML 的部分，甚至在正式環境下也可以快速的查看行為和處理問題。

Readable markup also means that others on your team can easily look at templates—or even the developer console on a production page—to quickly trace behavior or diagnose an issue.

## 總結
## The Water's Warm

現在是時候嘗試使用 Stimulus 並理解它是怎麼運作的。下一節我們將開始學習建立第一個 controller。
Now's a great time to dip your toes in and discover how Stimulus works. Keep reading to learn how to build your first controller.

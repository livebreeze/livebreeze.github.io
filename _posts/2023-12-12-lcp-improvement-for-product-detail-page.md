---
layout: post
title: SEO Performance - LCP Improvement for Product Detail Page
subtitle: Reduced from 2.8 seconds to 2.5 seconds
cover-img: /assets/img/cover/cover_2023-12-13_07-43-46.png
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [seo, performance]
---

## PDP Core Web Vitals Issues

- LCP issue - mobile site need improved URLs root major issue.
- INP issue - 分數偏差，google 將在 2024/03 納入評分指標

## 目標

在不同的手機效能與網速，可能影響出不同的效能瓶頸，在目標結果需要觀察整體指標要看的是 "現場數據(Field Data)"，在校驗與調整參考的是 "實驗數據(Lab Data)"，因此選定一個實驗數據的監控標準後，以此進行調整，最終檢視 Field Data 是否改善。

### 預期優化方向 (針對 Mobile 網速 (2M~4M))

- HTML Response Time
  - SSR 渲染速度
    - 產線環境有varnish cache
- CSS Download
  - css 包的大小
    - 執行下載時間點
- LCP Download Speed
  - Image Gallery Image
    - 執行下載時間點
- Others: 若能保證上述 3 個時間，暫先排除 browser 端渲染速度影響 (inject, react render)

## How to Validate LCP

### 測量目標

[https://web.dev/articles/lab-and-field-data-differences#lcp](https://web.dev/articles/lab-and-field-data-differences#lcp)

主要分為兩大類

1. Field data (RUM) Real User Monitoring
2. Lab data (SM) Synthetic Monitoring

建議在調整驗證時，參考的是 Lab data，但是在確定優化項的優先順序，應該要以 Field data 為依據，因為這才是對於使用者真實的影響。
因為在真實環境 (Field data) 有眾多不同的環境影響，包括並不限於：

- 螢幕尺寸影響展示畫面內容，導致 LCP 不同
- 業務差異 - 登入與非登入內容上的差異，A/B testing 的差異
- 使用者設定 - 字體大小也可能影響
- base URL - 使用者可能使用 hash 方式，移動到頁面指定的位置
- 緩存狀態 - 真實使用者，已經緩存了一些資源 包含回訪同頁面或者共用的靜態資源，eg. css, image, font. 又或者是 bfcache(回退緩存)
- 使用者交互 - scrolls or interacts 會停止監控 LCP element.

#### 測量環境

執行測試環境對於測量結果與問題方向有很大的影響，所以在盡可能解決真實使用者 (Field data) 為前提下檢測，因此檢測環境僅PRE/PRD，DEV&local 太多 server 因素所以排除。

#### 模擬網路環境

根據 chatGPT 建議，以及網路資源，設定一下幾種網路環境，其中

Network throttling

- Regular 4G: 為 75th 位數使用網路
- Slow 4G: 為 google lighthouse network speed
￼
![Network Throttling](/assets/img/post/post_2023-12-13_07-58-16.png)

模擬硬體效能
CPU 2~4x slow

### 測量工具

- 如何思考速度工具  \|  Articles  \|  web.dev
- 評估及回報網站體驗核心指標的工具
- [https://web.dev/articles/vitals-tools#step_2_debug_and_optimize](https://web.dev/articles/vitals-tools#step_2_debug_and_optimize)

#### Google Search Console (GSC)

Google 由瀏覽器搜集真實使用者的測量結果
這邊的 need improved URL，會以 Group 方式，將問題由相同的使用體驗來聚合
因此，當有部分頁面導致整體的平均時間上升，便會將相同的 link group 都算在其中。
eg. 以下 136K 的 PDP Link 類型，因為都在相同的 group 當中，其中有特定的性能問題頁面，則會將此 group 被定義為 need improvement.

From: [Core Web Vitals report - Search Console Help (google.com)](https://support.google.com/webmasters/answer/9205520#page_groups)

#### Lighthouse - PageSpeed Insights

Google 提供的線上工具，可直接查閱上方 Field Data & 下方 Lab Data

#### Lighthouse - Browser Extension

瀏覽器 extension，與上述 PageSpeed Insights 相同，另外可選擇使用 DevTools 的設定

#### Lighthouse - CLI

官方文檔: [lighthouse - npm (npmjs.com)](https://www.npmjs.com/package/lighthouse#using-the-node-cli)

#### Performance in Chrome DevTools

1. Network - 觀察下載的檔案和反應時間
2. Animations - 觀察各指標時間 (FCP, LCP)
3. 設定 - CPU, Network Speed, Caching

- Performance features reference - Chrome for Developers
- Chrome DevTools - Chrome for Developers

#### Web Vitals extension

Performance API

### Optimize Task

- Update Head Script
- Preload LCP Image
- 佔頻寬的字體檔案延遲加載
- Cloudflare Mirage affects the mobile LCP

## Reference

- [https://web.dev/articles/lab-and-field-data-differences#lcp](https://web.dev/articles/lab-and-field-data-differences#lcp)
- [最佳化最大內容繪製 \| Articles \| web.dev](https://web.dev/articles/optimize-lcp?hl=zh-tw)
- [今晚，我想來點 Web 前端效能優化大補帖！. 效能是工程師在維護專案時非常重視的要點，不論是 Web 還是… \| by 莫力全 Kyle Mo \| Starbugs Weekly 星巴哥技術專欄 \| Medium](https://medium.com/starbugs/%E4%BB%8A%E6%99%9A-%E6%88%91%E6%83%B3%E4%BE%86%E9%BB%9E-web-%E5%89%8D%E7%AB%AF%E6%95%88%E8%83%BD%E5%84%AA%E5%8C%96%E5%A4%A7%E8%A3%9C%E5%B8%96-e1a5805c1ca2)
- [關鍵轉譯路徑  \|  Articles  \|  web.dev](https://web.dev/articles/critical-rendering-path?hl=zh-tw)
- [網站使用體驗指標  \|  Articles  \|  web.dev](https://web.dev/articles/vitals?hl=zh-tw)
- [Optimize your mobile site or app - Think with Google](https://www.thinkwithgoogle.com/marketing-strategies/app-and-mobile/mobile-tools-to-optimize-site-and-app/)

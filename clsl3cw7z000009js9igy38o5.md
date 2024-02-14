---
title: "LINE Messaging API SDK 大改版"
datePublished: Wed Feb 14 2024 01:06:21 GMT+0000 (Coordinated Universal Time)
cuid: clsl3cw7z000009js9igy38o5
slug: line-messaging-api-sdk-revamped
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/W32ZoreSE0Y/upload/dfc0ec54f05c69904ff201e155966804.jpeg
tags: line

---

最近碰巧在升級一個 Chatbot 專案，意外發現官方的 Node.js LINE Bot SDK [@line/bot-sdk](https://www.npmjs.com/package/@line/bot-sdk) 有重大改版，版本號躍進到了 8.x.x，也會在程式碼當中看到一些既有 API 被淘汰的訊息（Deprecation），在本文當中將說明如何升級與緣由。

## SDK 升級指南，以 Node.js 為例

雖然目前過往的程式碼都能運作，但在 SDK 上最主要的修改有兩大部分：

### 初始化 SDK Client

首先在初始化 Client 的部分，會發現 Class 名稱已經有改變：

```javascript
const line = require('@line/bot-sdk');
const clientConfig = {
  channelAccessToken: 'YOUR_CHANNEL_ACCESS_TOKEN',
  channelSecret: 'YOUR_CHANNEL_SECRET'
};

// SDK v8+ 新的做法 
const client = new line.messagingApi.MessagingApiClient(clientConfig);

// 先前做法
const client = new line.Client(clientConfig);
```

### 呼叫 API 的參數格式

而在呼叫 function 所需要的參數，也有一些格式上的改變：

```javascript
const message: TextMessage = {
  type: 'text',
  event.message.text, // 舉例
};

// SDK v8+ 新的做法 
await client.replyMessage({
  replyToken: replyToken,
  messages: [message],
});

// 先前做法
await client.replyMessage(replyToken, message);
```

詳細各個 API 的格式改變，可以閱讀[官方的 API 文件](https://line.github.io/line-bot-sdk-nodejs/apidocs/classes/messagingApi.MessagingApiClient.html)。其他語言 SDK 的修改也大多大同小異，可以透過專案 Changelog 來了解狀況。

## 日新月異的 LINE API

至於為什麼會有這麼大的變更？原因在於 LINE 本身 API 功能日新月異在進步，光是今年 2024 至今才過一個半月，就已經推出了：

* [可以確認使用者是否有付費訂閱 LINE 官方帳號的功能（日本區才有的新機制）](https://developers.line.biz/en/news/2024/02/09/get-membership-plan-information/)
    
* [官方帳號是否曾經被加入封鎖又解封](https://developers.line.biz/en/news/2024/02/06/add-friends-and-unblock-friends-can-now-be-determined-by-webhook/)
    
* [訊息可以發送「複製到剪貼簿」的功能按鈕](https://developers.line.biz/en/news/2024/02/05/messaging-api-updated/)
    

那過往每次只要有新的 API 更新，SDK 也就得做出對應的功能調整，考慮到官方支援的語言相當的多（[Node.js](https://github.com/line/line-bot-sdk-nodejs) / [Python](https://github.com/line/line-bot-sdk-python) / [Ruby](https://github.com/line/line-bot-sdk-ruby) / [Swift](https://github.com/line/line-sdk-ios-swift) / [Java](https://github.com/line/line-bot-sdk-java) / [Go](https://github.com/line/line-bot-sdk-go) 等），其實這個過程往往會需要花上不少時間，使得開發者沒辦法在第一時間使用新功能。

作為 LINE API Expert 的我，在過去當中也[數次針對不同語言的 SDK 做出貢獻](https://github.com/line/line-bot-sdk-nodejs/pulls?q=is%3Apr+author%3Adlackty+is%3Aclosed)，然而這真的是得花費非常多時間，且每次修改之後也都需要對應的測試與程式碼審查等等，最後才能夠真正發布新版。

## LINE OpenAPI 的誕生

於是為了解決這個問題，LINE 官方推出了 [line-openapi](https://github.com/line/line-openapi) 的專案，此專案使用 [OpenAPI](https://www.openapis.org) 格式，以 API 文件紀錄了目前官方的最新 API 功能。

而 OpenAPI 作為一個業界通用的格式，其最大的優點是有豐富的生態系，除了可以自動化產生標準 API 文件之外，像是常用的 HTTP 開發工具 [Postman 也有內建支援匯入 OpenAPI 文件的功能](https://learning.postman.com/docs/integrations/available-integrations/working-with-openAPI/)，就可以很快地開始做 API 的對應測試。

而回到本文的重點，官方也決定透過 OpenAPI 配合對應的自動化程式碼產生 API Client，也因此才會有了這次的升級需求。
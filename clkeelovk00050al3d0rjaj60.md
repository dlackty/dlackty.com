---
title: "GCP 終於支援單一專案下多個 Firestore"
datePublished: Sat Jul 22 2023 19:29:05 GMT+0000 (Coordinated Universal Time)
cuid: clkeelovk00050al3d0rjaj60
slug: gcp-finally-supports-multiple-firestore-databases-in-a-project
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/boR7SuyzB7Q/upload/94c42a5d7c63c6d25962db3bfe946aaa.jpeg
tags: firebase, gcp, firestore

---

Cloud Firestore 是 Google 所提供的雲原生（Cloud Native）資料庫解決方案，效能好、完全託管，且支援部署到全球各地的 Google Cloud 機房。

看到名稱是 Firestore，你八成已經猜到此產品最早是源自於 Firebase，後續才又被整合到 GCP 當中。與許多 Firebase 的功能相同，同一個專案是可以在 GCP 和 Firebase 介面上共用、存取到同一個 Firestore 資料庫。

## 過往只能開單一資料庫

然而或許也因為當初是從 Firebase 的品牌出發，所以並沒有考慮到太多複雜、大規模的使用情境，設計邏輯上比較是「單一 App 就使用單一資料庫」的概念，所以並沒有辦法在同一個專案下建立多個 Firestore 資料庫。

這就會造成不少困擾，比如說沒辦法去分割測試與正式環境，非得要建立切換多個專案。又或是因為 Firestore 只能存在單一地區，所以如果要換區域使用資料庫，甚至得重新建立 GCP 專案。

## 終於支援多資料庫

就在本月初的更新當中，[Google 官方總算宣布支援了多個 Firestore 資料庫在單一專案中](https://cloud.google.com/blog/products/databases/manage-multiple-firestore-databases-in-a-project)，且隨著這次的更新，也針對相關功能做了調整：

* 提供了 Firestore 資料庫的增刪查改管理支援，包含 API、gcloud CLI 指令、firebase CLI 指令以及 Terraform 整合
    
* IAM 也加入了個別資料庫的權限控制
    
* 連帶的當然帳務以及監控工具都也支援了多資料庫的切換
    

在操作資料庫時也可以帶入資料庫名，以 Firebase Admin SDK 的 Java 版本為例：

```java
// 初始化 Firebase App
FirebaseApp app = FirebaseApp.getInstance();
​
// 使用資料庫 ID `test` 初始化 Firestore 資料庫
FirebaseFirestore db = FirebaseFirestore.getInstance(app, “test”)
​
// 取得文件
DocumentReference docRef = db.collection(“col”).document(“doc”);
​
// 寫入文件
waitFor(docRef.set(Collections.singletonMap(“foo”, “bar”)));
```

## 結論

支援多資料庫一直是 Firestore 長久被期待的功能，終於在 2023 年下半年得到實現。此舉將讓 Firestore 的使用彈性再更上一層樓，相當推薦還尚未嘗試 Firestore 資料庫的讀者們，可以在後續專案納入考慮使用。
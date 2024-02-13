---
title: "Google Cloud 的 Viewer 角色所造成的風險"
datePublished: Mon Jul 17 2023 16:45:43 GMT+0000 (Coordinated Universal Time)
cuid: clk73kbop000909mn60h97xgr
slug: google-cloud-viewer-role-risks
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/xit3LjRvKvM/upload/86ea36de83d4feb86ece6c4d652d2177.jpeg
tags: gcp

---

在 Google Cloud 的官方社群部落格上看到一篇有趣的文章：[The unexpected permissions in the Viewer role on Google Cloud](https://medium.com/google-cloud/the-unexpected-permissions-in-the-viewer-role-on-google-cloud-b0e816a478ad)，主要在提醒大家 GCP 的 Viewer 權限其實沒有大家想得這麼安全。

確實多數的時候，當今天有新的同仁需要了解 GCP 專案時，往往我們就是會很直覺得利用 GCP 內建的 Viewer 角色，將權限開給同事們。

畢竟 Viewer 角色，不過就是瀏覽資源而已、不會造成危害吧！？

## 實際上 Viewer 角色不只單純是觀看

然而實際上卻不是這樣！

根據這篇文章的分析，在 Viewer 角色當中，總計會開 3300 個權限，其中與「觀看」無關的權限包含了以下列表中的權限：

```plaintext
# 呼叫 AI 模型預測
aiplatform.endpoints.predict
automl.models.predict
cloudtranslate.XXX.XXXpredict # 許多 predict 相關權限
documentai.processorVersions.processBatch
documentai.processorVersions.processOnline
documentai.processors.processBatch
documentai.processors.processOnline
ml.models.predict
ml.versions.predict
retail.servingConfigs.predict
retail.placements.predict
speech.recognizers.recognize

# 執行 BigQuery 任務
bigquery.jobs.create

# 建立磁碟快照
compute.disks.createSnapshot
```

上面這些權限，都是會造成費用產生的！不管是做模型的使用，或者是查詢 BigQuery 內的資料，都有可能產生費用，更別說建立磁碟快照了。

## 那該如何處理權限問題比較好？

其實就是不要偷懶，針對個別使用情境，去[建立 Custom Roles](https://cloud.google.com/iam/docs/creating-custom-roles) 然後加入需要的權限。畢竟到頭來，就算 Viewer 角色沒有上述的費用風險，一口氣開三千多個權限仍然是有點令人擔心呀！

## 結論

Google Cloud 官方早已建議使用者避免使用內建的 Editor / Viewer 等角色，多年前 GCP 產品相當有限，或許那時候還適合用單純的內建角色來處理，隨著 GCP 十年來的發展已經成長為擁有巨多服務的超級雲端服務，所以也需要用更細緻的方式來做權限管理，各位在操作的時候不得不多加留意。
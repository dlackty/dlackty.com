---
title: "如何在 AWS ECS 上連線 GCP CloudSQL"
datePublished: Thu Jul 27 2023 16:24:52 GMT+0000 (Coordinated Universal Time)
cuid: clkld814c000609ldfu38b4nz
slug: aws-ecs-gcp-cloudsql
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/iM8dxccK1sY/upload/b508059cb193d4f6ece66e23b7b83fa5.jpeg
tags: aws, ecs, gcp, cloud-sql

---

近期遇到一個業務需求，需要在 AWS ECS 上透過 BI 工具 Metabase 連線到 GCP 上一個 CloudSQL 的資料庫。

## CloudSQL 遠端連線做法

CloudSQL 官方推薦兩種方式做遠端連線：

1. 透過 IP 白名單的方式，將會連線進來的 IP 做設定
    
2. 透過 CloudSQL Proxy 來做連線
    

由於在我本次的環境當中主機的 IP 並不固定，所以使用後者的作法。然而多數 CloudSQL Proxy 的教學，包含官方都是使用 Kubernetes（應該一點都意外？），但卻沒有使用 ECS 的範例。

## CloudSQL Proxy 在 ECS 上的設定

大邏輯就是在 task definition 上多跑一個 `containerDefinitions`，然後透過主要 container 的 `links` 將兩邊串在一起。

如下方為例，請注意 `cloudsql` 相關定義：

```yaml
{
    "family": "awesome-service",
    "containerDefinitions": [
        {
            "name": "awesome-service",
            "image": "awesome-service/awesome-service:v1.0.0",
            "cpu": 0,
            "links": [
                "cloudsql"
            ],
            "portMappings": [],
            "essential": true,
            "environment": [],
            "mountPoints": [],
            "volumesFrom": []
        },
        {
            "name": "cloudsql",
            "image": "asia.gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.6.0",
            "cpu": 0,
            "portMappings": [],
            "essential": false,
            "command": [
                "PROJECT:REGION:INSTANCE",
                "--address",
                "0.0.0.0",
                "--json-credentials",
                "{\"type\": \"service_account\", ...}"
            ],
            "environment": [],
            "mountPoints": [],
            "volumesFrom": []
        }
    ],
    "networkMode": "bridge",
    "memory": "1024"
}
```

其中主要設定的點在 `command` 的部分：

* 第一個參數 `PROJECT:REGION:INSTANCE` 可以從 CloudSQL 的管理介面當中取得
    
* 第五個參數則為 Service Account 的 JSON，注意要處理雙引號等字串跳脫
    

## 在 Container 中連線到 CloudSQL

設定完成之後，就可以在主要的 container 中，透過 `cloudsql` 這個 hostname 來進行連線，例如：`mysql://username:password@cloudsql:3306` 。
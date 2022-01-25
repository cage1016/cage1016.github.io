# Taiwan Hong Kong Cloud Study Jam Guide


<!--more-->

## Taiwan Hong Kong Cloud Study Jam (Google offical)

> 邀請台灣與港澳的開發人員一起學習 Google Cloud 的相關技術。
>
> Cloud Study Jam 是一項在線自學計劃，為台灣及港澳的開發人員提供操作與學習 Google Cloud Lab的機會，同時提供與在地 Google 開發者社群一起學習的機會。


2022 第一場 Taiwan Hong Kong Cloud Study Jam 已經開始

- [Taiwan & Hong Kong Cloud Study Jam - 活動首頁](https://events.withgoogle.com/taiwan-cloud-study-jam/)
- 活動時間: **Jan. 19, 2022 17:00 - Feb. 9, 2022 23:59**

## Taiwan & Hong Kong Cloud Study Jam Together (GDG Cloud Taipei)

GDG Cloud Taipei 也開了一個線上的 Meetup，邀請大家一起上線完成 Cloud Study Jam Lab

- [Meetup event link](https://gdg.community.dev/events/details/google-gdg-cloud-taipei-presents-taiwan-hong-kong-cloud-study-jam-together/) 👈 請上 GDG Cloud Taipei 的活動頁面報名
- 活動時間: **2022/1/25 19:30 - 21:30**

## Guide

GDG Cloud Taipei Meetup 活動開立之後，也有朋友寫信來問 Qwiklab 的問題，再來官方的[使用手冊](https://docs.google.com/presentation/d/1ZCbzzAmKKGDdxRxmNachL2pcO2M8rTUI6prdy_7DnNo/edit?usp=sharing)是 2021 的版本，對於參加過 Cloud Study Jam 的朋友應該沒有太大的問題，不過寫信來的朋友問的朋友看起來比較是沒有參加過 Cloud Study Jam，所以 GDG Cloud Taipei 就 fork 了一份更新了一些內容

[Cloud Study Jam 2022 TW/HK User Guide - GDG Cloud Taipei Updated - Google Slides](https://docs.google.com/presentation/d/1H6E-fUFpUFFCdU7sD8ccsqaqq1SzqLTudAbTkz0X3ng/edit#slide=id.g4c0cd0eecf_0_1867) 👇 這個連結內容跟上面是一至的

{{< gslides src="https://docs.google.com/presentation/d/e/2PACX-1vRBrucJBUtwHRt4Lokyg-yC7PhJcFFOl-lShHASlTSbpCg9j-83RD5CxU8YTBFt0EgdZ_bXHrBxoJEK/embed?start=false&loop=false&delayms=3000" >}}

{{<youtube o0SK9UoJwq4>}}

**這邊有幾點說明一下**

1. 活動是 Google 官方舉辦的，請依照官方的活動報名頁面，如果報名了沒有收到 Email，請連絡 support@qwiklabs.com or 至 [2022 TWHK Cloud Study Jam - Google Groups](https://groups.google.com/g/2022-twhk-cloud-study-jam) (mailing list) 發問請求協助
1. 報名成功登入後，Qwiklabs 會預先給你 9 個 credit (不過我老闆反應他只有 5 個 credit 😅)。原則上就是拿這 9 個 credit 去完成 Quest list starter 👇 其中一個 Lab 來換 30 天免費的Google Cloud Skills Boost 訂閱, **記得 Start Lab 的時間要 > 5 分鐘**
   - [Create and Manage Cloud Resources](https://www.cloudskillsboost.google/quests/120?qlcampaign=6s-cloudstudy-67)
   - [Set Up and Configure a Cloud Environment in Google Cloud](https://www.cloudskillsboost.google/quests/119?qlcampaign=6s-prodev-63)
   - [Implement DevOps in Google Cloud](https://www.cloudskillsboost.google/quests/141?qlcampaign=6s-studyjams-32)
1. 在選點任何一個 Quest list starter ☝ 連結都會帶有一個 `qlcampaign` 的值 👇 ，開通當下不要亂點切換 URL，這個值不見了會導至開通失敗 (**非常重要**)

   {{<image src="img/2.jpg" alt="qlcampaign">}}

1. 使用無痕視窗來操作，直接在 Open Google Console 按鈕右鍵開啟無痕視窗即可，UserName 會直接帶過去

   {{<image src="img/1.jpg" alt="Open GCP Console with in Incognito window">}}

1. 這一次指定的 QWIKLABS QUESTS

   - Introductory
      - [Create and Manage Cloud Resources](https://www.cloudskillsboost.google/quests/120?catalog_rank=%7B%22rank%22%3A2%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Perform Foundational Infrastructure Tasks in Google Cloud](https://www.cloudskillsboost.google/quests/118catalog_rank=%7B%22rank%22%3A4%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
   - Fundamental
      - [Build a Website on Google Cloud](https://www.cloudskillsboost.google/quests/115?catalog_rank=%7B%22rank%22%3A21%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Automating Infrastructure on Google Cloud with Terraform](https://www.cloudskillsboost.google/quests/159?catalog_rank=%7B%22rank%22%3A28%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Implement DevOps in Google Cloud](https://www.cloudskillsboost.google/quests/141?catalog_rank=%7B%22rank%22%3A33%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Monitor and Log with Google Cloud Operations Suite](https://www.cloudskillsboost.google/quests/143?catalog_rank=%7B%22rank%22%3A37%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Ensure Access & Identity in Google Cloud](https://www.cloudskillsboost.google/quests/150?catalog_rank=%7B%22rank%22%3A51%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Service Serverless Cloud Run Development](https://www.cloudskillsboost.google/quests/152?catalog_rank=%7B%22rank%22%3A47%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Cloud Architecture](https://www.cloudskillsboost.google/quests/24?catalog_rank=%7B%22rank%22%3A7%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Networking in Google Cloud](https://www.cloudskillsboost.google/quests/31?catalog_rank=%7B%22rank%22%3A18%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [DevOps Essentials](https://www.cloudskillsboost.google/quests/96?catalog_rank=%7B%22rank%22%3A32%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Understanding Your Google Cloud Costs](https://www.cloudskillsboost.google/quests/90?catalog_rank=%7B%22rank%22%3A54%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
   - Advanced
      - [Cloud Architecture: Design, Implement, and Manage](https://www.cloudskillsboost.google/quests/124?catalog_rank=%7B%22rank%22%3A11%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Deploy to Kubernetes in Google Cloud](https://www.cloudskillsboost.google/quests/116?catalog_rank=%7B%22rank%22%3A13%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Set Up and Configure a Cloud Environment in Google Cloud](https://www.cloudskillsboost.google/quests/119?catalog_rank=%7B%22rank%22%3A15%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Build and Secure Networks in Google Cloud](https://www.cloudskillsboost.google/quests/128?catalog_rank=%7B%22rank%22%3A19%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Secure Workloads in Google Kubernetes Engine](https://www.cloudskillsboost.google/quests/142?catalog_rank=%7B%22rank%22%3A34%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Optimize Costs for Google Kubernetes Engine](https://www.cloudskillsboost.google/quests/157?catalog_rank=%7B%22rank%22%3A42%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Migrate MySQL data to Cloud SQL using Database Migration](https://www.cloudskillsboost.google/quests/180?catalog_rank=%7B%22rank%22%3A44%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [VM Migration](https://www.cloudskillsboost.google/quests/87?catalog_rank=%7B%22rank%22%3A35%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Migrating MySQL data to Cloud SQL using Database Migration Service](https://www.cloudskillsboost.google/quests/161?catalog_rank=%7B%22rank%22%3A45%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Anthos Service Mesh](https://www.cloudskillsboost.google/quests/151?catalog_rank=%7B%22rank%22%3A52%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
      - [Network Performance and Optimization](https://www.cloudskillsboost.google/quests/46?catalog_rank=%7B%22rank%22%3A95%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)
   - Expert
      - [Deploy and Manage Cloud Environments with Google Cloud](https://www.cloudskillsboost.google/quests/121?catalog_rank=%7B%22rank%22%3A8%2C%22num_filters%22%3A0%2C%22has_search%22%3Afalse%7D)


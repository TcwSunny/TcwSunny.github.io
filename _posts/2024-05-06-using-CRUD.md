---
layout: post
title:  "《JDBC專題》3. 製作CRUD功能"
date:   2024-5-6 00:05:25 +0000
categories: coding
tags: JDBC
comments: 1
published: false
---
> CRUD是網頁執行最基礎的四個操作，本文會教大家如何應用JDBC執行CRUD操作

## 基本步驟

1. 建立connection
2. 建立statement（statement/prepareStatement)
3. 將sql語句放入statement中
4. 如果是prepareStatement，把變數放進sql語句中
5. 執行
6. 關閉資源

## 資料庫介紹

我自己的專題使用的是台灣銀行匯率資料庫，

## Insert

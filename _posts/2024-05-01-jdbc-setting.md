---
layout: post
title:  "《JDBC專題》1. 系統設定：使用JDBC的前置作業"
date:   2024-5-1 22:35:00 +0000
categories: coding introduction
tags: JDBC
comments: 1
---
>JDBC提供API讓JAVA使用者能夠透過程式訪問資料庫，這篇文章是在說明撰寫JDBC前的系統設定，step by step 教你如何設定資料庫與防火牆

## 事前準備

* 下載eclipse和SSMS和SQL server
* 確保以上都可以正常運作

## 開始設定

### 打開SSMS

1. 打開SSMS，在資料庫的地方右鍵新增資料庫
![打開SSMS，在資料庫的地方右鍵新增資料庫](https://github.com/TcwSunny/TcwSunny.github.io/blob/main/assets/res/2024-05-01-jdbc-setting-res/image.png)

1. 打上資料庫名稱後按確定
![打上資料庫名稱後按確定](https://github.com/TcwSunny/TcwSunny.github.io/blob/main/assets/res/2024-05-01-jdbc-setting-res/image-1.png)

1. 在”安全性” “登入”中新增一筆登入
![在”安全性” “登入”中新增一筆登入](https://github.com/TcwSunny/TcwSunny.github.io/blob/main/assets/res/2024-05-01-jdbc-setting-res/image-2.png)

1. 輸入登入名稱->勾選SQL server驗證後輸入密碼
![輸入登入名稱->勾選SQL server驗證後輸入密碼](https://github.com/TcwSunny/TcwSunny.github.io/blob/main/assets/res/2024-05-01-jdbc-setting-res/image-3.png)

1. 使用者對應中選擇剛剛建立的資料庫->下面勾選db_owner->按確定
![使用者對應中選擇剛剛建立的資料庫](..\assets\res\2024-05-01-jdbc-setting-res\image-4.png)

### SQL server設定管理員

1. 打開SQL server設定管理員
![打開SQL server設定管理員](..\assets\res\2024-05-01-jdbc-setting-res\image-5.png)

2. 到SQLEXPRESS的通訊協定->TCP/IP按右鍵啟用
![到SQLEXPRESS的通訊協定](..\assets\res\2024-05-01-jdbc-setting-res\image-6.png)

3. 在TCP/IP再按一次右鍵->內容->選擇IP位址->滑到最底下TCP通訊埠->改成1433
![在TCP/IP再按一次右鍵](..\assets\res\2024-05-01-jdbc-setting-res\image-7.png)

### Windows防火牆

1. 打開windows defender防火牆(不要開錯!!!)->進階設定
![打開windows defender防火牆](..\assets\res\2024-05-01-jdbc-setting-res\image-8.png)

2. 輸入規則->新增規則
![輸入規則->新增規則](..\assets\res\2024-05-01-jdbc-setting-res\image-9.png)

3. 選擇連接埠->下一步
![選擇連接埠->下一步](..\assets\res\2024-05-01-jdbc-setting-res\image-10.png)

4. 選擇TCP->輸入1433->下一步
![選擇TCP->輸入1433->下一步](..\assets\res\2024-05-01-jdbc-setting-res\image-11.png)

5. 下一步->全部打勾
![下一步->全部打勾](..\assets\res\2024-05-01-jdbc-setting-res\image-12.png)

6. 輸入名稱SQL Server->完成
![輸入名稱SQL Server->完成](..\assets\res\2024-05-01-jdbc-setting-res\image-13.png)

### 再次打開SSMS

1. 最上層右鍵屬性
![最上層右鍵屬性](..\assets\res\2024-05-01-jdbc-setting-res\image-14.png)

2. 安全性->勾選SQL Server及Windows驗證模式->確定
![安全性](..\assets\res\2024-05-01-jdbc-setting-res\image-15.png)

### 回到SQL Server設定管理員

1. 右鍵將資料庫重新啟動
![右鍵將資料庫重新啟動](..\assets\res\2024-05-01-jdbc-setting-res\image-16.png)

以上做完系統設定就大功告成啦，下一篇會介紹如何透過java程式連結資料庫。

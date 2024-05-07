---
layout: post
title:  "《JDBC專題》3. 製作CRUD功能"
date:   2024-5-6 23:33:00 +0000
categories: coding
tags: JDBC
comments: 1
published: true
---
> CRUD是網頁執行最基礎的四個操作，本文會教大家如何應用JDBC執行CRUD操作

## 資料庫介紹

我自己的專題使用的是[台灣銀行匯率資料庫](https://rate.bot.com.tw/xrt?Lang=zh-TW)，下載csv後可以發現官方檔案的欄位有很多，這個專題只會使用即期匯率跟現金匯率。

首先先在 SQL server 加入 exchangeRate 資料庫，這邊我將幣別與日期設為共同主鍵，確保資料不會重複。

{% highlight sql linenos %}
CREATE TABLE [exchangeRate](
    name NVARCHAR(50) NOT NULL, --幣別
    date DATE NOT NULL, --日期
    buy_cash decimal(10,5), --現金買入
    buy_spot decimal(10,5), --現金賣出
    sell_cash decimal(10,5), --即期買入
    sell_spot decimal(10,5) --即期賣出
    CONSTRAINT name_date_PK PRIMARY KEY (name,date)
)
{% endhighlight %}

## 建立匯率物件

為了方便操作資料，我會先建立一個匯率的 model ，內容如下：

{% highlight java linenos %}
public class ExchangeRate {
    private String name;
    private Date date = null;
    private Float buyCash;
    private Float buySpot;
    private Float sellCash;
    private Float sellSpot;
}
{% endhighlight %}

後面再加上每個屬性的 getter & setter

## 操作CRUD

這邊使用 DAO(Data Access Objects) 模式，將資料庫相關操作封裝起來，先測試是否能成功操作基本CRUD後再進行下一步設計

### 基本步驟

1. 建立SQL語句
2. 建立connection
3. 建立statement（statement/prepareStatement)，因為prepareStatement基本上可以覆蓋statement的使用範圍，本次專題只會使用prepareStatement
4. 將sql語句放入statement中
5. 如果是prepareStatement，把變數放進sql語句中
6. 視需求將prepareStatement加入batch
7. 執行
8. 如果是查詢需要取得回傳的resultSet
9. 關閉資源

### 新增、修改、刪除

以上三個功能其實基本概念都一樣，這邊使用新增來當作範例，因為這裡輸入是直接用ExchangeRate的List，代表一次會執行多筆插入，所以也會示範使用Batch的方式以節省時間。

{% highlight java linenos %}
public void InsertExchangeRate(List<ExchangeRate> erList) {

    //建立SQL語句，這邊先判斷插入的資料是否重複，若沒有重複則insert
    //問號的地方是prepareStatement需要加入變數的位置
    String sql = "IF NOT EXISTS (SELECT * FROM exchangeRate WHERE (name = ? AND date = ?)) BEGIN INSERT INTO exchangeRate VALUES (?,?,?,?,?,?)END";

    //因為connection和prepareStatement會需要包在try裡面並且在finally的地方關閉，故需要在外面先宣告
    Connection connection = null;
    PreparedStatement preparedStatement = null;

    try {

        //建立connection及prepareStatement
        connection = C3p0Util.getConnection();
        preparedStatement = connection.prepareStatement(sql);

        //將list的物件取出，加入prepareStatement
        for(ExchangeRate er : erList) {

            //代表第一個問號加入er的name(對應資料庫第一欄)
            preparedStatement.setString(1, er.getName());

            //因為當日資料不會在csv中寫日期，也就是說建立物件時可能不會有日期，所以多新增一個判斷式將今天的日期手動加入
            if(er.getDate()!=null) {
                preparedStatement.setDate(2, er.getDate());
            }else {
                preparedStatement.setDate(2, java.sql.Date.valueOf(LocalDate.now()));
            }
            preparedStatement.setString(3, er.getName());
            if(er.getDate()!=null) {
                preparedStatement.setDate(4, er.getDate());
            }else {
                preparedStatement.setDate(4, java.sql.Date.valueOf(LocalDate.now()));
            }

            //繼續加變數
            preparedStatement.setFloat(5, er.getBuyCash());
            preparedStatement.setFloat(6, er.getBuySpot());
            preparedStatement.setFloat(7, er.getSellCash());
            preparedStatement.setFloat(8, er.getSellSpot());

            //將prepareStatement加入batch中
            preparedStatement.addBatch();
        }

        //執行batch，因為大部分情況資料庫匯入只有幾十到幾百筆，故只執行一次，若筆數過多也可以使用判斷式設定每多少筆執行一次
        //若不使用batch則需使用execute指令，一次執行一筆新增
        preparedStatement.executeBatch();

        //執行完後清空batch
        preparedStatement.clearBatch();

        //顯示執行成功
        System.out.println("新增成功");

    } catch (SQLException e) {
        e.printStackTrace();
    }finally {

        //最後將connection和prepareStatement關閉
        C3p0Util.release(connection, preparedStatement);
    }
}
{% endhighlight %}

而修改刪除邏輯相似，基本上只要依需求修改SQL指令及batch的部分即可。

### 查詢

查詢語句較不一樣的地方是執行時需要使用executeQuery，且會回傳resultSet，需要將resultSet中的資料進行整理。

{% highlight java linenos %}
public List<ExchangeRate> selectAll() {

    //以下步驟同增刪改，但需要多一個resultSet去接住回傳的資料
    String sql = "SELECT * FROM exchangeRate ORDER BY name,date";
    ResultSet resultSet = null;
    Connection connection = null;
    PreparedStatement preparedStatement = null;
    try {
        connection = C3p0Util.getConnection();
        preparedStatement = connection.prepareStatement(sql);

        //執行時需使用executeQuery
        resultSet = preparedStatement.executeQuery();

        //建立準備回傳的list
        List<ExchangeRate> list = new ArrayList<ExchangeRate>();

        //當有下一筆resultSet時讀取下一筆
        while (resultSet.next()) {

            //建立匯率物件
            ExchangeRate exchangeRate = new ExchangeRate();

            //設定屬性值，依據不同型別使用getXXX，括號內可以放第幾欄或欄位名稱（此處使用第幾欄做示範）
            exchangeRate.setName(resultSet.getString(1));
            exchangeRate.setDate(resultSet.getDate(2));
            exchangeRate.setBuyCash(resultSet.getFloat(3));
            exchangeRate.setBuySpot(resultSet.getFloat(4));
            exchangeRate.setSellCash(resultSet.getFloat(5));
            exchangeRate.setSellSpot(resultSet.getFloat(6));

            //將匯率加入list中
            list.add(exchangeRate);
        }

        //回傳list
        return list;
    } catch (SQLException e) {
        e.printStackTrace();
    }finally {

        //關閉資源
        C3p0Util.release(connection, preparedStatement, resultSet);
    }

    //若失敗則回傳null
    return null;
}
{% endhighlight %}

有了基本的CRUD後，接下來就是依需求自行增加更多的DAO操作，下一篇會介紹整個專題的資料流程及爬蟲相關內容。
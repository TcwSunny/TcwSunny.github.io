---
layout: post
title:  "《JDBC專題》4. 資料處理與簡易Jsoup爬蟲"
date:   2024-5-8 19:06:00 +0000
categories: coding
tags: JDBC
comments: 1
published: true
---
> 本文會介紹此專題讀取資料的方法，以及如何運用簡易爬蟲抓取匯率資料

以下圖片是這個專題的架構，在前面的文章中，我們已經完成了model、util及DAO的建置，接著會介紹我如何取得資料。

![架構]({{ site.image_baseurl }}\2024-05-08-jsoup-data-res\image.png?raw=true)
![流程圖]({{ site.image_baseurl }}\2024-05-08-jsoup-data-res\flowChart.png?raw=true)

## getData

這個class的主要目的在於取得資料中的文字，並且以字串形式回傳，這個專題的輸入資料支援當日匯率csv、自行輸出的json、爬蟲取得的單日資料及單一匯率資料。

### CSV

{% highlight java linenos %}
public static List<String> getCsvData(String dataPath) {

    //建立inputStream，並以bufferReader加快讀取速度
    FileInputStream fileInputStream = null;
    InputStreamReader inputStreamReader = null;
    BufferedReader bufferedReader = null;
    List<String> list = new ArrayList<String>();
    try {
        fileInputStream = new FileInputStream(new File(dataPath));
        inputStreamReader = new InputStreamReader(fileInputStream);
        bufferedReader = new BufferedReader(inputStreamReader);

        //當bufferReader準備好後，就可以將內容放進list中
        while(bufferedReader.ready()) {
            list.add(bufferedReader.readLine());
        }

        //因為第一行會讀到標題，所以將首行移除
        list.remove(0);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        try {
            fileInputStream.close();
            inputStreamReader.close();
            bufferedReader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    return list;
}
{% endhighlight %}

這樣就會回傳一個字串list，其中一個string代表一筆資料，不同欄位之間使用逗號隔開，就可以傳入service進行處理。

### JSON

這邊IO的方法會和上面不太一樣，是我刻意練習不同方式實作，其實兩種是可以互通的。

{% highlight java linenos %}
public static String getJsonData(String dataPath) {

    //建立一個stringBuilder
    StringBuilder jsonBuilder = new StringBuilder();

    //這邊把IO放進括號，try結束時會auto close，不需要另外關閉
    try (FileInputStream fileInputStream = new FileInputStream(new File(dataPath));
        InputStreamReader inputStreamReader = new InputStreamReader(fileInputStream);
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader)){

        //把json檔全部放進一個字串中
        String line;
        while ((line = bufferedReader.readLine()) != null) {
            jsonBuilder.append(line);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    return jsonBuilder.toString();
}
{% endhighlight %}

這樣就可以產生一整個Json字串，之後再將字串使用GSON解析即可（這部分會在service介紹）。

### Jsoup爬蟲

這邊會分為兩個含式，getNowData是爬取當日資料，而getPastData則是爬單一幣別整個月的資料（datTime為月份、name不為空）或是過去的單日資料（datTime為日期、name為空字串）。

{% highlight java linenos %}
public static String[] getNowData() {
    String[] dataList = null;
    try {

        //使用Jsoup抓取網路資料
        Document document = Jsoup.connect("https://rate.bot.com.tw/xrt/flcsv/0/day").get();

        //print出來後可以發現資料都包在body中，所以使用body().text()取出內部文字
        //<html>
        //    <head></head>
        //    <body>
        //        幣別,匯率,現金,即期,遠期10天...
        //    </body>
        //</html>
        String body = document.body().text();

        //因為在不同筆中間是用空格隔開，因此使用split就可以將字串轉換為String的陣列
        dataList = body.split(" ");
    } catch (IOException e) {
        e.printStackTrace();
    }
    return dataList;
}

//getPastData也是使用一樣的方法，只是爬取網址不同
public static String[] getPastData(String name, String dayTime) {
    String[] dataList = null;
    try {
        Document document = Jsoup.connect("https://rate.bot.com.tw/xrt/flcsv/0/"+dayTime+"/"+name).get();
        String body = document.body().text();
        dataList = body.split(" ");
    } catch (IOException e) {
        e.printStackTrace();
    }
    return dataList;
}
{% endhighlight %}

這樣就可以回傳String陣列，再使用service處理相應的格式變成物件即可。

取得資料後，下一篇會分享我如何將回傳的資料轉換為物件。
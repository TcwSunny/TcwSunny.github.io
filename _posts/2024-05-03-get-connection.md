---
layout: post
title:  "《JDBC專題》2. 使用java與SQL server建立連結"
date:   2024-5-3 00:24:00 +0000
categories: coding
tags: JDBC
comments: 1
published: true
---
>在執行SQL語句前需要建立Connection和Statement，這篇會介紹建立connection的2種方法：直接建立connection及connection pool

實作這一篇之前請先確認系統設定都正確，還沒設定的請看[上一篇](https://tcwsunny.github.io/2024/05/01/jdbc-setting)

## 建立connection：直接連線

在建立connection時需要前一篇設定的帳號密碼進行登入，因為這個功能之後會很常用到，所以建議直接建一個工具class，在裡面新增static函式

1. 首先需要建立一個preperties檔
  檔名叫XXXX.properties(範例叫jdbc.properties)，裡面放url、user和password三個參數

  {% highlight properties linenos %}
  url=jdbc:sqlserver://localhost:1433;DatabaseName=你的資料庫名稱;encrypt = false
  user=帳號名稱
  password=你的密碼
  {% endhighlight %}

  建在properties是為了讓其他使用者更方便修改，不然讓別人動code是一件有點危險的事，如果沒有這個需求也可以直接寫在code裡面
2. 開始建立connection，程式如下

  {% highlight java linenos %}
  public class JDBCutil {
    public static Connection getConnection() {
      Connection connection = null;
      FileInputStream fileInputStream = null;
      try {
        //註冊JDBC驅動程式
        Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
        
        //讀properties檔案，取出帳號密碼
        Properties properties = new Properties();
        fileInputStream = new FileInputStream(new File("你的properties位置"));
        properties.load(fileInputStream);
        String url = properties.getProperty("url");
        String user = properties.getProperty("user");
        String password = properties.getProperty("password");

        //建立connection
        connection = DriverManager.getConnection(url,user,password);

        //確認connection是否連線
        boolean status = !connection.isClosed();
        System.out.println("連線狀態"+status);
      } catch (ClassNotFoundException e) {
        e.printStackTrace();
      } catch (FileNotFoundException e) {
        e.printStackTrace();
      } catch (IOException e) {
        e.printStackTrace();
      } catch (SQLException e) {
        e.printStackTrace();
      }finally {
        //關閉fileInputString
        try {
          fileInputStream.close();
        } catch (IOException e) {
          e.printStackTrace();
        }
      }
      return connection;
    }
  }
  {% endhighlight %}
3. 在使用完connection後需要關閉連線
  可以在工具class底下新增closeResource()，這邊我會使用overload的方式把statement和resultset(請見下篇)的關閉函式一起寫好
  {% highlight java linenos %}
  Statement statement = null;
  ResultSet resultSet = null; 
  public static void closeResource(Connection connection) {
    try {
      if(connection!=null) {
        connection.close();
      }
    } catch (SQLException e) {
      e.printStackTrace();
    }
  }
  public static void closeResource(Connection connection,Statement statement) {
    try {
      if(connection!=null) {
        connection.close();
      }
      if(statement!=null) {
        statement.close();
      }
    } catch (SQLException e) {
      e.printStackTrace();
    }
  }
  
  public static void closeResource(Connection connection,Statement statement,ResultSet resultSet) {
    try {
      if(connection!=null) {
        connection.close();
      }
      if(statement!=null) {
        statement.close();
      }
      if(resultSet!=null) {
        resultSet.close();
      }
    } catch (SQLException e) {
      e.printStackTrace();
    }
  }
  {% endhighlight %}
4. 在main裡面呼叫函式JDBCutil.getConnection()
  執行後如果看到”連線狀態true”表示連線成功
  {% highlight java linenos %}
  public static void main(String[] args) {
    Connection connection = JDBCUtil.getConnection();
    JDBCUtil.closeResource(connection);
  }
  {% endhighlight %}

## 建立Connection：連接池(Connection pool)

連接池是在系統執行時建立多個連結，需要用時把連結拿來使用，用完後再放回池子中，而連接池也是我在這次專題中主要使用的方法，以下code是示範連接池工具中如何getConnection，使用的是c3p0套件

[下載連結](https://www.mchange.com/projects/c3p0/)：滑到下面installation下載c3p0–0.10.0.jar 和mchange-commons-java-0.3.0.jar(有最新版就下載最新版)，然後加到build path中

1. 以下是getConnection的實作方式
  可以看到我在openDataSource中開了一個連接池，並將帳號密碼設定好，接著用一個變數代表連接池(dataSource)，然後在getConnection的地方把connection取出並回傳，這裡我直接把exception丟出，如果要在函式中直接try catch也是可以。
  {% highlight java linenos %}
  public static ComboPooledDataSource openDataSource() {
    //連接池設定
    ComboPooledDataSource cpds = new ComboPooledDataSource();
    try {
      cpds.setDriverClass( "com.microsoft.sqlserver.jdbc.SQLServerDriver" );
      cpds.setJdbcUrl( "jdbc:sqlserver://localhost:1433;DatabaseName=資料庫名稱;encrypt=false" );
      cpds.setUser("帳號");
      cpds.setPassword("密碼");
    } catch (PropertyVetoException e) {
      e.printStackTrace();
    }
      return cpds;
    }
  
    private static ComboPooledDataSource dataSource = C3p0Util.openDataSource();
  
    public static Connection getConnection() throws SQLException {
      return dataSource.getConnection();
    }
  {% endhighlight %}
2. 然後就是關連結的函式，一樣使用overload
  {% highlight java linenos %}
  public static void release(Connection connection) {
    try {
      if(connection!=null) {
        connection.close();
    }
    } catch (SQLException e) {
      e.printStackTrace();
    }
  }
  
  public static void release(Connection connection,Statement statement) {
    try {
      if(connection!=null) {
        connection.close();
      }
      if(statement!=null) {
        statement.close();
      }
    } catch (SQLException e) {
      e.printStackTrace();
    }
  }
  
  public static void release(Connection connection,Statement statement,ResultSet resultSet) {
    try {
      if(connection!=null) {
        connection.close();
      }
      if(statement!=null) {
        statement.close();
      }
      if(resultSet!=null) {
        resultSet.close();
      }
    } catch (SQLException e) {
      e.printStackTrace();
    }
  }
  {% endhighlight %}
測試連線成功後就可以開始建statement和跑SQL程式了!!!(下篇待續)

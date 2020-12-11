---
layout:    post
title:     DVWA之BruteForce
subtitle:  学习学习
date:      2020-12-09
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:

   - DVWA
---

#### BruteForce(暴力破解)

##### burp之intruder模块

介绍一下攻击类型吧：

[![rP3naD.md.png](https://s3.ax1x.com/2020/12/09/rP3naD.md.png)](https://imgchr.com/i/rP3naD)

1.**Sniper模式：** 狙击手模式，就如其名一样，该种模式每次都只针对一个目标，payload有20个时，就执行20次；假设爆破点有俩个时，如有a, b两个爆破点，词典为1，2，3，3，4，共执行5+5=10次，则执行顺序如下：

[![rP3osx.md.jpg](https://s3.ax1x.com/2020/12/09/rP3osx.md.jpg)](https://imgchr.com/i/rP3osx)

``该类型适合对常见漏洞中的请求参数单独地进行测试。``

2.**Batterint ram模式(攻城锤)：** 当有一个爆破点时，爆破方式与狙击手模式相同；当有两个爆破点时，同时验证两个爆破点，如a, b两个点，词典为1，2，3，4，5，共执行5次，执行顺序如下：

[![rP8wTO.md.jpg](https://s3.ax1x.com/2020/12/09/rP8wTO.md.jpg)](https://imgchr.com/i/rP8wTO)

``该模式适合需要在请求中把相同的输入放到多个位置的情况。``

3.**Pitchfork(草叉模式)：** 该模式需要两个爆破点，两个payload，攻击会同步迭代所有的payload组，把payload放入每个定义的位置中。假设两个爆破点分别为a, b，a对应词典a, b对应词典b, 则执行次数为：两个词典中payload小的，下图中执行次数为4，执行顺序如下：

[![rPG939.md.jpg](https://s3.ax1x.com/2020/12/09/rPG939.md.jpg)](https://imgchr.com/i/rPG939)

``该模式适合不同位置需要插入不同但相关的输入的情况。``

4.**Cluster bomb(集束炸弹)：** 与pitchfork类似，至少两个爆破点，但执行次数会计算两个payload的笛卡儿积。假设两个爆破点分别为a、b，a对应词典a、b对应词典b，则执行次数为连个词典的乘积，执行顺序如下：

[![rPGyUU.md.jpg](https://s3.ax1x.com/2020/12/09/rPGyUU.md.jpg)](https://imgchr.com/i/rPGyUU)
``这种攻击适用于那种位置中需要不同且不相关或者未知的输入的攻击。``

上方参考博文--->[here](https://blog.csdn.net/weixin_40950781/article/details/99693893?utm_source=app)

##### low

```
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Get username
    $user = $_GET[ 'username' ];

    // Get password
    $pass = $_GET[ 'password' ];
    $pass = md5( $pass );

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

**mysqli_query(connection,query,resultmode):** 执行一条MySQL查询，connection必须，规定要使用的MySQL连接；query必须，规定查询字符串。resultmode可选，一个常量，可以是下列中任意一个：

+ MYSQLI_USE_RESULT (如果需要检索大量数据，请使用这个)
+ MYSQLI_STORE_RESULT (默认)

针对成功的select，show，describe或explain查询，返回一个mysql_result对象。针对其他查询，成功返回true，反之false。

**$GLOBALS:**  引用全局作用域中可用的全部变量。

**die(status) ：** 输出一条消息，并退出当前脚本
status 必需。规定在退出脚本之前需要输出的消息或状态号。状态号不会被写入输出
如果 status 是字符串，则该函数会在退出前输出字符串。
如果 status 是整数，这个值会被用作退出状态。退出状态的值在 0 至 254 之间。退出状态 255 由 PHP 保留，不会被使用。状态 0 用于成功地终止程序
注释：如果 PHP 的版本号大于等于 4.2.0，那么在 status 是整数的情况下，不会输出该参数

**is_object($var) ：** 用于检测变量是否是一个对象
$var：要检测的变量
如果指定变量为对象，则返回TRUE，否则返回FALSE。

可以看到其直接获得了用户输入的账号和密码，密码又进行了MD5加密，排除了从密码注入的可能。这里对输入的账号和密码没有进行任何过滤与检查，所以我们也可以进行SQL注入。

方法一： 存在SQL注入，所有直接尝试SQL注入。

payload：``admin'#``   用#直接注释掉后面的password验证。

[![rPNrQJ.png](https://s3.ax1x.com/2020/12/09/rPNrQJ.png)](https://imgchr.com/i/rPNrQJ)

方法二：

采用burp爆破处理：

1.将浏览器代理设置好，打开burp，设置为拦截模式：

[![rFnVhR.png](https://s3.ax1x.com/2020/12/10/rFnVhR.png)](https://imgchr.com/i/rFnVhR)

2.在用户输入处输入账号密码，账号：admin，密码随意。

[![rFnG4A.png](https://s3.ax1x.com/2020/12/10/rFnG4A.png)](https://imgchr.com/i/rFnG4A)

3.点击登陆，burp自会拦截：

[![rFnDEQ.md.png](https://s3.ax1x.com/2020/12/10/rFnDEQ.md.png)](https://imgchr.com/i/rFnDEQ)

4.右键选择send to intruder,也可快捷键 CTRL I

[![rFnXDO.md.png](https://s3.ax1x.com/2020/12/10/rFnXDO.md.png)](https://imgchr.com/i/rFnXDO)

5.到intruder窗口：选择狙击手模式，爆破密码

[![rFK20U.md.png](https://s3.ax1x.com/2020/12/10/rFK20U.md.png)](https://imgchr.com/i/rFK20U)

6.加载字典，字典就是自己收集的各类账号，密码等，一个强大的字典也是很重要的。

[![rFM8N4.md.png](https://s3.ax1x.com/2020/12/10/rFM8N4.md.png)](https://imgchr.com/i/rFM8N4)

7.长度与其他不同的一般为正确的：

[![rFMWKP.png](https://s3.ax1x.com/2020/12/10/rFMWKP.png)](https://imgchr.com/i/rFMWKP)

##### medium

```
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Sanitise username input
    $user = $_GET[ 'username' ];
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_GET[ 'password' ];
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass = md5( $pass );

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        sleep( 2 );
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

**mysqli_real_escape_string(connection,escapestring):** 转义在SQL语句中使用的字符串中的特殊字符：

connection 必须，规定要使用的MySQL连接。

escapestring 必须，要转义的字符串。编码的字符是null, \\n, \\r, \\, ', " 和control-Z

该级别代码用mysqli_real_escape_string()函数对用户输入的账号密码进行了转义，防止了利用单引号，双引号等参数构造进行SQL注入，还使用了sleep()函数，在登陆失败时延时返回信息，延长了爆破时间，其余正常。

因为过程与low相同，直接放结果图了，还是根据长度判断即可：

[![rFYnmR.md.png](https://s3.ax1x.com/2020/12/10/rFYnmR.md.png)](https://imgchr.com/i/rFYnmR)

##### high

```
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Sanitise username input
    $user = $_GET[ 'username' ];
    $user = stripslashes( $user );
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_GET[ 'password' ];
    $pass = stripslashes( $pass );
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass = md5( $pass );

    // Check database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        sleep( rand( 0, 3 ) );
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```

high级别代码使用了Anti-CSRF token来抵御CSRF的攻击，使用了stripslashes函数和mysql_real_esacpe_string来抵御SQL注入和XSS的攻击。

stripslashes(string): 删除反斜杠，string必须，规定要检查的字符串。

由于使用了Anti-CSRF token, 每次服务器返回的登陆页面都会包含一个随机的user_token的值，用户每次登陆时都要将user_token一起提交。服务器收到请求后，会优先做user_token的检查，再进行sql查询。

1.抓包，放到intruder中，将Attack type设置为pitchfork(草叉模式)，为password和user_token值添加payload标记。

[![rk7RiR.md.png](https://s3.ax1x.com/2020/12/11/rk7RiR.md.png)](https://imgchr.com/i/rk7RiR)

2.线程设置，在option框中，将threads：设置为1。

3.在options栏找到Grep-Extract, 点击add。然后点击refetch response, 进行一个请求，看到响应报文，直接选取需要提取的字符串，上面的会自动填入数据的起始和结束标识，并将此值保存，之后会用到。

[![rk7IsO.png](https://s3.ax1x.com/2020/12/11/rk7IsO.png)](https://imgchr.com/i/rk7IsO)

4.设置payload，因为这个有两个payload，第一个为password的，步骤和之前一样不说了，第二个，选择参数为Recursive grep,后将options中的token作为一开始的初始值。

[![rkHopq.png](https://s3.ax1x.com/2020/12/11/rkHopq.png)](https://imgchr.com/i/rkHopq)

5.可以开始爆破了

[![rkqNIH.md.png](https://s3.ax1x.com/2020/12/11/rkqNIH.md.png)](https://imgchr.com/i/rkqNIH)

[![rkqHwF.png](https://s3.ax1x.com/2020/12/11/rkqHwF.png)](https://imgchr.com/i/rkqHwF)

成功。

##### imposs

```
<?php

if( isset( $_POST[ 'Login' ] ) && isset ($_POST['username']) && isset ($_POST['password']) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Sanitise username input
    $user = $_POST[ 'username' ];
    $user = stripslashes( $user );
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_POST[ 'password' ];
    $pass = stripslashes( $pass );
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass = md5( $pass );

    // Default values
    $total_failed_login = 3;
    $lockout_time       = 15;
    $account_locked     = false;

    // Check the database (Check user information)
    $data = $db->prepare( 'SELECT failed_login, last_login FROM users WHERE user = (:user) LIMIT 1;' );
    $data->bindParam( ':user', $user, PDO::PARAM_STR );
    $data->execute();
    $row = $data->fetch();

    // Check to see if the user has been locked out.
    if( ( $data->rowCount() == 1 ) && ( $row[ 'failed_login' ] >= $total_failed_login ) )  {
        // User locked out.  Note, using this method would allow for user enumeration!
        //echo "<pre><br />This account has been locked due to too many incorrect logins.</pre>";

        // Calculate when the user would be allowed to login again
        $last_login = strtotime( $row[ 'last_login' ] );
        $timeout    = $last_login + ($lockout_time * 60);
        $timenow    = time();

        /*
        print "The last login was: " . date ("h:i:s", $last_login) . "<br />";
        print "The timenow is: " . date ("h:i:s", $timenow) . "<br />";
        print "The timeout is: " . date ("h:i:s", $timeout) . "<br />";
        */

        // Check to see if enough time has passed, if it hasn't locked the account
        if( $timenow < $timeout ) {
            $account_locked = true;
            // print "The account is locked<br />";
        }
    }

    // Check the database (if username matches the password)
    $data = $db->prepare( 'SELECT * FROM users WHERE user = (:user) AND password = (:password) LIMIT 1;' );
    $data->bindParam( ':user', $user, PDO::PARAM_STR);
    $data->bindParam( ':password', $pass, PDO::PARAM_STR );
    $data->execute();
    $row = $data->fetch();

    // If its a valid login...
    if( ( $data->rowCount() == 1 ) && ( $account_locked == false ) ) {
        // Get users details
        $avatar       = $row[ 'avatar' ];
        $failed_login = $row[ 'failed_login' ];
        $last_login   = $row[ 'last_login' ];

        // Login successful
        echo "<p>Welcome to the password protected area <em>{$user}</em></p>";
        echo "<img src=\"{$avatar}\" />";

        // Had the account been locked out since last login?
        if( $failed_login >= $total_failed_login ) {
            echo "<p><em>Warning</em>: Someone might of been brute forcing your account.</p>";
            echo "<p>Number of login attempts: <em>{$failed_login}</em>.<br />Last login attempt was at: <em>${last_login}</em>.</p>";
        }

        // Reset bad login count
        $data = $db->prepare( 'UPDATE users SET failed_login = "0" WHERE user = (:user) LIMIT 1;' );
        $data->bindParam( ':user', $user, PDO::PARAM_STR );
        $data->execute();
    } else {
        // Login failed
        sleep( rand( 2, 4 ) );

        // Give the user some feedback
        echo "<pre><br />Username and/or password incorrect.<br /><br/>Alternative, the account has been locked because of too many failed logins.<br />If this is the case, <em>please try again in {$lockout_time} minutes</em>.</pre>";

        // Update bad login count
        $data = $db->prepare( 'UPDATE users SET failed_login = (failed_login + 1) WHERE user = (:user) LIMIT 1;' );
        $data->bindParam( ':user', $user, PDO::PARAM_STR );
        $data->execute();
    }

    // Set the last login time
    $data = $db->prepare( 'UPDATE users SET last_login = now() WHERE user = (:user) LIMIT 1;' );
    $data->bindParam( ':user', $user, PDO::PARAM_STR );
    $data->execute();
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```

这里记录了登录失败的次数，如果登陆失败次数大于3就锁定了账号。这样子暴力破解就会非常缓慢，15分钟只能跑3个。所以就相当于没用了。

同时采用了PDO(PHP Data Object, php数据对象)机制更加安全，不会在本地对SQL进行拼接。当调用prepare()时，将SQL模板传给MySQLServer，传过去是占位符"?", 不包含用户数据，当调用execute() 时， 用户的变量值才传递到MySQL Server, 分开传递，阻止了SQL语句被破坏而执行恶意代码。


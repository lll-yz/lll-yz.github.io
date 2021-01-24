---
layout:    post
title:     DVWA之Weak Session IDs
subtitle:  学习学习
date:      2020-12-28
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:

   - DVWA
---

#### Weak Session IDs

##### session IDs简介

HTTP是无连接的协议，当客户端访问通过HTTP协议访问服务器时，服务器无法确认访问者身份。这会导致一系列问题，比如无法判断是哪个用户登陆或无法向用户提供差异化服务，于是Session ID上场了，用户访问服务器的时候，一般服务器都会分配一个身份证Session ID就对应一个客户端，用户只要拿到Session ID后就会保存到cookie上，之后只要拿到cookies再访问服务器，服务器就知道你是谁了。但是Session ID过于简单 就会很容易被人伪造，根本都不需要知道用户的密码就能登陆服务器。

##### low

```
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if (!isset ($_SESSION['last_session_id'])) {
        $_SESSION['last_session_id'] = 0;
    }
    $_SESSION['last_session_id']++;
    $cookie_value = $_SESSION['last_session_id'];
    setcookie("dvwaSession", $cookie_value);
}
?> 
```

可以看到，low级别的cookie产生策略是：在上一个cookie值的基础上加一(初始值为0)。当用户访问服务器的时候，low级别的代码会先检测服务器是否存在名为last_session_id的session，如果存在，取出其值，加一后赋值给名为dvwaSession的cookie。没有进行过滤。

在浏览器A中使用burp抓包，复制其cookie于URL

[![rI1N8g.png](https://s3.ax1x.com/2020/12/27/rI1N8g.png)](https://imgchr.com/i/rI1N8g)

再去打开另一页面，用hackbar构造URL和cookie，点击execute，可以看到直接登陆成功无需账号密码

[![rI1hrR.md.png](https://s3.ax1x.com/2020/12/27/rI1hrR.md.png)](https://imgchr.com/i/rI1hrR)

##### medium

```
 <?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $cookie_value = time();
    setcookie("dvwaSession", $cookie_value);
}
?>

```

medium级别的代码为基于时间戳生成的dvwaSession(使用了time()函数设置cookie的值)

通过设置时间戳，可以诱骗受害者在某个时间点进行点击。

**测试：**

同样先用burp抓包，查看一下cookie

[![rI3WY8.md.png](https://s3.ax1x.com/2020/12/27/rI3WY8.md.png)](https://imgchr.com/i/rI3WY8)

可以去``https://tool.lu/timestamp``进行时间戳的转换。

[![rI8lAP.png](https://s3.ax1x.com/2020/12/27/rI8lAP.png)](https://imgchr.com/i/rI8lAP)

然后继续去用hackbar构造payload：

[![rI8Uns.md.png](https://s3.ax1x.com/2020/12/27/rI8Uns.md.png)](https://imgchr.com/i/rI8Uns)

同样绕过了账号密码，直接登陆进去。

##### high

```
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if (!isset ($_SESSION['last_session_id_high'])) {
        $_SESSION['last_session_id_high'] = 0;
    }
    $_SESSION['last_session_id_high']++;
    $cookie_value = md5($_SESSION['last_session_id_high']);
    setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], false, false);
}

?> 
```

[![r7pyrQ.md.jpg](https://s3.ax1x.com/2020/12/28/r7pyrQ.md.jpg)](https://imgchr.com/i/r7pyrQ)

从high级别代码可知，其同low规律一样多了个MD5加密。

##### impossible

```
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $cookie_value = sha1(mt_rand() . time() . "Impossible");
    setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], true, true);
}
?> 
```

impossible级别代码，首先使用mt_rand()函数随机选取一个数+时间戳+字符串"Impossible"，然后将这段字符串进行了sha1运算，给浏览器作为cookie。

这样就很难找到规律进行伪造了。


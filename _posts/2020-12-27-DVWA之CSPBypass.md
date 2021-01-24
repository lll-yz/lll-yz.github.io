---
layout:    post
title:     DVWA之CSP Bypass
subtitle:  学习学习
date:      2020-12-27
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:

   - DVWA
---

#### CSP Bypass

##### CSP简介

**csp** 全称是：Content-Security-Policy,内容安全策略。是指HTTP返回报文头中的标签，浏览器会根据标签中的内容，判断哪些资源可以加载或执行。主要是为了 **缓解潜在的跨站脚本问题(XSS)** ，浏览器的扩展程序系统引入了内容 **安全策略** 这个概念。原来应对XSS攻击时，主要采用函数过滤转义输入中的同时字符、标签、文本来规避攻击。CSP的实质就是 **白名单制度** ，开发人员明确告诉客户端，哪些外部资源可以加载和执行。开发者只需要提供配置，实现和执行全部由浏览器完成。

**说明：**

+ script-src脚本：只信任当前域名
+ object-src：不信任任何URL，即不加载任何资源
+ style-src样式表：只信任``http://cdn.example.org``和``http://third-party.org`` 

##### low

```
<?php

$headerCSP = "Content-Security-Policy: script-src 'self' https://pastebin.com hastebin.com example.com code.jquery.com https://ssl.google-analytics.com ;"; // allows js from self, pastebin.com, hastebin.com, jquery and google analytics.

header($headerCSP);

# These might work if you can't create your own for some reason
# https://pastebin.com/raw/R570EE00
# https://hastebin.com/raw/ohulaquzex

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    <script src='" . $_POST['include'] . "'></script>
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>You can include scripts from external sources, examine the Content Security Policy and enter a URL to include here:</p>
    <input size="50" type="text" name="include" value="" id="include" />
    <input type="submit" value="Include" />
</form>
';
```

可以看到

```
script-src 'self' https://pastebin.com hastebin.com example.com code.jquery.com https://ssl.google-analytics.com ;"; // allows js from self, pastebin.com, hastebin.com, jquery and google analytics.
```

允许访问pastebin网站，所以去这个网站看看，emmm打不开，源码中也有现成测试站点：``https://pastebin.com/raw/R570EE00``和``https://hastebin.com/raw/ohulaquzex``。第二个可以使用：

[![r5v3K1.png](https://s3.ax1x.com/2020/12/27/r5v3K1.png)](https://imgchr.com/i/r5v3K1)

原理为因为可以访问白名单，所以我们可以在其白名单的网站编辑XSS脚本，然后包含该网站实现XSS攻击。

##### medium

```
<?php

$headerCSP = "Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=';";

header($headerCSP);

// Disable XSS protections so that inline alert boxes will work
header ("X-XSS-Protection: 0");

# <script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert(1)</script>

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>Whatever you enter here gets dropped directly into the page, see if you can get an alert box to pop up.</p>
    <input size="50" type="text" name="include" value="" id="include" />
    <input type="submit" value="Include" />
</form>
';
```

**unsafe-inline：** 允许使用内联资源，如内联``<script>``元素，``javascript:URL``，内联事件处理程序(如：onclick)和内联``<style>``元素。必须包括单引号。

**nonce-source：** 仅允许特定的内联脚本块，``nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA="`` 

这里将所有的安全站点都去掉了，只能使用script元素且脚本块值nonce=TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=。

构建payload：``<script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert('hhh')</script>``

[![rISgIK.png](https://s3.ax1x.com/2020/12/27/rISgIK.png)](https://imgchr.com/i/rISgIK)

##### high

```
<?php
$headerCSP = "Content-Security-Policy: script-src 'self';";

header($headerCSP);

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>The page makes a call to ' . DVWA_WEB_PAGE_TO_ROOT . '/vulnerabilities/csp/source/jsonp.php to load some code. Modify that page to run your own code.</p>
    <p>1+2+3+4+5=<span id="answer"></span></p>
    <input type="button" id="solve" value="Solve the sum" />
</form>

<script src="source/high.js"></script>
';
```

"Content-Security-Policy: script-src 'self';"; 只允许代码只能从本地运行，最后会运行一个JavaScript代码

```
function clickButton() {
    var s = document.createElement("script");
    s.src = "source/jsonp.php?callback=solveSum";
    document.body.appendChild(s);
}

function solveSum(obj) {
    if ("answer" in obj) {
        document.getElementById("answer").innerHTML = obj['answer'];
    }
}

var solve_button = document.getElementById ("solve");

if (solve_button) {
    solve_button.addEventListener("click", function() {
        clickButton();
    });
}
```

在点击网页的按钮使js生成一个script标签(src 指向source/jsonp.php?callback=solveSum)，**document对象**使我们可以从脚本中对HTML页面中的所有元素进行访问，**createElement()**方法通过指定名称创建一个元素，**body属性**提供对``<body>``元素的直接访问，对于定义了框架集的文档将引用最外层的``<frameset>``。**appendChild()方法**可向结点的子节点列表的末尾添加新的子节点，也就是网页会把"source/jsonp.php?callback=solveSum"加入到DOM中。源码定义了solveSum的函数，函数传入参数obj，如果字符串"answer"在obj中就会执行。**getElementById()方法**可返回对拥有指定ID的第一个对象的引用，**innerHTML属性**设置或返回表格行的开始和结束标签之间的HTML。这里的script标签会把远程加载的solveSum({"answer":"15"})当作js代码执行，然后这个函数就会在页面显示答案。

返回上方源码：

```
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
```

发现还有一个接收include参数，从这里入手：

```
include=<script src="source/jsonp.php?callback=alert('oh');"></script>
```

[![rIkZQO.md.png](https://s3.ax1x.com/2020/12/27/rIkZQO.md.png)](https://imgchr.com/i/rIkZQO)

##### impossible

```
<?php

$headerCSP = "Content-Security-Policy: script-src 'self';";

header($headerCSP);

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>Unlike the high level, this does a JSONP call but does not use a callback, instead it hardcodes the function to call.</p><p>The CSP settings only allow external JavaScript on the local server and no inline code.</p>
    <p>1+2+3+4+5=<span id="answer"></span></p>
    <input type="button" id="solve" value="Solve the sum" />
</form>

<script src="source/impossible.js"></script>
';
```

js代码：

```
 function clickButton() {
    var s = document.createElement("script");
    s.src = "source/jsonp_impossible.php";
    document.body.appendChild(s);
}

function solveSum(obj) {
    if ("answer" in obj) {
        document.getElementById("answer").innerHTML = obj['answer'];
    }
}

var solve_button = document.getElementById ("solve");

if (solve_button) {
    solve_button.addEventListener("click", function() {
        clickButton();
    });
}

```

impossible级别代码中，仍然调用了jsonp但不使用callback参数无法从这里入手了，无法攻击。


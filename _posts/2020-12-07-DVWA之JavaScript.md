---
layout:    post
title:     DVWA之JavaScript
subtitle:  学习学习
date:      2020-12-07
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:

   - DVWA
---

#### JavaScript

##### low

```
<?php
$page[ 'body' ] .= <<<EOF
<script>
	function rot13(inp) {
        return inp.replace(/[a-zA-Z]/g,function(c){return String.fromCharCode((c<="Z"?90:122)>=(c=c.charCodeAt(0)+13)?c:c-26);});
    }

    function generate_token() {
        var phrase = document.getElementById("phrase").value;
        document.getElementById("token").value = md5(rot13(phrase));
    }

    generate_token();
</script>
EOF;
?>
```

首先根据提示输入 success，提示token不对

[![Dx7kdO.png](https://s3.ax1x.com/2020/12/07/Dx7kdO.png)](https://imgchr.com/i/Dx7kdO)

查看页面源码

[![DxXPOA.md.png](https://s3.ax1x.com/2020/12/07/DxXPOA.md.png)](https://imgchr.com/i/DxXPOA)

[![DxXnSg.md.png](https://s3.ax1x.com/2020/12/07/DxXnSg.md.png)](https://imgchr.com/i/DxXnSg)

发现token生成在前端，将ChangeMe修改为success，再在控制台执行一次js完成修改，之后在输入框中输入success，成功。

[![Dxju4K.png](https://s3.ax1x.com/2020/12/07/Dxju4K.png)](https://imgchr.com/i/Dxju4K)

##### medium

```

<?php
$page[ 'body' ] .= '<script src="' . DVWA_WEB_PAGE_TO_ROOT . 'vulnerabilities/javascript/source/medium.js"></script>';
?>
```

与low一样，但js代码放在另一文件中

```
function do_something(e){for(var t="",n=e.length-1;n>=0;n--)t+=e[n];return t}setTimeout(function(){do_elsesomething("XX")},300);function do_elsesomething(e){document.getElementById("token").value=do_something(e+document.getElementById("phrase").value+"XX")}
```

**do_elsesomething(e)：**  将一个字符串翻转，就是倒放。

**setTimeout(function(){do_elsesomething("XX")},300):** 每300毫秒执行一次do_elsesomething("XX")

**do_elsesomething(e):** 调用do_something(e)函数来设置token值，其中e=XX+phrase+XX

总体含义：该函数通过将phrase进行字符串拼接，即XX+phrase+XX，最后将该值反转设置为token。

在输入框输入success，再在控制台输入do_elsesomething("XX")执行，提交即可。

##### high

```
<?php
$page[ 'body' ] .= '<script src="' . DVWA_WEB_PAGE_TO_ROOT . 'vulnerabilities/javascript/source/high.js"></script>';
?>
```

同medium差不多，但high.js 这里的js代码被混淆了。在-->[here](http://deobfuscatejavascript.com
)可以破解混淆。关键代码如下：

```
function do_something(e) {
    for (var t = "", n = e.length - 1; n >= 0; n--) t += e[n];
    return t
}
function token_part_3(t, y = "ZZ") {
    document.getElementById("token").value = sha256(document.getElementById("token").value + y)
}
function token_part_2(e = "YY") {
    document.getElementById("token").value = sha256(e + document.getElementById("token").value)
}
function token_part_1(a, b) {
    document.getElementById("token").value = do_something(document.getElementById("phrase").value)
}
document.getElementById("phrase").value = "";
setTimeout(function() {
    token_part_2("XX")
}, 300);
document.getElementById("send").addEventListener("click", token_part_3);
token_part_1("ABCD", 44);
```

token生成步骤为：

1.由于有300毫秒延时，所以先执行了token_part_1("ABCD", 44);

2.然后执行token_part_2("XX")

3.token_part_3被添加到提交按钮的click事件上，也就是提交时会触发执行。

在输入框输入success，再在控制台输入token_part_1("ABCD",44)和token_part_2("XX")，最后提交即可。

##### Impossible

You can never trust anything that comes from the user or prevent them from messing with it and so there is no impossible level. 

哦豁，不相信我们! 那就没办法了。


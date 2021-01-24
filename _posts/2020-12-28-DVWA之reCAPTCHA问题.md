---
layout:    post
title:     DVWA之reCAPTCHA API key missing from config file问题解决
subtitle:  学习学习
date:      2020-12-28
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:

   - DVWA
---

##### reCAPTCHA API key missing from config file问题解决

进入DVWA之Insecure CAPTCHA(不安全的验证码)时，出现报错：

[![r7FBnO.png](https://s3.ax1x.com/2020/12/28/r7FBnO.png)](https://imgchr.com/i/r7FBnO)

原因是因为使用reCAPTCHA没有申请密钥，因此需要手动填入密钥，打开提示的配置文件，找到

```
$_DVWA[ 'recaptcha_public_key' ]  = '';
$_DVWA[ 'recaptcha_private_key' ] = '';
```

改为：

```
$_DVWA[ 'recaptcha_public_key' ]  = '6LdK7xITAAzzAAJQTfL7fu6I-0aPl8KHHieAT_yJg';
$_DVWA[ 'recaptcha_private_key' ] = '6LdK7xITAzzAAL_uw9YXVUOPoIHPZLfw2K1n5NVQ';
```

刷新一下成功。
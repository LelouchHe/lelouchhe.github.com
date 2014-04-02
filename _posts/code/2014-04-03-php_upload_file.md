---
layout: post
title: php的文件上传
category: code
tag: php
---

## 问题的来源

以前为公司的移动端开发了一个php的代理接口,调用方可以通过这个接口,调用其他接口.加这个间接层的目的,主要是为了给调用添加一些调用方无法获取的验证参数,这些参数,只有有限几个地方可以获取并使用

代理接口本来有个雏形,但前几天的代理请求,一直有问题,仔细一看,原来是没有考虑php的文件上传功能,导致一些有上传文件的请求会失败

## 接收上传文件

上传的文件,肯定是以POST的方式传给php服务器的,但这个文件,却不在$\_POST参数中,相反,php内部有个专门的内部变量$\_FILES,用来专门保存这些文件信息的.

$\_FILES是预定义的全局变量,是一个保存了上传文件基本信息的词典,key是上传文件的字段名称,value同样是一个词典,保存了基本信息.假设上传文件时使用的字段是'upload_file',原文件是一张大小12345B的图片,名称为'original_file',那么下面就是一项$\_FILES:

    $_FILES['upload_file']['name'] = 'original_file';
    $_FILES['upload_file']['type'] = 'image/jpg';
    $_FILES['upload_file']['size'] = 12345;
    $_FILES['upload_file']['tmp_name'] = '/tmp/phpabcdef'; // 临时存放处
    $_FILES['upload_file']['error'] = UPLOAD_ERR_OK;

上传到服务器的文件路径保存在tmp_name中,我们可以当做普通文件来进行处理,唯一的不同时,脚本结束后,该临时文件会被删除,所以如果需要一些持久化的操作,需要手动人工来弄.

## 通过http上传文件

我们主要通过curl来模拟http的上传文件.我们来看一个上传的样例:

    $url = 'http://sample.com/upload.php';
    $data = array();
    $data['pic'] = '@' . $_FILES['upload_file']['tmp_name'];

    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

    $r = curl_exec($ch);

    curl_close($ch);

### 文件的表示

我们不直接处理文件数据,相反,我们需要直接指明文件名称,并把对应文件名的绝对路径前加上'@'来赋值给请求字段即可

例子中的字段为'pic',如果'upload.php'接收到了请求,那么'upload.php'中的$\_FILES中,就会出现'pic'这个key了

### POST的类型

就目前碰到的而言,POST一般有两个类型:

* **application/x-www-form-urlencoded**: 这个是常规的POST请求,会把每个key-value都urlencode之后,用'&'连接起来.作用相当于'http_build_query'的调用效果.设置'CURLOPT_POSTFIELDS'时,如果采用字符串形式的话,就是这种类型
* **multipart/form-data**: 将POST请求分割成不同的段,可以在各个段中添加任意的内容(包括二进制内容).设置'CURLOPT_POSTFIELDS'时,如果采用词典格式(如本例直接传了array),就会赋值这个类型.上传文件一般只能使用这个类型.

两种类型,除了上传文件是由$\_FILES处理,其余的参数走的还是$\_POST,所以这两种类型,本质上没有太大差别.

注意,虽然上面这些都属于提交的'Content-type',是属于http头部信息的,但是要是我们随后对head进行赋值,也不会覆盖这些的(可以把这些看做是系统的高优先级设置).比如,就算像下面这样设置了'Content-type',也是不起作用的

    curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-type: text/plain'));

所以,我们可以大胆的添加我们自己的head信息,而不用担心与原系统的冲突

## php相关配置

上传文件除了php文件本身的处理外,php的配置也是比较重要的,可以控制的性质有临时文件夹路径,临时文件大小等,我们都可以在'php.ini'里找到.一个建议是grep一下'upload',就能看到大部分配置项了

## 总结

大体上,php中比较方便的文件上传,就这些了.主要还是$\_FILES的处理,和curl对http POST的模拟.一些细节问题比较多,所以在开发过程中还是要多看文档,多调试.

其他内容,可以参考[php官网][php]

[php]: http://www.php.net/manual/en/features.file-upload.post-method.php "Handling file uploads"

---
title: PHP异步工作避免程序运行超时
date: 2017-04-24 14:08:10
tags:
  - php
  - 超时处理
  - 异步
categories: 
  - 开发
---

# 案例
某SNS社区要求用户给自己好友（好友数量上百个）发送邮件，每封邮件内容不一，发送后提示发送完毕！
# 常用PHP写法
`sendmail.php`
```php
<?php
$count=count($emailarr);//$emailarr数组为好友的邮件地址
for($i=0;$i<$count;$i++){
    //sendmail(.....);//发送邮件
}
echo '发送完毕';
```
假设该次发送100封邮件。本次操作会出现什么结果呢?  

用户体验：用户等待->发送数十封邮件出去->系统超时返回错误信息
 
本次操作由于需要发送大量的邮件，导致php执行时间过长，用户烦躁的等待。当apache或者nginx等待超过允许执行时间，返回超时错误。这个时候用户不明确本次操作到底成功与否，到底发出了几封邮件。
我们可以看出该代码用户体验极差，并且不能够顺利完成任务。  

那应该怎么操作呢?

# 异步执行
这里提到一个概念，异步执行
用户体验：用户等待->发送完毕
朋友们就会问，怎么缺少发信环节？
OK，发信环节就在用户提交请求的时候，把发信任务转给了一个单独处理发信的php程序处理了，当用户看见“发送完毕”的时候其实信还没发送完，这个时候，发信程序正在后台努力的工作着，一封一封的向外发送
 
## sendmail.php
```php
<?php
$domain="www.***.com";
$url="/system_mail.php";
$par="email=".implode(',',$emailarr)."&........";
$header = "POST $url HTTP/1.0\r\n";
$header .= "Content-Type: application/x-www-form-urlencoded\r\n";
$header .= "Content-Length: " . strlen($par) . "\r\n\r\n";
$fp = @fsockopen ($domain, 80, $errno, $errstr, 30);
fputs ($fp, $header . $par);
fclose($fp);
echo '发送完毕';
```
## system_mail.php
```php
<?php
//保证php执行不超时
set_time_limit (0);
 
//设置当客户机与服务器断开连接后是否终止脚本的执行， 这里设置为true表示继续在后台执行脚本
ignore_user_abort(true);
 
//获取email地址，发信，此处为发信代码
```
好了，改成异步方式后，用户提交信息，可以立即得到结果“发送完毕”。信呢，会在后台一封一封的发送，直到发送完毕
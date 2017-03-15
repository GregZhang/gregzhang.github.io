---
title: php加密解密操作
date: 2017-03-01 11:40:07
tags:
  - php
  - 对称加密
categories: 
  - 开发
---
# 加密算法
```php
<?php
function encrypt($data, $key) {
    $key = md5($key);
    $x  = 0;
    $len = strlen($data);
    $l  = strlen($key);
    $char ='';
    for ($i = 0; $i < $len; $i++)
    {
        if ($x == $l) 
        {
         $x = 0;
        }
        $char .= $key{$x};
        $x++;
    }
    $str ='';
    for ($i = 0; $i < $len; $i++) {
        $str .= chr(ord($data{$i}) + (ord($char{$i})) % 256);
    }
    return base64_encode($str);
}

```

# 解密算法
```php
<?php
function decrypt($data, $key){
    $key = md5($key);
    $x = 0;
    $data = base64_decode($data);
    $len = strlen($data);
    $l = strlen($key);
    $char ='';
    for ($i = 0; $i < $len; $i++)
    {
        if ($x == $l) 
        {
         $x = 0;
        }
        $char .= substr($key, $x, 1);
        $x++;
    }
    $str ='';
    for ($i = 0; $i < $len; $i++) {
        if (ord(substr($data, $i, 1)) < ord(substr($char, $i, 1)))
        {
            $str .= chr((ord(substr($data, $i, 1)) + 256) 
                - ord(substr($char, $i, 1)));
        }
        else
        {
            $str .= chr(ord(substr($data, $i, 1)) 
                - ord(substr($char, $i, 1)));
        }
    }
    return $str;
}
```

# 演示

上述加密解密的过程均需要用到一个加密密钥(即参数$key).
```php
<?php
function encrypt($data, $key) {
    $key = md5($key);
    $x  = 0;
    $len = strlen($data);
    $l  = strlen($key);
    $char ='';
    for ($i = 0; $i < $len; $i++)
    {
        if ($x == $l) 
        {
         $x = 0;
        }
        $char .= $key{$x};
        $x++;
    }
    $str ='';
    for ($i = 0; $i < $len; $i++) {
        $str .= chr(ord($data{$i}) + (ord($char{$i})) % 256);
    }
    return base64_encode($str);
}
function decrypt($data, $key){
    $key = md5($key);
    $x = 0;
    $data = base64_decode($data);
    $len = strlen($data);
    $l = strlen($key);
    $char ='';
    for ($i = 0; $i < $len; $i++)
    {
        if ($x == $l) 
        {
         $x = 0;
        }
        $char .= substr($key, $x, 1);
        $x++;
    }
    $str ='';
    for ($i = 0; $i < $len; $i++) {
        if (ord(substr($data, $i, 1)) < ord(substr($char, $i, 1)))
        {
            $str .= chr((ord(substr($data, $i, 1)) + 256) 
                - ord(substr($char, $i, 1)));
        }
        else
        {
            $str .= chr(ord(substr($data, $i, 1)) 
                - ord(substr($char, $i, 1)));
        }
    }
    return $str;
}
$data = 'PHP加密解密算法';  // 被加密信息
$key = '123';     // 密钥
$encrypt = encrypt($data, $key);
$decrypt = decrypt($encrypt, $key);
echo $encrypt, "\n", $decrypt;
```

上述将输出类似如下结果：
```bash
gniCSOzZG+HnS9zcFea7SefNGhXF
PHP加密解密算法
```
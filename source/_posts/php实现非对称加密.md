---
title: php实现非对称加密[转载]
date: 2017-03-03 14:02:27
tags:
  - php
  - 加密
  - 非对称加密
categories: 
  - 开发
---
php实现非对称加密[[转载]](http://blog.csdn.net/su_dong/article/details/52684202)
使用非对称加密主要是借助openssl的公钥和私钥，用公钥加密私钥解密，或者私钥加密公钥解密。

# 安装
安装openssl和PHP的openssl扩展

# 生成私钥
openssl genrsa 用于生成rsa私钥文件，生成是可以指定私钥长度和密码保护

```bash
openssl genrsa -out rsa_private_key.pem 1024    
```

# 生成公钥
rsa命令用于处理RSA密钥、格式转换和打印信息

```bash
openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem   
```
# 加密解密
这里我们使用私钥加密，公钥解密
```php
<?php
//密钥文件的路径
$privateKeyFilePath = 'rsa_private_key.pem';
//公钥文件的路径
$publicKeyFilePath = 'rsa_public_key.pem';

extension_loaded('openssl') or die('php需要openssl扩展支持');

(file_exists($privateKeyFilePath) && file_exists($publicKeyFilePath))
    or die('密钥或者公钥的文件路径不正确');
//生成Resource类型的密钥，
//如果密钥文件内容被破坏，
//openssl_pkey_get_private函数返回false
$privateKey = openssl_pkey_get_private(file_get_contents($privateKeyFilePath));
//生成Resource类型的公钥，
//如果公钥文件内容被破坏，
//openssl_pkey_get_public函数返回false
$publicKey = openssl_pkey_get_public(file_get_contents($publicKeyFilePath));

($privateKey && $publicKey) or die('密钥或者公钥不可用');
//原数据
$originalData = '加密前hahahaha';
//加密以后的数据，用于在网路上传输 
$encryptData = '';

echo '原数据为:', $originalData, PHP_EOL;

///////////////////////////////用私钥加密////////////////////////
if (openssl_private_encrypt($originalData, $encryptData, $privateKey)) { 

    /**    
     * 加密后 可以base64_encode后方便在网址中传输 或者打印  否则打印为乱码    
     */
    echo '加密成功，加密后数据(base64_encode后)为:'
        , base64_encode($encryptData)
        , PHP_EOL;

} else {
    die('加密失败');
}


///////////////////////////////用公钥解密//////////////////////// 

//解密以后的数据
$decryptData ='';

if (openssl_public_decrypt($encryptData, $decryptData, $publicKey)) { 

    echo '解密成功，解密后数据为:', $decryptData, PHP_EOL;

} else {
    die('解密成功');
}
```
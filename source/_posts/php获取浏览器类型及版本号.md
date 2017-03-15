---
title: php获取浏览器类型及版本号
date: 2017-03-10 13:47:13
tags:
  - php
  - 浏览器类型
  - 浏览器版本号
categories: 
  - 开发
---

直接上代码
```php
<?php
function getBrowser(){
    $agent=$_SERVER["HTTP_USER_AGENT"];
    if(strpos($agent,'MSIE')!==false || strpos($agent,'rv:11.0')) //ie11判断
    return "ie";
    else if(strpos($agent,'Firefox')!==false)
    return "firefox";
    else if(strpos($agent,'Chrome')!==false)
    return "chrome";
    else if(strpos($agent,'Opera')!==false)
    return 'opera';
    else if((strpos($agent,'Chrome')==false)&&strpos($agent,'Safari')!==false)
    return 'safari';
    else 
    return 'unknown';
}

function getBrowserVer(){
    //当浏览器没有发送访问者的信息的时候
    if (empty($_SERVER['HTTP_USER_AGENT'])){    
        return 'unknow';
    }
    $agent= $_SERVER['HTTP_USER_AGENT'];   
    if (preg_match('/MSIE\s(\d+)\..*/i', $agent, $regs)){
        return $regs[1];
    } elseif (preg_match('/FireFox\/(\d+)\..*/i', $agent, $regs)){
        return $regs[1];
    } elseif (preg_match('/Opera[\s|\/](\d+)\..*/i', $agent, $regs)){
        return $regs[1];
    } elseif (preg_match('/Chrome\/(\d+)\..*/i', $agent, $regs)){
        return $regs[1];
    } elseif ((strpos($agent,'Chrome')==false)
        &&preg_match('/Safari\/(\d+)\..*$/i', $agent, $regs)){
        return $regs[1];
    } else {
        return 'unknow';
    }
}

print_r(getBrowser());
print_r(getBrowserVer());
echo '<br>'.$_SERVER["HTTP_USER_AGENT"];
```
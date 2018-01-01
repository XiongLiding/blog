---
title: 在 PHP 中实现带 WSDL 的 SOAP
categories:
  - 软件开发
tags:
  - PHP
  - WSDL
date: 2014-12-10 09:40:36
---

1. 用 composer 安装生成 WSDL 所需的库
```bash
composer require piotrooo/wsdl-creator
```
2. 实现用于外部访问的入口文件，代码示例请参考（其中方法名和参数中出现的 `Notify` 对应一个类名，该类的方法将成为可以通过 SOAP 调用的外部接口）：
```php
<?php
use WSDL\DocumentLiteralWrapper;
use WSDL\WSDLCreator;
use WSDL\XML\Styles\DocumentLiteralWrapped;
class Api
{
    public static function soapNotify()
    {
        $host = $_SERVER['HTTP_HOST'];
        $soapuri = "http://{$host}/Api/soapNotify";
        if (isset($_GET['wsdl'])) {
            $wsdl = new WSDLCreator('Notify', $soapuri);
            $wsdl-&gt;renderWSDL();
            exit;
        }
        $server = new SoapServer(null, [
            'uri' =&gt; $soapuri
        ]);
        $server-&gt;setClass('Notify');
        $server-&gt;handle();
    }
}
```
3. 实现包含接口功能逻辑的类文件，代码示例请参考（其中方法的注释相当重要，是 wsdl-creator 正确生成 WSDL 的依据，必须严格按照格式进行注释）：
```php
<?php
class Notify
{
    /**
     * @desc sendText 向患者的微信发送文本信息
     * @param string $toUser 发给哪个用户
     * @param string $content 发送的内容
     * @return string $result
     */
    public function sendText($toUser, $content)
    {
        if (!$toUser || !$content) {
            return E::INVALID;
        }
        $user = G::xpdo()->row("SELECT * FROM `users` WHERE `patientid` = ?", [$toUser]);
        if (!$user) {
            return E::NODATA;
        }
        $api = G::xpdo()->row("SELECT * FROM `api` WHERE `token` = ?", [$user['token']]);
        $weixin = new Weixin($api['appid'], $api['appsecret']);
        $weixin->sendText($user['openid'], $content);
        return 0;
    }
}
4.  使用 SoapUI 软件对接口进行调试
5.  编制相应的接口调用说明文档 

###### 参考资料：
* 了解 SOAP 的基本文档结构请参考 [http://www.sitepoint.com/series/creating-web-services-with-php-and-soap/]](http://www.sitepoint.com/series/creating-web-services-with-php-and-soap/])
* 关于生成 WSDL 的库 wsdl-creator 详见： [https://github.com/piotrooo/wsdl-creator](https://github.com/piotrooo/wsdl-creator)

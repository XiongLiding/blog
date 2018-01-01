---
title: 'Hash, PushState 和微信 JSSDK 授权'
categories:
  - 微信
tags:
  - 微信
  - JSSDK
  - JavaScript
date: 2016-12-15 15:33:00
---

最近将 riot.js 升级到了 3.0，并用上了新版本的 riot-route，原先用了一年多的 2.2.4 版本内置的 riot.route 只支持 hash 形式的 SPA 单页面应用，riot-route 则支持 pushState。

Hash 方式有个缺点，就是服务器不知道地址栏中 # 之后的内容，放在微信里，就导致了未授权用户授权后想返回原界面需要借助 JS 来实现，导致历史记录多了一条，用户按返回键无法退出（返回上一页面后又被 JS “前进”了）。

PushState 方式就没有这个问题，可以直接用 HTTP 302 重定向过来，不影响历史记录和返回按钮功能。

于是在新项目中开始采用 pushState 方式。

不过等到应用到了微信环境中，又冒出来一个问题，通过 pushState 改变地址在 iOS 中会导致 JSSDK 的授权失败。

在 config 中开启 debug，可见 invalid signature 。原因是 location.href 中的地址变化了，但是微信客户端认为地址还是打开页面时的地址，微信“复制链接”得到的地址可以作为旁证。

## 解决方法

先按标准方法用当前地址计算 signature，如果失败，再用打开浏览器时的地址计算 signature。

由于之前已经把加载微信 JSSDK 和相关 Api 的功能独立成函数，因此这次解决起来也比较简单。

###### 修改前的代码

```JavaScript
function runWx(api, fn, loadError) {  
    if (typeof api === 'string') {
        api = [api];
    }

    // 用 require.js 加载 JSSDK 文件，如果你已经用 <script> 方式加载，可以省掉这一步
    require(['jweixin'], function (wx) {
        var url = location.href.split('#')[0];

        // 将 url 发往服务器计算相应的 signature
        $.get('/signature', { url: url }, function (ans) {
            // wx.config 执行成功时调用
            wx.ready(function () {
                wx.checkJsApi({
                    jsApiList: _.clone(api), // 坑，微信会改变此参数的内容
                    success: function success(res) {
                        if (!res || !res.checkResult || _.any(api, function (v) {
                            return !res.checkResult[v];
                        })) {
                            alert('您的微信版本过低，无法使用此功能，请升级微信');
                            return;
                        }
                        fn(wx);
                    }
                });
            });

            // wx.config 执行失败时调用
            wx.error(function () {
                alert('授权失败，您可能无法使用部分功能');
            });

            // 校验用的参数来自服务器
            var config = {
                // debug: true,
                appId: ans.data.appId,
                timestamp: ans.data.timestamp,
                nonceStr: ans.data.nonceStr,
                signature: ans.data.signature,
                jsApiList: _.union(['checkJsApi'], api)
            };

            // 进行校验
            wx.config(config);
        });
    }, loadError);
}
```

先简单展示一下这个 runWx 函数的用法：

```JavaScript
// 当需要用到特定的微信接口时，运行 runWx
runWx(['uploadImage', 'chooseImage'], function (wx) {  
    // 可以在这里使用 wx.uploadImage 和 wx.chooseImage 功能
    wx.uploadImage(...);
    wx.chooseImage(...);
}, function () {
    // 加载失败时要做的事
});
```

runWx 做了这么几件事：

* 用 require.js 加载 weixin sdk 的 js 文件
* 获取当前页面地址 location.href.split('#')[0]
* 将 url 作为参数发送到服务器计算出 signature
* 调用 wx.config
* 成功时调用 wx.ready 并最终调用回调函数 fn
* 失败时调用 wx.error 提示错误

###### 修改后的代码

```JavaScript
// 解决部分机型 pushState 不能正确改变地址导致授权失败

// 在第一次打开页面时加载此文件，记录当时的地址作为原始地址
var originUrl = location.href.split('#')[0];

// 增加 tryOrigin 参数
function runWx(api, fn, loadError, tryOrigin) {  
    if (typeof api === 'string') {
        api = [api];
    }
    require(['jweixin'], function (wx) {
        // tryOrigin 为真时使用原始地址
        var url = tryOrigin ? originUrl : location.href.split('#')[0];

        $.get('/signature', { url: url }, function (ans) {
            wx.ready(function () {
                wx.checkJsApi({
                    jsApiList: _.clone(api), // 坑，微信会改变此参数的内容
                    success: function success(res) {
                        if (!res || !res.checkResult || _.any(api, function (v) {
                            return !res.checkResult[v];
                        })) {
                            alert('您的微信版本过低，无法使用此功能，请升级微信');
                            return;
                        }
                        fn(wx);
                    }
                });
            });

            wx.error(function () {
                // 已经试了原始地址
                if (originUrl === url) {
                    alert('授权失败，您可能无法使用部分功能');
                    return;
                }
                // 没有使用原始地址且 signature 不匹配，尝试用原始地址计算 signature
                runWx(api, fn, loadError, true);
            });

            var config = {
                debug: true,
                appId: ans.data.appId,
                timestamp: ans.data.timestamp,
                nonceStr: ans.data.nonceStr,
                signature: ans.data.signature,
                jsApiList: _.union(['checkJsApi'], api)
            };
            wx.config(config);
        });
    }, loadError);
}
```

和原来相比，主要变化有：

* 记录了打开浏览器时的原始地址
* 先尝试用标准的 location.href.split('#')[0] 计算 signature
* 失败时用 originUrl 再试一次

这个改动的主要优点是原来用到 runWx 的地方，代码完全不需要进行变动，由 runWx 自己去尝试解决问题。

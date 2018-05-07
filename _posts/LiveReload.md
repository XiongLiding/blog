---
title: SSE 与 LiveReload
date: 2018-05-07 08:25:54
tags:
categories: DevOps
---

之前实现了自己的编译工具 [dev.js](/2018/02/13/dev-js) ，用了快三个月，感觉良好，可以彻底把 gulp 甩掉了。不过上次也留下一个小尾巴，整合自动重载功能。

最初的打算是用 live.js 这个方案，因为无需改动自己的代码，只要把 live.js 加载到网页中就可以。不过实际用起来，发现还是有点问题。live.js 本身功能是正常的，但其机制是轮询，会在 network 里留下很多请求记录，调试时看起来很乱。

备选方案考虑了 WebSocket ，不过看起来不如 SSE 简单，如果想要用起来简单，就得上 socket.io 这样的库。不过既然 sse 已经够用，省下一个库自然更好。

<!--more-->

# SSE
SSE 的全称是 Server-sent events，也就是服务器推送事件，SSE 走的也是 http 协议，连接建立完毕后，数据是单向的，只能从服务器往客户端发送。

SSE 非常像以前用隐藏的 iframe 实现长连接(long-polling)的加强版，客户端和服务器建立连接后，连接一直保持，并等待服务器返回数据，在一个连接中服务器可以多次发送数据给客户端，如果连接意外断开，客户端还会自动发起重连。SSE 的事件机制也简化了解析服务器发来的数据的过程。

# LiveReload
LiveReload 是指服务器上的代码发生变化后，自动刷新浏览器，这样就避免了手动切换到浏览器刷新，再回到编辑器这样的来回切换。如果屏幕够大，两个程序平铺开来，就可以只管在编辑器打字，然后顺便瞧瞧浏览器上的变化，在调整样式尤其高效。

再进一步，叫做 HotReload，不刷新整个网页，只更新发生变化的部分代码，同步效率更高，不过这个实现起来就比较复杂，而且状态处理不好就会出现各种问题。

# 实现
接下来就在 dev.js 的基础上实现 LiveReload 功能，首先在原先的编译结果中，加入一行代码建立 SSE 连接

```
http.createServer((req, res) => {
    res.setHeader('Content-Type', 'text/javascript');
    res.setHeader('Accept-Charset', 'utf-8');
    const tags = Object.values(cacheTags).sort().join('\n');
+   const liveReload = req.url !== '/release' ? `new EventSource('http://${req.headers.host}/live').addEventListener('reload', () => location.reload());\n` : ''; //<1>
    const hash = crypto.createHash('md5').update(tags).digest("hex");
    res.setHeader('ETag', hash);
+   res.end(liveReload + tags); //<2>
}).listen(process.env.PORT, (err) => {
    log(`端口：${ process.env.PORT }`);
});
```

1. 通过 `new EventSource()` 建立了一个 SSE 连接，参数指向服务器地址 `/live`，然后监听服务器发来的 `reload` 事件，触发刷新浏览器的操作。三元操作服的判断是为了让这行代码只在测试环境中启用，编译正式环境代码时忽略。
2. 把第一步的代码嵌入发送给浏览器的 JavaScript 文件中。

上面的代码发起了连接，服务器也得准备好相应的代码进行对接

```
http.createServer((req, res) => {
+   if (req.url == '/live') {
+        live = res;
+        live.setHeader('Access-Control-Allow-Origin', req.headers.origin);
+        live.setHeader('Content-Type', 'text/event-stream');
+        return;
+   }
    res.setHeader('Content-Type', 'text/javascript');
    res.setHeader('Accept-Charset', 'utf-8');
    const tags = Object.values(cacheTags).sort().join('\n');
    const liveReload = req.url !== '/release' ? `new EventSource('http://${req.headers.host}/live').addEventListener('reload', () => location.reload());\n` : ''; //<1>
    const hash = crypto.createHash('md5').update(tags).digest("hex");
    res.setHeader('ETag', hash);
    res.end(liveReload + tags); //<2>
}).listen(process.env.PORT, (err) => {
    log(`端口：${ process.env.PORT }`);
});
```

这里我们监听地址为 `/live` 的请求，如果客户端来建立 SSE 连接，我们就使之保持，等待代码发生变化时，再通知客户端。
这里设置了两个头部，一个是为了跨域，因为我的系统主体是 PHP，与 dev.js 运行在不同端口，需要跨域。
第二个是设置 Content-Type，text/event-stream 是 SSE 的标准类型。

这里把 res 保存在了全局的 live 变量中（不是一个好的做法，但够简单），后续想发送数据时，只要调用 live.send() 就可以了。将其稍加包装，做成一个函数

```
const clientReload = () => {
    if (live) {
        live.end('event: reload\ndata:{}\n\n');
        live = null
    }
};
```

这样就搞定了，然后在监听文件变化的代码中，等每次编译结束就调用 `clientReload()` ，即可刷新浏览器中的界面。

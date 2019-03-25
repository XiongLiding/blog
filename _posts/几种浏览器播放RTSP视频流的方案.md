---
title: 几种浏览器播放RTSP视频流的方案
date: 2019-01-20 12:24:38
tags:
    - FFmpeg
    - VLC
    - RTSP
    - HLS
---

上周又做了一次在浏览器中播放视频流的项目，不过以前做的是手机，这次做的是桌面。停车场出入口的摄像头作为源头提供RTSP协议的视频流，而浏览器不能直接播放，只有通过插件或者转码来实现这个需求。

要实现这个目的，可以采用的方案非常得多，有商业的也有开源的，这里主要列举一些开源的方案。

<!--more-->

# 方案
这里的方案都是我尝试过了的，有些成功，有些没成功。但是因为每个项目情况不同，这次没成的方法，换个项目也许就能成，所以方案全部列在这里，失败的我也会说明原因。读者若走投无路，不妨也试试我失败了的方案。

由于项目原因，这里的浏览器主要考虑 Windows 平台。

## 顺序
由于方案很多，我们先归归类、排排序，从简单的到复杂的来。

# 1.通过浏览器插件直接播放RTSP
首先我们介绍没有中介的方案，也就是不预先进行转码，而在浏览器中直接解码RTSP流。

## 1.1 VLC插件
**优点：**
- 可以直接播放RTSP，无需任何中介服务器的帮助

**缺点：**
- 需要手动安装插件；
- 基于NPAPI，不被最新的 Chrome 和 Firefox 支持

**方法：**
安装 VLC 播放器时，可以选择安装浏览器插件，然后进入浏览器激活它们，最后在页面中嵌入一个简单的视频标签，完成。

```HTML    
    <embed type="application/x-vlc-plugin" id="vlc"
        pluginspage="http://www.videolan.org"
        target="rtsp://192.168.10.51:554"
        height="480" width="640" />
```

如果你项目的其他功能都能兼容客户电脑上的 IE 浏览器，这个方案就是首选。
我们的项目就没有这个幸运，我们只支持了现代浏览器，而现代浏览器抛弃了 NPAPI 。 

**参考链接：**
- [VLC wiki](https://wiki.videolan.org/Documentation:WebPlugin/)
- [插件安装方法](https://www.5kplayer.com/video-music-player/vlc-web-plugin-free-download-install.htm)

## 1.2 Flash
**优点：**
- 可以直接播放RTSP，但通常需要中介服务器
- 支持几乎所有桌面浏览器 

**缺点：**
- 需要手动安装插件（但大多数情况下已经安装了）

**方法：**

Flash 被不少用户痛恨，漏洞之王，耗电也厉害，但它确实能把事给办了。

不少视频网站包括直播网站都在用 Flash ，有不少基于 Flash 的在线视频播放器，我这次尝试的是 locomote ，一个结合 JavaScript 和 Flash 的开源解决方案。

把 dist 目录下的 .js 和 .swf 文件放到自己的项目里，然后通过 JavaScript 来调用 Flash 播放器播放 RTSP 视频。

```HTML
    <div id="player" style="width: 640px; height: 480px;"></div>
    <script src="locomote.min.js"></script>
    <script src="http://code.jquery.com/jquery-2.1.1.min.js"></script>
    <script type="text/javascript">
      $(document).ready(function() {
        /* Load SWF and instantiate Locomote */
        var locomote = new Locomote('player', 'Player.swf');

        /* Set up a listener for when the API is ready to be used */
        locomote.on('apiReady', function() {
          console.log('API is ready. `play` can now be called');

          /* Tell Locomote to play the specified media */
          locomote.play('rtsp://192.168.10.51:554');
        });

        /* Start listening for streamStarted event */
        locomote.on('streamStarted', function() {
          console.log('stream has started');
        });

        /* If any error occurs, we should take action */
        locomote.on('error', function(err) {
          console.log(err);
        });
      });
    </script>
```

如果你足够幸运，到此就完成了，不过大多数情况下并没有。

出于安全考虑，从 Flash 10 开始，需要一个 Socket Policy Server 来“授权”，才能允许 Flash 读取其他服务器上的内容，而且你读的视频流在哪个IP地址，Socket Policy Server 也必须在那个服务器上（默认843端口），而摄像头显然没有条件让你在上面跑另外一个服务。

这时候就需要一个中介服务器，我用 Nginx 的 stream 功能做了一个二进制流的反向代理来解决这个问题

    stream {
        server {
            listen 9051;
            proxy_pass 192.168.10.51:554;
        }
    }

这样，就把摄像头的 554 端口代理到了这个服务器的 9051 端口，Flash 那边的 locomote.play 里的地址要改成运行 Nginx 的服务器的 9051 端口。
然后在同一台服务器上运行一个服务，监听 843 端口，并返回授权信息。

下面的参考链接里可以下载到 flashpolicyd_v0.6.zip 这个压缩包，解压后找到并运行（需要 python 环境） 

    ./flashpolicyd.py --file=../policyfile.xml --port=843

plicyfile.xml 是政策文件，你可以先把里面的 domain 和 to-ports 改成 * 放开限制，把功能调通，确定功能正常了，再把设置改成实际的地址和端口增加安全性。

正确开启这个服务后，就可以在浏览器中看到视频了。

非常不幸，我栽在了就要抵达胜利的时候。

类似这个问题 [issues 117](https://github.com/AxisCommunications/locomote-video-player/issues/117)

摄像头发过来的协议头不标准，返回的 Content-Base 中没有协议

摄像头只返回了 `Content-Base 192.168.10.51`，[按照标准中的例子](https://tools.ietf.org/html/rfc7826#section-18.14)，应该是 `Content-Base rtsp://192.168.10.51`，当然，这个视频在VLC里是可以播放的，也可以说是播放器的兼容性不好，不管怎样，我失败了，希望你够幸运。

**参考链接：**

- [locomote@github](https://github.com/AxisCommunications/locomote-video-player)
- [Setting up a socket policy file server](https://www.adobe.com/devnet/flashplayer/articles/socket_policy_files.html)

# 2.转码方案
优点：
- 可以转换成各种编码来适应客户端的要求

缺点：
- 服务器需要较大的性能开销
- 需要维护额外的转码进程 

转码方案我也尝试了两类，一类是需要专门的视频流服务器的，另一类是基于 http 的

两种方案都用到了 FFmpeg 这个开源软件，FFmpeg 在编解码领域是标杆性的，大量的播放器和其他涉及视频编解码的软件都用采用了它。

在 FFmpeg3 中，内置了 ffserver 视频流服务软件，可惜在 2017 年初的 FFmpeg4 中已经被移除了，但是相当多的 Linux 发型版仍然使用 FFmpeg3 并包含 ffserver 。

## 2.1 ffserver
**优点：**
- 方便

**缺点：**
- ffserver 已经过时了

请注意，ffserver 已经**过时**了，项目组已经停止开发。

如果你的环境允许，使用 ffserver 会比直接使用 ffmpeg 免去很多进程管理上的工作，其配置文件大概是这个形式

    HTTPPort 9999
    HTTPBindAddress 0.0.0.0
    MaxHTTPConnections 2000
    MaxClients 1000
    MaxBandwidth 1000000
    CustomLog -

    <Stream stat.html>
    Format status
    ACL allow 127.0.0.1
    ACL allow 192.168.0.0 192.168.255.255
    </Stream>

    <Feed feed51.ffm>
    File /tmp/feed51.ffm
    FileMaxSize 200M
    Launch ffmpeg -i rtsp://192.168.10.51:554
    </Feed>

    <Stream feed51.webm>
    Format webm
    Feed feed51.ffm

    VideoCodec libvpx
    VideoFrameRate 30
    VideoBitRate 800
    VideoSize 720x576
    AVOptionVideo crf 23
    AVOptionVideo me_range 16
    AVOptionVideo qdiff 4
    AVOptionVideo qmin 10
    AVOptionVideo qmax 51
    AVOptionVideo flags +global_header

    NoAudio
    </Stream>

其中 `<Stream stat.html>` 可以提供一个页面，让你通过浏览器查看整个 ffserver 的运行状况

`<Feed feed51.ffm>` 是视频来源的入口，通过设置 `Launch ffmpeg` 可以调用 ffmpeg 将参数中的视频源转成 ffserver 的内部格式，这个是 ffserver 的一大优点，你不用另外管理这些转码的进程，ffserver 会调用和执行它们

`<Stream feed51.webm>` 是视频供外部消费的出口，我选择 webm 这个格式，因为主流浏览器可以直接在 video 标签中播放这个格式，而不需要 JavaScript 插件。Stream 里有一行配置是 Feed feed51.ffm ，用来指明视频的来源，当你有多路视频的时候，根据这个来把各路 Feed 和 Stream 对应起来，后面的参数则是调整视频流的帧率、尺寸、声音等具体表现的参数。

这个方法成功了，不过不知道是服务器不给力还是 FFmpeg3 的性能不如 FFmpeg4 ，用这个方法转码的时候容易断掉，导致播放体验不好。

后来专门用 FFmpeg3 和 FFmpeg4 在同一台服务器里运行了一下，FFmpeg3 挂掉次数明显比 FFmpeg4 多，即使没挂掉的时候，命令行里提示的 Warning 也要多很多。

**参考资料：**
- [ffserver wiki](https://trac.ffmpeg.org/wiki/ffserver)

## 2.2 FFmpeg4 和 RTMP
既然 FFmpeg4 转码成功率更高，只好选择它，但是 FFmpeg4 没有对应的 ffserver 可以使用，必须另外找一个视频流服务软件作为代替，然后 Nginx 又登场了。

Linux 上需要安装一个组件 `nginx-mod-rtmp`，当然，不同发行版可能名称略有不同，Windows 上可以找第三方编译好的版本，官方下载的没有带这个组件。

在 Nginx 的配置文件中加入这些

    rtmp {
        server {
            listen 1935;
            chunk_size 4096

            application live {
                live on;
                record off;
            }
        }
    }

然后使用 ffmpeg 转码视频流，并将转码结果推到这个端口

    ffmpeg -i rtsp://192.168.10.51:554 -f flv rtmp://127.0.0.1:1935

RTMP 可以用前面提到的 Flash 播放器来播放，而且不需要另外配置 Socket Policy Server。

但在实际使用中，这个转码过的流在 VLC 中能够播放，在 Flash 中却失败了。证明转码是以及服务器的配置都是成功的，但此时已是周五下午，我实在没时间再进一步了，另一方面我拿 Flash 也没辙，它不提供任何错误信息，我就束手无策。

**参考资料：**
- [在 Nginx 中启用 RTMP 服务](https://obsproject.com/forum/resources/how-to-set-up-your-own-private-rtmp-server-using-nginx.50/)
- [Youtube 上的视频演示](https://www.youtube.com/watch?v=e_7bS8AOPlY)

## 2.3 FFmpeg4 和 HLS
这下只能回到以前成功过的方法了，想想还是有点凄惨，虽然花了两天时间学到不少东西，但终究走回了老路。

HLS 大体上是一种把视频切分成多个片段存在硬盘上，然后由一个 .m3u8 文件做索引的体系。RTSP 转 HLS 的过程，就是把 RTSP 流一段一段录像的过程，然后客户端再按照 .m3u8 文件中的指示，到服务器上把一段一段的录像下载过去再连起来播放，服务器上不停录下新的片段，客户端不停去下载新片段来播放，就实现了“直播”。与一般的基于视频流服务器的方式相比有以下特点：

优点：
- 基于 HTTP 协议，无需专门的视频流服务器
- 中间产物可见，方便调试
- 移动端的原生支持

缺点：
- 因为传输的最小单位是一段文件，而且需要先缓冲几段来保证视频顺畅，所以延迟较高

另外原先还有个缺点是需要自己管理已经播放过的视频片段，但做到最后发现 FFmpeg 已经实现自我管理，能把过时的文件清除掉了。

转码也只需要一条命令：

    ffmpeg -i rtsp://192.168.10.51:554 -c:v h264 -flags +cgop -g 30 -hls_time 1 -hls_flags delete_segments 51/live.m3u8

- `-c:v` 后面跟的是编码方式
- `-flags +cgop` 和 `-g 30` 说实话我还没弄清楚，只是官方的列子都有这个参数我就照抄了
- `-hls_time 1` 是视频片段的长度，单位是秒，可以适当长一点
- `-hls_flags delete_segments` 是让 ffmpeg 自动删除已经不在 .m3u8 中的旧文件
- `51/live.m3u8` 是索引文件的名称，视频片段会和索引文件放在同一个目录下。

我有四路视频，所以需要开 4 个转码进程，这样管理起来会比较麻烦，加上视频源可能意外中断，导致解码进程退出，所以必须有一个工具来管理他们。最后写了一个简单的 node.js 脚本来做这件事。

```JavaScript
    var respawn = require('respawn');

    [51,52,56,57].forEach((v) => {
        var monitor = respawn([
            'ffmpeg.exe',
            '-rtsp_transport', 'tcp',
            `-i`, `rtsp://192.168.10.${v}`,
            '-c:v', 'h264',
            '-flags', '+cgop',
            '-g', '30',
            '-hls_flags', 'delete_segments',
            '-hls_time', '10',
            `${v}/live.m3u8`
        ], {
            name: `live${v}`,          // set monitor name
            cwd: '.',              // set cwd
            maxRestarts:-1,        // how many restarts are allowed within 60s
                                    // or -1 for infinite restarts
            sleep:1000,            // time to sleep between restarts,
            kill:30000,            // wait 30s before force killing after stopping
        });

        monitor.start();
    });
```

`[51,52,56,57]` 这个数组里面是四个摄像头 IP 的尾部，然后利用 respawn 在一个循环里将它们都开起来。respawn 最重要的工作，是根据你设定的参数在进程意外退出时将他们重新开启，免除了手工维护。

转化开启后，需要配置 web 服务器，将生成文件所在的目录放到可以通过 http 协议访问到的地方，具体过程不再赘述。

最后，考虑到不是所有浏览器都支持 HLS，我们需要使用 hls.js 来播放视频流。

```HTML
<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
<video id="video"></video>
<script>
  var video = document.getElementById('video');
  if(Hls.isSupported()) {
    var hls = new Hls();
    hls.loadSource('http://192.168.20.45:5444/51/live.m3u8');
    hls.attachMedia(video);
    hls.on(Hls.Events.MANIFEST_PARSED,function() {
      video.play();
  });
 } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
    video.src = 'http://192.168.20.45:5444/51/live.m3u8';
    video.addEventListener('loadedmetadata',function() {
      video.play();
    });
  }
</script>
```

过程非常简单，放上一个 video 标签，引入 hls.js，然后用 hls.js 来加载刚才生成的视频流，完成。

**参考资料**

- [respawn@github](https://github.com/mafintosh/respawn)
- [ffmpeg hls](https://www.ffmpeg.org/ffmpeg-formats.html#hls-2)
- [hls.js](https://github.com/video-dev/hls.js)


# 杂项
这里是我在尝试的过程中遇到的问题，与一些相关的技术，如果你在上面的方法中仍然没有找到解决方案，可以试试下面的关键字。

- 在使用 ffmepg 转码的时候可能会遇到 `UDP timeout, retrying with TCP` 的提示，但是它并不会自己去换成 TCP，你需要在命令中加入 -rtsp_transport tcp 这个参数
- 如果你的客户可以接受弹出一个播放器的方式，你也可以考虑安装类似 `Open in VLC media player` 这样的浏览器插件将视频弹到播放器中进行播放。
- WebRTC 也是一种可以实现视频流的方式，但是我没找到一个直观的开源的服务器端解决方案，但是有看到正在讨论直接支持 RTSP 的动向，也许若干年以后，就可以直接用 WebRTC 技术来播放 RTSP 了。
- 还有一个叫 Simple STMP Server 的独立的开源流媒体服务器软件，可以代替上面说的 Nginx 插件。
- VLC 既是播放器，也可以作为流媒体格式转换工具（与视频流服务器），虽然支持的输出格式没有 FFmpeg 广泛，但是有图形界面（也支持命令行调用），视频路数不多的时候，也是一个可以考虑的方案。
- 最后，不差钱的话，可以考虑类似 wowza 这样的商业解决方案，前后配套，方便可靠，支持多种设备和格式，还可以搭在内网。

这是近年来写过最长的单篇了，感谢阅读。

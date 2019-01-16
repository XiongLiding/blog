---
title: nginx 实现类似 RAID 0 的读取效果
date: 2019-01-16 11:10:50
categories:
  - DevOps
tags: nginx
---

# 问题
这几天做了一个停车场的项目，出入口的相机会拍照并保存在前端的机器里，也就是收费岗亭里的那台电脑。当一个收费站有两个出入口时，照片分布在AB两台机器中。当你需要调取照片时，有可能在A机器中，也可能在B机器中。

为了能够统一的调取照片，利用 nginx 的反向代理把两台电脑的目录整合成了一个整体，对外使用一个入口。如果把电脑看成硬盘，那就是实现类似 RAID 0 的读取效果。

<!--more-->

# 基本配置
两台电脑分别安装 nginx ，并配置目录，主要是修改 nginx 的配置文件，将 

    location / {
        root html;
        index index.htm index.html;
    }

改成

    location / {
        root ../BackupPhoto;
    }

这个是出入口照片保存的目录，因为收费机的系统是 windows ，不太清楚绝对路径怎么写，就用了对应 nginx.exe 文件的相对路径。

    D:\
        BackupPhoto
        nginx-1.15.8
            nginx.exe

这样两个电脑各自都能通过 http 对外提供照片了。

# 统一入口
现在要把入口合并到一起，让用户通过其中一台电脑就能访问到所有的图片，继续修改其中一台的配置，
    
    location / {
        root ../BackupPhoto;
        try_files $uri @photoproxy;
    }

    location @photoproxy {
        proxy_pass  http://192.168.10.55; # 另一台的 ip
    }

这样，当访问这台服务器，并且根据文件名找不到图片的时候，会触发 try_files 规则，从而将请求通过 proxy_pass 转发到另一台电脑。那台电脑也找不到图片时，返回 404 。

如果有更多的设备，也可以按照这个方式把他们依次串联起来。

# 其他
电脑关机或重启后，为了让 nginx 可以开机启动，最简单的方式是使用“启动”目录。以前的 windows 在附件下面有个“启动”目录，这次用的 windows 10 ，找不到这个目录了，最好通过在运行框(win+R)输入 shell:startup 打开了这个目录。然后为 nginx.exe 创建一个快捷方式，拖到这个目录就可以了。

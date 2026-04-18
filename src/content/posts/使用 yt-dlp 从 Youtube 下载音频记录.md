---
title: 使用 yt-dlp 从 Youtube 下载音频记录
pubDate: 2026-4-18
categories: ['Tools']
description: '通过 yt-dlp 以及插件 Get cookies.txt Clean 从 Chrome 浏览器上下载 Youtube 视频的实践。'
slug:
---

今天突然想从 Youtube 上下载一些国内被 ban 掉的歌，但是发现需要充钱。于是笔者转头开始扒有没有小工具，还真找到一个：https://github.com/yt-dlp/yt-dlp

<img src ="/Tools/yt-dlp/install.png">

把它下载下来：https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp.exe

首先测试了使用 edge 浏览器对 Youtube 内容进行爬取，命令如下：

```
yt-dlp --cookies-from-browser edge "https://www.youtube.com/watch?v=arGVWzrCnvM"
```

然后报错了，这个工具拿不到 edge 的 cookie：

```
Extracting cookies from edge ERROR: Could not copy Chrome cookie database. See https://github.com/yt-dlp/yt-dlp/issues/7271 for more info ERROR: Could not copy Chrome cookie database. See https://github.com/yt-dlp/yt-dlp/issues/7271 for more info
```

后面我查到一种方法，可以直接把浏览器的 cookie 保存下来供 yt-dlp 使用：即使用一个叫 get cookies txt 的插件（edge上叫这个）。但是实际测试结果不佳，这个插件获得不了 Youtube 页面的 cookie，如下图所示：

<img src ="/Tools/yt-dlp/error.png">

后续因为 edge 上开的页面太多了不方便测试排查失败原因，我直接转到 Chrome 浏览器。这里使用了一个叫 Get cookies.txt Clean 1.0 的插件，如下图所示：

<img src ="/Tools/yt-dlp/Get-cookies.png">

使用方法：打开 chrome 浏览器，点开 Youtube 界面，使用插件将 cookies 导出到 txt：

<img src ="/Tools/yt-dlp/get-youtube-cookies.png">

然后就可以使用 yt-dlp 和 youtube.com_cookies.txt 来拉取想要的 mp3 文件。注意这里还有一个关键点：yt-dlp 在解析 Youtube 时需要执行 Youtube 的解密逻辑（Youtube 现在的视频流是被 signatureCipher 保护的，需要 js 解密参数）。所以 yt-dlp 必须下载 player JS，执行解密函数来还原真实 URL。

因为我的电脑上已经有 node.js，所以这里不过多讲解环境配置问题，而是分享工具使用的过程。

使用命令如下，即可得到想要的音频：

```
yt-dlp --cookies youtube.com_cookies.txt --js-runtimes node -f bestaudio --extract-audio --audio-format mp3 "https://www.youtube.com/watch?v=arGVWzrCnvM"
```
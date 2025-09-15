---
title: hexo博客搭建
pubDate: 2024-04-15
categories: ['environment']
description: ''
slug: 
---

搭建个人博客使用：Github + Hexo

如果有条件，可以部署在自己的服务器上，发布到自己服务器的话，Negix 代理

## 一、前置条件

+ Github 账号
  + 都能搭建个人博客了肯定会用 Github 吧~
+ NodeJS
  + nodejs 要好好配置哦，不然之后会很崩溃的，嘻
  + 可以参考下这个[NodeJS安装及配置(Windows)](https://blog.csdn.net/yaorongke/article/details/119084295)
+ Git，(理论上cmd也是可以的，我就是用的cmd)

## 二、Github部分

+ 新建一个仓库

<img src ="/hexo/create_repo.png">

+ 按要求填写

<img src ="/hexo/repo_fillin.png">

+ 测试一下是否能正常使用
  + 在新仓库中添加 `index.html`
  + 保存

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>tsglz</title>
</head>
<body>
    <h1>This is tsglz</h1>
</body>
</html>
```

+ 在 settings 的 page 中 可以访问到自己的网站
  + 如果显示正常，即部署成功，可以进行下一步
  + 如果不成功，自查上面的步骤是否出现遗漏

<img src ="/hexo/settings_deploy.png">

### Token

+ 除了一个仓库，你还需要一个 Token

<img src="/hexo/github_settings.png">

+ 注意使用 classic 哦~
+ select 一个 repo 就能用了
+ expiration 按照需要配置
  + 并不建议一直用一个

<img src="/hexo/github_token.png">

## 三、Hexo部分

### 1.安装hexo

```powershell
npm install -g hexo-cli
```

+ 查看版本

```powershell
hexo -v
```

+ 出现版本号
  + 有一大串是正常情况，请不要感到害怕

<img src="/hexo/version.png">

### 2.创建 hexo-blog

+ 创建一个项目 `hexo-blog` 并初始化

```powershell
hexo init hexo-blog
cd hexo-blog
npm install
```

+ 更换主题

[Release 1.9.7 · fluid-dev/hexo-theme-fluid (github.com)](https://github.com/fluid-dev/hexo-theme-fluid/releases/tag/v1.9.7)

+ 下载解压到 `themes` 目录，并将解压出的文件夹重命名为 `fluid`。
  + 注意，fluid目录下应该直接是多个文件
  + 如果存在多余的文件夹，请移出内部文件后删除它

+ 修改根目录下的 `_config.yml`：

<img src="/hexo/sets_1.png">

<img src="/hexo/sets_2.png">

**创建「关于页」**

首次使用主题的「关于页」需要手动创建：

```powershell
hexo new page about
```

+ 创建成功后，编辑博客目录下 `/source/about/index.md`，添加 `layout` 属性。

```text
---
title: about
date: xxxxxx
layout: about
---
```

+ 注意**冒号**后面有空格，否则会报错

#### 关于移动 hexo-blog 的补充说明

+ 一般来说全局状态下的 hexo-blog 会出现在 `C:\Windows\System32` 中
+ 但对于一些 C盘 看上去不太友好（快爆炸）的人来说（比如我），还是更希望把它移到别的地方去

1. `C:\Windows\System32\hexo-blog`找到那个文件夹
2. 把它剪切到你希望它出现的位置
   + 这个时候使用 `cd hexo-blog` 大概率是不成功的
   + 为了让我们更好的使用它，可以配置一下环境变量（如果你连环境变量在哪里或者是什么都不知道的话，问问万能的Google吧）
3. 把你的路径塞到 path 里

<img src="/hexo/path.png">

### 3.启动！

+ 先在本地运行下

```powershell
hexo g
hexo server
```

+ 如果启动不了，请使用 `管理员模式` 进行尝试

### 4.创建文章

+ 根目录下 `_config.yml`
  + 在生成文章的时候生成一个同名的资源目录用于存放图片文件

```yml
post_asset_folder: true
```

+ 插入图片
  + 这种方法在 md 编辑器里看不到图片，但在博客里是可以看到的
  + 更多方法参考官方文档（最下面会给的，别急）

```text
![图片引用方法二](test.png)
```

+ 想要使图片显示出来，需要装一个依赖
  + 它可以帮你更好地对路径进行转换

```powershell
npm install https://github.com/CodeFalling/hexo-asset-image --save
```

## 四、fluid 部分

### 覆盖配置

[配置指南 | Hexo Fluid 用户手册 (fluid-dev.com)](https://hexo.fluid-dev.com/docs/guide/#覆盖配置)

+ 好东西就要狠狠学习
+ 建议马上配置好

### 1.申请LeanCloud账号并创建应用

[开发者信息 · 账号 · LeanCloud](https://console.leancloud.cn/account/profile)

+ 注册一下
  + 如果可以请使用国际版，不然你的浏览量是看不到的
    + 国际版的使用可能需要一些**特殊的小技巧**
  + 笔者在测试过程中发现评论功能国际版有些问题（笔者是不会承认自己是菜鸡的QAQ）
    + 个人建议两个都注册下（以防万一）
+ 验证邮箱
+ 创建应用
  + 开发版就够了

<img src="/hexo/create_application.png">

+  `设置->应用凭证`，找到 `AppID` 和 `AppKey`
   + 留着等下用

### 2. 来个记录浏览数的功能吧

<img src="/hexo/web_analytics.png">

+ 配置一下 `app_id` 和 `app_key`
+ 顺手把底部的也解决一下吧

<img src="/hexo/bottom.png">

<img src="/hexo/set_id_key.png">

+ 打开计数功能，统计来源改为 `leancloud`

<img src="/hexo/set_source.png">

+ **注：统计的功能或许也需要一些特殊的小技巧，当你看不到眼睛和浏览次数的时候，请不用太过着急**
  + 以下是示例
  + 如果你能看到它，恭喜你，你成功了~

<img src="/hexo/example_by_tsglz.png">

### 3.在来个评论的功能吧

+ 先这样

<img src="/hexo/comment.png">

+ 再这样

<img src="/hexo/valine.png">

+ 评论的查看在 `leancloud` 的 `数据存储 >> 结构化数据 >> Comment` 里
  + 删除也在这里，不要因为测试输入的乱码删不掉而苦恼 ：)

## 五、一些使用技巧

### 快速导航

```txt
- 清除缓存：hexo clean
- 生成：hexo g
- 部署到远端：hexo d
- 在自己的主机上部署：hexo s
- 创建文章：hexo new post $文章名$
- 图片插入方法一：{% asset_img test.png 图片引用方法一 %} 
- 图片插入方法二：![图片引用方法二](test.png)
```

### 关于多级分类

#### 父子分类

```
categories:
- 一级
- 二级
```

#### 同级分类

```
categories:
- [一级,二级,二级中的1]
- [一级,二级,二级中的2]
```

此时该文章同时处于 `一级-二级` 下的 `1和2` 中

### 参考文章

#### hexo

+ [GitHub Pages + Hexo搭建个人博客网站，史上最全教程_hexo博客-CSDN博客](https://blog.csdn.net/yaorongke/article/details/119089190)

#### fluid

+ https://hexo.fluid-dev.com/docs/guide/
+ [Front-matter | Hexo](https://hexo.io/zh-cn/docs/front-matter)

#### valine

[快速开始 | Valine 一款快速、简洁且高效的无后端评论系统。](https://valine.js.org/quickstart.html)

如有遗漏，欢迎补充 ：）


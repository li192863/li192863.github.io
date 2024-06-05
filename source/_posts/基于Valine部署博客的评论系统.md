---
title: 基于Valine部署博客的评论系统
cover: cover.png
categories: '技术'
tags: ['博客', '评论', 'Valine']
date: 2024-06-05 20:35:03
---

## 集成Valine

[Valine](https://github.com/xCss/Valine)是一个基于LeanCloud的快速而又简单的评论系统。首先，我们需要将自己的网站集成Valine，这一步我们可以通过查阅[官方文档](https://valine.js.org/)来进行操作。

如果你使用的是Hexo，那么大多数的主题系统本身也对评论提供了支持，例如本站使用的主题`hexo-theme-butterfly`可以通过简单的操作即可完成配置，具体步骤参考[这里](https://butterfly.js.org/posts/ceeb73f/#%E8%A9%95%E8%AB%96)。

Valine的集成需要使用[LeanCloud](https://www.leancloud.cn/)，在LeanCloud中可以创建一个名为`blog-comment`的应用，用于管理和部署自己的评论系统。在LeanCloud应用控制台中`数据存储`-`结构化数据`-`Comment</>`可以删除和管理评论。

![LeanCloud管理界面](LeanCloud管理界面.png)

## 邮件通知

为了使得站长以及评论者得知被评论或被回复时，添加一个邮件通知功能是十分有用的。[Valine-Admin](https://github.com/DesertsP/Valine-Admin)是一个常见的评论系统，但是在部署该系统的时候，我出现了错误，可能是该项目太久没有更新的缘故。因此，我找到了[Valine-Admin的一个fork项目](https://github.com/wiidede/Valine-Admin)，基于此进行部署。

部署的过程均是可视化的，首先进入LeanCloud应用控制台，在`云引擎`-`管理部署`中新建一个分组，例如`EmailValineAdmin`。

![LeanCloud新建分组](LeanCloud新建分组.png)

进入该部署，在`设置`中添加如下环境变量，具体可以参考下图，其中的云引擎域名可有可无。其中`SMTP_PASS`需要通过邮箱申请。以QQ邮箱为例，进入QQ邮箱后，通过`设置`-`账号`-`POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务`中获取，过程中需要手机验证以及一些其他操作。

![自定义环境变量](自定义环境变量.png)

自定义环境变量的含义如下，这也是部署中最复杂的一步。

| 变量          | 示例                   | 说明 |
| --            | --                     | --   |
| SITE_NAME     | Deserts                | [必填]博客名称
| SITE_URL      | https://panjunwen.com  | [必填]首页地址
| SMTP_SERVICE  | QQ                     | [必填]邮件服务提供商，支持QQ、163、126、Gmail以及更多
| SMTP_USER     | xxxxxx@qq.com          | [必填]SMTP登录用户
| SMTP_PASS     | xxxxxxxx               | [必填]SMTP登录密码（QQ邮箱需要获取独立密码）
| SENDER_NAME   | Deserts                | [必填]发件人
| SENDER_EMAIL  | xxxxxx@qq.com          | [必填]发件邮箱
| ADMIN_URL     | https://xxx.leanapp.cn/| [建议]Web主机二级域名（云引擎域名），用于自动唤醒
| BLOGGER_EMAIL | xxxxx@gmail.com        | [可选]博主通知收件地址，默认使用SENDER_EMAIL
| AKISMET_KEY   | xxxxxxxx               | [可选]Akismet Key 用于垃圾评论检测，设为MANUAL_REVIEW开启人工审核，留空不使用反垃圾

设置好环境变量后，在`部署`中点击Git部署，地址填入`https://github.com/wiidede/Valine-Admin`，分支填入`master`，然后点击部署即可。如果部署成功，那么部署日志中会有相应的提示。如果部署失败，那么可能需要更换仓库地址进行再次部署。

![部署到生产环境](部署到生产环境.png)

如果提示部署成功，那么邮件通知服务也就基本完成了，下一步直接进行测试即可。完成的效果是，若A在博文中评论，那么站长(`BLOGGER_EMAIL`)会收到邮件提醒。如果A回复了B的评论，那么除了站长会收到邮件外，B也会收到邮件提醒。

## 管理回复

如果在部署邮件通知时指定了域名，那么可以通过可视化界面对评论进行管理，可以参考[这里](https://github.com/DesertsP/Valine-Admin#%E8%AF%84%E8%AE%BA%E7%AE%A1%E7%90%86)。如果没有指定域名，那么只能通过LeanCloud应用控制台中`数据存储`-`结构化数据`-`Comment</>`管理评论。对于自己的博客系统，评论数相对较少，通常使用后一种方法就足够了。

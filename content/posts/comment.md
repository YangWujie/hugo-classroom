+++
title = "给Hugo网站增加评论功能"
date = 2022-05-27T17:32:04+08:00
draft = false
author = "杨武杰"
categories = [ "工具" ]
tags = [ "Hugo", "评论功能", "Disqus", "静态网站", "giscus" ]
year = "2022"
month = "2022/05"
+++

Hugo创建的是静态网站，自身无法实现评论功能。
<!--more-->
但这不是问题，借助第三方API可以轻松解决。Hugo对Disqus提供了非常好的内建支持(参考https://gohugo.io/content-management/comments/)，可惜被墙了。所以，需要另找一个比较靠谱的API。

[giscus](https://giscus.app/zh-CN)来救场了！感觉是个中国人的开源项目。
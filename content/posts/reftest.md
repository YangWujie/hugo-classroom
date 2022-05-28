+++
title = "站内引用"
date = 2022-05-28T11:23:40+08:00
draft = false
author = "杨武杰"
categories = [ "工具" ]
tags = [ "Hugo" ]
year = "2022"
month = "2022/05"
+++

这是一个测试贴。试一下能不能链接到站内的其他网页。
<!--more-->
[第一个水贴]({{< ref "posts/first-post.md" "html" >}})

[又一个水贴]({{< relref "posts/hello-world.md" >}})

```
    [第一个水贴]({{< ref "posts/first-post.md" "html" >}})
    [又一个水贴]({{< relref "posts/hello-world.md" >}})
```

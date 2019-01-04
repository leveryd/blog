---
title: Disucz ssrf一处
date: 2017-11-01 23:14:59
tags:
---
# 漏洞证明
更新:10.20的补丁已经修复此漏洞
1. 前提条件
用户可以发帖
2. 复现步骤
发表帖子后,访问如下链接：
```
forum.php?mod=ajax&action=setthreadcover&inajax=1&fid=2&wysiwyg=1&imgurl=ftp://127.0.0.1:8089&tid=26815&pid=1
```
其中的imgurl是ssrf探测的地址,另外需要将tid修改为自己发表的帖子的tid值.这个值在chrome浏览器前端中搜索tid可以找到.

![](/img/6.png)

# 漏洞分析
漏洞触发的代码在source/module/forum/forum_ajax.phpforum_ajax.php文件大约211行处,调用了setthreadcover函数.

函数会根据第五个参数imgurl发起网络请求,此处imgurl刚好可控.
![](/img/7.png)

最终在source/class/class_image.php的init函数中发起请求
![](/img/8.png)

# 漏洞总结
1.source/class/class_image.php中看到init函数发起网络请求,向上找调用init函数的地方挖掘漏洞.

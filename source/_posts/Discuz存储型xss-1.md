---
title: Discuz存储型xss(1)
date: 2017-10-11 17:13:06
tags:
---

# 漏洞证明
1.漏洞需要开启后台四方格功能
![](/img/1.jpeg)
2.发表新帖子,在帖子标题中设置为payload
`&#x003c;img src=1 onerror=alert(1)&#x003e;`
3.然后在首页鼠标放在帖子准备点击我们刚才的发表的帖子时，触发onmouseover事件，执行我们的代码。
![](/img/2.jpeg)
# 漏洞分析
鼠标放在帖子上准备点击时，触发onmouseover事件。onmouseover事件中调用的showTip函数最终调用到了_showTip函数
![](/img/3.jpeg)

_showTip函数中使用getAtrribute取了tip属性值，然后又放入innerHTML。
重点是getAtrribute函数获取属性值时会自动解码实体编码后的值，这样`&#x003c;img src=1 onerror=alert(1)&#x003e;`就变成`<img src=1 onerror=alert(1)>`。
![](/img/4.png)

另外此漏洞中的payload只能用
`&#x003c;img src=1 onerror=alert(1)&#x003e;`
而不能用
`&#x3c;img src=1 onerror=alert(1)&#x3e;`
这是由于Discuz中dhtmlspecialchars函数实现问题。
```
function dhtmlspecialchars($string, $flags = null) {
	if(is_array($string)) {
		foreach($string as $key => $val) {
			$string[$key] = dhtmlspecialchars($val, $flags);
		}
	} else {
		if($flags === null) {
			$string = str_replace(array('&', '"', '<', '>'), array('&amp;', '&quot;', '&lt;', '&gt;'), $string);
			if(strpos($string, '&amp;#') !== false) {
				$string = preg_replace('/&amp;((#(\d{3,5}|x[a-fA-F0-9]{4}));)/', '&\\1', $string);
			}  //在这一行又将 &amp; 解码成 &,导致htmlspecailchars和php中的htmlspecailchars函数差异
		} else {
			if(PHP_VERSION < '5.4.0') {
				$string = htmlspecialchars($string, $flags);
			} else {
				if(strtolower(CHARSET) == 'utf-8') {
					$charset = 'UTF-8';
				} else {
					$charset = 'ISO-8859-1';
				}
				$string = htmlspecialchars($string, $flags, $charset);
			}
		}
	}
	return $string;
}
```
这个差异可以这么解释
```
htmlspecialchars(htmlspecialchars("&#x003c;")); 值是"&amp;amp;#x003c;"
dhtmlspecialchars(dhtmlspecialchars("&#x003c;")); 值是"&#x003c;"
```
而上面tip属性值就是两次dhtmlspecialchars了标题的值,输入存到数据库一次，输出时一次。
```
tip=dhtmlspecialchars(dhtmlspecialchars(标题));
```

# 漏洞总结
1.dhtmlspecialchars函数和实际的htmlspecialchars函数有差异
2.getAttribute函数会将属性值解码

---
title: iframe问题总结
date: 2019-09-11 15:07:10
tags:
---

#### iframe内部内容被添加了`<pre>`标签

今天在解决iframe上传文件的跨域问题时，遇到一个奇怪的问题，后台返回的json数据，放到iframe中时，莫名加上了`<pre>`标签，通过查询，最后在stackoverflow上找到这么一段话

> Assuming that the user POST the request in a form setting the target to an iframe. The JSON response will be sent back to the user on his/her iframe with content type set as "text/html". It is set as "text/html" instead of "application/json" because I want to avoid having a "pre" tag injected by the browser around the JSON response. Anyway, how does the user read that JSON response if the iframe and the parent window have different domain? There is going to be a cross domain policy issue.

大概意思就是，在form表单提交的是后，返回的json数据会根据form表单设置的target属性放到对应的iframe中，将返回的头信息改成`text/html`而不是默认的`application/json`就能避免返回的数据被包裹在`<pre>`标签中。


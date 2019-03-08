---
layout:     post
title:      "跨域"
subtitle:   ""
author:     "wml"
header-img: "img/cross/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - 跨域
---

## 前言

`同源策略(Same-origin Policy)`,这个概念已经耳熟能详了,简单来说就是浏览器的一种安全策略,不允许浏览器中一个网站访问另一个网站的数据,除非这两个网站具有相同的`源(origin)`,即: 相同的协议(eg:http/https)、相同的主机地址（eg: www.a.com/www.b.com）、相同的端口号（www.a.com:3000/www.a.com:4000），只要有一项不同，都被认为是不同源而不被允许访问。

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://test-circle.seeyouyima.com/v3/topic_detail?topic_id=36185682');
xhr.send();
xhr.onreadystatechange = function() {
    if(xhr.readyState == 4 && xhr.status == 200) {
        console.log(xhr.responseText);
    }else {
        console.log(xhr.statusText);
    }
}
```

这个是一个简单的get请求，本地发起请求，由于源不同，所以提示跨域，看下图，可以看出请求是成功的，只是服务器在没有特殊处理的情况下返回给浏览器时被浏览器拦截了，浏览器根据响应头`Access-Control-Allow-Origin`的值判断是否有权限获取数据。

![cross](/img/cross/2.jpg)

## 怎么解决跨域

同源策略的安全策略不置可否，但是有时候我们也希望不同源如和子域之间能进行数据交换，那如何绕过这一限制呢？业界已经有许多比较成熟的方法解决，都各有利弊，根据自己的情况选择。

### [document.domain](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/domain)

`document.domain`属性是最直接方法，在根域范围内，Mozilla允许吧domain属性设置它的上一级域，eg:

```js
var root = 'dev.a.com';
document.domain = 'a.com'; // right
document.domain = 'b.com'; // error
```

缺点也很明显，就是只限于父子域之间，甚至有些浏览器并不能设置改属性。

### 跨文档消息 [postMessage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage)

> 语法：otherWindow.postMessage(message, targetOrigin, [transfer]);

`window.postMessage`是H5的API，通过它可以在不同域之间进行通信，不过要注意两点：

* window.postMessage()中的window始终是要通信的目标页面的window；
* 只能在iframe、window.open()的方式下通信

```html
<!-- window.open -->

<!-- pageA -->
<h1>Page A</h1>
<button id="newWin">open new window</button>

<script>
var test = {
    "code": 0,
    "name": "wml",
    "des": 'this page from pageA'
}
window.onload = function() {
    var openBtn = document.getElementById('newWin');
    openBtn.addEventListener('click', openwindow);
}

function openwindow() {
    var pageB = window.open('http://192.168.152.69:3000/pageB');

    //这里加setTImeout是因为需要等到PageB页面加载完成才能进行postMessage通信，但是在PageA内是无法对PageB进行onload事件监听，所以做延迟处理。
    setTimeout(function() {
        pageB.postMessage(test, 'http://192.168.152.69:3000/pageB');
    }, 500);
}
</script>

<!-- page B -->
<h1>Page B</h1>

<script>
window.onload = function() {
    window.addEventListener("message", function(e) {
        console.log(e, 999);
    }, false)
}
</script>
```

![cross](/img/cross/3.jpg)

该方法的缺点也是适用场景不普遍，只是和window.open和iframe出来的页面。
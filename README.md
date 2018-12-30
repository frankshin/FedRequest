# FedRequest

> summary of some font-end request APIS

目录结构:
— FONT END REQUEST APIS
  — XMLHttpRequest
  — Fetch
— CROSS DOMAIN
  — Ajax规避浏览器同源策略methods
  　 |— 图片ping
  　 |— comet
  　 |— 服务器发送事件
  　 |— window.name+iframe
  　 |— window.postMessage()
  　 |— 修改document.domain跨子域
  　 |— 服务器代理
  　 |— JSONP
  　 |— WebSocket
  　 |— SSE与WebSocket
  　 |— CORS
  　   　|— 简单请求
  　　　　|—非简单请求
  — http协议10种请求类型介绍
    |— OPTIONS
    |— HEAD
    |— GET
    |— POST
    |— PUT
    |— DELETE
    |— TRANCE
    |— CONNECT
    |— LINK
    |— UNLINK

## FONT-END REQUEST APIS

### XMLHttpRequest

> XMLHttpRequest是大家普遍比较熟悉的一个浏览器内置对象，所有现代浏览器 (IE7+、Firefox、Chrome、Safari 以及 Opera) 都内建了 XMLHttpRequest 对象。

#### 历史回顾

1996年，IE 中首先添加了 iframe 用来实现异步请求获取服务器内容
1998年，微软 Outlook 在客户端 script 中实现了 XMLHttp 对象
1999年，微软在 IE5 中添加了 XMLHTTP ActiveX 对象用来异步获取服务器内容，该对象直到 Edge 浏览器才废弃。其它浏览器陆续实现了类似的对象称为 XMLHttpRequest
2004年，Google Gmail 中大量使用 XMLHttpRequest
2005年，Google Map 中大量使用 XMLHttpRequest
2005年，Jesse James Garrett 发表了文章 "Ajax: A New Approach to Web Applications"，Ajax 诞生
2006年，XMLHttpRequest 被 W3C 采纳，最后更新时间是 2014年1月

#### usage

```javascript {cmd='node'}
// 通过XMLHttpRequest创建请求对象（兼容新老版浏览器）
function createXHR() {
  if (typeof XMLHttpRequest != 'undefined') {
    // > ie7
    return new XMLHttpRequest();
  } else if (typeof ActiveXObject != 'undefined') {
    // ie7之前的版本
    if (typeof arguments.callee.activeXString != "string") {
      var versions = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0", "MSXML2.XMLHttp"],
      i,
      len;
      for (i = 0, len = versions.length; i < len; i++) {
        try {
          new ActiveXObject(versions[i]);
          arguments.callee.activeXString = versions[i];
          break;
        } catch(ex) {
          //  
        }
      }
    }
    return new ActiveXObject(arguments.callee.activeXString);
  } else {
    throw new Error('NO XHR object available');
  }
}
// begin request
function sendAjax(method, url, asy) {
  var handler = function() {
    if (this.readyState === 4) {
      if (this.status === 200) {
        console.error(this.status);
        console.log(this.response);
      } else {
        console.log("请求失败: " + new Error(this.statusText));
      }
    }
  }
  var xhr = new XMLHttpRequest();
  xhr.onreadystatechange = handler;
  xhr.open(method, url, asy); // 规定请求的类型、URL 以及是否异步处理请求
  xhr.responseType = 'json'; // 设置该值能够改变响应类型。就是告诉服务器你期望的响应格式。
  xhr.setRequestHeader('MyHeader', 'Myvalue'); // 设置自定义头部
  xhr.timeout = 1000; // 设置超时时间为1秒，仅适用于IE8+
  xhr.ontimeout = function() {
    // code
  }
  xhr.send(params); // 将请求发送到服务器 其中send(string)仅用于post请求
}
```

### Fetch

#### 历史回顾

>

#### usage

>

## CROSS DOMAIN

### Ajax规避浏览器同源策略methods

#### 图片ping

#### comet

#### 服务器发送事件

#### window.name+iframe

#### window.postMessage()

#### 修改document.domain跨子域

#### nginx服务器代理

#### JSONP

#### WebSocket

#### SSE与WebSocket

#### CORS

> 历史&原理？

##### 原理解析

![](https://smallpang.oss-cn-shanghai.aliyuncs.com/blog/images/CORS_pri.png)

##### 简单请求

##### 非简单请求

### http协议10种请求类型介绍

#### OPTIONS

#### HEAD

#### GET

#### POST

#### PUT

#### DELETE

#### TRANCE

#### CONNECT

#### LINK

#### UNLINK

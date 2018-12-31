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

      |— Preflighted Request

      |— Simple Request

      |— Requests with Credential

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

For security reasons, browsers restrict cross-origin HTTP requests initiated from within scripts. For example, XMLHttpRequest and the Fetch API follow the same-origin policy. This means that a web application using those APIs can only request HTTP resources from the same origin the application was loaded from, unless the response from the other origin includes the right CORS headers(details see the follows demo).

The CORS mechanism supports secure cross-origin requests and data transfers between browsers and web servers. Modern browsers use CORS in an API container such as XMLHttpRequest or Fetch to help mitigate the risks of cross-origin HTTP requests.

details here: [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

##### 原理解析 & demo

![](https://smallpang.oss-cn-shanghai.aliyuncs.com/blog/images/CORS_pri.png)

Access-Control-Allow-Origin: http://xxxx.com:
该响应头用来记录可以访问该资源的域。在接收到服务端响应后，浏览器将会查看响应中是否包含Access-Control-Allow-Origin响应头。如果该响应头存在，那么浏览器会分析该响应头中所标示的内容。如果其包含了当前页面所在的域，那么浏览器就将知道这是一个被允许的跨域访问，从而不再根据Same-origin Policy来限制用户对该数据的访问

##### Preflighted Request

1、请求类型除GET，HEAD或POST以外的任何一种类型；
2、请求类型POST，Content-Type不是application/x-www-form-urlencoded，multipart/form-data或text/plain之一.

eg:

```javascript {cmd='node'}
// 首先执行请求代码follows：
var request = new XMLHttpRequest(),
payload = ......;
request.open('POST', 'http://frankshin/someData', true);
request.setRequestHeader('TEST-CROESS-HEADER', 'value');
request.onreadystatechange = handler;
request.send(payload);

// 打开浏览器debug可以看到如下，该阶段相当于在询问服务端是否可以请求数据：
OPTIONS /someData/ HTTP/1.1
Host: frankshin.com
......
Origin: http://frankshin.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: TEST-CROESS-HEADER

// 服务端相应头：
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://frankshin.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: TEST-CROESS-HEADER

// 接着，浏览器会分析这个返回数据（Response Headers）,如果发现是被允许的请求后，浏览器会开始向服务端发送真正的post请求，follows：
POST /someData/ HTTP/1.1
Host: frankshin.com
TEST-CROESS-HEADER: value
......
[Payload Here]

// 服务端接着处理并返回
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://frankshin.com
Content-Type: application/xml
......
[Payload Here]

```

##### Simple Request

1、请求类型是GET，HEAD或POST之一，没有包含任何自定义请求头；
2、使用POST作为请求，该请求的Content-Type是application/x-www-form-urlencoded，multipart/form-data或text/plain之一。

##### Requests with Credential

> XMLHttpRequest.withCredentials  属性是一个Boolean类型，它指示了是否该使用类似cookies,authorization headers(头部授权)或者TLS客户端证书这一类资格证书来创建一个跨站点访问控制（cross-site Access-Control）请求。在同一个站点下使用withCredentials属性是无效的。如果在发送来自其他域的XMLHttpRequest请求之前，未设置withCredentials 为true，那么就不能为它自己的域设置cookie值

带凭证的请求相比简单请求和预检请求，在请求发送时多了y用户凭证（[withCredentials](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/withCredentials)）

```javascript {cmd='node'}
// 带withCredentials的前端请求代码：
var request = new XMLHttpRequest();
request.open('GET', 'http://frankshin.com/someData', true);
request.withCredentials = true;
request.onreadystatechange = handler;
request.send();

// 服务端的返回x响应头
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://frankshin.com
Content-Type: application/xml
......
[Payload Here]
```

一个跨域请求包含了当前页面的用户凭证，那么其就属于Requests with Credential（ps：一般情况下，一个跨域请求不会包含当前页面的用户凭证）。

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

## 参考资料

[cors简介](https://www.cnblogs.com/loveis715/p/4592246.html)
# FedRequest

> summary of some font-end request APIS

目录结构:

- FONT END REQUEST APIS
  - [XMLHttpRequest](#XMLHttpRequest)
    - Fetch
      - 历史
      - usage
      - 存在的问题
    - 当前流行的ajax库剖析
      - axios
  - [CROSS DOMAIN](#CROSS-DOMAIN)
    - 图片ping
    - comet
    - 服务器发送事件(SSE)
    - window.name+iframe
    - window.postMessage()
    - 修改document.domain跨子域
    - nginx反向代理
    - JSONP
    - WebSocket
    - SSE与WebSocket
    - CORS
      - Preflighted Request
      - Simple Request
      - Requests with Credential
- http协议10种请求类型介绍
  - OPTIONS
  - HEAD
  - GET
  - POST
  - PUT
  - DELETE
  - TRANCE
  - CONNECT
  - LINK
  - UNLINK

## FONT-END REQUEST APIS

### XMLHttpRequest

> XMLHttpRequest是大家普遍比较熟悉的一个浏览器内置对象，所有现代浏览器 (IE7+、Firefox、Chrome、Safari 以及 Opera) 都内建了 XMLHttpRequest 对象。

#### 历史回顾

1996年，IE 中首先添加了 iframe 用来实现异步请求获取服务器内容

1998年，微软 Outlook 在客户端 script 中实现了 XMLHttp 对象

1999年，微软在 IE5 中添加了 XMLHTTP ActiveX 对象用来异步获取服务器内容，该对象直到 Edge浏览器才废弃。其它浏览器陆续实现了类似的对象称为 XMLHttpRequest

2004年，Google Gmail 中大量使用 XMLHttpRequest

2005年，Google Map 中大量使用 XMLHttpRequest

2005年，Jesse James Garrett 发表了文章 "Ajax: A New Approach to Web Applications"，Ajax 诞生

2006年，XMLHttpRequest 被 W3C 采纳，最后更新时间是 2014年1月

#### usage

```javascript
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

> fetch api是一个用以请求数据的简洁接口，相比XMLHttpRequest需要额外的逻辑（如：处理重定向）而言，fetch在发送站点请求以及处理返回数据上更加简单

#### 历史

Fetch目前还不是W3C规范，由[whatwg](https://en.wikipedia.org/wiki/WHATWG)(Web Hypertext Application Technology Working Group 超文本应用技术工作组)负责出品

#### usage

- 检测浏览器是否支持fetch

```javascript
if(!('fetch' in window)) {
  console.log('fetch api not found, try including the polyfill')
  return
}
```

- 腻子库

[polyfill](https://github.com/github/fetch)for browsers are not currently supported

```javascript
import 'whatwg-fetch'
window.fetch(...)
```

- response返回数据对象：

fetch规范定义的response对象具有如下方法：

```javascript
//
arrayBuffer()
//
blob()
//
json()
// 
text()
//
formData()
```

- fetch获取http头信息

```javascript
fetch('url').then(function(response) {
  console.log(response.headers.get('Content-Type'))
  console.log(response.headers.get('Date'))
})
```

- 发起请求说明

```javascript
postData('http://example.com/answer', {answer: 42})
  .then(data => console.log(data)) // JSON from `response.json()` call
  .catch(error => console.error(error))

function postData(url, data) {
  // Default options are marked with *
  return fetch(url, {
    body: JSON.stringify(data), // must match 'Content-Type' header
    cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
    // credentials: 'include'  Fetch 跨域请求时默认不会带 cookie，需要时得手动指定 credentials: 'include'，类比XHR的withCredentials: true
    // credentials: 'same-origin': 只在请求URL与调用脚本位于同一起源处时发送凭据
    // credentials: 'omit':确保浏览器不在请求中包含凭据
    credentials: 'same-origin',
    headers: {
      'user-agent': 'Mozilla/4.0 MDN Example',
      'content-type': 'application/json'
    },
    method: 'POST',          // *GET, POST, PUT, DELETE, etc.
    mode: 'cors',            // no-cors, cors, *same-origin
    redirect: 'follow',      // manual, *follow, error
    referrer: 'no-referrer', // *client, no-referrer
  })
  .then(response => {
    if (response.status >= 200 && response.status < 300) {
      return Promise.resolve(response)
    } else {
      return Promise.reject(new Error(response.statusText))
    }
  })
  .catch(function(error) {
    console.log('Looks like there was a problem: \n', error);
  })
}
```

- 上传文件

```javascript
// Files can be uploaded using an HTML <input type="file" /> input element, FormData() and fetch()
var formData = new FormData();
var fileField = document.querySelector("input[type='file']");
formData.append('username', 'abc123');
formData.append('avatar', fileField.files[0]);
fetch('https://example.com/profile/avatar', {
  method: 'PUT',
  body: formData
})
.then(response => response.json())
.catch(error => console.error('Error:', error))
.then(response => console.log('Success:', response));
```

- fetch链式调用

```javascript
fetch('doAct.action')
  .then((response) => {
    if (response.status >= 200 && response.status < 300) {
      return Promise.resolve(response)
    } else {
      return Promise.reject(new Error(response.statusText))
    }
  })
  .then((response) => {
    return response.json()
  })
  .then(function(data) {
    console.log('Request succeeded with JSON response', data)
  })
  .catch(function(error) {
    console.log('Request failed', error)
  })
```

#### 存在的问题

#### fetch vs xhr

### 当前流行的ajax库剖析

#### axios

## CROSS DOMAIN

### 图片ping

> 图像Ping是与服务器进行简单、单向的跨域通信的一种方式。请求的数据是通过查询字符串形式发送的，而且响应可以是任意内容，但通常是像素图或204响应。通过图像ping，浏览器得不到任何具体的数据，但通过侦听load和error事件，可以知道响应是什么时候接收到的。

```javascript
var img = new Image();
img.onload = function(){
    console.log('success...');
}
img.onerror = function(){
    console.log('failed...');
}
img.src = 'xxxxxxxxx';
```

图像ping最常用于跟踪用户点击页面或动态广告曝光次数，图像ping有两个主要缺点：

1、只能监听服务端是否相应，不能访问响应的文本；

2、只能发送get请求。

应用demo：利用图像ping可以初略检测网络延迟：

```javascript
function ping(){
  var statrt = (new Date()).getTime()
  var img = new Image()
  var calcueTime = function(){
    var overtime = (new Date()).getTime() - statrt
    console.log('ping img的时长：' + overtime)
  }
  img.onload = img.onerror = function(){
    calcueTime()
  }
  img.src = 'https://www.baidu.com?=' + new Date().getTime().toString();
}

```

### comet

Comet是Alex Russell发明的一个技术名词，指的是一种基于服务器推送的更高级的Ajax技术，与普通Ajax从页面向服务器请求数据相比，Comet则是由服务器向页面推送数据。

Comet的实现方式有长轮询和流两种：

长轮询：

长轮询类似于传统轮询（短轮询）的升级，浏览器定时发送数据请求，只有有更新的数据时，服务端才会返回数据（如下图21-1，21-2分别为短轮询和长轮询示意图）

![1](https://smallpang.oss-cn-shanghai.aliyuncs.com/blog/images/1098800-20180301164535057-563650997.jpg)

![](https://smallpang.oss-cn-shanghai.aliyuncs.com/blog/images/1098800-20180301164632242-40102811.jpg)

http流：

流不同于长轮询，因为在页面的整个生命周期，只使用一个HTTP连接，就是浏览器向服务器发送一个请求，而服务器保持连接打开，然后周期性地向浏览器发送数据。

在Firefox、safari、opera和chrome中，通过侦听readyStatechange事件及检测readyState的值是否为3，就可以利用xhr对象实现HTTP流。在上述浏览器环境中，随着不断从服务器接收数据，readyState的值会周期性地变为3。当readyState值变为3时，responseText属性中就会保存接收到的所有数据。此时，就需要比较之前接收到的数据，决定从什么位置开始取得最新的数据，code如下：

```javascript
function createStreamingClient(url, progress, finished){
  var xhr = new XMLHttpRequest(), received = 0
  xhr.open('get', url, true);
  xhr.onreadystatechange = function(){
    var result
    if(xhr.readyState === 3){
      result = xhr.responseText.substring(received);
      received += result.length
      // 调用progress回调函数
      progress(result)
    }else if(xhr.readyState === 4){
      finished(xhr.responseText)
    }
  }
  xhr.send(null);
  return xhr;
}
// begin call
var api = 'http://baidu.com/xxxxx.xml'
var client = createStreamingClient(api, function(res){
    console.log(res)
}, function(res){
    console.log(res)
})

```

### 服务器发送事件(SSE)

> SSE是围绕只读Comet交互推出的API或模式。SSE API用于创建到服务器的单向连接，服务器通过这个连接可以发送任意数量的数据。服务器响应的MIME类型必须是test/event-stream。而且是浏览器中的javascript API能解析格式输出。SSE支持短轮询、长轮询和http流。而且能在断开连接时自动确定何时重新连接。

支持SSE的浏览器有Firefox6+、Safari5+、Opera11+、Chrome和ios4+版Safari。

1、SSE API

SSE的javascript api与其他传递消息的javascript api很相似。要预定新的时间流，首先要创建一个新的EventSource('myevents.php');

备注：传入的url必须与创建对象的页面同源（url模式、域及端口均相同）。EventSource实例有一个readyState属性，值为0表示正链接到服务器，值为1表示打开了连接，值为2表示关闭了连接。

实例有以下三个事件：

open：在建立连接时触发。

message：在从服务器接收到新事件时触发。

error：在无法建立连接时触发。

```javascript
source.onmessage = function(event){
  var data = event.data;
  // 数据处理
}
```

EventSource默认会保持与服务器的活动连接，如果连接断开，还会重新连接。如果需要强制断开连接且不再重新连接，可以调用close()方法。

source.close()

2、事件流

服务器事件会通过一个持久的HTTP响应发送，找个响应的MIME类型为text/event-stream。响应的格式是纯文本，最简单的情况是每个数据项都带有前缀data：。例如

data：foo

data：bar

data：foo

data：bar

以上响应中，事件流的第一个message事件返回的event.data值为“foo”，第二个message事件返回的event.data值为“bar”，第三个message事件返回的event.data值为“foo\nbar”（注意中间的换行符）。对于多个连续的以data：开头的数据行，将作为多段数据解析，每个值之间以一个换行符分隔。只有在包含data：的数据行后面有空行时，才会触发message事件，因此在服务器上生成事件流时不能忘记多添加这一行。

通过id：前缀可以给特定的事件指定一个关联的ID，找个id行位于data：行前面或后面皆可：

data：foo

id：1

设置了id后，EventSource对象会跟踪上一次触发的事件。如果连接断开，会向服务器发送一个包含名为last-event-id的特殊http头部请求，以便服务器知道下一次该触发哪个事件。在对此连接的事件流中，这种机制可以确保浏览器以正确的顺序收到连接的数据段。

### window.name+iframe

### window.postMessage()

### 修改document.domain跨子域

### nginx反向代理服务器

> 反向代理（Reverse Proxy）方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求连接的客户端

因为服务器之间没有跨域,所以能够请求到数据

#### nginx配置demo：

```javascript
// 这里使用代理服务器61.200.166.130进行请求www.domain.com
server {
  listen 80;
  server_name www.domain.com;
  # access_log  /var/log/nginx/www.domain.com.access.log;
  location / {
    proxy_set_header Host www.domain.com;
    add_header 'Access-Control-Allow-Origin' 'http://www.domain.com';
    add_header 'Access-Control-Allow-Credentials' 'true';
    proxy_pass http://61.200.166.130;
  }
}

// another scenes
server
{
  listen 80;
  server_name www.aaa.top;
  location / {
    proxy_pass http://www.bbb.com;
    add_header 'Access-Control-Allow-Origin' '*';
    add_header 'Access-Control-Allow-Credentials' 'true';
  }
  ##### other directive
}
```

### JSONP

### WebSocket

### SSE与WebSocket

### CORS

For security reasons, browsers restrict cross-origin HTTP requests initiated from within scripts. For example, XMLHttpRequest and the Fetch API follow the same-origin policy. This means that a web application using those APIs can only request HTTP resources from the same origin the application was loaded from, unless the response from the other origin includes the right CORS headers(details see the follows demo).

The CORS mechanism supports secure cross-origin requests and data transfers between browsers and web servers. Modern browsers use CORS in an API container such as XMLHttpRequest or Fetch to help mitigate the risks of cross-origin HTTP requests.

details here: [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

#### 原理解析 & demo

![](https://smallpang.oss-cn-shanghai.aliyuncs.com/blog/images/CORS_pri.png)

Access-Control-Allow-Origin: http://xxxx.com:
该响应头用来记录可以访问该资源的域。在接收到服务端响应后，浏览器将会查看响应中是否包含Access-Control-Allow-Origin响应头。如果该响应头存在，那么浏览器会分析该响应头中所标示的内容。如果其包含了当前页面所在的域，那么浏览器就将知道这是一个被允许的跨域访问，从而不再根据Same-origin Policy来限制用户对该数据的访问

#### Preflighted Request

1、请求类型除GET,HEAD或POST以外的任何一种类型；

2、请求类型POST，Content-Type不是application/x-www-form-urlencoded，multipart/form-data或text/plain之一.

eg:

```javascript
// 首先执行请求代码follows：
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

// 服务端相应头：
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://frankshin.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: TEST-CROESS-HEADER

// 接着，浏览器会分析这个返回数据（Response Headers）,如果发现是被允许的请求后，浏览器会开始向服务端发送真正的post请求，follows：
POST /someData/ HTTP/1.1
Host: frankshin.com
TEST-CROESS-HEADER: value
......
[Payload Here]

// 服务端接着处理并返回
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://frankshin.com
Content-Type: application/xml
......
[Payload Here]

```

#### Simple Request

1、请求类型是GET，HEAD或POST之一，没有包含任何自定义请求头；

2、使用POST作为请求，该请求的Content-Type是application/x-www-form-urlencoded，multipart/form-data或text/plain之一。

#### Requests with Credential

> XMLHttpRequest.withCredentials  属性是一个Boolean类型，它指示了是否该使用类似cookies,authorization headers(头部授权)或者TLS客户端证书这一类资格证书来创建一个跨站点访问控制（cross-site Access-Control）请求。在同一个站点下使用withCredentials属性是无效的。如果在发送来自其他域的XMLHttpRequest请求之前，未设置withCredentials 为true，那么就不能为它自己的域设置cookie值

带凭证的请求相比简单请求和预检请求，在请求发送时多了y用户凭证（[withCredentials](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/withCredentials)）

```javascript
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

## http协议10种请求类型介绍

### OPTIONS

> 说明：询问服务器支持的方法,支持的http协议版本: 1.0、1.1

### HEAD

> 说明：获得报文首部,支持的http协议版本: 1.0、1.1

### GET

> 说明：获取资源,支持的http协议版本: 1.0、1.1

get的常见参数提交方式为Query String Parameter，如下图：

![](https://smallpang.oss-cn-shanghai.aliyuncs.com/blog/images/1098800-20180626185813168-647235402.png)

### POST

> 传输实体主体,支持的http协议版本: 1.0、1.1

HTTP 协议是以 ASCII 码传输，建立在 TCP/IP 协议之上的应用层规范。规范把 HTTP 请求分为三个部分：状态行、请求头、消息主体，协议规定 POST 提交的数据必须放在消息主体（entity-body）中，但协议并没有规定数据必须使用什么编码方式。实际上，开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以

#### 1 application/x-www-form-urlencoded

> 即参数提交方式application/x-www-form-urlencoded

这应该是最常见的 POST 提交数据的方式了。浏览器的原生 form 表单，如果不设置 enctype 属性，那么最终就会以application/x-www-form-urlencoded方式提交数据，注意，如果写原生的ajax请求，提交的数据需要按照 key1=val1&key2=val2 的方式进行序列化编码，key和val都进行了URL转码（一些库如jquery等其内部已做好了封装，所以不需要进行序列化操作）

![](https://smallpang.oss-cn-shanghai.aliyuncs.com/blog/images/1098800-20180626184636340-1356927996.png)

![](https://smallpang.oss-cn-shanghai.aliyuncs.com/blog/images/1098800-20180626184726974-155389537.png)

#### 2 multipart/form-data

> 即参数提交方式multipart/form-data

这种方式一般用来上传文件，各大服务端语言对它也有着良好的支持。

#### 3 application/json

> 参数提交方式application/json

用来告诉服务端消息主体是序列化后的 JSON 字符串。由于 JSON 规范的流行，除了低版本 IE 之外的各大浏览器都原生支持 JSON.stringify，服务端语言也都有处理 JSON 的函数，使用 JSON 不会遇上什么麻烦，这种方案，可以方便的提交复杂的结构化数据，特别适合 RESTful 的接口，请求如下图：

![](https://smallpang.oss-cn-shanghai.aliyuncs.com/blog/images/1098800-20180626193409703-1342591225.png)

![](https://smallpang.oss-cn-shanghai.aliyuncs.com/blog/images/1098800-20180626192143334-1218422902.png)

#### 4 text/xml

> 即参数提交方式text/xml

一种使用HTTP作为传输协议，XML作为编码方式的远程调用规范，不过XML结构还是过于臃肿，一般场景用JSON会更灵活方便

### PUT

> 传输文件, 支持的http协议版本: 1.0、1.1

### DELETE

> 删除文件, 支持的http协议版本: 1.0、1.1

### TRANCE

> 追踪路径,支持的http协议版本: 1.1

### CONNECT

> 要求用隧道协议连接代理,支持的http协议版本: 1.1

### LINK

> 说明：建立和资源之间的联系,支持的http协议版本: 1.0

### UNLINK

> 断开连接关系,支持的http协议版本: 1.0

## github

[FedRequest](https://github.com/frankshin/FedRequest)

## 参考资料

[cors简介](https://www.cnblogs.com/loveis715/p/4592246.html)
《javacript高级程序设计》
[Working with the Fetch API](https://developers.google.com/web/ilt/pwa/working-with-the-fetch-api)
[fetch:下一代ajax技术](https://www.cnblogs.com/snandy/p/5076512.html)
[使用fetch](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)
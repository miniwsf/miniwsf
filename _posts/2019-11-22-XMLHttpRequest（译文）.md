---
layout:     post
title:      XMLHttpRequest（译文）
subtitle:   
date:       2019-11-22
author:     wsf
header-img: img/20191122/451571454809_2.pic_hd.jpg
catalog: true
tags:
    - XMLHttpRequest
    - 译文
---
> 为行己方便（WHATWG竟然没有中文版，网上也没有现成的资源，不得不自己翻译。文档部分翻译是
之前W3C现有的，所以自然是......）。因第一次翻译，若有不当之处望指正。原文地址[XMLHttpRequest](https://xhr.spec.whatwg.org/)。

### 摘要
XMLHttpRequest标准定义了一套API: 提供在客户端和服务端之间传输数据的脚本客户端功能。

### 1. 介绍
本节是非规范的。
XMLHttpRequest是用于获取资源的API。
XMLHttpRequest这个名称具有历史性，和功能无关。
```javascript
// 处理从网络上获取到的 XML 文档数据的简易代码：
function processData(data) {
  // taking care of data
}
function handler() {
  if(this.status == 200 &&
    this.responseXML != null &&
    this.responseXML.getElementById('test').textContent) {
    // success!
    processData(this.responseXML.getElementById('test').textContent);
  } else {
    // something went wrong
    …
  }
}
var client = new XMLHttpRequest();
client.onload = handler;
client.open("GET", "unicorn.xml");
client.send();
// 如果你只希望在服务器上记录消息：
function log(message) {
  var client = new XMLHttpRequest();
  client.open("POST", "/log");
  client.setRequestHeader("Content-Type", "text/plain;charset=UTF-8");
  client.send(message);
}
// 或者如果你希望检测服务器上的文档状态：
function fetchStatus(address) {
  var client = new XMLHttpRequest();
  client.onload = function() {
    // in case of network errors this might not give reliable results
    returnStatus(this.status);
  }
  client.open("HEAD", address);
  client.send();
}
```

#### 1.1 规范史
XMLHttpRequest 对象初定义于 WHATWG 的 HTML 项目中，很久之后微软发起了一个实现。它于 2006 年被移到 W3C。直到 2011 年底，XMLHttpRequest 的扩展（XMLHttpRequest Level 2）被开发于一份独立的草案中，这时两份草案被合并，
XMLHttpRequest 又从一个 standards perspective 变成一个单一的实体。2012年底，它又移回了WHATWG。
历史讨论可以从以下邮件列表档案中找到：
- [whatwg@whatwg.org](https://lists.w3.org/Archives/Public/public-whatwg-archive/)
- [public-webapi@w3.org](https://lists.w3.org/Archives/Public/public-webapps/)
- [public-appformats@w3.org](https://lists.w3.org/Archives/Public/public-webapi/)
- [public-webapps@w3.org](https://lists.w3.org/Archives/Public/public-appformats/)

### 2. 一致性
同所有章节中明确标明了非规范的部分一样，这份规范中的所有的图表、范例及注释都是非规范的。其它的所有内容都是规范的。
在这份规范的规范部分中，关键词“必须”、“必须不”、“需要”、“应该”、“不该”、“被推荐的”、“可能”和“可选的”，都要被作为 RFC2119 中的描述来解读。
为了确保可读性，这些单词不会以全大写的形式出现在这份文档中。[[RFC2119]](https://xhr.spec.whatwg.org/#refsRFC2119)

#### 2.1 扩展性
强烈建议用户代理，工作组和其它有兴趣的团体与WHATWG社区讨论新特性。

### 3. 术语
本规范使用的术语在DOM，DOM解析和序列化，编码，功能策略，访存，文件API，HTML，HTTP，URL，Web IDL和XML中贯穿始终。
它使用HTML的印刷约定。

### 4. XMLHttpRequest 接口
```javascript
[Exposed=(Window,DedicatedWorker,SharedWorker)]
interface XMLHttpRequestEventTarget : EventTarget {
  // event handlers
  attribute EventHandler onloadstart;
  attribute EventHandler onprogress;
  attribute EventHandler onabort;
  attribute EventHandler onerror;
  attribute EventHandler onload;
  attribute EventHandler ontimeout;
  attribute EventHandler onloadend;
};
[Exposed=(Window,DedicatedWorker,SharedWorker)]
interface XMLHttpRequestUpload : XMLHttpRequestEventTarget {
};
enum XMLHttpRequestResponseType {
  "",
  "arraybuffer",
  "blob",
  "document",
  "json",
  "text"
};
[Constructor,
 Exposed=(Window,DedicatedWorker,SharedWorker)]
interface XMLHttpRequest : XMLHttpRequestEventTarget {
  // event handler
  attribute EventHandler onreadystatechange;
  // states
  const unsigned short UNSENT = 0;
  const unsigned short OPENED = 1;
  const unsigned short HEADERS_RECEIVED = 2;
  const unsigned short LOADING = 3;
  const unsigned short DONE = 4;
  readonly attribute unsigned short readyState;
  // request
  void open(ByteString method, USVString url);
  void open(ByteString method, USVString url, boolean async, optional USVString? username = null, optional USVString? password = null);
  void setRequestHeader(ByteString name, ByteString value);
           attribute unsigned long timeout;
           attribute boolean withCredentials;
  [SameObject] readonly attribute XMLHttpRequestUpload upload;
  void send(optional (Document or BodyInit)? body = null);
  void abort();
  // response
  readonly attribute USVString responseURL;
  readonly attribute unsigned short status;
  readonly attribute ByteString statusText;
  ByteString? getResponseHeader(ByteString name);
  ByteString getAllResponseHeaders();
  void overrideMimeType(DOMString mime);
           attribute XMLHttpRequestResponseType responseType;
  readonly attribute any response;
  readonly attribute USVString responseText;
  [Exposed=Window] readonly attribute Document? responseXML;
};
```
XMLHttpRequest 对象有一个与之关联的 XMLHttpRequestUpload。  
XMLHttpRequest 对象有一个与之关联的状态：unsend,opened,headers received, loading, done之一。若无声明，那么状态为unsent。  
XMLHttpRequest 对象有一个与之关联的send（）标志。若无声明，则未设置。

#### 4.1 构造器
```javascript
    // 非规范
    client = new XMLHttpRequest();  // 返回一个新的XMLHttpRequest对象
```
XMLHttpRequest()方法必须返回一个新的XMLHttpRequest对象。

#### 4.2 垃圾回收
如果 XMLHttpRequest 对象的状态是 OPENED 并且已设置 send() 标识，或者它的状态是 HEADERS_RECEIVED，又或者它的状态是 LOADING 
且它对以下事件注册了一个以上的事件监听：readystatechange、progress、about、error、load、timeout、loadend，
凡满足以上这些情况的 XMLHttpRequest 就必须不被垃圾回收机制回收。
如果 XMLHttpRequest 对象在连接尚存打开时被垃圾回收机制回收了，用户代理必须终止请求。

#### 4.3 事件处理
下列是事件处理（它们对应事件处理的事件类型），在对象实现一个继承于 XMLHttpRequestEventTarget 的接口时必须实现它们：
事件处理程序 | 事件处理程序事件类型
---|---
onloadstart | loadstart
onprogress | progress
onabort | abort
onerror | error
onload | load
ontimeout | timeout
onloadend | loadend
下面是必须由 XMLHttpRequest 对象单独支持的的事件处理（它对应事件处理的事件类型）：
事件处理程序 | 事件处理程序事件类型
---|---
onreadystatechange | readystatechange

#### 4.4 状态
```
    // 非规范
    client . readyState
    返回当前状态。
```
XMLHttpRequest 对象可以有几个状态。readyState属性必须返回当前状态，它必须是下列值的其中之一：
状态 | 状态值 | 描述
---|---|---
unsent | UNSENT(数字值为0) | 对象已创建
opened | OPENED(数字值为1) | open() 方法已经成功调用。在此期间可以使用 setRequestHeader() 方法来设置请求头，并且可以调用 send() 方法来请求。
headers received | HEADERS_RECEIVED(数字值为2) | 所有的重定向都已经跳转（如果有的话），并且已经接收到了所有的HTTP响应头。
loading | LOADING(数字值为3) | 响应的主体部分正在被接收。
done | DONE(数字值为4) | 数据传输已经完成或者传输过程中出现错误（例如无限重定向）。

#### 4.5 请求
每个 XMLHttpRequest 对象都有下面与请求相关的概念：请求方法、请求URL、作者请求头、请求主体、异步标识、上传完成标识以及上传事件标识。  
作者请求头初始为空头列表。
请求主体初始为 null。
异步标识、上传完成标识、上传事件标识，初始都未设置。
注：在一个 XMLHttpRequest 对象上注册一个或者多个事件监听会导致CORS预检请求。（因为注册事件监听会导致上传监听标志被设置，从而导致
use-CORS-preflight标志被设置）

##### 4.5.1 open()方法
```
    // 非规范
   client . open(method, url [, async = true [, username = null [, password = null]]])
   设置请求方法、请求URL和异步标识。
   如果方法不是有效的 HTTP 方法或 url 无法解析则抛出 "SyntaxError" 异常。
   如果方法是大小写不敏感的 `CONNECT`、`TRACE` 或 `TRACK` 其中之一则抛出一个 "SecurityError" 异常。
   如果 async 为 false、JavaScript 全局环境是一个文档环境，并且满足下列其中之一：timeout 属性非零、 withCredentials 属性为 true、
   responseType 属性不是空字符串，抛出一个 "InvalidAccessError" 异常。
```
警告：开发者必须不在 JavaScript 全局环境是文档环境时给 async 参数传递 false，它会造成不良的终端用户体验。
强烈鼓励用户代理对这样的用法在开发者工具中抛出警告，而且当这种情况出现时候可以尝试抛出一个 "InvalidAccessError" 异常，
如此一来最终可以将这个特性从平台上彻底移除。
open(method, url)或open(method, url, async, username, password)方法必须以下面步骤执行：
1. 让配置对象作为上下文对象的相关设置对象。
2. 如果配置对象的可靠文档不是充分活跃的则抛出一个 "InvalidStateError" 异常。
3. 如果 method 不是一个方法则抛出一个 "SyntaxError" 异常。
4. 如果 method 是一个被禁用的方法则抛出一个 "SecurityError" 异常。
5. 规范化 method。
6. 令 parsedURL 为以 base 解析 url 的结果。
7. 如果 parsedURL 是失败的则抛出一个 "SyntaxError" 异常。
8. 如果 async 参数被省略则将 async 设置为 true，并且将 username 和 password 设置为 null。  
注：遗憾的是旧版中的 async 参数不能被省略，如果省略的话会被作为 undefined 处理。
9. 如果 parsedURL 的相对标志有设置则以下面的步骤执行：
    1. 如果 username 参数不为 null 则令 username 为 parsedURL 的 用户名。
    2. 如果 password 参数不为 null 则令 password 为 parsedURL 的 密码。
10. 如果 async 为 false 且 JavaScript 全局环境是文档环境并且满足下列其中之一：timeout 属性非零、 withCredentials 属性为 true、responseType 属性不是空字符串，抛出一个 "InvalidAccessError" 异常。
11. 终止请求。  
注：在这个点上抓取会被持续。
12. 设置变量与对象的关联:
    1. 复原send()标志和上传监听标志。
    2. 将 请求方法 设置为 method。
    3. 将 请求 URL 设置为 parsedURL。
    4. 如果 async 为 false 且同步标志未设置，则设置同步标志。
    5. 清空作者请求头。
    6. 将响应设置为网络错误。
    7. 设置接收字节为空的字节序列。  
    注：此处不覆盖媒体类型，因为在open()方法之前可以调用overrideMimeType()方法来实现。
13. 如果其状态不为 OPENED 则以下列步骤执行：
    1. 将其状态设置为 OPENED。
    2. 触发一个名为 readystatechange 的事件。
    
注：定义了两个open()方法的原因是由于用于编写XMLHttpRequest标准的编辑软件的限制。

##### 4.5.2 setRequestHeader() 方法
```
    // 非规范
    client . setRequestHeader(name, value)
    并入一个头到作者请求头中。
    如果状态不是 OPENED 或 send() 标识有设置则抛出一个 "InvalidStateError" 异常。
    如果 name 不是请求头名称或 value 不是请求头值则抛出一个 "SyntaxError"。
```
 setRequestHeader(name, value)方法必须运行以下步骤：
 1. 如果状态不是 opened，则抛出一个 "InvalidStateError" 异常。
 2. 如果send() 标志有设置，则抛出一个 "InvalidStateError" 异常。
 3. 规范化 value。
 4. 如果 name 不是请求头名称或 value 不是请求头值则抛出一个 "SyntaxError"。  
 注：空字节序列表示空请求头值
 5. 如果 name 是一个禁止的请求头名称，终止步骤。
 6. 并入name/value到作者请求头中。
 ```javascript
 // 示范当设置同样的请求头两次时会发生什么的简易代码：
// The following script:
var client = new XMLHttpRequest();
client.open('GET', 'demo.cgi');
client.setRequestHeader('X-Test', 'one');
client.setRequestHeader('X-Test', 'two');
client.send();
// …results in the following header being sent:
// X-Test: one, two
```
 
##### 4.5.3 timeout属性
 ```
    // 非规范
    client . timeout
    可以设置时间为毫秒。当设置一个非0时间，在超过给定的时间后会导致抓取停止。当时间过去但请求未完成，并且异步标志未设置，将调度超时时间，
    否则将抛出 "TimeoutError" 异常（对于send方法）。
    
    设置时，如果异步标志被设置并且当前全局对象是一个window对象，则抛出 "InvalidAccessError" 异常。
```
 timeout属性必须返回该值。默认值必须是0。  
 设置timeout属性必须执行这些步骤：
 1. 如果当前对象是一个window对象并且设置了异步标志，则会抛出 "InvalidAccessError" 异常。
 2. 设置它的值为新值。
注：这意味着可以在抓取过程中设置timeout。如果发生这种情况，则仍将相对于抓取开始处进行测量。
##### 4.5.4 withCredentials属性
```
    // 非规范
    client . withCredentials
    当包含跨域请求时为true。当不包括跨域请求并且在响应中忽略cookie时为false。默认为false。
    设置时，如果状态不是unsent或者opened，或者send()标志有设置，则抛出 "InvalidStateError" 异常。
```
withCredentials属性必须返回该值。默认值必须为false。  
设置withCredentials属性必须执行这些步骤：
1. 如果状态不是unsent或者opened，则抛出 "InvalidStateError" 异常。
2. 如果send()标志设置，则抛出"InvalidStateError" DOMException。
3. 将withCredentials属性值设置为给定值。
注：当获取同源资源时，withCredentials属性无效。

##### 4.5.5 upload属性
```
    // 非规范
    client . upload
    返回关联的XMLHttpRequestUpload对象。当数据传输到服务器时，它可用于收集传输信息。
```
upload属性必须返回与之关联的XMLHttpRequestUpload对象。
注：如前所诉，每个XMLHttpRequest对象都有一个与之关联的XMLHttpRequestUpload对象。

##### 4.5.6 === send()方法 ===
```
    // 非规范
    client . send([body = null])
    
```
send(body)必须运行以下步骤：
1. 如果状态不是opened，抛出"InvalidStateError" DOMException。
2. 如果send()标志未设置，抛出"InvalidStateError" DOMException。
3. 如果请求方法是get或者head，设置body为空。
4. 如果body不为空，那么
- 设置extractedContentType为空
- 如果body是document，则将请求体设置为body，转换为unicode和UTF-8编码。
- 否则，将请求体和extractedContentType设置为extracting body
- 如果作者请求头包含Content-Type，那么：
   1. 如果body是doucument或USVString，那么：
       1. 令originalAuthorContentType为标头的值，该标头的名与作者请求标头中的“ Content-Type”匹配（不区分大小写）。
       2. 让contentTypeRecord是解析originalAuthorContentType的结果。
       3. 如果contentTypeRecord没有失败，并且contentTypeRecord的参数 [“ charset”]存在，并且参数 [“ charset”]不是ASCII区分大小写的“ UTF-8”，则：
           1. 设置contentTypeRecord的参数["charset"]为"UTF-8".
           2. 令newContentTypeSerialized为序列化contentTypeRecord的结果。
           3. 在作者请求标头中设置`Content-Type` / newContentTypeSerialized。
- 否则:
   1. 如果body是一个HTML文档，在作者请求标头中设置Content-Type / text / html; charset = UTF-8。
   2. 否则，如果body是XML文档，则在作者请求标头中设置Content-Type / application / xml; charset = UTF-8。
   3. 否则，如果extractContentType不为null，则在作者请求标头中设置`Content-Type` / extractedContentType。
- 如果在关联的XMLHttpRequestUpload对象上注册了一个或多个事件侦听器，则设置upload侦听器标志。
- 令req为一个新请求，初始化如下：
method: 请求方法
url： 请求地址
header list： 作者头
unsafe-request flag： set？ 
body： 请求体
client： 上下文对象的相关设置对象
同步标志：如果设置了同步标志，则设置。
mode：'cors'
使用CORS预检标志：如果设置了上载侦听器标志，则设置
凭证模式：如果withCredentials属性值为true，否则为“ include”和“ same-origin”。
use-URL-credentials标志：如果请求URL的用户名不是空字符串或者请求URL的密码非空，则设置。
- 取消上载完成标志
- 取消timeout标志
- 如果req的body为空，设置上载完成标志
- 设置send()标志
- 如果同步标志未设置，那么：
   1. 使用0和0触发一个名为loadstart的进度事件。
   2. 如果上载完成标志未设置，上载监听标志设置，那么then fire a progress event named loadstart at this’s XMLHttpRequestUpload object with 0 and req’s body’s total bytes.
   3. 如果状态不是opened或者send()标志未设置，那么直接返回
   4. 提取req。请按照以下方式处理在网络任务源上排队的任务。
      并行运行这些步骤：
      1. 等待直到req的done标志设置或者
          1. 从这些步骤开始以来，timeout属性值已经过毫秒数
          2. 而timeout属性值不为零。
      2. 如果req的done标志未设置，那么设置timeout标志和终止获取。
      为了处理请求正文，执行这些步骤：
      1. 如果自上次调用这些步骤以来未经过大约50毫秒，请终止这些步骤。
      2. If upload listener flag is set, then fire a progress event named progress at this’s XMLHttpRequestUpload object with request’s body’s transmitted bytes and request’s body’s total bytes.
      为了处理请求的请求主体，请运行以下步骤：
      
      要处理响应以进行响应，请运行以下步骤：
      
- 如果同步标志设置了，那么执行这些步骤：（现在已经不鼓励使用同步标志了，所以我不想翻译了）

##### 4.5.7 abort()方法
```
    // 非规范
   client . abort() 
   取消任何网络活动
```
abort()方法必须执行这些步骤：
1. 在设置了中止标志的情况下终止正在进行的抓取。
2. 如果状态是opened并且send()标志已设置，或者是headers received，或是loading，运行事件中止的请求错误步骤。
3. 如果状态done，则将状态设置为unsent并响应网络错误。
注：不调度readystatechange事件。

#### 4.6 响应
XMLHttpRequest对象有一个与之关联的响应。除非有声明，否则为网络错误。  
XMLHttpRequest对象还有一个与之关联的接收字节（一个字节序列）。除非有声明，否则为空字节序列。

##### 4.6.1 responseURL属性
如果响应的url为null，则responseURL属性返回空字符串。  ？？？？？？？？

##### 4.6.2 status属性
status属性必须返回响应的状态。

##### 4.6.2 statusText属性
statusText属性必须返回响应的状态消息。

##### 4.6.4 getResponseHeader()方法   ?????????
调用getResponseHeader（name）方法时，必须返回从响应的标头列表中获取名称的结果。

##### 4.6.5 getAllResponseHeaders()方法
如果以下步骤返回true，则字节序列a比字节序列b的大写字节小：
- 让A为a，字节大写
- 让B为a，字节大写
- 返回A比B少字节
getAllResponseHeaders()方法必须执行这些步骤：
1. 令输出output为空字节序列。
2. 让initialHeaders是运行排序的结果，并与响应的标头列表相结合。
3. 让标头是按升序对initialHeaders进行排序的结果，如果a的名称小于b的名称，则a小于b。
4. 对于头文件中的每个头文件，将头文件的名称、后面跟着0x3A 0x20字节对、头文件的值、后面跟着0x0D 0x0A字节对追加到输出。
5. 返回输出。
```
    // 以下脚本:
    
    var client = new XMLHttpRequest();
    client.open("GET", "narwhals-too.txt", true);
    client.send();
    client.onreadystatechange = function() {
      if(this.readyState == this.HEADERS_RECEIVED) {
        print(this.getAllResponseHeaders());
      }
    }
    
    // The print() function will get to process something like:
    
    connection: Keep-Alive
    content-type: text/plain; charset=utf-8
    date: Sun, 24 Oct 2004 04:58:38 GMT
    keep-alive: timeout=15, max=99
    server: Apache/1.3.31 (Unix)
    transfer-encoding: chunked
```

##### 4.6.6 响应体
响应MIME类型是运行以下步骤的结果：
1. 假设mimeType是从响应的标题列表中提取MIME类型的结果。
2. 如果mimeType失败，则将mimeType设置为text / xml。
3. 返回mimeType
覆盖MIME类型最初为null，并且在调用overlayMimeType（）时可以获取值。最终的MIME类型是覆盖MIME类型，除非该值为null，在这种情况下，它是响应MIME类型。
最终的字符集是这些步骤的返回值：
1. 让label为null
2. 如果存在响应mime type的参数["charset"]存在，将label设为此值
3. 如果存在覆盖mime type的参数["charset"]存在，将label设为此值
4. 如果label为null，那么返回null
5. 让encoding成为从label获得编码的结果。
6. 如果encoding失败，那么返回null
7. 返回encoding
XMLHttpRequest对象有一个相关的响应对象(对象、失败或null)。除非另有说明，否则为空。
arraybuffer响应是这些步骤的返回值：
1. 将响应对象设置为表示接收字节的新ArrayBuffer对象。如果抛出异常，则将response对象设置为failure并返回null。
2. 返回响应对象。
blob响应是这些步骤的返回值：
1. 将response对象设置为一个新的Blob对象，该对象表示接收的字节，类型设置为最终的MIME类型。
2. 返回响应对象。
document响应是这些步骤的返回值：
1. 如果响应体为空，则返回空。
2. 如果最后的MIME类型不是HTML MIME类型或XML MIME类型，则返回null。
3. 如果responseType是空字符串，最后的MIME类型是HTML MIME类型，那么返回null。
4. 如果最后的MIME类型是HTML MIME类型，那么:
    - 让charset成为最终的charset。
    - 如果charset为null，则保留接收字节的前1024字节，如果不成功终止，则让charset作为返回值。
    - 如果charset为空，则将charset设置为UTF-8。
    - 假设document是一个文档，它表示根据HTML标准中规定的规则解析接收到的字节的结果，该标准适用于HTML解析器，并且禁用了脚本和已知的明确的编码字符集。[HTML
    - 将文档标记为HTML文档。
5. 否则，让document成为一个表示运行XML解析器的结果的文档，在接收的字节上禁用XML脚本支持。如果失败(不支持字符编码、名称空间格式良好性错误等)，则返回null。[HTML
6. 如果charset为空，则将charset设置为UTF-8。
7. 将文档编码设置为字符集。
8. 将文档的内容类型设置为最终的MIME类型。
9. 将文档的URL设置为响应的URL。
10. 将文档origin设置为上下文对象的相关设置对象的origin。
11. 将response对象设置为document并返回。
    
JSON响应是这些步骤的返回值：
1. 如果响应体为空，则返回空。
2. 让jsonObject是在接收的字节上从字节运行解析JSON的结果。如果抛出异常，则返回null。
3. 将响应对象设置为jsonObject并返回它。
text响应是这些步骤的返回值：
1. 如果响应体为空，则返回空字符串。
2. 让charset成为最终的charset。
3. 如果responseType是空字符串，charset是null，最后的MIME类型是XML MIME类型，那么使用XML规范中规定的规则来确定编码。设字符集为确定的编码。(XML) (XMLNS
4. 如果charset为空，则将charset设置为UTF-8。
5. 使用回退编码字符集返回对接收字节运行解码的结果。

##### overrideMimeType() 方法
当overrideMimeType(mime)被调用时，必须执行这些步骤：
1. 如果状态是loading或done，那么抛出"InvalidStateError" DOMException
2. 将替代MIME类型设置为解析mime的结果。
3. 如果覆盖MIME类型失败，则将覆盖MIME类型设置为application / octet-stream。

##### responseType属性
responseType属性必须返回其值，初始化为一个空的字符串。
设置responseType属性需要执行这些步骤：
1. 如果当前值不是window对象，并且给定值为"document"，终止步骤。
2. 如果状态为loading或者done，那么抛出"InvalidStateError" DOMException
3. 如果当前全局对象是window对象，并且设置了同步标志，那么抛出"InvalidAccessError" DOMException
4. 设置responseType属性为给定的值

##### response属性
response属性的值是执行了这些步骤的返回值：
- 如果responseType是一个空的字符串或者"text"
1. 如果状态不是loading或者done，返回空的字符串
2. 返回文本响应
- 否则
1. 如果状态不是done，返回null
2. 如果response对象失败，那么返回null
3. 如果response对象不为空，那么返回这个值
4. 如果responseType是arraybuffer....

##### responseText属性
responseText属性值是执行了以下步骤的返回值：
1. 如果responseType是一个空的字符串或者"text"，那么抛出"InvalidStateError" DOMException.
2. 如果状态不是loading或者done，返回空的字符串
3. 返回text response

##### responseXML属性
response属性的值是执行了这些步骤的返回值：
1. 如果responseType是一个空的字符串或者"text"，那么抛出"InvalidStateError" DOMException.
2. 如果状态不是done，返回null
3. 断言：response对象不是失败
4. 如果response对象不为null，返回该值
5. 返回document响应

##### 事件摘要
本节非规范性。
在XMLHttpRequest或XMLHttpRequestUpload对象上调度以下事件：

##### 功能政策整合
该规范定义了由字符串“ sync-xhr”标识的策略控制功能。其默认允许列表为*。

##### Formdata接口
```javascript
typedef (File or USVString) FormDataEntryValue;
[Constructor(optional HTMLFormElement form),
 Exposed=(Window,Worker)]
interface FormData {
  void append(USVString name, USVString value);
  void append(USVString name, Blob blobValue, optional USVString filename);
  void delete(USVString name);
  FormDataEntryValue? get(USVString name);
  sequence<FormDataEntryValue> getAll(USVString name);
  boolean has(USVString name);
  void set(USVString name, USVString value);
  void set(USVString name, Blob blobValue, optional USVString filename);
  iterable<USVString, FormDataEntryValue>;
};
```

尚未译完......

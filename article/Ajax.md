## Ajax
> Fetch 似乎是更好替代物

### 1. 实现异步

Ajax 是基于 XMLHttpRequest API 的异步方案，通过 Ajax 可以让 client 实现无需刷新页面就能从 server 获取数据。

#### 1.1 基本用法

需要注意的是，事件监听需要在 `open()` 方法之前添加。基本的使用如下：

	function listener() {
	  if (xhr.readyState == 2) {
	    console.log(xhr.getResponseHeader(<header>));
	    console.log(xhr.getAllResponseHeaders());
	  }
	
	  if (xhr.readyState == 4) {
	    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
	      console.log(xhr.responseText);
	    }
	  }
	};
	
	var xhr = new XMLHttpRequest();
	xhr.onreadystatechange = listener;
	xhr.open(<method>, <URL>, <isAsync>);
	xhr.setRequestHeader(<header>, <value>);
	xhr.send(<empty or data>);

其中，`readyState` 的值：

- 0：unsent 状态，即调用 `open()` 前
- 1：opened 状态，即调用 `open()` 后，调用 `send()` 前
- 2：headers received 状态，即调用 `send()` 后，但未收到数据
- 3：loading 状态，即接收数据的整个过程
- 4：done 状态，即完成所有数据的接收

#### 1.2 定位进度

通过 `readyState` 值和 `onreadystatechange` 事件组合的方式处理 response 不够语义化，也无法定位进度，可以使用自解释性很强的 Progress Event 事件来替代：

- loadstart：开始，回调的执行次数为 1 次，执行时机是最开始的时候
- progress：进行中，回调的执行次数为 1 次以上，执行时机是 loadstart 回调执行后
- abort：被终止，回调的执行次数为 0 或 1 次
- error：失败，回调的执行次数为 0 或 1 次
- load：成功，回调的执行次数为 0 或 1 次
- timeout：因预设时间到了而被终止，回调的执行次数为 0 或 1 次
- loadend：结束，回调的执行次数为 1 次，执行时机是 abort、error、load、timeout 中任意一个回调之后

其中，abort、error、load 和 timeout 的执行时机是 最后一个 progress（进行中状态）的回调执行后，且四个回调彼此互斥，同一个 progress（进度）内只会执行一个。

另外，为了获取最新的数据，防止 XHR 对象从缓存中读取数据，可以给 URL 添加时间戳：

	url += (/\?/).test(url) ? '&' : '?';
	url += 'timestamp=';
	url += (new Date()).getTime();

### 2. 同源策略

Ajax 最大的限制是同源策略（Same-origin policy），它限制了不同源之间的交互，一个源的文档或脚本不能与另一个源的资源进行交互。

同源是指两个 URL 的协议、域名、端口完全一致。`itunes.apple.com` 与 `apple.com` 也不符合同源策略，因为域名的一致要求子域也相同。有个简单的方法可以改变这种不合理性：修改 `document.domain` 的值。该值可以被设置为父域的值，从而通过同源策略的检查，这也是最基本的跨源解决方案。

#### 2.1 跨源操作

包含了三类交互操作：读、写、嵌入：

- 读操作是被同源策略限制的，除非通过嵌入手段
- 写操作是被允许的，如链接、重定向、表单提交
- 嵌入操作也是被允许的，如通过 `<img>` `<audio>` `<video>` 嵌入媒体资源、通过 `@font-face` `<link>` `<script>` 嵌入字体样式脚本、通过 `<object>` 或 `<iframe>` 嵌入其他资源

浏览器提供了一些可以实现跨源访问脚本和数据的方法：

- 跨源脚本访问：`window` 提供了一些具有一定跨源访问能力的 API，如使用 `window.postMessage` 实现通信
- 跨源数据访问：`localStorage` 和 `indexedDB` 的数据是按源进行分割，无法跨源访问

从上述讨论可以看出，浏览器通常阻止的跨源请求，即必须遵守同源策略的请求，是从脚本中发起的读操作，如通过 XHR 或 Fetch 发起的请求。

#### 2.2 CORS

跨域资源共享（CORS）机制让跨源访问成为可能，并且能够安全的进行。该机制需要 client 和 server 同时支持才能实现。通常有以下场景：

简单请求：可以直接跨域访问的请求。特征是请求使用 `HEAD` `GET` `POST`，且使用了对 CORS 安全的请求字段，同时也满足符合要求的 `Content-Type` 请求字段值。简单请求下，`Origin` 请求字段说明请求来源，而`Access-Control-Allow-Origin` 响应字段的值与 `Origin` 相同或者为 `*`。此外，简单请求也可以通过设置 Cookies 来附带身份凭证，然后由服务器决定是否返回请求内容。

预检请求：需要先检查能够跨域访问。特征是请求使用 `PUT` `PATCH` `DELETE` `TRACE` `CONNECT` `OPTIONS`，或者使用了对 CORS 安全的请求字段外的其他字段，或者 `Content-Type` 请求字段值不符合要求。预检请求下，实际的请求之前，需要先发送一个使用 `OPTIONS` 方法的预检请求，然后由服务器决定是否允许实际的请求。

#### 2.3 JSONP

跨源的嵌入操作是被允许的，JSONP 的原理即在于此。简单使用如下：

	// 设置 url 和 callback
	var say = res => console.log(res);
	var url = 'freegeoip.net/json/github.com?callback=say';
	
	// 生成请求
	var script = document.createElement('script');
	script.setAttribute('src', url);
	
	// 执行回调
	var head = document.querySelector('head');
	head.appendChild(script);

当流程从前端走到后台时，后台将其视为一个 URL 为 `freegeoip.net/json/github.com` 的 `GET` 请求，然后将一个特殊的字符串作为返回值交还给前端：`say(/* 这里是 JSON 数据 */)`，这个字符串就像是一个特殊的 JSON 数据外加了左右 padding 一样，并且可以被前端直接执行。

JSONP 的本质是前后端的协作，即前端把想要的资源以及后续的处理都告诉后台，后台封装好返回给前端执行。其缺陷是明显的：每一个 JSONP 请求必须对应一个脚本注入，可能存在一定的安全隐患。 

#### 2.4 window.postMessage()

这是一个浏览器原生支持的 web API，基于 message 事件的安全跨源通信。

`http://one.com/index.html` 的脚本如下：

	var popup = window.open('http://two.com/index.html');
	
	window.addEventListener('message', e => {
	  if (e.data === 'ready') {
	    popup.postMessage('give u some data', 'http://two.com');
	  }
	}, false);

`http://two.com/index.html` 的脚本如下：

	opener.postMessage('ready', 'http://one.com');
	
	window.addEventListener('message', e => {
	  if (e.origin === 'http://one.com') {
	    console.log(e.data);
	  }
	}, false);

### 3. Fetch

XHR 确实做到了异步请求，但是它将输入、输出和状态的管理全部纳入到了同一个对象，并且基于事件的状态追踪与当前的异步编程风格（如 Promise 和 generator）不一致。

jQuery 和 axios 一定程度上缓解了这个风格问题，而 Fetch API 则是一种更根本的解决方案：

	fetch(url, options).then(function(res) {
	  // handle HTTP response
	}, function(err) {
	  // handle network error
	})

### 参考
1. [Same-origin policy - web security | MDN](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
2. [HTTP access control - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
3. [axios - mzabriskie | GitHub](https://github.com/mzabriskie/axios)
4. [fetch polyfill and documentation - github | GitHub](https://github.com/github/fetch)

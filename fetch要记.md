作者：Yang

日期：2022-9-19

## 介绍

- `fetch()`是 XMLHttpRequest 的升级版，用于在 JavaScript 脚本里面发出 HTTP 请求。
- 浏览器原生提供这个对象。

### fetch与XMLHttpRequest的区别

`fetch()`的功能与 XMLHttpRequest 基本相同，但有三个主要的差异：

​		（1）`fetch()`使用 Promise，不使用回调函数，因此大大简化了写法，写起来更简洁。

​		（2）`fetch()`采用模块化设计，API 分散在多个对象上（Response 对象、Request 对象、Headers 对象）；相比之下，XMLHttpRequest 的 API 设计并不是很好，输入、输出、状态都在同一个接口管理，容易写出非常混乱的代码。

​		（3）`fetch()`通过数据流（Stream 对象）处理数据，可以分块读取，有利于提高网站性能表现，减少内存占用，对于请求大文件或者网速慢的场景相当有用。XMLHTTPRequest 对象不支持数据流，所有的数据必须放在缓存里，不支持分块读取，必须等待全部拿到后，再一次性吐出来。

## 基本用法

基本语法：fetch()接受一个 URL 字符串作为参数，默认向该网址发出 GET 请求，返回一个 Promise 对象

```js
fetch(url)
  .then(...)
  .catch(...)
```

例子：`fetch()`接收到的`response`是一个 [Stream 对象](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API)，`response.json()`是一个异步操作，取出所有内容，并将其转为 JSON 对象。

```js
fetch('https://api.github.com/users/ruanyf')
  .then(response => response.json())
  .then(json => console.log(json))
  .catch(err => console.log('Request Failed', err)); 
```

Promise 可以使用 await 语法改写，使得语义更清晰。注意：`await`语句必须放在`try...catch`里面，这样才能捕捉异步操作中可能发生的错误。

```js
async function getJSON() {
  let url = 'https://api.github.com/users/ruanyf';
  try {
    let response = await fetch(url);
    return await response.json();
  } catch (error) {
    console.log('Request Failed', error);
  }
}
```

## Response对象：处理HTTP回应

### Response 对象的同步属性

- `fetch()`请求成功以后，得到的是一个 [Response 对象](https://developer.mozilla.org/en-US/docs/Web/API/Response)。它对应服务器的 HTTP 回应。

- Response 包含的数据通过 Stream 接口异步读取，但是它还包含一些同步属性，对应 HTTP 回应的标头信息（Headers），可以立即读取。

  | 属性                | 描述                                                         | 值                                                           |
  | ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | Response.ok         | 返回一个布尔值，表示请求是否成功                             | `true`对应 HTTP 请求的状态码 200 到 299<br />`false`对应其他的状态码。 |
  | Response.status     | 返回一个数字，表示 HTTP 回应的状态码（例如200，表示成功请求）。 |                                                              |
  | Response.statusText | 返回一个字符串，表示 HTTP 回应的状态信息（例如请求成功以后，服务器返回"OK"）。 |                                                              |
  | Response.url        | 返回请求的 URL。如果 URL 存在跳转，该属性返回的是最终 URL。  |                                                              |
  | Response.type       | 返回请求的类型                                               | `basic`：普通请求，即同源请求。 <br />`cors`：跨域请求。 `error`：网络错误，主要用于 Service Worker。 <br />`opaque`：如果`fetch()`请求的`type`属性设为`no-cors`，就会返回这个值<br /> `opaqueredirect`：如果`fetch()`请求的`redirect`属性设为`manual`，就会返回这个值 |
  | Response.redirected | 返回一个布尔值，表示请求是否发生过跳转                       |                                                              |

### 判断请求是否成功

`fetch()`发出请求以后，有一个很重要的注意点：只有网络错误，或者无法连接时，`fetch()`才会报错，其他情况都不会报错，而是认为请求成功。

这就是说，即使服务器返回的状态码是 4xx 或 5xx，`fetch()`也不会报错（即 Promise 不会变为 `rejected`状态）。

- 通过`Response.status`属性

  得到 HTTP 回应的真实状态码，`response.status`属性只有等于 2xx （200~299），才能认定请求成功。这里不用考虑网址跳转（状态码为 3xx），因为`fetch()`会将跳转的状态码自动转为 200。

```js
async function fetchText() {
  let response = await fetch('/readme.txt');
  if (response.status >= 200 && response.status < 300) {
    return await response.text();
  } else {
    throw new Error(response.statusText);
  }
}
```

- 通过response.ok属性

  判断`response.ok`是否为`true`

### Response.headers 属性

- Response 对象还有一个`Response.headers`属性，指向一个 [Headers 对象](https://developer.mozilla.org/en-US/docs/Web/API/Headers)，对应 HTTP 回应的所有标头。

- Headers 对象可以使用`for...of`循环进行遍历。

- Headers 对象提供了以下方法，用来操作标头。

  | 方法名            | 描述                                                     |
  | ----------------- | -------------------------------------------------------- |
  | Headers.get()     | 根据指定的键名，返回键值                                 |
  | Headers.has()     | 返回一个布尔值，表示是否包含某个标头                     |
  | Headers.set()     | 将指定的键名设置为新的键值，如果该键名不存在则会添加     |
  | Headers.append()  | 添加标头                                                 |
  | Headers.delete()  | 删除标头                                                 |
  | Headers.keys()    | 返回一个遍历器，可以依次遍历所有键名                     |
  | Headers.values()  | 返回一个遍历器，可以依次遍历所有键值                     |
  | Headers.entries() | 返回一个遍历器，可以依次遍历所有键值对（`[key, value]`） |
  | Headers.forEach() | 依次遍历标头，每个标头都会执行一次参数函数               |

  例子：

  ```js
  let response =  await  fetch(url);  
  response.headers.get('Content-Type')
  // application/json; charset=utf-8
  
  // 键名
  for(let key of myHeaders.keys()) {
    console.log(key);
  }
  
  // 键值
  for(let value of myHeaders.values()) {
    console.log(value);
  }
  
  let response = await fetch(url);
  response.headers.forEach(
    (value, key) => console.log(key, ':', value)
  );
  ```

### 读取内容的方法

​	`Response`对象根据服务器返回的不同类型的数据，提供了不同的读取方法。5个读取方法都是`异步`的，返回的都是 Promise 对象。必须等到异步操作结束，才能得到服务器返回的完整数据。

| 方法名                 | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| response.text()        | 得到文本字符串，获取文本数据，如.ht                          |
| response.json()        | 得到 JSON 对象，用于获取服务器返回的 JSON 数据               |
| response.blob()        | 得到二进制 Blob 对象，如img                                  |
| response.formData()    | 得到 FormData 表单对象，主要用在 Service Worker 里面，拦截用户提交的表单，修改某些数据以后，再提交给服务器。 |
| response.arrayBuffer() | 得到二进制 ArrayBuffer 对象，主要用于获取流媒体文件，如音频文件 |

### Response.clone()

> Stream 对象只能读取一次，读取完就没了。这意味着，前一节的五个读取方法，只能使用一个，否则会报错。

Response 对象提供`Response.clone()`方法，创建`Response`对象的副本，实现多次读取。

```js
const response1 = await fetch('flowers.jpg');
const response2 = response1.clone();

const myBlob1 = await response1.blob();
const myBlob2 = await response2.blob();

image1.src = URL.createObjectURL(myBlob1);
image2.src = URL.createObjectURL(myBlob2);
```

### Response.body 属性

`Response.body`属性是 Response 对象暴露出的底层接口，返回一个 ReadableStream 对象，供用户操作。

它可以用来分块读取内容，应用之一就是显示下载的进度。

## `fetch()`的第二个参数：定制 HTTP 请求

`fetch()`的第一个参数是 URL，还可以接受第二个参数，作为配置对象，定制发出的 HTTP 请求。

HTTP 请求的方法、标头、数据体都在这个对象里面设置。

```js
fetch(url, optionObj)
```

`fetch()`第二个参数的完整 API 如下:

```js
const response = fetch(url, {
  method: "GET",
  headers: {
    "Content-Type": "text/plain;charset=UTF-8"
  },
  body: undefined,
  referrer: "about:client",
  referrerPolicy: "no-referrer-when-downgrade",
  mode: "cors", 
  credentials: "same-origin",
  cache: "default",
  redirect: "follow",
  integrity: "",
  keepalive: false,
  signal: undefined
});
```

> `fetch()`请求的底层用的是 [Request() 对象](https://developer.mozilla.org/en-US/docs/Web/API/Request/Request)的接口，参数完全一样，因此上面的 API 也是`Request()`的 API。



### 示例

**POST 请求**

配置对象用到了三个属性

```js
/*
	method：HTTP 请求的方法，POST、DELETE、PUT都在这个属性设置。
	headers：一个对象，用来定制 HTTP 请求的标头。
	body：POST 请求的数据体。
*/
const response = await fetch(url, {
  method: 'POST',
  headers: {
    "Content-type": "application/x-www-form-urlencoded; charset=UTF-8",
  },
  body: 'foo=bar&lorem=ipsum',
});

const json = await response.json();
```

> 注意，有些标头不能通过`headers`属性设置，比如`Content-Length`、`Cookie`、`Host`等等。它们是由浏览器自动生成，无法修改。

**提交 JSON 数据**

```js
const user =  { name:  'John', surname:  'Smith'  };
const response = await fetch('/article/fetch/post/user', {
  method: 'POST',
  headers: {
   'Content-Type': 'application/json;charset=utf-8'
  }, 
  body: JSON.stringify(user) 
});
```

> 上面示例中，标头`Content-Type`要设成`'application/json;charset=utf-8'`。因为默认发送的是纯文本，`Content-Type`的默认值是`'text/plain;charset=UTF-8'`。

**提交表单**

```js
const form = document.querySelector('form');

const response = await fetch('/users', {
  method: 'POST',
  body: new FormData(form)
})
```

**文件上传**

**直接上传二进制数据**

## 取消`fetch()`请求

`fetch()`请求发送以后，如果中途想要取消，需要使用`AbortController`对象。

首先新建 AbortController 实例，然后发送`fetch()`请求，配置对象的`signal`属性必须指定接收 AbortController 实例发送的信号`controller.signal`。

`controller.abort()`方法用于发出取消信号。这时会触发`abort`事件，这个事件可以监听，也可以通过`controller.signal.aborted`属性判断取消信号是否已经发出。

一个1秒后自动取消请求的例子

```js
let controller = new AbortController();
setTimeout(() => controller.abort(), 1000);

try {
  let response = await fetch('/long-operation', {
    signal: controller.signal
  });
} catch(err) {
  if (err.name == 'AbortError') {
    console.log('Aborted!');
  } else {
    throw err;
  }
}
```

## 问题：fetch发送2次请求的原因

fetch发送post请求的时候，总是发送2次。第一次[状态码](https://so.csdn.net/so/search?q=状态码&spm=1001.2101.3001.7020)是204，第二次才成功；
因为在用fatch的post请求的时候，导致fetch第一次发送了一个Options请求，询问服务器是否支持修改的请求头，如果服务器支持，则在第二次中发送真正的请求。

## 参考文章

[阮一峰](https://www.ruanyifeng.com/blog/2020/12/fetch-tutorial.html)
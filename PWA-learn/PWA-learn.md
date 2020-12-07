# PWA学习笔记



## 1. serviceWorker API

首先要在网页中注册`serviceWorker`，具体操作如下： 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>PWA-LEARN</title>
    <link rel="stylesheet" href="index.css">
</head>
<body>
    <p>pwa学习</p>
    <script>
        navigator.serviceWorker.register('./sw.js', { scope: '/' }).then(ServiceWorkerRegistration => {
            console.log(ServiceWorkerRegistration)   
        }, error => {
            console.log(error)
        })
    </script>
</body>
</html>
```

在html中插入这样一段html代码，通过`navigator`对象上的`serviceWorker`使用`register`方法来进行注册，`register`接收两个参数，第一个参数是`serviceWorker`脚本的地址，这里是当前目录下的`sw.js`文件，第二个参数是一个对象，这个对象包含注册时可以指定的配置参数，目前这个对象只能有一个选项，就是`scope`，`scope`表示`service worker`可以控制的`URL范围`，通常我们指定为根目录即可。

​	接下来我们可以在`sw.js`里编写`serviceWorker`脚本，进行`serviceWorker编程`。`serviceWorker`是一个全新的上下文环境，这里不能修改`DOM`，也不能访问`window`，`localStorage`这些对象，只能使用一些特定的对象，比如`self`。`self`代表这个sw的全局作用域对象，操作`serviceworker`需要通过`self监听事件`完成，sw有三个事件可以进行监听，分别是`install（安装）`、`activate（激活）`、`fetch（获取）`，在对应的操作进行时触发。需要注意的是，只要sw的代码发生了任何改变，浏览器就会认为这是一个全新的sw版本，并`（install）安装`但是不会`（activate）立即激活`。`fetch`在资源请求时触发，比如引入css文件时，演示代码如下： 

```javascript
self.addEventListener('install', event => {
})

self.addEventListener('activate', event => {
    
})

self.addEventListener('fetch', event => {
    
})
```

install事件的回调中有两个操作需要注意，一是event.waitUntil()，这个函数可以在install和activate中使用，它传入一个promise，当这个promise完成时，install才真正完成，这也意味着它会推迟activate的执行。另一个是self.skipWaiting()，这个函数可以强制停止旧的sw并激活新的sw，通常event.waitUntil()和self.skipWaiting会联合使用，如下： 

```javascript
self.addEventListener('install', event => {
    event.waitUntil(self.skipWaiting())
})

```

这会使每当产生新的sw时，都会强制停止旧的sw并立即激活新的sw。

在activate中同样也有event.waitUntil()，用法与install相同，activate中第二个需要注意的是self.clients.claim(),clients指sw控制的所有页面，claim()会使一个激活的sw能够立即控制所有页面，当一个 service worker 被初始注册时，页面在下次加载之前不会使用它。 `claim()` 方法会立即控制这些页面,代码如下：

```javascript
self.addEventListener('activate', event => {
    event.waitUntil(self.clients.claim())
})
```



## 2.cache API

`cache API`提供了缓存网页资源的能力，通常我们在`install`中缓存需要缓存的资源，在`activate`中清除上一个sw版本中遗留的无用缓存，在fetch中捕获到资源请求后，去查询并返回缓存中的资源。

首先在`install`中的`waitUntil`中使用`caches`这个全局对象，它代表了所有缓存空间的集合，使用它的`open`方法开启一个缓存空间，open接收一个参数（缓存空间的名字），并用`promise`返回开启的缓存空间，具体操作如下：

```js
const CACHE_VERSION = 'cache_v1'	 //这里定义了一个静态变量
self.addEventListener('install', event => {
    event.waitUntil(caches.open(CACHE_VERSION).then(cache => {
        cache.addAll([
            '/',
            './index.css'
        ])
    }))  
})
```

这里把`CACHE_VERSION`传入了`open`,得到的`cache`使用`addAl`l来写入缓存，`addAll`接收一个数组，数组的每一项就是需要缓存资源的路径。

在`install`中缓存资源后，我们就可以在`fetch`中使用缓存，fetch中可以捕获到所有 的资源请求，请求时我们可以在`cache`中查找是否有缓存，如果查询到了则直接返回缓存，否则就用`fetch`来请求资源。代码如下： 

```js
self.addEventListener('fetch', event => {
    // check if request is made by chrome extensions or web page
  // if request is made for web page url must contains http.
    if (!(event.request.url.indexOf('http') === 0)) return; // skip the request. if request is not made with http protocol
    event.respondWith(caches.open(CACHE_VERSION).then(cache => {
        return cache.match(event.request).then(response => {
            if(response) {
                return response
            } else {
                return fetch(event.request).then(response => {
                    cache.put(event.request, response.clone())
                    return response
                })
            }
        })
    }))
})
```

`event.respondWith`作用和`waitUntil`类似，但只能在`fetch`中使用，当传入的` Promise` ` resolved `之后，才会将对应的 response 返回给浏览器。 我们同样先打开指定的缓存空间，得到缓存后使用match方法，match返回一个 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)解析为(resolve to)与 [`Cache`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache) 对象中的第一个匹配请求相关联的[`Response`](https://developer.mozilla.org/zh-CN/docs/Web/API/Response) 。如果没有找到匹配，[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 解析为 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)。找到则直接返回`response`，没找到则用`fetch`方法去请求资源，`fetch`可以传入一个[Request对象](<https://developer.mozilla.org/zh-CN/docs/Web/API/Request>)，得到response后使用`cache.put`将请求的资源进行键值对缓存，键位`request`，值为`response`，因为response是流式的只能读取一次，所以我们使用`clone`方法克隆一份，同时，我们返回`response`。

现在我们已经可以缓存和读取资源，我们也需要在新的sw启用前清理上一版本sw所缓存的资源，我们可以在activate中的waitUntil中进行这个操作，代码如下：

```js
self.addEventListener('activate', event => {
    event.waitUntil(caches.keys().then(cacheNames => {
        return Promise.all(cacheNames.map(cacheName => {
            if(cacheName !== CACHE_VERSION) {
               return caches.delete(cacheName)
            }
        }))
    }))
})
```

 [`Cache`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache) 接口的 **keys()** 方法返回一个 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) ，这个 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 将解析为一个[`Cache`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache) 键的数组。得到所有的`caches`键数组`cacheNames`后，使用`promise.all()`做并行处理，在里面遍历`cacheNames`，将缓存的`cacheName`与当前`cache`的版本号作对比，如果不同则使用`caches.delete()`方法删除,如果相同则什么都不做。

## 3.Notification API

Notification API可以给予`Web APP`推送通知的能力，即使用户关闭了网页，但是只要浏览器的线程还在，就能推送通知。

`Notification API`的`permission`属性：

- `default`：使用者尚未给予任何权限（因此不会显示任何通知）

- `granted`：使用者允许接收到网站的通知

- `denied`：使用者拒绝接收网站的通知

网站的默认`permission`是`default`，可以在页面上下文中使用Notification.requestPermission方法（不能在sw上下文中使用）在浏览器界面上产生一个允许通知的请求视窗，让使用者允许Web App可以启用通知功能。

![通知请求](https://mexion.xyz//images/study-notes/PWA-learn/request-notification.png)

当用户点击允许，我们就可以向用户推送通知了，在页面上下文中可以new一个`Notification`来推送通知，Notification接收两个参数，第一个参数是通知的标题，第二个参数是一个对象，为通知的配置属性，具体可以配置的参数详见[`Notification()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Notification/Notification)。

```js
new Notification('Hello Notification', {
	body: 'This is a Notification!'
})
```

![弹出通知](https://mexion.xyz//images/study-notes/PWA-learn/Notification.png)

而在sw的上下文中不能通过构造函数来弹出通知，需要使用`self.registration.showNotification(title, option)`来弹出通知。

## 4.Web App Manifest


  [Web应用程序清单](https://developer.mozilla.org/zh-CN/docs/Web/web%20app%20manifest)在一个JSON文本文件中提供有关应用程序的信息（如名称，作者，图标和描述）。manifest 的目的是将Web应用程序安装到设备的主屏幕，为用户提供更快的访问和更丰富的体验。

在页面的HTML头部添加manifest即可部署，如下：

```html
<link rel="manifest" href="/manifest.json">
```

manifest是一个json文件，具体配置信息参见[Web App Manifest](<https://developer.mozilla.org/zh-CN/docs/Web/Manifest>)。
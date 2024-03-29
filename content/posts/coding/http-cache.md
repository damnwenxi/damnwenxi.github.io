+++
title = "关于HTTP缓存策略"
date = 2021-08-22 21:54:48
author = "Damnwenxi"
tags = ["Technology"]
keywords = ["Cache", "HTTP"]
cover = "posts/coding/http.png"
readingTime = true
description = "为什么要有 HTTP 缓存？ 首先，我们知道每一次 http 请求都是需要耗费时间和资源的。在如今这个“越来越卷”的时代，浏览器性能优化和用户体验都如此重要，以至于我们需要花费大量时间来优化首屏渲染、做懒加载、做一系列让用户看起来“快”的事情。而 http 请求作为页面内容的来源，不仅出现频率高，而且极易被用户察觉。而页面中的静态资源大多数时候是不会发生改变的，如果没有缓存机制..."
+++

## 为什么要有 HTTP 缓存？

首先，我们知道每一次 http 请求都是需要耗费时间和资源的。在如今这个“越来越卷”的时代，浏览器性能优化和用户体验都如此重要，以至于我们需要花费大量时间来优化首屏渲染、做懒加载、做一系列让用户看起来“快”的事情。而 http 请求作为页面内容的来源，不仅出现频率高，而且极易被用户察觉。而页面中的静态资源大多数时候是不会发生改变的，如果没有缓存机制，那么每一次的页面刷新都将把所有的资源都重新请求一遍，这不仅会增加用户的时间成本，给用户带来不好的体验，而且还会极大的增加服务器的开销。对于任何一个给正常人使用的网站来说，http 缓存策略都是必不可少的。

## 强制缓存 VS 协商缓存

说到 http 缓存，那必然离不开目前存在的两种缓存机制：强制缓存机制、协商缓存机制。
所谓强制缓存，就是浏览器不需要经过询问服务器，也就是浏览器自己决定是否使用缓存。
而协商缓存则是基于浏览器和服务器协商这个过程的，也就是说，浏览器仍然会向服务器发送 http 请求，只不过服务器不一定会返回浏览器所需要的资源，他可能会告诉浏览器，使用本地缓存。即使 http 请求发送出去了，但是服务器返回的内容可能是一个简单的状态码，而不需要将数据量庞大的资源文件通过网络发送回来，从而达到节省资源的目的。

> 在说两种缓存策略之前，首先要明白，使用何种缓存是完全由服务端决定的，如果服务端不设置任何的缓存策略，那前端不会使用任何缓存。所以这为啥是前端要掌握的呢？我也不明白。。。

## 强制缓存

在 HTTP 1.0 以前，浏览器只能根据请求头（headers）里面的 "expires" 字段来决定是否使用缓存。在第一次向服务器发起某个请求时，服务器不仅会把资源返回给我们，还会在响应头里面设置 expires，注意 _expires 是一个绝对时间_ ，也就是说，由 expires 决定的缓存，是受客户端（浏览器端）本地时间的影响的。如果设置了错误的本地时间或者与服务器时间对不上的话，很可能会造成缓存策略不起作用。

浏览器在第二次请求同样的内容的时候，会现在本地进行决策，比对当前时间和上一次请求返回的 expires 时间，决定是否使用本地缓存。expires 长这样：

```javascript
Expires: Wed, 21 Oct 2000 07:28:00 GMT
```

而到了 HTTP 1.1 以后，cache-control 出现了，它是一种新的缓存控制手段，同样存在于 headers 中，同样由服务端设置，不同之处在于，它提供了丰富的配置，以供开发者们实现更为先进合理的缓存策略。

常见的 cache-control 字段有这么些：

```js
Cache-Control: max-age=20000, s-maxage=20000
Cache-Control: no-cache
Cache-Control: no-store
```

其中*max-age*和*s-maxage*都是相对时间，以上的配置表示在此之后的 20000 秒内，都可以使用该缓存。相对时间的好处在于，它不受客户端本地时间的影响，总是在当前时间之后的 20000 秒内有效。

> no-cache 才是真正的不使用缓存

*no-cache*表示：使用缓存前，强制要求把请求提交给服务器进行验证(协商缓存验证)。
*no-store*表示：不存储有关客户端请求或服务器响应的任何内容，即不使用任何缓存。

expires 和 cache-control 的优先级：如果 cache-control 里面设置了 max-age，则会忽略 expires。

## 协商缓存

前面说到，协商缓存需要有一个客户端和服务端进行协商的过程。而这个过程是如何实现的呢？也是通过 headers 配置，具体有以下几个：更准确的说，应该是几对。

> 注意，使用协商缓存的前提是：cache-control 需要设置为 no-cache，也就是让浏览器不使用强制缓存，走协商过程

### Etag & If-None-Match

在客户端的请求第一次到达服务器后，服务器首先会对当前需要返回的资源文件进行一次“指纹采集”，可以理解为标识文件当前状态唯一性的一串字符，然后将它放到响应头的*Etag*里面。客户端下一次请求同样的资源时，会在请求头上带上*If-None-Match*字段，服务端收到后拿去跟请求的资源比对，如果一致则表明资源没有发生改变，返回 304，让客户端使用缓存，否则返回 200 和最新的内容，然后设置新的*Etag*。

Etag 和 If-None-Match 的格式长这样：

```js
ETag: "W/611f74a1-5d16"
If-None-Match: "W/611f74a1-5d16"
```

### Last-Modified & If-Modified-Since

和 _Etag & If-None-Match_ 一样，这一对属性也是分别存在响应头和请求头里面的。不一样的是，这一对值是通过比对文件最后修改的时间来决定是否使用缓存。处理逻辑跟*Etag*一致，这里不再赘述。

### Etag VS Last-Modified

_Etag_ 和 _Last-Modified_ 虽说是两种不同的校验方式，但是他们的流程大致是一样的。区别在于，计算*Etag*的 hash 值可能需要耗费大量的计算性能，取决于文件大小，而获取修改时间所需要的开销是极小的。那我们为什么还要使用 Etag 呢？这是因为，在某些时候，可能文件内容并没有被修改，而只是经历了一次重命名啊什么的，这也会导致文件的修改时间发生变化。而实际上文件内容并没有发生改变，这样一来，本该使用缓存的请求会被错误的重新发送一次，导致额外的性能和网络开销。使用*Etag*则可以避免这种问题，因为*Etag*实际上是比对文件的内容。这也是为什么*Etag*和*Last-Modified*同时存在的时候*Etag*优先级会高于*Last-Modified*。

以上就是我 http 缓存策略的一些理解和总结。我个人认为，制定 http 缓存策略是非常有必要且重要的，一个好的缓存策略可能会让你的网站可以在特殊情况下承受更多的并发量，节约大量的服务器成本，合理运用缓存，不仅会给服务提供方节省成本，还可以增强用户体验，一举多得。

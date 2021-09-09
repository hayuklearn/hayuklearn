[TOC]

# 关于跨域

## 参考链接

1. [阮一峰《跨域资源共享 CORS 详解》](http://www.ruanyifeng.com/blog/2016/04/cors.html)
2. [Mozilla《跨源资源共享（CORS）》](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)
3. [博客园《Flutter web 踩坑关于浏览器的跨域的深思》](https://www.cnblogs.com/lizhanqi/p/13564660.html)
4. [UNI-APP H5 跨域问题解决方案](https://ask.dcloud.net.cn/m/article/35267)

## 概念

`CORS`

`cross-origin resource sharing`

`跨域资源共享`

## 作用

它允许浏览器向跨源服务器，发出`XMLHttpRequest`请求，从而克服了`AJAX`只能同源使用的限制。

## 简单请求与非简单请求

|简单请求<br/>`simple request`|非简单请求<br/>`not-so-simple request`|
|:--:|:--:|

### 简单请求

#### 条件

##### 条件一

以下三种请求方式之一：

1. `HEAD`
2. `GET`
3. `POST`

##### 条件二

不超过以下五种请求头部：

1. `Accept`
2. `Accept-Language`
3. `Content-Language`
4. `Last-Event-ID`
5. `Content-Type`：仅限`application/x-www-form-urlencoded` `multipart/form-data` `text/plain`之一

### 非简单请求

#### 条件

`PUT`或`DELETE`或者`Content-Type`字段的类型是`application/json`

#### 预检`preflight`

### 流程

```flow

st=>start: 浏览器开始
cond-is-simple-request=>condition: 是否简单请求
op01=>operation: 增加一个Origin头部
op02=>operation: 发送给服务端
cond-is-origin-allow=>condition: Origin
是否在许可范围内
op03=>operation: 服务端响应包含
Access-Control-Allow-Origin
Access-Control-Allow-Credentials
Access-Control-Expose-Headers
Content-Type
op04=>operation: 服务端响应不包含
Access-Control-Allow-Origin
op05=>operation: 预检preflight
cond-preflight=>condition: Origin
是否在许可范围内
op06=>operation: 正常请求
op07=>operation: 跨域报错

e=>end: 结束

st->cond-is-simple-request
cond-is-simple-request(yes)->op01->op02->cond-is-origin-allow
cond-is-simple-request(no)->op05->cond-preflight
cond-is-origin-allow(yes)->op03->e
cond-is-origin-allow(no)->op04->op07->e
cond-preflight(yes, right)->op06->e
cond-preflight(no)->op04
```

## 解决跨域报错问题

1. Chrome 浏览器禁用安全设置

`"{PATH_FOR_CHROME}\chrome.exe" --disable-web-security --user-data-dir={USER_DATA_PATH}`

2. 服务端设置允许跨域

3. `Nginx`转发客户端请求时，加上`Access-Control-Allow-Origin: *`请求头

4. `jsonp`解决方案是利用`script`标签支持跨域加载，但是`script`都是通过`get`请求，因此`josnp`发送的都是`get`请求，因此无法`post`数据
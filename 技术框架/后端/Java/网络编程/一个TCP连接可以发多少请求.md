---
title: '一个TCP连接可以发多少请求'
date: 2020-03-09
categories: Java
tags: TCP
---
# 一个TCP连接可以发多少请求

## 1、现代浏览器在与服务器建立一个TCP连接后是否会在一个HTTP请求完成后断开？什么情况下会断开？
默认情况下，已建立的TCP连接在一个HTTP请完成后不会断开，只有在请求头中声明 `connection: close`时才会在http请求完成后断开连接。

在http/1.0时，一个服务器在发送完一个http响应后，会断开TCP连接，这样每次请求都会重新建立和断开TCP连接，开销过大；
在http/1.1时，把`Connection`头写进了标准，并且默认开启`持久连接`。

## 2、一个TCP连接对应几个http请求
如果维持TCP连接，一个连接可以发送多个http请求。

## 3、一个TCP连接中，http请求可以可以一起发送么（比如一起发三个请求，再三个响应一起接收）？
在HTTP/1.1存在Pipelining技术可以完成一个TCP同时发送多个http请求，但由于浏览器默认关闭该功能，可认为行不通；
在http2中由于`Multiplexing`特性的存在，多个http请求可以在同一个TCP连接中并行进行。

在http/1.1时，单个TCP连接在同一时刻只能处理一个请求。

HTTP/1.1在规范中规定了`Pipelining`来试图解决这个问题，但该功能在浏览器中默认关闭。
`RFC 2616` 中定义的 `Piprlining`：
>A client that supports persistent connections MAY "pipeline" its requests (i.e., send multiple requests without waiting for each response). A server MUST send its responses to those requests in the same order that the requests were received.
>一个支持持久连接的客户端可以在一个连接中发送多个请求（不需要等待任意请求的响应）。收到请求的服务器必须按照请求收到的顺序发送响应。

在http/1.1时，浏览器提高页面加载效率的方法：
- 维持和服务器已经建立的TCP连接，在同一连接上顺序处理多个请求；
- 和服务器建立多个TCP连接。

在HTTP2时提供了`Multiplexing`多路传输特性，可以在一个TCP连接中同时完成多个http请求。

## 4、为什么有的时候刷新页面不需要重新建立 SSL 连接？
TCP连接有时会被浏览器和服务器维持一段时间，TCP不需要重新建立，SSL也会使用之前建立的。

## 5、浏览器对同一 Host 建立 TCP 连接到数量有没有限制？
有限制，不同浏览器有区别，chrome浏览器最多允许对同一个Host建立6个TCP连接。

## 6、收到的 HTML 如果包含几十个图片标签，这些图片是以什么方式、什么顺序、建立了多少连接、使用什么协议被下载下来的呢？
如果图片都是https连接并在同一个域名下，那浏览器在SSL握手后回合服务器商量能不能使用http2，如果可以使用http2，就是用Multiplexing特性在这个连接上进行多路传输。不过不一定所有挂在这个域名下的资源都会使用这一个TCP连接去获取，但可以确定Multiplexing很可能被用到。

如果发http2用不了，或用不了https（现实中的http2都是在https上实现的，这种情况下只能使用http/1.1），那浏览器就会在一个HOST建立多个TCP连接，连接数量的最大限制取决于浏览器设置，这些连接会在空闲时被浏览器用来发送新的请求，如果所有的连接都被用于发送请求，那其余请求只能等待空闲连接出现。
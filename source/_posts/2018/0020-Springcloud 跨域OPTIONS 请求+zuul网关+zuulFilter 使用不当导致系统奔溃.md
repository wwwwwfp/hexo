---
title: Springcloud 跨域OPTIONS 请求+zuul网关+zuulFilter 使用不当导致系统奔溃的原因
index_img: /img/cover/20.jpg
categories:
  - Spring Cloud
tags:
  - 跨域
  - zuul
abbrlink: 6e9fe614
date: 2018-07-15 01:10:53
---

当访问子微服务时候，如果存在跨域问题，浏览器会默认发送一个OPTIONS欲请求。

验证通过后才会调用真正的接口。

如果使用zuul调用接口，并且使用到zuulFilter时，在处理逻辑中需要注意如下几点：

    1：在向客户端返回数据时，不要对response直接操作，通过RequestContext API来操作。

    2：对特殊的请求做不同的处理,如OPTIONS 

比如：在使用zuulFilter做身份认证，这个时候需要用到一些参数 用户id/token 等，如果是我们自己的接口，这些参数肯定时存在的，但是像OPTIONS 请求，时不带这些参数的。那么在通过zuulFilter时，验证肯定会失败。一般会向客户端响应一个错误的数据。虽然相应的数据时错误的，但是OPTIONS欲请求已经验证通过。这是会调用我们自己的接口。这儿就存在一个问题，当OPTIONS 验证失败后会向客户端响应一条ERROR信息，而常规的请求也会响应请求数据。这样浏览器会一直接受不到第二个请求数据，导致数据访问不了。

解决方案：可以在是否使用Filter逻辑中做一个处理。对特殊请求不启用该拦截器

也可以在处理逻辑中对特殊请求做相应的处理。


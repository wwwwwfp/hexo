---
title: Nginx 配置和使用
index_img: /img/cover/13.jpg
categories:
  - Nginx
tags:
  - Nginx
abbrlink: e0aaee0e
date: 2019-03-20 10:48:59
---

NGINX参考文档：http://www.nginx.cn/doc/

+ **nginx**

  一个多进程/多线程高性能web服务器，在linux系统中，nginx启动后会以后台守护进程（daemon）的方式去运行，后台进程包含一个master进程和多个worker进程（这个数量可以在nginx.conf配置文件中worker_processes这个参数设置）。nginx工作模式是以多进程的方式来工作的，当然nginx也是支持多线程的方式的，只是我们主流的方式还是多进程的方式，也是nginx的默认方式。nginx在启动之后会有一个master进程和多个worker进程（默认是一个），多个worker子进程将监听同一个端口，并行处理请求。

  master主进程主要用来管理worker进程，主要作用是：读取并验正配置信息，接收来自客户端的的请求，向各worker进程发送信号，监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程。而用户的请求则是worker进程来响应的。

  nginx是通过信号来控制，比如关闭，重启等去控制nginx进程。nginx信号是属于nginx进程间的通信的一种机制，比如master主进程控制多个worker子进程，也是通过信号控制的，如下图。

  ![](1.png)

  worker 进程数应该设置为等于 CPU 的核数，高流量并发场合也可以考虑将进程数提高至 CPU 核数 * 2。

  安装目录 /ulsr/local/nginx, 看到如下4个目录
  
  ... conf 配置文件
  
  ... html 网页文件
  
  ... logs 日志文件
  
  ...sbin 主要二进制程序

+ **Nginx的信号控制**

  ![](2.png)
  
+ **具体语法:**
  Kill -信号选项 nginx的主进程号

  Kill -HUP 4873
  
  Kill -信号控制 `cat /xxx/path/log/nginx.pid`
  
  Kil; -USR1 `cat /xxx/path/log/nginx.pid`
  
  Nginx 配置段
  
  // 全局区
  
  worker_processes 1;
  
  // 有1个工作的子进程,可以自行修改,但太大无益,因为要争夺CPU,一般设置为 CPU数*核数
  
  Event {
  
  // 一般是配置nginx连接的特性
  
  // 如1个word能同时允许多少连接
  
  worker_connections 1024; // 这是指 一个子进程最大允许连1024个连接
  
  }
  
  Nginx 配置段
  
  日志格式 是指记录哪些选项
  
  默认的日志格式: main
  
  log_format mylog '$remote_addr - $remote_user [$time_local] "$request" '
  
  '$status $body_bytes_sent "$http_referer" '
  
  '"$http_user_agent" "$http_x_forwarded_for"';
  
  Nginx允许针对不同的server做不同的Log ,(有的web服务器不支持,如lighttp)，默认全局
  
  access_log logs/access_8080.log mylog;
  
  声明log log位置 log格式;
+ **Nginx日志切割**
  shell+定时任务+nginx信号管理,完成日志按日期存储

  分析思路:
  
  凌晨00:00:01,把昨天的日志重命名,放在相应的目录下
  
  再USR1信息号控制nginx重新生成新的日志文件
  
  echo命令用于在shell中打印shell变量的值，或者直接输出指定的字符串。
  
  echo `date -d yesterday +%Y%m%d`
  
  echo $(date -d yesterday +%Y%m%d) shell返回用反引号 或$
+ **具体脚本:**
  .#!/bin/bash
  
  base_path='/usr/local/nginx/logs'
  
  log_path=$(date -d yesterday +"%Y%m")
  
  day=$(date -d yesterday +"%d")
  
  mkdir -p $base_path/$log_path
  
  mv $base_path/access.log $base_path/$log_path/access_$day.log
  
  .#echo $base_path/$log_path/access_$day.log
  
  kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
  
  定时任务
  
  Crontab 编辑定时任务
  
  01 00 * * * /xxx/path/b.sh 每天0时1分(建议在02-04点之间,系统负载小)
  
  location 语法
  
  location [=|~|~*|^~] patt {
  
  }
  
  因此,大类型可以分为3种
  
  location = patt {} [精准匹配]
  
  location patt{} [一般匹配]
  
  location ~ patt{} [正则匹配]
  
  location / {
  
  root /usr/local/nginx/html;
  
  index index.html index.htm;
  
  }
  
  location ~ image {
  
  root /var/www/image;
  
  index index.html;
  
  }
  
  如果我们访问 http://xx.com/image/logo.png
  
  此时, “/” 与”/image/logo.png” 匹配
  
  同时,”image”正则 与”image/logo.png”也能匹配,谁发挥作用?
  
  正则表达式的成果将会使用.
  
  图片真正会访问 /var/www/image/logo.png
  
  location / {
  
  root /usr/local/nginx/html;
  
  index index.html index.htm;
  
  }
  
  location /foo {
  
  root /var/www/html;
  
  index index.html;
  
  }
  
  我们访问 http://xxx.com/foo
  
  对于uri “/foo”, 两个location的patt,都能匹配他们
  
  即 ‘/’能从左前缀匹配 ‘/foo’, ‘/foo’也能左前缀匹配’/foo’,
  
  此时, 真正访问 /var/www/html/index.html
  
  原因:’/foo’匹配的更长,因此使用之.;
  
  location 匹配规则

  ![](3.png)

+ **Rewrite 重写**
  重写中用到的指令

  if (条件) {} 设定条件,再进行重写
  
  set #设置变量
  
  return #返回状态码
  
  break #跳出rewrite
  
  rewrite #重写
  
  If 语法格式
  
  If 空格 (条件) {
  
  重写模式
  
  }
  
  条件又怎么写?
  
  答:3种写法
  
  1: “=”来判断相等, 用于字符串比较
  
  2: “~” 用正则来匹配(此处的正则区分大小写)
  
  ~* 不区分大小写的正则
  
  3: -f -d -e来判断是否为文件,为目录,是否存在.
  
  例子:
  
  if ($remote_addr = 192.168.1.100) {
  
  return 403;
  
  }
  
  if ($http_user_agent ~ MSIE) {
  
  rewrite ^.*$ /ie.htm;
  
  break; #(不break会循环重定向)
  
  }
  
  if (!-e $document_root$fastcgi_script_name) {
  
  rewrite ^.*$ /404.html break;
  
  }
  
  注, 此处还要加break,
  
  以 xx.com/dsafsd.html这个不存在页面为例,
  
  我们观察访问日志, 日志中显示的访问路径,依然是GET /dsafsd.html HTTP/1.1
  
  提示: 服务器内部的rewrite和302跳转不一样.
  
  跳转的话URL都变了,变成重新http请求404.html, 而内部rewrite, 上下文没变,
  
  就是说 fastcgi_script_name 仍然是 dsafsd.html,因此 会循环重定向.
  
  set 是设置变量用的, 可以用来达到多条件判断时作标志用.
  
  达到apache下的 rewrite_condition的效果
  
  如下: 判断IE并重写,且不用break; 我们用set变量来达到目的
  
  if ($http_user_agent ~* msie) {
  
  set $isie 1;
  
  }
  
  if ($fastcgi_script_name = ie.html) {
  
  set $isie 0;
  
  }
  
  if ($isie 1) {
  
  rewrite ^.*$ ie.html;
  
  }
  
  Rewrite语法
  
  Rewrite 正则表达式 定向后的位置 模式
  
  Goods-3.html ---->Goods.php?goods_id=3
  
  goods-([\d]+)\.html ---> goods.php?goods_id =$1
  
  location /ecshop {
  
  index index.php;
  
  rewrite goods-([\d]+)\.html$ /ecshop/goods.php?id=$1;
  
  rewrite article-([\d]+)\.html$ /ecshop/article.php?id=$1;
  
  rewrite category-(\d+)-b(\d+)\.html /ecshop/category.php?id=$1&brand=$2;
  
  rewrite category-(\d+)-b(\d+)-min(\d+)-max(\d+)-attr([\d\.]+)\.html /ecshop/category.php?id=$1&brand=$2&price_min=$3&price_max=$4&filter_attr=$5;
  
  rewrite category-(\d+)-b(\d+)-min(\d+)-max(\d+)-attr([\d+\.])-(\d+)-([^-]+)-([^-]+)\.html /ecshop/category.php?id=$1&brand=$2&price_min=$3&price_max=$4&filter_attr=$5&page=$6&sort=$7&order=$8;
  
  }
  
  注意:用url重写时, 正则里如果有”{}”,正则要用双引号包起来

+ **网页内容的压缩编码与传输速度优化 http gzip**

  我们观察news.163.com的头信息
  
  请求:
  
  Accept-Encoding:gzip,deflate,sdch
  
  响应:
  
  Content-Encoding:gzip
  
  Content-Length:36093
  
  再把页面另存下来,观察,约10W字节,实际传输的36093字节
  
  原因-------就在于gzip压缩上.

+ **原理:**

  浏览器 ---> 请求----> 声明可以接受(accept-encoding):gzip压缩 或 deflate压缩 或compress 或 sdch压缩

  从http的角度看 -----> 请求头声明accept-encoding:gzip deflate sdch (是指压缩算法,其中sdch是google倡导的一种压缩方式,目前支持的服务器尚不多) ---> 服务器-->回应 ---> 把内容用gzip方式压缩 ----> 发给浏览器 ---> 接收gzip压缩内容 ---> 解码gzip ----> 浏览

+ **gzip配置的常用参数**

  gzip on|off; #是否开启gzip
  
  gzip_buffers 32 4K| 16 8K #缓冲(压缩在内存中缓冲几块? 每块多大?)
  
  gzip_comp_level [1-9] #推荐6 压缩级别(级别越高,压的越小,越浪费CPU计算资源)
  
  gzip_disable #正则匹配UA 什么样的Uri不进行gzip
  
  gzip_min_length 200 # 开始压缩的最小长度(再小就不要压缩了,意义不在)
  
  gzip_http_version 1.0|1.1 # 开始压缩的http协议版本(可以不设置,目前几乎全是1.1协议)
  
  gzip_proxied # 设置请求者代理服务器,该如何缓存内容
  
  gzip_types text/plain application/xml # 对哪些类型的文件用压缩 如txt,xml,html ,css
  
  gzip_vary on|off # 是否传输gzip压缩标志
  
  gzip_types 类型 可以到 nginx/conf/mime.types 中查看
  
  zip作用域 : http, server, locationg
  
  gzip参考URL：http://www.nginx.cn/doc/standard/httpgzip.html
  
  参考博客：https://blog.csdn.net/liupeifeng3514/article/details/79018334

+ **注意: 图片/mp3这样的二进制文件,不必压缩**
  因为压缩率比较小, 比如100->80字节,而且压缩也是耗费CPU资源的.

  比较小的文件不必压缩,
  
  网页nginx的缓存设置，提高网站性能。(expires)
  
  对于网站的图片,尤其是新闻站, 图片一旦发布, 改动的可能是非常小的.我们希望 能否在用户访问一次后, 图片缓存在用户的浏览器端,且时间比较长的缓存.
  
  可以, 用到 nginx的expires设置 .
  
  nginx中设置过期时间,非常简单,
  
  在location或if段里,来写.
  
  格式 expires 30s;
  
  expires 30m;
  
  expires 2h;
  
  expires 30d;
  
  (注意:服务器的日期要准确,如果服务器的日期落后于实际日期,可能导致缓存失效)

+ **另: 304 也是一种很好的缓存手段**
  原理是: 服务器响应文件内容是,同时响应etag标签(内容的签名,内容一变,他也变), 和 last_modified_since 2个标签值

  浏览器下次去请求时,头信息发送这两个标签, 服务器检测文件有没有发生变化,如无,直接头信息返回 etag,last_modified_since
  
  浏览器知道内容无改变,于是直接调用本地缓存.
  
  这个过程,也请求了服务器,但是传着的内容极少.
  
  对于变化周期较短的,如静态html,js,css,比较适于用这个方式

  ![](4.png)

  Nginx对于图片,js等静态文件的缓存设置
  
  注:这个缓存是指针对浏览器所做的缓存,不是指服务器端的数据缓存.
  
  主要知识点: location expires指令
  
  location ~ \.(jpg|jpeg|png|gif)$ {
  
  expires 1d;
  
  }
  
  location ~ \.js$ {
  
  expires 1h;
  
  }
  
  设置并载入新配置文件,用firebug观察,
  
  会发现 图片内容,没有再次产生新的请求,原因--利用了本地缓存的效果.
  
  注: 在大型的新闻站,或文章站中,图片变动的可能性很小,建议做1周左右的缓存
  
  Js,css等小时级的缓存.
  
  如果信息流动比较快,也可以不用expires指令,
  
  用last_modified, etag功能(主流的web服务器都支持这2个头信息)
  
  原理是:
  
  响应: 计算响应内容的签名, etag 和 上次修改时间
  
  请求: 发送 etatg, If-Modified-Since 头信息.
  
  服务器收到后,判断etag是否一致, 最后修改时间是否大于if-Modifiled-Since
  
  如果监测到服务器的内容有变化,则返回304,
  
  浏览器就知道,内容没变,直接用缓存.
  
  304 比起上面的expires 指令
  
  多了1次请求,
  
  但是比200状态,少了传输内容.
  
  高304缓存原理：
  
  在第一次请求服务器的时候在获取资源之后是会先把该资源缓存在本地的，同时服务器response返回了一个响应头ETag，ETag全称Entity Tag，用来标识一个资源。在具体的实现中，ETag可以是资源的hash值，也可以是一个内部维护的版本号。但不管怎样，ETag应该能反映出资源内容的变化，这是Http缓存可以正常工作的基础。服务器对于hello world这个字符串使用上述返回的ETag来表示，只要hello world这个资源不变，这个Etag就不会变。
  
  客户端第二次请求服务器的时候，利用请求头If-None-Match来告诉服务器自己已经有个ETag为xxx的资源。如果服务器上的资源没有变化，也就是说服务器上的资源的ETag也是xxx的话，服务器就不会再返回该资源的内容，而是返回一个304的响应，告诉浏览器该资源没有变化，缓存有效，浏览器将直接调用本地缓存。
  
  响应头Cache-Control
  
  每个资源都可以通过Http头Cache-Control来定义自己的缓存策略，Cache-Control控制谁在什么条件下可以缓存响应以及可以缓存多久。 最快的请求是不必与服务器进行通信的请求：通过响应的本地副本，我们可以避免所有的网络延迟以及数据传输的数据成本。为此，HTTP 规范允许服务器返回一系列不同的 Cache-Control 指令，控制浏览器或者其他中继缓存如何缓存某个响应以及缓存多长时间。
  
  Cache-Control 头在 HTTP/1.1 规范中定义，取代了之前用来定义响应缓存策略的头(例如 Expires)。当前的所有浏览器都支持 Cache-Control，因此，使用它就够了。
  
  参考博客：https://blog.csdn.net/canot/article/details/76359917
  
  高Nginx反向代理+负载均衡
  
  用nginx做反向代理和负载均衡非常简单,
  
  支持两个用法 1个proxy, 1个upstream,分别用来做反向代理,和负载均衡
  
  以反向代理为例, nginx不自己处理php的相关请求,而是把php的相关请求转发给apache来处理.

  ![](5.png)

  反向代理后端如果有多台服务器,自然可形成负载均衡,
  
  但proxy_pass如何指向多台服务器?
  
  把多台服务器用 upstream指定绑定在一起并起个组名,
  
  然后proxy_pass指向该组
  
  默认的均衡的算法很简单,就是针对后端服务器的顺序,逐个请求.
  
  反向代理导致了后端服务器的IP,为前端服务器的IP,而不是客户真正的IP,怎么办?
  
  答案：X-ForWard-For 配置属性
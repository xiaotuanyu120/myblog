---
title: 面试题汇总-01
date: 2016-07-18 15:29:00
categories: interview
tags: [interview,ha,dns,http,lvs]

---
## heartbeat和keepalived的优缺点
> 引用来源: [**HAproxy作者对这个问题的看法**](http://www.formilux.org/archives/haproxy/1003/3259.html)
> 
> Well, I'm not sure whether you'll find a response here as this is purely a heartbeat question.
> 
> Anyway, I'd like to say that I'm amazed by the number of people who use heartbeat to get a redundant haproxy setup. It is not the best tool for *this* job, it was designed to build clusters, which is a lot different from having two redundant stateless network equipments. Network oriented tools such as keepalived or ucarp are the best suited for that task.
> 
> The difference between those two families is simple :
> 
> a cluster-oriented product such as heartbeat will ensure that a shared resource will be present at *at most* one place. This is very important for shared filesystems, disks, etc... It is designed to take a service down on one node and up on another one during a switchover. That way, the shared resource may never be concurrently accessed. This is a very hard task to accomplish and it does it well.
> a network-oriented product such as keepalived will ensure that a shared IP address will be present at *at least* one place. Please note that I'm not talking about a service or resource anymore, it just plays with IP addresses. It will not try to down or up any service, it will just consider a certain number of criteria to decide which node is the most suited to offer the service. But the service must already be up on both nodes. As such, it is very well suited for redundant routers, firewalls and proxies, but not at all for disk arrays nor filesystems.
> The difference is very visible in case of a dirty failure such as a split brain. A cluster-based product may very well end up with none of the nodes offering the service, to ensure that the shared resource is never corrupted by concurrent accesses. A network-oriented product may end up with the IP present on both nodes, resulting in the service being available on both of them. This is the reason why you don't want to serve file-systems from shared arrays with ucarp or keepalived.
> 
> The nature of the controls and changes also has an impact on the switch time and the ability to test the service offline. For instance, with keepalived, you can switch the IP from one node to another one in just one second in case of a dirty failure, or in zero delay in case of volunteer switch, because there is no need to start/stop anything. That also means that even if you're experiencing flapping, it's not a problem because even if the IP constantly moves, it moves between places where the service is offered. And since the service is permanently available on the backup nodes, you can test your configs there without impacting the master node.
> 
> So in short, I would not like to have my router/firewall/load balancer running on heartbeat, as well as I would not like to have my fileserver/ disk storage/database run on keepalived.
> 
> With keepalived, your setup above is trivial. Just configure two interfaces with their shared IP addresses, enumerate the interfaces you want to track, declare scripts to check the services if you want and that's all. If any interface fails or if haproxy dies, the IP immediately switches to the other node. If both nodes lose the same interface (eg: shared switch failure), you still have part of the service running on one of the nodes on the other interface.
> 
> Hoping this helps understanding the different types of architectures one might encounter,
> 
> Willy Received on 2010/03/25 22:38

<!--more-->

## http协议的包头的构造
> 引用来源： [**作业部落格里一个优秀解读**](https://www.zybuluo.com/yangfch3/note/167490)
> 
> ---
> 
> HTTP协议详解
> 
> http
> 
> http？
> 
> 全称：超文本传输协议(HyperText Transfer Protocol)
> 作用：设计之初是为了将超文本标记语言(HTML)文档从Web服务器传送到客户端的浏览器。现在http的作用已不局限于HTML的传输。
> 版本：http/1.0 http/1.1* http/2.0
> 
> 两篇佳文
> > [**输入网址之后发生了什么**](https://www.zybuluo.com/yangfch3/note/113028)
> > [**Web开发新人培训系列**](http://rapheal.sinaapp.com/)
> 
> URL详解
> 
> 一个示例URL
> 
> http://www.mywebsite.com/sj/test;id=8079?name=sviergn&x=true#stuff
> 
> Schema: http
> host: www.mywebsite.com
> path: /sj/test
> URL params: id=8079
> Query String: name=sviergn&x=true
> Anchor: stuff
> scheme：指定低层使用的协议(例如：http, https, ftp)
> host：HTTP服务器的IP地址或者域名
> port#：HTTP服务器的默认端口是80，这种情况下端口号可以省略。如果使用了别的端口，必须指明，例如 http://www.mywebsite.com:8080/
> path：访问资源的路径
> url-params
> query-string：发送给http服务器的数据
> anchor：锚
> 无状态的协议
> 
> http协议是无状态的：
> 
> 同一个客户端的这次请求和上次请求是没有对应关系，对http服务器来说，它并不知道这两个请求来自同一个客户端。
> 解决方法：Cookie机制来维护状态
> 
> 
> 既然Http协议是无状态的，那么Connection:keep-alive 又是怎样一回事？
> 
> 无状态是指协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。从另一方面讲，打开一个服务器上的网页和你之前打开这个服务器上的网页之间没有任何联系。
> http消息结构
> 
> Request 消息的结构：三部分
> 
> 第一部分叫Request line（请求行）， 第二部分叫http header, 第三部分是body
> 
> 请求行：包括http请求的种类，请求资源的路径，http协议版本
> http header：http头部信息
> body：发送给服务器的query信息 
> 当使用的是"GET" 方法的时候，body是为空的（GET只能读取服务器上的信息，post能写入） 
> 
> GET /hope/ HTTP/1.1   //---请求行
> Host: ce.sysu.edu.cn
> Accept: */*
> Accept-Encoding: gzip, deflate, sdch
> Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.6
> Cache-Control: max-age=0
> Cookie:.........
> Referer: http://ce.sysu.edu.cn/hope/
> User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.130 Safari/537.36
> 
> ---分割线---
> 
> POST /hope/ HTTP/1.1   //---请求行
> Host: ce.sysu.edu.cn
> Accept: */*
> Accept-Encoding: gzip, deflate, sdch
> Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.6
> Cache-Control: max-age=0
> Cookie:.........
> Referer: http://ce.sysu.edu.cn/hope/
> User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.130 Safari/537.36
> ...body...
> Response消息的结构
> 
> 也分为三部分，第一部分叫request line, 第二部分叫request header，第三部分是body
> 
> request line：协议版本、状态码、message
> request header：request头信息
> body：返回的请求资源主体 
> 
> 
> HTTP/1.1 200 OK
> Accept-Ranges: bytes
> Content-Encoding: gzip
> Content-Length: 4533
> Content-Type: text/html
> Date: Sun, 06 Sep 2015 07:56:07 GMT
> ETag: "2788e6e716e7d01:0"
> Last-Modified: Fri, 04 Sep 2015 13:37:55 GMT
> Server: Microsoft-IIS/7.5
> Vary: Accept-Encoding
> X-Powered-By: ASP.NET
> <!DOCTYPE html>
> ...
> <>
> ...
> get 和 post 区别
> 
> http协议定义了很多与服务器交互的方法，最基本的有4种，分别是GET,POST,PUT,DELETE。 一个URL地址用于描述一个网络上的资源，而HTTP中的GET, POST, PUT, DELETE 就对应着对这个资源的查，改，增，删4个操作。 我们最常见的就是GET和POST了。GET一般用于获取/查询资源信息，而POST一般用于更新资源信息.
> 
> GET 提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，如EditPosts.aspx?name=test1&id=123456。POST 方法是把提交的数据放在HTTP包的Body中。
> 
> GET 提交的数据大小有限制（因为浏览器对URL的长度有限制），而POST方法提交的数据没有限制.
> 
> GET 方式需要使用Request.QueryString 来取得变量的值，而POST方式通过Request.Form来获取变量的值。
> 
> GET 方式提交数据，会带来安全问题，比如一个登录页面，通过GET方式提交数据时，用户名和密码将出现在URL上，如果页面可以被缓存或者其他人可以访问这台机器，就可以从历史记录获得该用户的账号和密码. 
> 
> HTTP请求的Get与Post
> 
> 状态码
> 
> Response 消息中的第一行叫做状态行，由HTTP协议版本号， 状态码， 状态消息 三部分组成。
> 
> 状态码用来告诉HTTP客户端，HTTP服务器是否产生了预期的Response.
> 
> HTTP/1.1中定义了 5 类状态码。
> 
> 状态码由三位数字组成，第一个数字定义了响应的类别
> 
> 1XX 提示信息 - 表示请求已被成功接收，继续处理
> 2XX 成功 - 表示请求已被成功接收，理解，接受
> 3XX 重定向 - 要完成请求必须进行更进一步的处理
> 4XX 客户端错误 - 请求有语法错误或请求无法实现
> 5XX 服务器端错误 - 服务器未能实现合法的请求
> 
> 200 OK 
> 请求被成功地完成，所请求的资源发送回客户端
> 302 Found 
> 重定向，新的URL会在response中的Location中返回，浏览器将会使用新的URL发出新的Request
> 304 Not Modified 
> 文档已经被缓存，直接从缓存调用
> 400 Bad Request 
> 客户端请求与语法错误，不能被服务器所理解 
> 403 Forbidden 
> 服务器收到请求，但是拒绝提供服务 
> 404 Not Found 
> 请求资源不存在
> 500 Internal Server Error 
> 服务器发生了不可预期的错误 
> 503 Server Unavailable 
> 服务器当前不能处理客户端的请求，一段时间后可能恢复正常
> http reauest header
> 
> http 请求头包括很多键值对，这些键值对有什么意义与作用？如何根据功能为他们分一下组呢？
> 
> cache 头域
> 
> If-Modified-Since 
> 用法：If-Modified-Since: Thu, 09 Feb 2012 09:07:57 GMT
> 
> 把浏览器端缓存页面的最后修改时间发送到服务器去，服务器会把这个时间与服务器上实际文件的最后修改时间进行对比。如果时间一致，那么返回304，客户端就直接使用本地缓存文件。如果时间不一致，就会返回200和新的文件内容。客户端接到之后，会丢弃旧文件，把新文件缓存起来，并显示在浏览器中。 
> 
> 
> If-None-Match 
> 用法：If-None-Match: "03f2b33c0bfcc1:0"
> 
> If-None-Match和ETag一起工作，工作原理是在HTTP Response中添加ETag信息。 当用户再次请求该资源时，将在HTTP Request 中加入If-None-Match信息(ETag的值)。如果服务器验证资源的ETag没有改变（该资源没有更新），将返回一个304状态告诉客户端使用本地缓存文件。否则将返回200状态和新的资源和Etag. 使用这样的机制将提高网站的性能 
> 
> 
> Pragma：Pragma: no-cache 
> Pargma只有一个用法， 例如： Pragma: no-cache
> 
> 作用： 防止页面被缓存， 在HTTP/1.1版本中，它和Cache-Control:no-cache作用一模一样 
> 
> 
> Cache-Control 
> 用法：
> 
> Cache-Control:Public 可以被任何缓存所缓存（）
> Cache-Control:Private 内容只缓存到私有缓存中
> Cache-Control:no-cache 所有内容都不会被缓存
> 作用：用来指定Response-Request遵循的缓存机制
> 
> Client 头域
> 
> Accept 
> 用法：Accept: */*，Accept: text/html
> 
> 作用： 浏览器端可以接受的媒体类型； 
> Accept: */* 代表浏览器可以处理所有回发的类型，(一般浏览器发给服务器都是发这个） 
> Accept: text/html 代表浏览器可以接受服务器回发的类型为 text/html ；如果服务器无法返回text/html类型的数据，服务器应该返回一个406错误(non acceptable) 
> 
> 
> Accept-Encoding 
> 用法：Accept-Encoding: gzip, deflate
> 
> 作用： 浏览器申明自己接收的文件编码方法，通常指定压缩方法，是否支持压缩，支持什么压缩方法（gzip，deflate），（注意：这不是指字符编码） 
> 
> 
> Accept-Language 
> 用法：Accept-Language: en-us
> 
> 作用： 浏览器申明自己接收的语言。 
> 
> 语言跟字符集的区别：中文是语言，中文有多种字符集，比如big5，gb2312，gbk等等； 
> 
> 
> User-Agent 
> 用法： User-Agent: Mozilla/4.0......
> 
> 作用：告诉HTTP服务器， 客户端使用的操作系统和浏览器的名称和版本. 
> 
> 
> Accept-Charset 
> 用法：Accept-Charset：utf-8
> 
> 作用：浏览器申明自己接收的字符集，这就是本文前面介绍的各种字符集和字符编码，如gb2312，utf-8（通常我们说Charset包括了相应的字符编码方案） 
> 
> 
> Cookie/Login 头域
> 
> Cookie
> 
> Cookie: bdshare_firstime=1439081296143; ASP.NET_SessionId=rcqayd4ufldcke0wkbm1vhxb; pgv_pvi=7361416192; pgv_si=s6686106624; ce.sysu.edu.cn80.ASPXAUTH=9E099592DD5A414BEECD8CF43CFC71664
> 作用： 最重要的header, 将cookie的值发送给HTTP 服务器
> 
> Entity 头域
> 
> Content-Length 
> 用法：Content-Length: 38
> 
> 作用：发送给HTTP服务器数据的长度。 
> 
> 
> Content-Type 
> 用法：Content-Type: application/x-www-form-urlencoded
> 
> 不常出现，一般出现在response头部，用于指定数据文件类型 
> 
> 
> Miscellaneous 头域
> 
> Referer 
> 用法：Referer: http://ce.sysu.edu.cn/hope/
> 
> 作用：提供了Request的上下文信息的服务器，告诉服务器我是从哪个链接过来的，比如从我主页上链接到一个朋友那里，他的服务器就能够从HTTP Referer中统计出每天有多少用户点击我主页上的链接访问他的网站。 
> 
> 
> Transport 头域
> 
> Connection 
> Connection: keep-alive： 当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接
> 
> Connection: close： 代表一个Request完成后，客户端和服务器之间用于传输HTTP数据的TCP连接会关闭， 当客户端再次发送Request，需要重新建立TCP连接 
> 
> 
> Host 
> 用法：Host: ce.sysu.edu.cn
> 
> 作用: 请求报头域主要用于指定被请求资源的Internet主机和端口号（默认80），它通常从HTTP URL中提取出来的
> 
> HTTP Response header
> 
> Cache 头域
> 
> Date 
> 用法：Date: Sat, 11 Feb 2012 11:35:14 GMT
> 
> 作用: 生成消息的具体时间和日期 
> 
> 
> Expires 
> 用法：Expires: Tue, 08 Feb 2022 11:35:14 GMT 
> 作用: 浏览器会在指定过期时间内使用本地缓存 
> 
> Vary 
> 用法：Vary: Accept-Encoding 
> 
> Cookie/Login 头域
> 
> P3P 
> 用法： 
> P3P: CP=CURa ADMa DEVa PSAo PSDo OUR BUS UNI PUR INT DEM STA PRE COM NAV OTC NOI DSP COR
> 
> 作用: 用于跨域设置Cookie, 这样可以解决iframe跨域访问cookie的问题 
> 
> 
> Set-Cookie 
> 用法： 
> Set-Cookie: sc=4c31523a; path=/; domain=.acookie.taobao.com
> 
> 作用：非常重要的header, 用于把cookie 发送到客户端浏览器， 每一个写入cookie都会生成一个Set-Cookie.
> 
> Entity 头域
> 
> ETag 
> 用法：ETag: "03f2b33c0bfcc1:0"
> 
> 作用: 和request header的If-None-Match 配合使用 
> 
> 
> Last-Modified 
> 用法：Last-Modified: Wed, 21 Dec 2011 09:09:10 GMT
> 
> 作用：用于指示资源的最后修改日期和时间。（实例请看上节的If-Modified-Since的实例） 
> 
> 
> Content-Type 
> 用法：
> 
> Content-Type: text/html; charset=utf-8
> Content-Type:text/html;charset=GB2312
> Content-Type: image/jpeg
> 作用：WEB服务器告诉浏览器自己响应的对象的类型和字符集 
> 
> 
> Content-Encoding 
> 用法：Content-Encoding：gzip
> 
> 作用：WEB服务器表明自己使用了什么压缩方法（gzip，deflate）压缩响应中的对象。 
> 
> 
> Content-Language 
> 用法： Content-Language:da
> 
> WEB服务器告诉浏览器自己响应的对象的语言
> 
> Miscellaneous 头域
> 
> Server 
> 用法：Server: Microsoft-IIS/7.5
> 
> 作用：指明HTTP服务器的软件信息 
> 
> 
> X-AspNet-Version 
> 用法：X-AspNet-Version: 4.0.30319
> 
> 作用：如果网站是用ASP.NET开发的，这个header用来表示ASP.NET的版本 
> 
> X-Powered-By 
> 用法：X-Powered-By: ASP.NET
> 
> 作用：表示网站是用什么技术开发的 
> 
> Transport头域
> 
> Connection 
> 用法与作用： 
> Connection: keep-alive：当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接
> Connection: close：代表一个Request完成后，客户端和服务器之间用于传输HTTP数据的TCP连接会关闭， 当客户端再次发送Request，需要重新建立TCP连接
> Location头域
> 
> Location 
> 用法：Location：http://ce.sysu.edu.cn/hope/ 
> 作用： 用于重定向一个新的位置， 包含新的URL地址

## LVS  fullnat的原理
> [**官网文档**](http://www.linuxvirtualserver.org/VS-NAT.html)
> virtual ip经过DS转换成real server的ip，然后返回的时候realserver的ip再经过DS转换成vip

## DNS的迭代和递归的差别
> [**DNS.LA站长小百科**](https://www.dns.la/support/art_276.aspx)
> 
> 我们在阅读DNS相关资料的时候经常看到递归查询和迭代查询，在DNS中的这两种查询都是DNS查询方法。
> 递归查询
> 递归查询是最常见的查询方式，当一个客户机发送一个查询给本地域名服务器时，本地域名服务器必须返回一个IP地址，如果解析不到IP，域名服务器将代替提出请求的客户机（下级DNS服务器）进行域名查询，若域名服务器不能直接回答，则域名服务器会在域各树中的各分支的上下进行递归查询，最终将返回查询结果给客户机，在域名服务器查询期间，客户机将完全处于等待状态。具体的查询过程为：首先，客户端提出域名解析请求(无论以何种形式或方法)，并将该请求发或转发给本地的DNS服务器。 接着，本地DNS服务器收到请求后就去查询自己的缓存，如果有该条记录，则会将查询的结果返回给客户端。(也就是我们看到的“ “非权威性”的应答”)。如果DNS服务器本地没有搜索到相应的记录，则会把请求转发到根DNS(13台根DNS服务器的IP信息默认均存储在DNS服务器中，当需要时就会去有选择性的连接)。 然后，根DNS服务器收到请求后会判断这个域名是谁来授权管理，并会返回一个负责该域名子域的DNS服务器地址。比如，查询abc.com的IP，根DNS服务器就会在负责.com顶级域名的DNS服务器中选一个(并非随机，而是根据空间、地址、管辖区域等条件进行筛选)，返回给本地DNS服务器。可以说根域对顶级域名有绝对管理权，自然也知道他们的全部信息，因为在DNS系统中，上一级对下一级有管理权限，毫无疑问，根DNS是最高一级了。就这样一直轮回。
> 迭代查询又称重指引。当服务器使用迭代查询时能够使其他服务器返回一个最佳的查询点提示或主机地址，若此最佳的查询点中包含需要查询的主机地址，则返回主机 地址信息，若此时服务器不能够直接查询到主机地址，则是按照提示的指引依次查询，直到服务器给出的提示中包含所需要查询的主机地址为止，对一个迭代查询的响应就像一个DNS服务器说：“我不知道你找的IP地址是多少，但是我知道位于10.1.2.3的域名服务器可以告诉你。”一般的，每次指引都会更靠近根服务器（向上），查寻到根域名服务器后，则会再次根据提示向下查找。从上节的图中可以知道，B访问C、D、E、F、G，都是迭代查询，首先B 访问C，得到了提示访问D的提示信息后，开始访问D，这时因为是迭代查询，D又返回给B提示信息，告诉B应该访问E，依次类推。

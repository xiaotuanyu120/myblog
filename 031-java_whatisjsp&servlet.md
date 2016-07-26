---
title: JSP和Servlet(概念)
date: 2016-07-26 22:56:00
categories: java
tags: [java,jsp,servlet]

---
## 什么是JSP？
JSP全称： java server pages
jsp是由sun microsystems公司倡导的、多家公司参与建立的一种动态网页技术标准，是在传统的html文件中加入java程序片段(scriptlet)和jsp标记(tag)，从而形成的jsp文件(.jsp)。
和传统的html请求过程的区别是，客户的请求(XX.jsp)，服务器会将jsp文件加载转换成名为java文件(XXserverlet.java)，然后编译成class文件(XXserverlet.class)。
部署和执行jsp，需要servlet container，例如tomcat，jetty，wildfy等

<!--more-->

## 什么是servlet？
Servlet是一种独立于平台和协议的服务器端的Java应用程序，可以生成动态的Web页面。 它担当Web浏览器或其他HTTP客户程序发出请求，与HTTP服务器上的数据库或应用程序之间的中间层。

Servlet是位于Web 服务器内部的服务器端的Java应用程序，与传统的从命令行启动的Java应用程序不同，Servlet由Web服务器进行加载，该Web服务器必须包含支持Servlet的Java虚拟机。

在通信量大的服务器上，Java servlet的优点在于它们的执行速度更快于CGI程序。各个用户请求被激活成单个程序中的一个线程，而创建单独的程序，这意味着各个请求的系统开销比较小。

简单地说，servlet就是在服务器端被执行的java程序，它可以处理用户的请求，并对这些请求做出响应。Servlet编程是纯粹的java编程，而jsp则是html和java编程的中庸形式，它更有助于美工人员来设计界面。正是如此，所有的jsp文件都将被最终转换成java servlet来执行。

从jsp到java到class，jsp在首次被请求时是要花费一定的服务器资源的。但庆幸的是，这种情况只发生一次，一旦这个jsp文件被翻译并编译成对应的servlet，在下次请求来临时，将直接由servlet来处理，除非这个jsp已经被修改。
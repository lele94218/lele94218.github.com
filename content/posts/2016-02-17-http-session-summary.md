---
title: "深入理解 HTTP Session"
date: 2016-09-28
slug: "2016-02-17-http-session-summary"
tags:
  - "Java"
  - "Java Web"
categories:
  - "Java Web"
---

> 转载于 [深入理解 HTTP Session](http://lavasoft.blog.51cto.com/62575/275589)

Session 在 web 开发中是一个非常重要的概念, 这个概念很抽象,很难定义,也是最让人迷惑的一个名词, 也是最多被滥用的名字之一, 在不同的场合,session一次的含义也很不相同. 这里只探讨 HTTP Session.

为了说明问题,这里基于 Java Servlet 理解 Session 的概念与原理,这里所说 Servlet 已经涵盖了 JSP 技术,因为 JSP 最终也会被编译为 Servlet,两者有着相同的本质.

在 Java 中, HTTP 的 Session 对象用 `javax.servlet.http.HttpSession` 来表示.

1. 概念: Session 代表服务器与浏览器的一次会话过程, 这个过程是连续的, 也可以时断时续的. 在 Servlet 中, session 指的是 HttpSession 类的对象, 这个概念到此结束了, 也许会很模糊, 但只有看完本文,才能真正有个深刻理解.
2. Session 创建的时间是:

   一个常见的误解是以为 session 在有客户端访问时就被创建, 然而事实是直到某 server 端程序调用 `HttpServletRequest.getSession(true)` 这样的语句时才被创建,注意如果JSP没有显示的使用 `<% @page session="false"%>` 关闭 session, **则 JSP 文件在编译成 Servlet 时将会自动加上这样一条语句 `HttpSession session = HttpServletRequest.getSession(true)`; 这也是JSP中隐含的 session对象的来历.**

   由于 session 会消耗内存资源, 因此, 如果不打算使用 session, 应该在所有的 JSP 中关闭它.

   引申:

   * 访问 `*.html` 的静态资源因为不会被编译为 Servlet, 也就不涉及 session 的问题.
   * 当 JSP 页面没有显式禁止 session 的时候, 在打开浏览器第一次请求该 jsp 的时候, 服务器会自动为其创建一个 session, 并赋予其一个 sessionID, 发送给客户端的浏览器. 以后客户端接着请求本应用中其他资源的时候, 会自动在请求头上添加:  
     `Cookie:JSESSIONID` = 客户端第一次拿到的 session ID  
     这样, 服务器端在接到请求时候, 就会收到 session ID, 并根据 ID 在内存中找到之前创建的 session 对象, 提供给请求使用. 这也是 session 使用的基本原理 —- 搞不懂这个, 就永远不明白session的原理.
3. Session删除的时间是:

   * Session 超时: 超时指的是连续一定时间服务器没有收到该 Session 所对应客户端的请求,并且这个时间超过了服务器设置的 Session 超时的最大时间.
   * 程序调用 `HttpSession.invalidate()`
   * 服务器关闭或服务停止
4. session 存放在哪里: 服务器端的内存中. 不过 session 可以通过特殊的方式做持久化管理.
5. session 的 id 是从哪里来的, sessionID 是如何使用的: 当客户端第一次请求 session 对象时候, 服务器会为客户端创建一个 session, 并将通过特殊算法算出一个 session 的 ID, 用来标识该session对象, 当浏览器下次 (session 继续有效时) 请求别的资源的时候, 浏览器会偷偷地将sessionID放置到请求头中, 服务器接收到请求后就得到该请求的 sessionID, 服务器找到该 id 的 session 返还给请求者 (Servlet) 使用. 一个会话只能有一个 session 对象, **对 session 来说是只认 id 不认人**.
6. session 会因为浏览器的关闭而删除吗？

   不会, session 只会通过上面提到的方式去关闭.
7. 同一客户端机器多次请求同一个资源, session 一样吗？

   一般来说,每次请求都会新创建一个 session.

   其实, 这个也不一定的, 总结下: **对于多标签的浏览器 (比如 Chrome 浏览器) 来说, 在一个浏览器窗口中, 多个标签同时访问一个页面, session是一个. 对于多个浏览器窗口之间, 同时或者相隔很短时间访问一个页面, session是多个的, 和浏览器的进程有关. 对于一个同一个浏览器窗口, 直接录入url访问同一应用的不同资源, session是一样的.**
8. session 是一个容器,可以存放会话过程中的任何对象.
9. session 因为请求 (request对象) 而产生, 同一个会话中多个 request 共享了一个 session 对象, 可以直接从请求中获取到 session 对象.
10. **其实, session 的创建和使用总在服务端, 而浏览器从来都没得到过 session 对象.** 但浏览器可以请求 Servlet (jsp 也是 Servlet) 来获取 session 的信息. 客户端浏览器真正紧紧拿到的是 session ID, 而这个对于浏览器操作的人来说, 是不可见的, 并且用户也无需关心自己处于哪个会话过程中.

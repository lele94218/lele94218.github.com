---
title: "Java Web 系统总结 (一)"
date: 2016-09-28
slug: "2016-02-16-java-web-summary-1"
tags:
  - "Java"
  - "Java Web"
categories:
  - "Java Web"
---

### 一. JSP 总结

#### 9个隐含对象及其常用方法 (虽然为 Java 代码, 但是使用 _EL_ 表达式完全可以进行调用相关 get 方法):

**1. *Request (HttpServletRequest)***

- **Attribute** 相关:

可以理解在 `request` 作用域范围内的一个存储空间, 实现方法是将键值对放入一个 `HashMap`.

```java
Object getAttribute(String name);
void setAttribute(String name, Object o);
void removeAttribute(String name);
```

- **Paramater** 相关:

一般用于与 `html`, `jsp`页面 及 `url` 地址传递参数, 仅能传递字符串. 注意: 没有 `setParamater` 方法.

```java
String getParameter(String name);
String[] getParameterValues(String name);
```

- **Cookie** 相关:

作为浏览器端持久化储存的一种实现方法, 由服务器加入 `response` 中, 并在 `request` 中取出.

```java
Cookie[] getCookies();
```

- **Session** 相关:

作为服务器持久化储存的一种实现方法, 在浏览器第一次访问服务器资源时自动创建.

```java
HttpSession getSession();
```

- **URI** 路径:

**应用名称 + 页面路径**

比如: `/day_15/bbs/bbs.jsp` .

```java
String getRequestURI();
```

- **URL** 路径:

**站点地址 + URI路径**

比如: `http://localhost:8080/day_15/bbs/bbs.jsp`

```java
StringBuffer getRequestURL();
```

- **ServletPath** 路径:

**页面路径**

一般用于 `filter` 进行页面筛选. 比如: `/bbs/bbs.jsp`

```java
String getServletPath();
```

- 转发地址:

可以将 `request` 转发到另一页面或 `Servlet`. 注意: `path` 为 `Servlet context path`.

```java
RequestDispatcher getRequestDispatcher(String path);
```

- 编码问题:

对于携带中文作为 `Paramater` 的 `request` 在打印时会出现乱码, 所以需要进行编码.

```java
void setCharacterEncoding(String env);
```

**2. *Response (HttpServletResponse)***

- **Cookie** 相关:

作为浏览器端持久化储存的一种实现方法, 由服务器加入 `response` 中, 并在 `request` 中取出.

```java
void addCookie(Cookie cookie)
```

- 重定向地址

可以跳转到另一页面, 但不转发 `request`. `location` 必须包含 `application context path` 或为 `URL`.

```java
void sendRedirect(String location);
```

**3. *Session (HttpSession)***

- **Attribute** 相关:

和 `request` 类似, `session` 也可以携带键值对, 使用方法完全类似.

```java
Object getAttribute(String name);
void setAttribute(String name, Object value);
void removeAttribute(String name);
```

- 设置生命周期:

从创建到销毁的时间.

```java
void setMaxInactiveInterval(int interval);
int getMaxInactiveInterval();
```

立即销毁 `session`:

```java
void invalidate();
```

- **JSessionId**:

以 `JSessionId` 为名称存入 `Cookie` 中.

获取 `JSessionId`:

```java
String getId();
```

- 时间信息:

获取创建与最近访问时间:

```java
long getCreationTime();
long getLastAccessedTime();
```

- 获取 **ServletContext**

```java
ServletContext getServletContext();
```

以上总结的三个隐含对象 `request` `response` `session`, 不只是对于 `JSP` , 而在整个 `Java Web` 技术中都极为重要, 对其常用的实现方法必须使用熟练.

**4. *Application (ServletContext)***

- **Attribute** 相关:

和 `request` `session` 类似, `Servlet` 也可以携带键值对, 使用方法完全类似.

```java
Object getAttribute(String name);
void setAttribute(String name, Object value);
void removeAttribute(String name);
```

- **Paramater** 相关:

获取在 `web.xml` 中设置的 `Servlet` 初始化参数.

```java
String getInitParameter(String name);
```

- **Servlet Context Path**

为服务器应用的路径名, 如: `/day_15`.

```java
String getContextPath();
```

仅获取应用名称,  如: `day_15`.

```java
String getServletContextName();
```

**5. *Config (ServletConfig)***

`ServletContext` 可以覆盖其实现的所有功能.

- **Paramater** 相关:

获取在 `web.xml` 中设置的 `Servlet` 初始化参数.

```java
String getInitParameter(String name);
```

- 获取 **ServletContext**:

```java
ServletContext getServletContext();
```

- 获取应用名称:

如: `day_15`.

```java
String getServletContextName();
```

**6. *PageContext (PageContext)***

- 可以获取其他所有隐藏对象 **Request Response Session ServletContext ServletConfig Out Page Exception**:

直接调用 `get` 方法既可. 因为可以调用所有功能, 多用于 `EL` 表达式.

**7. 其余不常用的三个: Out Page Exception**:

以上9个隐藏对象包含 `Java Web` 最重要的四个作用域:

- **Application**: 作用于整个 Web 应用, 生命周期为 **应用的添加到移除**.
- **Session**: 作用于一次会话, 生命周期为 **浏览器第一次访问生成到生命周期结束自动注销**.
- **Request**: 作用于一次请求, 生命周期为 **发送请求到请求处理结束**.
- **Page**: 仅作用于本页面, 无生命周期.

# Web 容器是什么

> HTTP 服务器 + Servlet 容器

# HTTP 的本质

> 浏览器与服务器之间的数据传送协议（应用层协议），规定了客户端和服务器之间的通信格式；HTTP 是基于 TCP/IP 协议来传递数据的（HTML 文件、图片、查询结果等），HTTP 协议不涉及数据包（Packet）传输

## 从远程 HTTP 服务器获取一个 HTML 文本

1. 用户请求
2. 浏览器发起TCP连接请求
3. 服务器接受请求并建立连接
4. 生成HTTP格式数据包
5. 发送请求数据包
6. 解析HTTP格式数据包
7. 执行请求
8. 生成HTTP格式数据包
9. 发送响应数据包
10. 解析HTTP格式数据包
11. 浏览器呈现响应给用户

# HTTP 请求数据

1. 请求行
2. 请求报头
3. 请求正文

# HTTP 响应数据

1. 状态行
2. 响应报头
3. 报文主体

# Servlet 完成什么任务

> HTTP 服务器不直接跟业务类打交道，而是把请求交给 Servlet 容器去处理，Servlet 容器会将请求转发到具体的 Servlet，如果这个 Servlet 还没创建，就加载并实例化这个 Servlet，然后调用这个 Servlet 的接口方法

PS：Tomcat 和 Jetty 都按照Servlet 规范的要求实现了 Servlet 容器，同时它们也具有 HTTP 服务器的功能

Servlet 有五个核心方法

1. init

   初始化一些资源，并在destroy 方法里释放这些资源

2. getServletConfig

   封装 Servlet的初始化参数

3. service

4. getServletInfo

   ServletRequest 和 ServletResponse。ServletRequest 用来封装请求信息，ServletResponse 用来封装响应信息，因此本质上这两个类是对通信协议的封装

5. destroy

PS：实现自己的Servlet

通过继承 HttpServlet 重写两个方法：doGet 和 doPost

> 当客户请求某个资源时，HTTP 服务器会用一个 ServletRequest 对象把客户的请求信息封装起来，然后调用 Servlet 容器的 service 方法，Servlet 容器拿到请求后，根据请求的 URL 和 Servlet 的映射关系，找到相应的 Servlet，如果Servlet 还没有被加载，就用反射机制创建这个 Servlet，并调用 Servlet 的 init方法来完成初始化，接着调用 Servlet 的 service 方法来处理请求，把ServletResponse 对象返回给 HTTP 服务器，HTTP 服务器会把响应发送给客户端

PS：引入了 Servlet 规范后，你不需要关心 Socket 网络通信、不需要关心 HTTP 协议，也不需要关心你的业务类是如何被实例化和调用的，因为这些都被 Servlet规范标准化了，你只要关心怎么实现的你的业务逻辑，Servlet 规范提供了两种扩展机制：Filter 和 Listener

1. Filter

   允许你对请求和响应做一些统一的定制化处理，比如你可以根据请求的频率来限制访问，或者根据国家地区的不同来修改响应内容

   Web 应用部署完成后，Servlet 容器需要实例化
   Filter 并把 Filter 链接成一个 FilterChain。当请求进来时，获取第一个 Filter 并调用 doFilter 方法，doFilter 方法负责调用这个 FilterChain 中的下一个Filter

2. Listener

   提供了一些默认的监听器来监听Servlet 内部的一些事件

# Tomcat 日志作用

1. catalina.***.log   主要是记录 Tomcat 启动过程的信息，在这个文件可以看到启动的 JVM 参数以及操作系统等日志信息
2. catalina.out是 Tomcat 的标准输出（stdout）和标准错误
3. localhost.**.log主要记录 Web 应用在初始化过程中遇到的未处理的异常
4. localhost_access_log.**.txt存放访问 Tomcat 的请求日志，包括 IP地址以及请求的路径、时间、请求协议以及状态码等信息
5. manager.***.log/host-manager.***.log存放 Tomcat 自带的Manager 项目的日志信息
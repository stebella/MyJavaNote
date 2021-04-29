Servlet是javaweb的三大组件之一(Servlet、Filter、Listener)

Servlet:接口

HttpServlet：使用 doGet() doPost()

1. 编写servlet类

2. 在web.xml中进行配置  

   ```xml
   <servlet>
   	<servlet-name>MyFisrtServlet</servlet-name>
       <servlet-class>com.wxy.servlet.MyFisrtServlet</servlet-class>
   </servlet>
   <servlet-mapping>
       <servlet-name>MyFirstServlet</servlet-name>
       <url-pattern>/hello</url-pattern>
   </servlet-mapping>
   ```

重要的几个知识点：

> 1.一个Servlet对应一个ServletConfig封装当前Servlet的配置信息；
>
> 2.ServletContext:四大域对象之一(PageContext、HttpServletRequest、HttpSession、ServletContext)；
>
> 域对象作用：在域中保存一些数据，跨页面获取到保存的数据；多个资源之间共享数据
>
> ServletContext作用：
>
> - 1）域对象
>
> - 2）ServleContext获取当前项目信息；一个项目对应一个ServletContext
>
>   ​	context.getRealPath("/pics/a.jpg");只能获取当前项目下某个资源的路径
>
> 3.HttpServletRequest：代表请求；
>
> ​	每一次发送请求，请求的详细信息会被tomcat自动封装成一个reuqest对象，我们以后要获取当前请求的一些信息，我们使用request对象即可
>
> 作用：1）获取请求参数 request.getParameter("hello")
>
> ​			2）作为域对象保存数据：同一个请求期间可以共享数据
>
> ​			3）获取到HttpSession对象：request.getSession()
>
> ​			4）转发：request.getRequestDispatcher("/index.jsp").forward(request,response);
>
> ​					//将当次请求和响应交给另外一个程序处理，在服务器内部进行的
>
>  服务器上保存了所有的页面和图片、视频、动态程序等等
>
> 4.HttpServletResponse:响应
>
> 作用：交给浏览器的数据(响应首行、响应头(对浏览器的一些命令)和响应体(浏览器收到要解析的数据))；
>
> **JSP最终也会被转化为一个Java类，JSP本质就是一个Servlet**
>
>  请求index.jsp页面 (在服务器中是一个servlet)index.jsp---->index_jsp.class---->response.getWriter().write("");把页面write进去
>
> 回归到底层，服务器给浏览器发送数据，都是写出字符流或者字节流
>
> 重定向：浏览器收到重定向命令要发送新请求
>
> ​				response.sendRedirect("重定向的地址")
>
> ​				response.sendRedirect(reuqest.getContextPath() + "/index.jsp")
>
> ​               reuqest.getContextPath();//获取项目名，以/开始不以/结束

**request:** 每一次请求都是一个新的request对象,如果在web组件之间需要共享同一个请求中的数据,只能使用请求转发.
**session:** 每一次会话都是一个新的session对象,如果如果需要在一次会话中的多个请求之间需要共享数据,只能使用session.
**application:** 应用对象,Tomcat启动到关闭,表示一个应用,在一个应用中有且只有一个application对象,作用于整个Web应用,可以实现多次会话之间的数据共享.

![在这里插入图片描述](E:\笔记\Java笔记\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDk3NjgzNQ==,size_16,color_FFFFFF,t_70)


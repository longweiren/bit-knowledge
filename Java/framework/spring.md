### 读源码理解springframework

----------


##### 理解DispatchServlet

**web服务器启动时**，实例化DispatchServlet，配置文件作为参数传入。  

DispatchServlet实例化时，调用父类FrameworkServlet.initServletBean()。

* 调用initWebApplicationContext(), 读取配置文件，实例化bean

		获取WebApplicationContext实例  
	 	从context中获取 multipartResolver bean实例
	 	从context中获取 localeResolver bean实例
	 	从context中获取 themeResolver bean实例
	 	从context中获取 handlerMappings bean实例
	 	从context中获取 handlerAdapters bean实例
	 	从context中获取 handlerExceptionResolver bean实例
	 	从context中获取 requestToViewNameTranslator bean实例
	 	从context中获取 viewResolver bean实例
		

* 调用initFrameworkServlet()


**前端发送请求时**：

调用DispatchServlet.doDispatch方法

	首先判断是否 multipart request
	根据 request 得到 mappedHandler
	执行前置 handlerInterceptor
	处理请求 handler.handle()
	执行后置 handlerInterceptor
	render(modelAndView, request, response)
Spring MVC 的原理和流程如下：

- 请求先到DispatcherServlet
= DispatcherServlet通过处理器映射器(HandlerMapping)找到具体的Controller
- DispatcherServlet将请求交给处理器适配器(HandlerAdapter)。
- HandlerAdapter调用Handler执行，并且返回执行结果ModelAndView给DispatcherServlet。
- DispatcherServlet将ModelAndView传递给视图解析器(ViewResolver)，得到视图对象。
- DispatcherServlet对视图进行渲染，即将模型数据填充至视图中。
- DispatcherServlet响应用户。

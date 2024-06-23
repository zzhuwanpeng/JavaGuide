Spring MVC 的原理和流程如下：

- 请求先到DispatcherServlet
= DispatcherServlet通过处理器映射器(HandlerMapping)找到具体的Controller
- DispatcherServlet将请求交给处理器适配器(HandlerAdapter)。
- HandlerAdapter调用Handler执行，并且返回执行结果ModelAndView给DispatcherServlet。
- DispatcherServlet将ModelAndView传递给视图解析器(ViewResolver)，得到视图对象。
- DispatcherServlet对视图进行渲染，即将模型数据填充至视图中。
- DispatcherServlet响应用户。

@ResponseBody 注解用于将控制器方法的返回值直接写入 HTTP 响应体，而不是解析为视图。
这对于 RESTful Web 服务的开发非常有用，因为它允许你直接返回 JSON、XML 或其他格式的数据，而无需通过视图解析器。

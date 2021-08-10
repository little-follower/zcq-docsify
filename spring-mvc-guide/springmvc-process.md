## 初探Spring MVC请求处理流程

![](https://gitee.com/zt888/zcq-pic-manage/raw/master/springmvc/springmvc-process-1.png)

Spring MVC相对于前面讲的只是一个概述，现在首先通过上面的一张流程图来了解Spring MVC的核心组件和大致处理流程，如上图图所示。

①**DispatcherServlet**是SpringMVC中的前端控制器（**Front Controller**），负责接收Request并将Request转发给对应的处理组件。 

②**HanlerMapping**是SpringMVC中完成URL到Controller映射的组件。DispatcherServlet接收Request，然后从HandlerMapping查找处理 Request的Handler，即Controller。

④**HandlerAdapter**是处理器适配器，负责调用处理器（Handler|Controller）的方法。

⑤Controller|Handler处理Request，并返回ModelAndView对象。Controller是 SpringMVC中负责处理Request的组件（类似于Struts 2中的Action）， ModelAndView是封装结果视图的组件。 

⑧、⑨、⑩ 是视图解析器解析**ModelAndView**对象并返回对应的视图给客户端的过程。 

容器初始化时会建立所有URL和Controller中方法的对应关系，保存到HandlerMapping中，用户请求时根据请求的URL快速定位到 Controller中的某个方法。在Spring中先将URL和Controller的对应关系保存到Map<url,Controller>中。

Web容器启动时会通知Spring初始化容器（加载Bean的定义信息和初始化所有单例Bean），然后Spring MVC会遍历容器中的Bean，获取每一个Controller中的所有方法访问的 URL，将URL和Controller保存到一个Map中。这样就可以根据请求快速定位到Controller，因为最终处理请求的是Controller中的方法，Map中只 保留了URL和Controller的对应关系，所以要根据请求的URL进一步确认 Controller中的方法。

其原理就是拼接Controller的URL （Controller 上 @RequestMapping 的值）和方法的 URL （Method 上**@RequestMapping** 的值），与请求的URL进行匹配，找到匹配的方法。确定处理请求的方 法后，接下来的任务就是参数绑定，把请求中的参数绑定到方法的形式 参数上，这是整个请求处理过程中最复杂的一步。
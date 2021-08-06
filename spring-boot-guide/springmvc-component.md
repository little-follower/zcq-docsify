# SpringMVC九大组件

## HandlerMapping

HandlerMapping 是用来查找 Handler 的，也就是**处理器**，具体的表现形式可以是类，也可以是方法。比如，标注了@RequestMapping的每个方法都可以看成一个Handler。Handler负责实际的请求处理，在请求到达后，HandlerMapping 的作用便是**找到请求相应的处理器 Handler 和 Interceptor（拦截器）**。

## HandlerAdapter

从组件的名字上看，HandlerAdapter是一个**适配器**。因为Spring MVC中 Handler可以是任意形式的，只要能够处理请求便可。但是把请求交给 Servlet 的时候，由于 Servlet 的方法结构都是 doService(HttpServletRequest req，HttpServletResponse resp)形式的， 要让固定的**Servlet处理方法调用Handler来进行处理**，这一步工作便是 HandlerAdapter要做的事。

## HandlerExceptionResolver

从组件的名字上看，HandlerExceptionResolver是用来处理 **Handler 产生的****异常情况的组件**。具体来说，此组件的作用是根据异常设置 ModelAndView，之后交给渲染方法进行渲染，渲染方法会将 ModelAndView渲染成页面。不过要注意，HandlerExceptionResolver只
用于解析对请求做处理阶段产生的异常，渲染阶段的异常不归它管，这也是Spring MVC 组件设计的一大原则—分工明确、互不干涉。

## ViewResolver

ViewResolver即**视图解析器**，相信大家对这个组件应该很熟悉了。 通常在Spring MVC的配置文件中，都会配上一个实现类来进行视图解 析。这个组件的主要作用是将 String 类型的视图名和Locale 解析为 View 类型的视图，只有一个 resolveViewName()方法。从方法的定义可以看出，Controller层返回的String类型的视图名viewName最终会在这里被解析成为View。View是用来渲染页面的，也就是说，它会将程序返回的参数和数据填入模板中，生成HTML文件。ViewResolver在这个过程中主要做两件大事：ViewResolver 会找到渲染所用的模板（第一件大事）和所用的技术（第二件大事，其实也就是找到视图的类型，如 JSP）并填入参数。默认情况下，Spring MVC会为我们自动配置一个 InternalResourceViewResolver，是针对JSP类型视图的。

## RequestToViewNameTranslator

RequestToViewNameTranslator组件的作用是从请求中获取 ViewName。因为ViewResolver根据ViewName查找View，但有的 Handler处理完成之后，没有设置View，也没有设置ViewName，便要通过这个组件来从请求中查找ViewName。

## LocaleResolver

ViewResolver组件的resolveViewName()方法需要两个参数，一个 是视图名，另一个就是Locale。参数Locale是从哪来的呢？这就是 LocaleResolver组件要做的事。LocaleResolver用于从请求中解析出 Locale，比如在中国Locale当然就是zh-CN，用来表示一个区域。这个组件也是i18n的基础。

## ThemeResolver

从名字便可看出，ThemeResolver组件是用来解析主题的。主题就是样式、图片及它们所形成的显示效果的集合。Spring MVC中一套主题对应一个properties文件，里面存放着与当前主题相关的所有资源，如图片、CSS 样式等。创建主题非常简单，只需准备好资源，然后新建一 个“主题名.properties”并将资源设置进去，放在classpath下，之后便可以 在页面中使用了。Spring MVC中与主题有关的类有ThemeResolver、 ThemeSource和Theme。ThemeResolver负责从请求中解析出主题名， ThemeSource 则根据主题名找到具体的主题，其抽象也就是 Theme，可 以通过 Theme来获取主题和具体的资源。

## MultipartResolver

MultipartResolver 是一个大家很熟悉的组件，用于处理上传请求， 通过将普通的请求包装成**MultipartHttpServletRequest**来实现。 MultipartHttpServletRequest可以通过**getFile()**方法直接获得文件。如 果上传多个文件，还可以调用 **getFileMap()**方法得到 Map＜ FileName，File＞ 这样的结构。MultipartResolver的作用就是封装普通的请求，使其拥有文件上传的功能

## FlashMapManager

<font color=#00ffff>说到FlashMapManager组件</font>，得先说一下**FlashMap**。 FlashMap用于重定向时的参数传递， FlashMapManager就是用来管理FlashMap的。

```
FlashMap eg:	
比如在处理用户订单时，为了避免重复提交，可以处理完post请求后重定向到一个get请求，
这个get请求可以用来显示订单详情之类的信息。
这样做虽然可以规避用户重新提交订单的问题，
但是在这个页面上要显示订单的信息，这些数据从哪里获取呢？
因为重定向是没有传递参数这一功能的，
如果不想把参数写进 URL（其实也不推荐这么做，除了URL有长度限制，把参数都直接暴露也不安全），
那么就可以通过FlashMap来传递。
只需要在重定向之前将要传递的数据写入请求
（可以通过 ServletRequestAttributes.getRequest()方法获得）的属性 OUTPUT_FLASH_MAP_ATTRIBUTE中，
这样在重定向之后的Handler 中Spring就会自动将其设置到Model中，
在显示订单信息的页面上就可以直接从Model中获得数据。
```

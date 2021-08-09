# SpringMVC源码分析

根据上面分析的Spring MVC的九大组件，Spring MVC的处理过程可分为如下三步： 

```Text
  - ApplicationContext初始化时用Map保存所有URL和Controller类的对应关系。 
  - 根据请求URL找到对应的Controller，并从Controller中找到处理请求的方法。 
  - 将Request参数绑定到方法的形参上，执行方法处理请求，并返回结果视图。
```

##   初始化阶段

首先找到DispatcherServlet类，寻找init（）方法。我们发现init（） 方法其实在父类HttpServletBean中，其源码如下：

```java
public final void init() throws ServletException {

   PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
   if (!pvs.isEmpty()) {
      try {
         //定位资源
         BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
         //加载配置信息
         ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
         bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
         initBeanWrapper(bw);
         bw.setPropertyValues(pvs, true);
      }
      catch (BeansException ex) {
         if (logger.isErrorEnabled()) {
            logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
         }
         throw ex;
      }
   }
    //☆☆
   initServletBean();
}
```

在这段代码中调用了一个重要的方法：initServletBean()。 initServletBean()方法的源码如下：

```java
protected final void initServletBean() throws ServletException {
   getServletContext().log("Initializing Spring " + getClass().getSimpleName() + " '" + getServletName() + "'");
   if (logger.isInfoEnabled()) {
      logger.info("Initializing Servlet '" + getServletName() + "'");
   }
   long startTime = System.currentTimeMillis();

   try {
      //初始化ioc容器并调用onRefresh方法
      this.webApplicationContext = initWebApplicationContext();
      initFrameworkServlet();
   }
   catch (ServletException | RuntimeException ex) {
      logger.error("Context initialization failed", ex);
      throw ex;
   }

   if (logger.isDebugEnabled()) {
      String value = this.enableLoggingRequestDetails ?
            "shown which may lead to unsafe logging of potentially sensitive data" :
            "masked to prevent unsafe logging of potentially sensitive data";
      logger.debug("enableLoggingRequestDetails='" + this.enableLoggingRequestDetails +
            "': request parameters and headers will be " + value);
   }

   if (logger.isInfoEnabled()) {
      logger.info("Completed initialization in " + (System.currentTimeMillis() - startTime) + " ms");
   }
}
```

这段代码主要就是初始化IoC容器，最终会调用refresh（）方 法，对于IoC容器的初始化这里就不再叙述。 我们看到，在IoC容器初始化之后，又调用了onRefresh（）方法，它是 在DisptcherServlet类中实现的，来看源码：

```java
protected void initStrategies(ApplicationContext context) {
   //多文件上传的组件
   initMultipartResolver(context);
   //初始化本地语言环境
   initLocaleResolver(context);
   //初始化模板处理器
   initThemeResolver(context);
   //初始化处理器映射器
   initHandlerMappings(context);
   //初始化参数适配器
   initHandlerAdapters(context);
   //初始化异常拦截器
   initHandlerExceptionResolvers(context);
   //初始化视图预处理器
   initRequestToViewNameTranslator(context);
   //初始化视图转换器
   initViewResolvers(context);
   //初始化FlashMap管理器
   initFlashMapManager(context);
}
```

到这就完成了Spring MVC的九大组件的初始化。接下来，我们来看 URL和Controller的关系是如何建立的。HandlerMapping 的子类 AbstractDetectingUrlHandlerMapping 实现了initApplicationContext（）方 法，我们直接看子类中的初始化容器方法：

```java
protected void detectHandlers() throws BeansException {
   ApplicationContext applicationContext = obtainApplicationContext();
   // 获取当前ApplicationContext容器中所有的Bean的名字
   String[] beanNames = (this.detectHandlersInAncestorContexts ?
         BeanFactoryUtils.beanNamesForTypeIncludingAncestors(applicationContext, Object.class) :
         applicationContext.getBeanNamesForType(Object.class));

   // Take any bean name that we can determine URLs for.
   //遍历beanNames，并找到这些Bean对应的URL
   for (String beanName : beanNames) {
      //查询Bean上的所有的URL（Controller上的URL和方法上的URL），该方法由对应的子类实现
      String[] urls = determineUrlsForHandler(beanName);
      if (!ObjectUtils.isEmpty(urls)) {
         //保存urls和beanName的对应关系，放入Map<urls,beanName>,该方法在父类AbstractUrlHandlerMapping中实现
         registerHandler(urls, beanName);
      }
   }

   if ((logger.isDebugEnabled() && !getHandlerMap().isEmpty()) || logger.isTraceEnabled()) {
      logger.debug("Detected " + getHandlerMap().size() + " mappings in " + formatMappingName());
   }
}


/**
 * 获取Controller中所有方法的URL，由子类实现，典型的模板模式
 */
protected abstract String[] determineUrlsForHandler(String beanName);
```

**determineUrlsForHandler**（String beanName）方法的作用是获取每 个Controller中的URL，不同的子类有不同的实现，这是典型的模板模式。因为开发中用得最多的就是用注解来配置Controller中的URL， BeanNameUrlHandlerMapping是AbstractDetectingUrlHandlerMapping的子 类，用于处理注解形式的URL映射。我们这里以BeanNameUrlHandlerMapping为例来进行分析，看看如何查找beanName 上所有映射的URL。

## 运行调用阶段

运行调用是由请求触发的，所以入口为DispatcherServlet的核心方法 doService（），doService（）中的核心由doDispatch（）实现，源代码 如下：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
   HttpServletRequest processedRequest = request;
   HandlerExecutionChain mappedHandler = null;
   boolean multipartRequestParsed = false;

   WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

   try {
      ModelAndView mv = null;
      Exception dispatchException = null;

      try {
         //1、检查是否是文件上传的参数
         processedRequest = checkMultipart(request);
         multipartRequestParsed = (processedRequest != request);

         // 2、取得处理当前请求的Controller，这里也称为Handler，即处理器
         // 第一步意义就在这里体现了，这里并不是直接返回Controller，而是
         // 返回了HandlerExecution 请求处理器链对象，该对象封装了Handler和interceptor
         mappedHandler = getHandler(processedRequest);
         //判断handler是否为空，为空则返回404
         if (mappedHandler == null) {
            noHandlerFound(processedRequest, response);
            return;
         }

         //3、获取处理请求的处理器适配器HandlerAdapter
         HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

         // 处理last-modified 请求头
         String method = request.getMethod();
         boolean isGet = "GET".equals(method);
         if (isGet || "HEAD".equals(method)) {
            long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
            if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
               return;
            }
         }

         if (!mappedHandler.applyPreHandle(processedRequest, response)) {
            return;
         }

         // 4、实际处理器处理请求，返回结果视图对象
         mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

         if (asyncManager.isConcurrentHandlingStarted()) {
            return;
         }

         //结果视图对象的处理
         applyDefaultViewName(processedRequest, mv);
         mappedHandler.applyPostHandle(processedRequest, response, mv);
      }
      catch (Exception ex) {
         dispatchException = ex;
      }
      catch (Throwable err) {
         // As of 4.3, we're processing Errors thrown from handler methods as well,
         // making them available for @ExceptionHandler methods and other scenarios.
         dispatchException = new NestedServletException("Handler dispatch failed", err);
      }
      processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
   }
   catch (Exception ex) {
      triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
   }
   catch (Throwable err) {
      triggerAfterCompletion(processedRequest, response, mappedHandler,
            new NestedServletException("Handler processing failed", err));
   }
   finally {
      if (asyncManager.isConcurrentHandlingStarted()) {
         // Instead of postHandle and afterCompletion
         if (mappedHandler != null) {
            // 请求成功响应之后的方法
            mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
         }
      }
      else {
         // Clean up any resources used by a multipart request.
         if (multipartRequestParsed) {
            cleanupMultipart(processedRequest);
         }
      }
   }
}
```

getHandler（processedRequest）方法实际上从HandlerMapping中找 到URL和Controller的对应关系，也就是Map＜url，Controller＞。我们知道，最终处理请求的是Controller中的方法，现在只是知道了 Controller，如何确认Controller中处理请求的方法呢？继续往下看。 从 Map＜urls，beanName＞中取得 Controller 后，经过拦截器的预处理方法，再通过反射获取该方法上的注解和参数，解析方法和参数上的注解，然后反射调用方法获取 ModelAndView结果视图。最后调用 RequestMappingHandlerAdapter的handle（）中的核心代码，由 handleInternal （request，response，handler）实现：

```java
protected ModelAndView handleInternal(HttpServletRequest request,
      HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

   ModelAndView mav;
   checkRequest(request);

   // Execute invokeHandlerMethod in synchronized block if required.
   if (this.synchronizeOnSession) {
      HttpSession session = request.getSession(false);
      if (session != null) {
         Object mutex = WebUtils.getSessionMutex(session);
         synchronized (mutex) {
            mav = invokeHandlerMethod(request, response, handlerMethod);
         }
      }
      else {
         // No HttpSession available -> no mutex necessary
         mav = invokeHandlerMethod(request, response, handlerMethod);
      }
   }
   else {
      // No synchronization on session demanded at all...
      mav = invokeHandlerMethod(request, response, handlerMethod);
   }

   if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
      if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
         applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
      }
      else {
         prepareResponse(response);
      }
   }

   return mav;
}
```

整个处理过程中最核心的步骤其实就是拼接Controller的URL和方法 的URL，与Request的URL进行匹配，找到匹配的方法。方法源码如下：


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

```java
protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
   //如果url为http://localhost:8080/web/hello-world，则lookupPath=web/hello-world
   String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
   request.setAttribute(LOOKUP_PATH, lookupPath);
   this.mappingRegistry.acquireReadLock();
   try {
      //遍历controller上的所有的方法，获取url匹配的方法
      HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);
      return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);
   }
   finally {
      this.mappingRegistry.releaseReadLock();
   }
}
```

通过上面的代码分析，已经找到处理请求的Controller中的方法了， 下面看如何解析该方法上的参数，并反射调用该方法。

```java
/**
 * 获取处理请求的方法，执行并返回结果视图
 */
@Nullable
protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
      HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

   ServletWebRequest webRequest = new ServletWebRequest(request, response);
   try {
      WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
      ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

      ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
      if (this.argumentResolvers != null) {
         invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
      }
      if (this.returnValueHandlers != null) {
         invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
      }
      invocableMethod.setDataBinderFactory(binderFactory);
      invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);

      ModelAndViewContainer mavContainer = new ModelAndViewContainer();
      mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
      modelFactory.initModel(webRequest, mavContainer, invocableMethod);
      mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

      AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
      asyncWebRequest.setTimeout(this.asyncRequestTimeout);

      WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
      asyncManager.setTaskExecutor(this.taskExecutor);
      asyncManager.setAsyncWebRequest(asyncWebRequest);
      asyncManager.registerCallableInterceptors(this.callableInterceptors);
      asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

      if (asyncManager.hasConcurrentResult()) {
         Object result = asyncManager.getConcurrentResult();
         mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
         asyncManager.clearConcurrentResult();
         LogFormatUtils.traceDebug(logger, traceOn -> {
            String formatted = LogFormatUtils.formatValue(result, !traceOn);
            return "Resume with async result [" + formatted + "]";
         });
         invocableMethod = invocableMethod.wrapConcurrentResult(result);
      }

      //这一步，完成请求中的参数和方法参数上数据的绑定
      invocableMethod.invokeAndHandle(webRequest, mavContainer);
      if (asyncManager.isConcurrentHandlingStarted()) {
         return null;
      }

      return getModelAndView(mavContainer, modelFactory, webRequest);
   }
   finally {
      webRequest.requestCompleted();
   }
}
```

invocableMethod.invokeAndHandle（）最终要实现的目的是：完成 请求中的参数和方法参数上数据的绑定。

Spring MVC中提供两种从请求 参数到方法中参数的绑定方式： 

（1）通过注解进行绑定，**@RequestParam**。 

（2）通过参数名称进行绑定。 通过注解进行绑定，只要在方法的参数前面声明 @RequestParam（＂name＂），就可以将请求中参数name的值绑定到方 法的该参数上。 通过参数名称进行绑定的前提是必须获取方法中参数的名称，Java 反射只提供了获取方法参数类型的方法，并没有提供获取参数名称的方 法。Spring MVC解决这个问题的方法是用asm框架读取字节码文件。 asm 框架是一个字节码操作框架，更多介绍可以参考其官网。个人建议 通过注解进行绑定，如下代码所示，这样就可以省去**asm**框架的读取字节码的操作。

```java
public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
      Object... providedArgs) throws Exception {

   Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
   if (logger.isTraceEnabled()) {
      logger.trace("Arguments: " + Arrays.toString(args));
   }
   return doInvoke(args);
}

protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
      Object... providedArgs) throws Exception {

   MethodParameter[] parameters = getMethodParameters();
   if (ObjectUtils.isEmpty(parameters)) {
      return EMPTY_ARGS;
   }

   Object[] args = new Object[parameters.length];
   for (int i = 0; i < parameters.length; i++) {
      MethodParameter parameter = parameters[i];
      parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
      args[i] = findProvidedArgument(parameter, providedArgs);
      if (args[i] != null) {
         continue;
      }
      if (!this.resolvers.supportsParameter(parameter)) {
         throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
      }
      try {
         args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
      }
      catch (Exception ex) {
         // Leave stack trace for later, exception may actually be resolved and handled...
         if (logger.isDebugEnabled()) {
            String exMsg = ex.getMessage();
            if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
               logger.debug(formatArgumentError(parameter, exMsg));
            }
         }
         throw ex;
      }
   }
   return args;
}
```

关于asm框架获取方法参数的内容这里就不再进行分析了，感兴趣 的“小伙伴 可以继续自行深入了解。 到这里，方法的参数列表也获取到了，可以直接进行方法的调用 了。最后我们再来梳理一下Spring MVC核心组件的关联关系，如下图所示。

![](https://gitee.com/zt888/zcq-pic-manage/raw/master/springmvc/%E4%B8%80%E6%AD%A5%E4%B8%80%E6%AD%A5%E6%89%8B%E7%BB%98Spring%20MVC%E8%BF%90%E8%A1%8C%E6%97%B6%E5%BA%8F%E5%9B%BE.jpg)
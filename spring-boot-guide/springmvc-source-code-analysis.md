# SpringMVC源码分析

根据上面分析的Spring MVC的九大组件，Spring MVC的处理过程可分为如下三步： 

```
  - ApplicationContext初始化时用Map保存所有URL和Controller 类的对应关系。 
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
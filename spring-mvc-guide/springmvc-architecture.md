

## 1、xml的方式配置SpringWebMvc

如下这个xml的配置，就是通过maven的骨架在web.xml中编写一些SpringMVC环境的配置，但是随着时间的推移，Springboot的出现，现在基本都是基于注解的方式进行开发了，因为这里所讲的SpringMVC的环境搭建都是主要偏向基于注解的方式生成。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
<!--dispatcherServlet-->
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:config/dispatcherServlet.xml</param-value>
    </init-param>
    <!--设置加载此servlet的优先级-->
    <load-on-startup>1</load-on-startup>
</servlet>
<!--spring 前端请求映射-->
<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>

<!--配置post请求乱码，让所有请求和响应都设置我们指定的编码-->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
    <!--强制请求HttpServletRequest请求对象使用encoding的字符编码的值-->
    <init-param>
        <param-name>forceRequestEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
    <!--强制请求HttpServletResponse请求对象使用encoding的字符编码的值-->
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <!--配置所有的请求都经过此过滤器-->
    <url-pattern>/*</url-pattern>
</filter-mapping>

<!--配置加载spring的监听器 ContextLoadListener-->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<!--配置监听器加载spring配置文件的位置-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:config/applicationContext.xml</param-value>
</context-param>
</web-app>
```
## 2、基于注解的方式搭建SpringMVC的环境

 2.1这里以grade进行构建，必须先引入SpringMVC的核心webmvc的jar包

```groovy
plugins {
    id 'java'
    id 'war'
}

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    //引入springwebmvcjar，这里是基于Spring源码的项目，所以这里直接是project
    compile(project(':spring-webmvc'))
    //引入servlet相关的jar
    compile("javax.servlet:javax.servlet-api:4.0.1")
}
```

2.2 创建一个类CustomizeWebInitializer 继承 AbstractAnnotationConfigDispatcherServletInitializer，实现其三个方法

```java
protected Class<?>[] getRootConfigClasses() 	//配置spring容器方法

protected Class<?>[] getServletConfigClasses()	//配置springmvc方法

protected String[] getServletMappings()			//设置DispatcherServlet的拦截规则
```

2.3 为什么要继承这个类呢？简单的说明一下

- 首先web容器启动（tomcat容器为例）的时候，会扫描每个jar包中META-INF\services\javax.servlet.ServletContainerInitializer文件， 加载其中指定的ServletContainerInitializer的实现类，并调用onStartup方法。我们的spring-web模块也是有这个配置文件，所以容器在启动时就会加载此类并调用此类的方法。

  ![](https://gitee.com/zt888/zcq-pic-manage/raw/master/springmvc/springservletContainerInitializer-config-01.png)

  

- SpringServletContainerInitializer 类上的有个@HandlesTypes(WebApplicationInitializer.class)用于指定一个接口，所有该接口的实现类会传给onStartup方法的第一个参数，因为有多个实现类，所以第一个接口的参数一个Class类集合。

  

- WebApplicationInitializer的实现类有很多，主要的三个实现类:

1. AbstractAnnotationConfigDispatcherServletInitializer：注解方式配置的DispatcherServlet初始化器
2. AbstractContextLoaderInitializer ：创建根容器
3.  AbstractDispatcherServletInitializer：创建webioc容器 创建DispatcherServlet 将创建的DispatcherServlet添加到ServletContext

![](https://gitee.com/zt888/zcq-pic-manage/raw/master/springmvc/WebApplicationInitializer-01.png)


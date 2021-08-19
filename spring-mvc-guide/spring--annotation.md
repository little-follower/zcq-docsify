## 原始web.xml配置SpringMVC

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


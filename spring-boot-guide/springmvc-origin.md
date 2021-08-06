# MVC模型的由来

在SpringMVC还没有普及或者诞生之前，大多数的Java程序猿（当然当时我可能还在上初中:koala:）开发还是基于两个Model，哪两个Model呢？

## Model 1 模型

**Model 1** ：是很早以前项目开发的一种常见模型，项目主要由 jsp 和 JavaBean 两部分组成。 

- 优点：结构简单。开发小型项目时效率高。 

- 缺点：1、JSP 的职责兼顾于**展示数据**和**处理数据**（也就是干了控制器和视图的事）；

  ​				 2、所有逻辑代码都是写在 JSP 中的，导致代码重用性很低；

  ​				 3、由于展示数据的代码和部分的业务代码交织在一起，维护非常不便。 

- 结论：结论是此种设计模型已经被淘汰没人使用了。

  model 1 web运行图：

  ![](https://gitee.com/zt888/zcq-pic-manage/raw/master/springmvc/model1-1.png)

  model1-2 图

## Model 2 模型

**Model 2**：是基于MVC架构的模式。  在Model 2架构中，**Servlet**作为前端控制器，负责接收客户端发送的请求  在**Servlet**中只包含控制逻辑和简单的前端处理； 后端JavaBean来完成实际的逻辑处理; 最后，转发到相应的JSP页面处理显示逻辑。

简单概括：Java Bean(*Model*) + JSP(*View*) + Servlet(*Controller*） 

- 优点：更适用于大规模应用的开发。 
- 缺点：抽象和封装程度不够，开发时不可避免地会重复造轮子，降低了程序的可维护性和复用性。
- 结论：于是很多JavaWeb开发相关的 MVC 框架应运而生比如**Struts2**，但是 **Struts2** 比较笨重。随着 Spring 轻量级开发框架的流行，**Spring** 生态圈出现了 **Spring MVC** 框架， **Spring MVC** 是当前最优秀的 MVC 框架。相比于 **Struts2** ， **Spring MVC** 使用更加简单和方便，开发效率更高，并且 **Spring MVC** 运行速度更快。


![](https://gitee.com/zt888/zcq-pic-manage/raw/master/springmvc/model2-1.png)

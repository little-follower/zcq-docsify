# [BiYing Copilot](https://https://www.bing.com//)
![image.png](/img/copilot1.png)

> 1. **注册方法与使用**请参考这个[链接](https://biying.com/)进行创建
>你必须使用Microsoft Edge 登录账号，然后在升级你的Edge在Copilot界面绑定账号，我这里使用的Github账号。


> 2. **生成代码：**例如给定一个[JHipster](https://www.jhipster.tech/)的jdl要求的描述，帮我生成jdl内容代码

1. **问题**：给我创建一个jhipster的jdl。要求如下：有两个类，一个是User类，一个是Department类。User类中的字段有用户Id：Long类型，用户名称、用户账号、用户密码、部门id；部门类中有部门Id，部门名称、部门编号。部门与用户的管理关系是一对一的关系，在user类中使用部门id作为外键。
2. **回答**：![image.png](/img/copilot2.png)
> 3. **解决项目问题**

1. **问题**：Caused by: java.lang.IllegalArgumentException: jdbcUrl is required with driverClassName.
2. **回答**：![image.png](/img/copilot4.png)![image.png](/img/copilot6.png)![image.png](/img/copilot7.png)

> 4. **优化代码**

1. **问题**：我按照group排序后，我想把这个元素对象中属性defaultQuota为true的排在第一位该怎么写
```java
List<SchemeQuotaEntryData> vSort = v.stream().
    sorted(Comparator.comparing(SchemeQuotaEntryData::getGroupNo))
    .collect(Collectors.toList());
```
  我按照group排序后，我想把这个元素对象中属性defaultQuota为true的排在第一位该怎么写

2. **回答**：![image.png](/img/copilot8.png)
> 5. **优化SQL**

1. **问题**：针对mysql like 查询量很大的数据 优化有什么方法
2. **回答**：![image.png](/img/copilot9.png)
> 6. **查询资料**

1. **问题**：@Resource 和 @Autowired的区别
2. **回答**：![image.png](/img/copilot10.png)



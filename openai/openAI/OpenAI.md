# [Open Chat-Gpt 3.5](https://chat.openai.com/)
![image.png](https://gitee.com/zt888/zcq-pic-manage/raw/master/OPENAI/gpt.png)

> 1. **注册方法与使用： **请参考这个[链接](https://mp.weixin.qq.com/s/Vdw3WJigk4XAgRAuuOCxtg)进行创建 & [VPN](http://ww2.xsus.buzz/) 请参考github上面的教程


> 2. **生成代码：**例如给定一个[JHipster](https://www.jhipster.tech/)的jdl要求的描述，帮我生成jdl内容代码

1. **问题**：给我创建一个jhipster的jdl。要求如下：有两个类，一个是User类，一个是Department类。User类中的字段有用户Id：Long类型，用户名称、用户账号、用户密码、部门id；部门类中有部门Id，部门名称、部门编号。部门与用户的管理关系是一对一的关系，在user类中使用部门id作为外键。
2. **回答**：
> 3. **解决项目问题**

1. **问题**：Caused by: java.lang.IllegalArgumentException: jdbcUrl is required with driverClassName.
2. **回答**：![jdl文件.png](https://gitee.com/zt888/zcq-pic-manage/raw/master/OPENAI/SpringBootJDBC%E9%97%AE%E9%A2%98.png)

> 4. **优化代码**

1. **问题**：我按照group排序后，我想把这个元素对象中属性defaultQuota为true的排在第一位该怎么写
```java
List<SchemeQuotaEntryData> vSort = v.stream().
    sorted(Comparator.comparing(SchemeQuotaEntryData::getGroupNo))
    .collect(Collectors.toList());
```
  我按照group排序后，我想把这个元素对象中属性defaultQuota为true的排在第一位该怎么写

2. **回答**：![优化代码.png](https://cdn.nlark.com/yuque/0/2023/png/25756928/1698297606938-b9f2f332-bc70-4065-936b-75335fdc8ec9.png#averageHue=%2344cd93&clientId=u3729d949-7438-4&from=drop&id=udb3578e5&originHeight=1145&originWidth=1921&originalType=binary&ratio=1.3625000715255737&rotation=0&showTitle=false&size=130714&status=done&style=none&taskId=ua91b62f6-91c8-4657-82b6-c61e04de9eb&title=)
> 5. **优化SQL**

1. **问题**：针对mysql like 查询量很大的数据 优化有什么方法
2. **回答**：![mysql优化.png](https://cdn.nlark.com/yuque/0/2023/png/25756928/1698297665310-b986712c-102a-407f-8098-b6abe76214ae.png#clientId=u3729d949-7438-4&from=drop&id=ue26e5920&originHeight=15510&originWidth=1921&originalType=binary&ratio=1.3625000715255737&rotation=0&showTitle=false&size=3784988&status=done&style=none&taskId=u89da7866-b6a8-452a-a041-f045e766967&title=)
> 6. **查询资料**

1. **问题**：@Resource 和 @Autowired的区别
2. **回答**：![spring技术.png](https://cdn.nlark.com/yuque/0/2023/png/25756928/1698297722219-19d8a153-6e17-4488-b938-6bfe2ddc9961.png#averageHue=%2394cfe2&clientId=u3729d949-7438-4&from=drop&id=u1c4f1e9f&originHeight=1196&originWidth=1921&originalType=binary&ratio=1.3625000715255737&rotation=0&showTitle=false&size=222124&status=done&style=none&taskId=ub66ad1b5-f4c2-4431-b42e-1b6f1785c2e&title=)




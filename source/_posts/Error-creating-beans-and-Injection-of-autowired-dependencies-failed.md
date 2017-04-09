title: Error creating beans and Injection of autowired dependencies failed
date: 2017-04-06 20:03:45
tags: [Java,Web]
categories: Java笔记
---
Spring中需要用到spring的事务回滚,注解方式是最简单易用的，一开始发现spring的事务没有起作用，最后查了很多资料,最后发现是配置文件中没有添加事务注解的配置,所以在spring的配置文件里面加了下面这行:
```
<tx:annotation-driven  transaction-manager="transactionManager" />
```
结果发现之前可以运行的项目突然不能运行了,涉及到事务注解的service全部无法注入

一般出现这个问题都会去网上查查是不是自己用的注入方式不对,有一些人还建议使用`@Resource`注解替换`@Autowried`,我试过时候发现还是不行．后来网上查了一下，其实很多人这么说都是人云亦云，很多人都不知道为什么，就知道`@Resource`是java自带的注解，所以比`@Autowried`靠谱,真的是这样吗?其实这个通过查资料就很容易知道并不是这样的:
* @Autowire 

默认按照类型装配，默认情况下它要求依赖对象必须存在如果允许为null，可以设置它required属性为false，如果我们想使用按照名称装配，可 以结合@Qualifier注解一起使用;

* @Resource

默认按照名称装配，当找不到与名称匹配的bean才会按照类型装配，可以通过name属性指定，如果没有指定name属 性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找 依赖对象.

所以说,正常情况下你可以理解为没啥区别,如果非要说后者比前者的优势,那就是减少外部依赖,因为`@Autowried`是spring提供的.

但是我还是照着网上说的,既然注入失败,那肯定是bean的名字还有类型找不到,所以我尝试强制指定`Service`的名字:
```
@Service("serviceName")
```
然后在`Controller`里面注入的时候使用`@Resource`注解指定名字:
```
@Resource(name = "serviceName")
```
结果奇迹出现了,还是报错,不过这个错误和之前不一样了,关键信息如下:
```
but was actually of type [com.sun.proxy.$Proxy17
```
这句话什么意思呢?意思是bean的名字是找到了，但是这个类型不符合,所以注入依然失败,总而言之就是我们要注入的`Service`的类型变成了`com.sun.proxy.$Proxy17`这个类型了,这是个啥类型？然后我网上查了很久,最后终于在stackoverflow上找到了答案,原来很多人都碰到了这个问题,原帖地址
> 
http://stackoverflow.com/questions/841231/fixing-beannotofrequiredtypeexception-on-spring-proxy-cast-on-a-non-singleton-be

摘一段Spring官方的文档:
> 
Applies to proxy mode only. Controls what type of transactional proxies are created for classes annotated with the @Transactional annotation. If "proxy-target-class" attribute is set to "true", then class-based proxies will be created. If "proxy-target-class" is "false" or if the attribute is omitted, then standard JDK interface-based proxies will be created. (See the section entitled Section 6.6, “Proxying mechanisms” for a detailed examination of the different proxy types.)

大概翻译一下就是:是否只采用代理模式来做事务管理,并且有个关键字`class-based proxies`,这样就不会有类型不匹配的了.
而且这个作者的这个问题也是由于他使用的事务注解而导致的,和我的这个问题非常相像，核心意思是这个注解默认是使用系统的代理模式,即`com.sun.proxy`这个里面的类,但是一般`spring`的项目都是使用的`cglib,aspectj`这类的库来做代理的默认实现,所以导致了以上的这种问题,那么解决方法就是将最开始的配置改为:
```
<tx:annotation-driven  transaction-manager="transactionManager" proxy-target-class="true"/>
```

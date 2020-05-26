title: 枚举类型全字段序列化转JSON
date: 2019-06-23 16:55:36
tags: [Java]
categories: Java笔记
---
### 背景
最近在构思做一个通用化的字典工具,其中有一个功能就是自动扫描枚举类，将枚举类序列化成一张表，对比更新到数据库中。但是在实际中使用发现，如果不做任何限制，直接用`fastjson`的`JSON.toJSONString(obj)` 方法，得到的只是枚举的名字，并没有得到一个全字段的json串。即`SUCCESS(0, "成功")`得到的将是`SUCCESS`这个字符串

fastjson版本:1.2.56

#### 解决方案

1. 重写覆盖枚举类的toString() 方法

```
@Getter
@AllArgsConstructor
public enum EnumTest {
    SUCCESS(0, "成功"),
    FAIL(-1, "失败");

    private int code;
    private String msg;

    @Override
    public String toString() {
        return "{\n" +
                "  \"code\": " + getCode() + ",\n" +
                "  \"msg\": " + getMsg() + "\n" +
                "}";
    }
}
```
这样的话，如果直接用`EnumTest.SUCCESS.toString()`就可以得到想要的结果，但是如果用`JSON.toJSONString(EnumTest.SUCCESS)`得到的仍然是`SUCCESS`。这个比较直白简单，但是不支持`fastjson`的方法,就是手动控制了枚举的`toString`输出内容，但是也有一个不好的问题就是，新增字段或者修改字段，还得改`toString`方法，万一忘了改，那可能会发生一些莫名其妙的Bug,而且还不易察觉,所以不推荐。
**NOTE:**注解是用了`lombok`包里面的一些方法

2. 自定义SerializeConfig

其实仔细看`JSON.toJSONString()`方法，有一些其他的重载方法提供了一些其他的参数，其中
```
public static String toJSONString(Object object, SerializeConfig config, SerializerFeature... features) {
        return toJSONString(object, config, (SerializeFilter) null, features);
    }
```
这个里面有一个自定义序列化的配置参
使用如下:
```
SerializeConfig config = new SerializeConfig();
config.configEnumAsJavaBean(EnumTest.class);
System.out.println(JSON.toJSONString(EnumTest.SUCCESS, config));
```
我们对比下几个的输出结果:
```
System.out.println("1:" + EnumTest.SUCCESS.toString());
System.out.println("2:" + JSON.toJSONString(EnumTest.SUCCESS));

SerializeConfig config = new SerializeConfig();
config.configEnumAsJavaBean(EnumTest.class);
System.out.println("3:" + JSON.toJSONString(EnumTest.SUCCESS, config));
```
结果如下:
```
1:{
  "code": 0,
  "msg": 成功
}
2:"SUCCESS"
3:{"code":0,"msg":"成功"}
```
通过这种方式，可以比较灵活自由的达到我们想要的序列化效果，而没有破坏掉其他的一些引用到枚举类的地方，因为如果直接重载了枚举本身的`toString()`方法，会产生一些不可预知的错误。

#### 枚举嵌套
对于简单的枚举，这种应该没有啥问题，如果枚举出现了嵌套呢?我们写个例子测试一下,再申明一个`EnumTest2`
```
// 枚举定义
@Getter
@AllArgsConstructor
public enum EnumTest2 {
    SUCCESS(0, "成功", EnumTest.SUCCESS),
    FAIL(-1, "失败", EnumTest.FAIL);

    private int code;
    private String msg;
    private EnumTest enumTest;

}

// 自定义序列化配置
SerializeConfig config = new SerializeConfig();
config.configEnumAsJavaBean(EnumTest2.class);
System.out.println(JSON.toJSONString(EnumTest2.SUCCESS, config));

// 输出
{"code":0,"enumTest":"SUCCESS","msg":"成功"}
```
可以看到`EnumTest2`本身序列化没问题，但是他的`enumTest`属性没有按照我们想要的方式来，需要改一些:
```
// 这个地方改一下
config.configEnumAsJavaBean(EnumTest.class, EnumTest2.class);
// 输出结果
{"code":0,"enumTest":{"code":0,"msg":"成功"},"msg":"成功"}
```

#### 枚举转字典
其实上面的基本是为了搞清楚枚举的序列化问题，主要目的其实还是为了我们的字典如何同步。因为并不是所有的枚举都需要入库，所以我们需要实现一个注解，当有这个注解的枚举，那么我们会把他同步到字典中。当然还有一个问题就是上面探讨的，如果一个枚举他嵌套了其他的枚举，我们还需要把他所引用的枚举都配置到自定义序列化的配置里，所以实现如下:

##### 同步注解


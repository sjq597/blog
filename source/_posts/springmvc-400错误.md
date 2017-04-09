title: springmvc 400错误
date: 2017-04-09 11:43:11
tags: [Java,Web]
categories: Java笔记
---
前后端交互数据的时候，经常需要前端把数据提交给后端，前端一般就用ajax提交数据，就像下面这样:
```
$.ajax({
        type: "post",
        url: "pos_url.do",
        contentType: "application/json",
        data: JSON.stringify(params),
        success: function (resp) {
	    xxxx;
        },
        error: function (resp) {
            xxxx;
        }
    });
```
然后后端采用spring的代码如下:
```
@RequestMapping(value = "post_url", method = RequestMethod.POST)
@ResponseBody
public String postURL(@RequestBody JsonParam[] params) {

}
```
然后就是`JsonParam`这个类的定义:
```
public class JsonParam implements Serializable {

    private String name;

    private String value;
```
这样的话是可以从后端接受参数的，但是后来觉得这个`JsonParam`定义的不是很好，所以就把`name`改成了`key`,结果就报错了`400 bad request`,不知道为啥，连请求都报错了，更别说参数的接收和解析了，网上查资料，找到一个答案提示了我:
> 
Verify the following things:
Whether the name of JSON matches the User class's field name.
Also check whether JSON value is supported by the corresponding field name datatype in User class.
Do try it, I have faced the same issue numerous times.

然后我才明白，之前我自定义的Json类的两个属性就是`name,value`，然后改成`key,value`就报错了,所以错误的根源应该就是前端传过来的数据是`name=xxx,value=xxx`的数据格式,然后后端想按`key=xxx,value=xxx`这样来接收解析，然后就报`400 bad request`这个错误了,如果直接使用request来接收应该就没有这个问题了，但是参数和参数的值就得自己解析了.

具体的详细策略可以看spring源码,大概就是需要实现一些转化器来实现`http-->bean`之间的互转

title: DataTables 常用操作
date: 2016-11-04 17:12:34
tags: [Js, datatables]
categories: JScript
---
DataTables这个插件功能非常强大，也很灵活，所以这里介绍一下几个项目里用到的常用操作,基本使用可以参考上一篇[DataTables使用样例](2016/10/01/DataTables使用样例)

### 改变列渲染方式
有时候需要给某些列单独渲染，或者在已有基础上增加一点特效，类似于状态之类的，比如:
![改变列渲染模式](http://7xn9y9.com1.z0.glb.clouddn.com/DataTables%20%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C01.png)
像这种可能状态列可能就是个0,1,2枚举值，需要根据不同的枚举值显示不同的效果，普通情况下当然就是显示数字了，现在假设状态列返回`True/False`代表激活/未激活，实现效果的部分具体代码为:
```
var myTable = $('#dynamic-table').DataTable({
                "bDestroy": true,
                //"bProcessing": true,
                "bServerSide": true,
                "sAjaxSource": "xxxxx",
                "fnServerData": function (sSource, aoData, fnCallback) {
                    $.ajax({
                        'dataType': 'json',
                        'type': 'POST',
                        'url': sSource,
                        'data': aoData,
                        'success': fnCallback
                    });
                },
                "columnDefs": [
                    {
                        "render": function (data, type, row) {
                            if (data == true) {
                                return '<span class="label label-sm label-success">激活</span>'
                            } else {
                                return '<span class="label label-sm label-inverse">未激活</span>'
                            }
                        },
                        "targets": 2
                    }
                ],
```
参数解释:

参数 | 含义
-----|-----
data | 单元格的实际值，这里为0/1,即True/False
targets | 指定的是哪一列，下标从0开始,即你指定了哪一列，data就是那一列的值了
row | 表格一行的值

最后的效果就像上面那样了。

### 单元格填充控件
有时候表格的元素是控件而不是简单的字符串填充，并且这个控件还必须有对应的事件监听，这个时候也可以使用上面的方法同样来实现，现在假设我要给表格的最后一列添加两个按钮，这两个按钮可以操作这行的元素:
```
var edit_str =
       '<div class="hidden-sm hidden-xs btn-group">' +
       '<button id="edit" class="btn btn-xs btn-success">' +
       '<i class="ace-icon fa fa-pencil-square-o bigger-120"></i>' +
       '</button>' +
       '<button id="delete" class="btn btn-xs btn-danger">' +
       '<i class="ace-icon fa fa-trash-o bigger-120"></i>' +
       '</button>' +
       '</div>'

    //initiate dataTables plugin
    var myTable = $('#dynamic-table').DataTable({
        bAutoWidth: false,
        "columnDefs": [
            {
                "render": function (data, type, row) {
                    row.edit = edit_str // 往每一行最后一列添加一个属性,即一列
                    if (data == true) {
                        return '<span class="label label-sm label-success">激活</span>'
                    } else {
                        return '<span class="label label-sm label-inverse">未激活</span>'
                    }
                },
                "targets": 2
            }
        ],
```
看一下最后的显示效果:
![单元格填充控件](http://7xn9y9.com1.z0.glb.clouddn.com/DataTables%20%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C02.png)
进一步需要给这两个按钮添加操作，例如编辑按钮可以编辑状态，删除按钮会删除这一行,注意我们添加的两个按钮的id分别为`edit`,`delete`,表格的id为`dynamic-table`:
```
$('#dynamic-table tbody').on('click', 'button#edit', function () {
    var data = myTable.row($(this).parents('tr')).data();

    $.ajax({
        url: "xxx",
        type: "POST",
        dataType: "json",
        data: data
    }).done(function (resp) {
        if (resp.ret == true) {
            myTable.ajax.reload();
        } else {
            alert("编辑失败!")
        }
    }).fail(function () {
        alert("网络错误！")
    });
});


$('#dynamic-table tbody').on('click', 'button#delete', function () {
    var data = myTable.row($(this).parents('tr')).data();

    var title_msg;
    var text_msg;
    $.ajax({
        type: "post",
        url: "xxxx",
        dataType: "json",
        data: data,
        success: function (resp) {
            if (resp.ret) {
                    alert("成功删除!");
                    myTable.ajax.reload();
                }
            } else {
                alert("删除失败!");
            }
        }
    });
})
```

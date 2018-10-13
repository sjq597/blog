title: DataTables使用样例
date: 2016-10-01 14:01:32
tags: [JS,Datatables]
categories: JScript
---
最近开发公司内部系统里面大量要使用到数据的展示,所以最后就研究了一下`DataTables`这个Js的插件，功能确实很强大，总结一下如何使用这个插件以及使用过程中碰到的一些问题。
我会先做一个简单的事例,然后逐渐扩展功能达到系统想要的要求，环境准备:

环境　| 版本
------|------
OS | Ubuntu 16.04 64bits <br>python2.7
Web框架 | [Flask 0.11.1](http://docs.jinkan.org/docs/flask/) 
模板引擎 | [Jinjia2 2.8](http://docs.jinkan.org/docs/jinja2/)
前端框架 | [ACE](http://ace.jeka.by/)
Js插件 | [DataTables](https://www.datatables.net/)


### 初始环境搭建
把需要的东西都下载下来即可，如果嫌麻烦，可以直接用第三方的cdn,国内的话推荐:[Bootstrap官方推荐](http://www.bootcdn.cn/),但是Ace这个框架的东西你还是要下载的，毕竟里面包含的东西太多了，github上直接clone到本地即可[Ace Github地址](https://github.com/bopoda/ace):下载之后把`assets`整个文件夹拷过来即可，由于是演示，所以这个项目可能会又其他一些演示，这里我使用蓝图对不同的演示进行分模块,基本的项目结构如下:
```
.
├── README.md
└── web
    ├── config.py
    ├── datatables
    │   ├── __init__.py
    │   ├── templates
    │   │   └── data_table
    │   │       └── index.html
    │   └── views.py
    ├── __init__.py
    ├── run.py
    ├── static
    │   └── assets
    │       ├── css
    │       ├── font-awesome
    │       ├── fonts
    │       ├── images
    │       ├── js
    │       └── swf
    └── templates
        └── base.html

```
数据都是用Ajax异步请求获取,其中异步获取数据包括两个部分:
* 异步获取表格的头部列信息,即表格头部一般是之前不知道的，这种情况尤其适用于BI平台这种专门展示数据的获取数据方式。
* 异步获取表格的填充数据信息,这个毫无疑问了，后端分页必须这么做

### 后端
后端主要就两个接口，由于只是演示，所以数据不是从数据库里面查的，但是是一样的效果
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from flask import Blueprint, render_template, jsonify, request

__author__ = 'anonymous'

data_table = Blueprint('data_table', __name__, template_folder='templates')


@data_table.route('/')
def index():
    return render_template('data_table/index.html')


@data_table.route('/cols', methods=['GET', 'POST'])
def cols():
    """
    获取表格需要展示列信息
    :return:
    """
    return jsonify(ret=True, columns=['col1', 'col2', 'col3', 'col4', 'col5'])


@data_table.route('/data', methods=['GET', 'POST'])
def data():
    """
    获取表格需要展示数据
    :return:
    """
    # 生成一个100行 5列的二维数组
    ao_data = map(lambda row: map(lambda col: '(%s,%s)' % (row, col), range(1, 6)), range(1, 101))

    start = int(request.form.get('iDisplayStart'))
    end = int(request.form.get('iDisplayStart')) + int(request.form.get('iDisplayLength'))

    return jsonify({
        "sEcho": request.form.get('sEcho'),
        "iTotalRecords": 100,
        "iTotalDisplayRecords": 100,
        "aaData": ao_data[start:end]
    })
```
**备注:**默认是解析`aaData`里面的数据进行填充，当然这个也可以在前端设置的。由于是后端获取数据进行分页，并且是用`POST`方式传输参数，所以可以用`request.form.get()`来获取各种`dataTables`的参数，又很多，将几个主要的:

参数 | 含义
-----|----
sEcho | 点击标记位,每次查询这个数会自增，需要原封不动的给传回去，不然不能分页
iTotalRecords | 总记录条数,显示用
iTotalDisplayRecords | 过滤之后的条数
aaData | 填充表格的数据，二维数组
iDisplayStart |　数据开始位置,分页用
iDisplayLength | 每页显示数据条数,分页用
sSortDir_[index] | 第index列数据顺序,例如 sSortDir_0='desc'表示最后返回的数据按第一列升序排序
sSearch | 搜索，即过滤条件


所以如果你是从数据库查询数据的话，可以把这些参数解析了然后拼出你要的sql，这样就可以按条件查询出你要的结果了，不过这个搜索过滤条件就比较随意，这里我就假定是对第一列进行搜索,例如:
```
select 
  col1,
  col2,
  col3,
  col4,
  col5
from
  user  
where
  col1 like ${sSearch}
order by
  col1 desc
limit 
  ${iDisplayLength} offset ${iDisplayStart};
```
这样后端的工作就基本完成了。


### 前端
前端的代码就一个文件，主要是用到了[Jinjia2](http://docs.jinkan.org/docs/jinja2/),不熟悉这个语法的可以去官网大概看一下，非常好上手，就是一个模板替换引擎,有一个公共的基础模板，里面引入了一些外部的文件和样式:
```
{% extends 'base.html' %}

{% block title %}
    DataTables演示
{% endblock %}

{% block body %}
    <div>
        <div class="row">
            <div class="col-xs-12">
                <!-- PAGE CONTENT BEGINS -->
                <div class="row">
                    <div class="col-xs-12">
                        <h3 class="header smaller lighter blue">
                            表格大标题
                        </h3>

                        <div class="clearfix">
                            <div class="pull-right tableTools-container"></div>
                        </div>
                        <div class="table-header">
                            我是表头
                        </div>

                        <!-- div.dataTables_borderWrap -->
                        <div>
                            <table id="dynamic-table"
                                   class="table table-striped table-bordered table-hover">
                                <thead>
                                    <tr>
                                    </tr>
                                </thead>

                                <tbody>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>

                <div id="modal-table" class="modal fade" tabindex="-1">
                    <div class="modal-dialog">
                        <div class="modal-content">
                            <div class="modal-header no-padding">
                                <div class="table-header">
                                    <button type="button" class="close" data-dismiss="modal"
                                            aria-hidden="true">
                                        <span class="white">&times;</span>
                                    </button>
                                    Results for "Latest Registered Domains
                                </div>
                            </div>

                            <div class="modal-body no-padding">
                                <table class="table table-striped table-bordered table-hover no-margin-bottom no-border-top">
                                    <thead>

                                    </thead>

                                    <tbody>

                                    </tbody>
                                </table>
                            </div>

                            <div class="modal-footer no-margin-top">
                                <button class="btn btn-sm btn-danger pull-left" data-dismiss="modal">
                                    <i class="ace-icon fa fa-times"></i>
                                    Close
                                </button>

                                <ul class="pagination pull-right no-margin">
                                    <li class="prev disabled">
                                        <a href="#">
                                            <i class="ace-icon fa fa-angle-double-left"></i>
                                        </a>
                                    </li>

                                    <li class="next">
                                        <a href="#">
                                            <i class="ace-icon fa fa-angle-double-right"></i>
                                        </a>
                                    </li>
                                </ul>
                            </div>
                        </div>
                        <!-- /.modal-content -->
                    </div>
                    <!-- /.modal-dialog -->
                </div>

            </div>
            <!-- /.page-content -->
        </div>
    </div>  <!-- 单独一个完整报表展示-->
{% endblock %}

{% block js %}
    {{ super() }}

    <!-- DataTables　Js-->
    <script src="{{ url_for('static', filename='assets/js/jquery.dataTables.min.js') }}"></script>
    <script src="{{ url_for('static', filename='assets/js/jquery.dataTables.bootstrap.min.js') }}"></script>
    <script src="{{ url_for('static', filename='assets/js/dataTables.buttons.min.js') }}"></script>
    <script src="{{ url_for('static', filename='assets/js/buttons.flash.min.js') }}"></script>
    <script src="{{ url_for('static', filename='assets/js/buttons.html5.min.js') }}"></script>
    <script src="{{ url_for('static', filename='assets/js/buttons.print.min.js') }}"></script>
    <script src="{{ url_for('static', filename='assets/js/buttons.colVis.min.js') }}"></script>
    <script src="{{ url_for('static', filename='assets/js/dataTables.select.min.js') }}"></script>

    <script type="text/javascript">
        var table = $('#dynamic-table')
        var aoColumns = []
        $.ajax({
            contentType: "application/json; charset=utf-8",
            url: "{{ url_for('data_table.cols') }}",
            type: 'post',
            success: function (resp) {
                $.each(resp.columns, function (i, v) {
                    table.find('thead > tr').append('<th>' + v + '</th>')
                    aoColumns.push({"title": v})
                });
                showTables(table, aoColumns)
            }
        });

        function showTables(dom_table, cols) {

            //initiate dataTables plugin
            var myTable = dom_table.DataTable({
                bAutoWidth: false,
                "aoColumns": cols,
                "aaSorting": [],

                //"bProcessing": true,
                "bServerSide": true,
                "sAjaxSource": "{{ url_for('data_table.data') }}",
                "fnServerData": function (sSource, aoData, fnCallback) {
                    $.ajax({
                        'dataType': 'json',
                        'type': 'POST',
                        'url': sSource,
                        'data': aoData,
                        'success': fnCallback
                    });
                },
                //,
                //"sScrollY": "200px",
                //"bPaginate": false,

                //"sScrollX": "100%",
                //"sScrollXInner": "120%",
                //"bScrollCollapse": true,
                //Note: if you are applying horizontal scrolling (sScrollX) on a ".table-bordered"
                //you may want to wrap the table inside a "div.dataTables_borderWrap" element

                //"iDisplayLength": 50


                select: {
                    style: 'multi'
                }
            });


            $.fn.dataTable.Buttons.defaults.dom.container.className = 'dt-buttons btn-overlap btn-group btn-overlap';

            new $.fn.dataTable.Buttons(myTable, {
                buttons: [
                    {
                        "extend": "colvis",
                        "text": "<i class='fa fa-search bigger-110 blue'></i> <span class='hidden'>Show/hide columns</span>",
                        "className": "btn btn-white btn-primary btn-bold",
                        columns: ':not(:first):not(:last)'
                    },
                    {
                        "extend": "copy",
                        "text": "<i class='fa fa-copy bigger-110 pink'></i> <span class='hidden'>Copy to clipboard</span>",
                        "className": "btn btn-white btn-primary btn-bold"
                    },
                    {
                        "extend": "csv",
                        "text": "<i class='fa fa-database bigger-110 orange'></i> <span class='hidden'>Export to CSV</span>",
                        "className": "btn btn-white btn-primary btn-bold"
                    },
                    {
                        "extend": "excel",
                        "text": "<i class='fa fa-file-excel-o bigger-110 green'></i> <span class='hidden'>Export to Excel</span>",
                        "className": "btn btn-white btn-primary btn-bold"
                    },
                    {
                        "extend": "pdf",
                        "text": "<i class='fa fa-file-pdf-o bigger-110 red'></i> <span class='hidden'>Export to PDF</span>",
                        "className": "btn btn-white btn-primary btn-bold"
                    },
                    {
                        "extend": "print",
                        "text": "<i class='fa fa-print bigger-110 grey'></i> <span class='hidden'>Print</span>",
                        "className": "btn btn-white btn-primary btn-bold",
                        autoPrint: false,
                        message: 'This print was produced using the Print button for DataTables'
                    }
                ]
            });
            myTable.buttons().container().appendTo($('.tableTools-container'));

            //style the message box
            var defaultCopyAction = myTable.button(1).action();
            myTable.button(1).action(function (e, dt, button, config) {
                defaultCopyAction(e, dt, button, config);
                $('.dt-button-info').addClass('gritter-item-wrapper gritter-info gritter-center white');
            });


            var defaultColvisAction = myTable.button(0).action();
            myTable.button(0).action(function (e, dt, button, config) {

                defaultColvisAction(e, dt, button, config);


                if ($('.dt-button-collection > .dropdown-menu').length == 0) {
                    $('.dt-button-collection')
                            .wrapInner('<ul class="dropdown-menu dropdown-light dropdown-caret dropdown-caret" />')
                            .find('a').attr('href', '#').wrap("<li />")
                }
                $('.dt-button-collection').appendTo('.tableTools-container .dt-buttons')
            });

            ////

            setTimeout(function () {
                $($('.tableTools-container')).find('a.dt-button').each(function () {
                    var div = $(this).find(' > div').first();
                    if (div.length == 1) div.tooltip({container: 'body', title: div.parent().text()});
                    else $(this).tooltip({container: 'body', title: $(this).text()});
                });
            }, 500);


            myTable.on('select', function (e, dt, type, index) {
                if (type === 'row') {
                    $(myTable.row(index).node()).find('input:checkbox').prop('checked', true);
                }
            });
            myTable.on('deselect', function (e, dt, type, index) {
                if (type === 'row') {
                    $(myTable.row(index).node()).find('input:checkbox').prop('checked', false);
                }
            });


            /////////////////////////////////
            //table checkboxes
            $('th input[type=checkbox], td input[type=checkbox]').prop('checked', false);

            //select/deselect all rows according to table header checkbox
            $('#dynamic-table > thead > tr > th input[type=checkbox], #dynamic-table_wrapper input[type=checkbox]').eq(0).on('click', function () {
                var th_checked = this.checked;//checkbox inside "TH" table header

                $('#dynamic-table').find('tbody > tr').each(function () {
                    var row = this;
                    if (th_checked) myTable.row(row).select();
                    else  myTable.row(row).deselect();
                });
            });

            //select/deselect a row when the checkbox is checked/unchecked
            $('#dynamic-table').on('click', 'td input[type=checkbox]', function () {
                var row = $(this).closest('tr').get(0);
                if (this.checked) myTable.row(row).deselect();
                else myTable.row(row).select();
            });


            $(document).on('click', '#dynamic-table .dropdown-toggle', function (e) {
                e.stopImmediatePropagation();
                e.stopPropagation();
                e.preventDefault();
            });


            //And for the first simple table, which doesn't have TableTools or dataTables
            //select/deselect all rows according to table header checkbox
            var active_class = 'active';
            $('#simple-table > thead > tr > th input[type=checkbox]').eq(0).on('click', function () {
                var th_checked = this.checked;//checkbox inside "TH" table header

                $(this).closest('table').find('tbody > tr').each(function () {
                    var row = this;
                    if (th_checked) $(row).addClass(active_class).find('input[type=checkbox]').eq(0).prop('checked', true);
                    else $(row).removeClass(active_class).find('input[type=checkbox]').eq(0).prop('checked', false);
                });
            });

            //select/deselect a row when the checkbox is checked/unchecked
            $('#simple-table').on('click', 'td input[type=checkbox]', function () {
                var $row = $(this).closest('tr');
                if ($row.is('.detail-row ')) return;
                if (this.checked) $row.addClass(active_class);
                else $row.removeClass(active_class);
            });


            /********************************/
            //add tooltip for small view action buttons in dropdown menu
            $('[data-rel="tooltip"]').tooltip({placement: tooltip_placement});

            //tooltip placement on right or left
            function tooltip_placement(context, source) {
                var $source = $(source);
                var $parent = $source.closest('table')
                var off1 = $parent.offset();
                var w1 = $parent.width();

                var off2 = $source.offset();
                //var w2 = $source.width();

                if (parseInt(off2.left) < parseInt(off1.left) + parseInt(w1 / 2)) return 'right';
                return 'left';
            }


            /***************/
            $('.show-details-btn').on('click', function (e) {
                e.preventDefault();
                $(this).closest('tr').next().toggleClass('open');
                $(this).find(ace.vars['.icon']).toggleClass('fa-angle-double-down').toggleClass('fa-angle-double-up');
            });
            /***************/


            /**
             //add horizontal scrollbars to a simple table
             $('#simple-table').css({'width':'2000px', 'max-width': 'none'}).wrap('<div style="width: 1000px;" />').parent().ace_scroll(
             {
               horizontal: true,
               styleClass: 'scroll-top scroll-dark scroll-visible',//show the scrollbars on top(default is bottom)
               size: 2000,
               mouseWheelLock: true
             }
             ).css('padding-top', '12px');
             */


        }
    </script>
{% endblock %}
```
这里主要就说说后面的js获取数据这一块:
```
var table = $('#dynamic-table')
var aoColumns = []
$.ajax({
    contentType: "application/json; charset=utf-8",
    url: "{{ url_for('data_table.cols') }}",
    type: 'post',
    success: function (resp) {
        $.each(resp.columns, function (i, v) {
            table.find('thead > tr').append('<th>' + v + '</th>')
            aoColumns.push({"title": v})
        });
        showTables(table, aoColumns)
    }
});

function showTables(dom_table, cols) {

    //initiate dataTables plugin
    var myTable = dom_table.DataTable({
        bAutoWidth: false,
        "aoColumns": cols,
        "aaSorting": [],

        //"bProcessing": true,
        "bServerSide": true,
        "sAjaxSource": "{{ url_for('data_table.data') }}",
        "fnServerData": function (sSource, aoData, fnCallback) {
            $.ajax({
                'dataType': 'json',
                'type': 'POST',
                'url': sSource,
                'data': aoData,
                'success': fnCallback
            });
        },
        //,
    }
}
```
第一个`ajax`从后端获取列的名字,然后动态插入表头元素:
```
table.find('thead > tr').append('<th>' + v + '</th>')
```
并且生成表头`aoColumns`供后面的`myTable`初始化用,格式就是{title:col_name}的一个数组，采用后端分页，所以需要设置
```
"bServerSide": true
```
及获取数据的源
```
"sAjaxSource": "{{ url_for('data_table.data') }}"
```
然后就是前后端数据交互:
```
                "fnServerData": function (sSource, aoData, fnCallback) {
                    $.ajax({
                        'dataType': 'json',
                        'type': 'POST',
                        'url': sSource,
                        'data': aoData,
                        'success': fnCallback
                    });
                },
```
这样就完成了一个简单的前后端交互表格的服务器端分页的例子,访问[http://127.0.0.1:5000/data_table](http://127.0.0.1:5000/data_table)完成后的效果如下:

![DataTables样例效果](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/DataTables%E4%BD%BF%E7%94%A8%E6%A0%B7%E4%BE%8B01.jpg)

### DataTables一些配置解释
这里又一份非常详细的参数说明，不太清楚的可以先看看这个,这个是别人总结的，可以参考一下这些参数的含义
```
$('#dataTable_ID1').dataTable({
    //"aaSorting" : [[1, "asc"]], //默认的排序方式，第1+1列，升序排列
    "aLengthMenu" : [5, 10, 25, 50, 100], //更改显示记录数选项(默认：[10,25,50,100])
    "bAutoWidth" : false, // 禁用自适应宽度(默认：true)
    //"bDeferRender":false,//推迟创建表元素每个元素,直到它们都创建完成(默认：false)
    "bDestroy" : true,//重新初始化表格，未匹配到表格则新建 (默认：false)
    "bFilter" : false,// 不适用搜索框过滤(默认：true)
    //"bInfo" : true, //显示页脚信息，左下角显示记录数(默认：true)
    //"bJQueryUI" : false, //不使用使用 jQury的UI theme(默认：false)
    "bLengthChange" : true,//显示每页几条数据的显示框(默认：true)
    //"bPaginate" : true, //显示(应用)分页器，不开启全显示(默认：true)
    "bProcessing" : true,//加载进度提示(默认：false)
    //"bScrollInfinite" : true, //启动初始化滚动条(默认：false)
    //"bRetrieve":false,//使用指定的选择器检索表格，注意，如果表格已经被初始化，该参数会直接返回已经被创建的对象，并不会顾及你传递进来的初始化参数对象的变化，将该参数设置为true说明你确认已经明白这一点，如果你需要的话，bDestroy可以用来重新初始化表格(默认：false)
    "bServerSide" : true,//启动服务器端数据导入(默认：false)
    "bSort" : true,//启用字段可排序(默认：true) TODO:单个列排序可禁用
    //"bStateSave" : true,//开启状态缓存，如分页信息，展示长度，开启后在ajax刷新纪录的时候不会将个性化设定重置为初始化状态，如: 会导致默认的aaSorting设置失效(默认：false)
    //"bScrollCollapse" : true, //开启高度自适应，当数据条数不够分页数据条数的时候，插件高度随数据条数而改变
    //"bScrollAutoCss":true,//指明滚动的标题元素是否被允许设置内边距和外边距等(默认：true)
    //"bScrollCollapse":false,//当垂直滚动被允许的时候，不强制强制表格视图在任何时候都是给定的高度(默认：false)
    //"bSortCellsTop":false,//允许使用底部的单元格，true为顶部(默认：false)
    //"iCookieDuration":7200,//cookie储存时长(单位:s)(默认：7200)
    //"iDeferLoading":null,//延时加载(type：int)(默认：null)
    //"iDisplayLength":10,//每页显示几条数据(默认：10)
    //"iDisplayStart":0,//当前页开始的记录序号(默认：0)
    //"iScrollLoadGap":100,//当前页面还有多少条数据可供滚动时自动加载新的数据(默认：100)
    "sDom": '<"top"l>rt<"bottom_left"i><"bottom_right"p><"clear">',//布局定义
        //格式指定：包括分页，显示多少条数据和搜索等
        //The following options are allowed:
        //    'l' - 左上角按个下拉框，10个，20个，50个，所有的哪个
        //    'f' - 快速过滤框
        //    't' - 表格本身
        //    'i' - 分页信息
        //    'p' - 分页按钮
        //    'r' - 现在正在加载中……
        //The following constants are allowed:
        //    'H' - jQueryUI theme "header" classes ('fg-toolbar ui-widget-header ui-corner-tl ui-corner-tr ui-helper-clearfix')
        //    'F' - jQueryUI theme "footer" classes ('fg-toolbar ui-widget-header ui-corner-bl ui-corner-br ui-helper-clearfix')
        //The following syntax is expected:
        //    '<' and '>' - div 元素
        //    '<"class" and '>' - 给div加clasa
        //    '<"#id" and '>' - 给div加上id
        //Examples:
        //    '<"wrapper"flipt>'
        //    '<lf<t>ip>'
        //例子：
        //'<"top"i>rt<"bottom"flp><"clear">'
        //解析结果：
        //    <div class="top">
        //    i
        //    </div>
        //    rt
        //    <div class="bottom">
        //    flp
        //    </div>
        //    <div class="clear"></div>
    "sPaginationType" : "full_numbers",//全页数显示 || "two_button"：显示两个按钮(默认：two_button)
    "sAjaxSource" : mediaHost+'/wxUsers/getDataTable1',
    //"sAjaxDataProp" : "aaDataName",//指定返回的数据对象名称(默认：aaData)
    //"sScrollX" : 720, //DataTables的宽，可以是css设置，或者一个数字(单位:px)，大于则开启水平滚动(默认:"blank string - i.e. disabled")
    //"sScrollY" : 480, //DataTables的高，可以是css设置，或者一个数字(单位:px)，大于则开启垂直滚动(默认:"blank string - i.e. disabled")
    //"sCookiePrefix" : "SpryMedia_DataTables_",//指定cookie前缀(默认："SpryMedia_DataTables_")


    //初始化过滤状态
    //"oSearch":{
    //    "sSearch":"value",
    //    "bRegex":false, //value不当成正则式
    //    "bSmart":true //灵活匹配策略
    //},

    //数据表列值
    "aoColumns" : [ {
        "mDataProp" : "data_properties0",
        "sClass" : "center",
        "bSortable" : false
        //"sDefaultContent":"",//此列默认值为""，防数据无值报错
        //"bVisible" : false //不显示此列
        }, {
            "mDataProp" : "data_properties1",
            "sClass" : "center",
            "bSortable" : false
        },  {
            "mDataProp" : "data_properties2",
            "sClass" : "center",
            "bSortable" : false
        }, {
            "mDataProp" : "data_properties2",
            "sClass" : "center",
            "bSortable" : false
        },
    ],

    //国际化配置
    "oLanguage" : {
        "sProcessing" : "正在加载数据，请稍后...",
        "sLengthMenu" : "每页显示 _MENU_ 条记录",
        "sZeroRecords" : "没有数据！",
        "sEmptyTable" : "表中无数据存在！",
        "sInfo" : "当前显示 _START_ 到 _END_ 条，共 _TOTAL_ 条记录",
        "sInfoEmpty" : "显示0到0条记录",
        "sInfoFiltered" : "数据表中共有 _MAX_ 条记录",
        //"sInfoPostFix": "",
        //"sSearch": "搜索:",
        //"sUrl": "",
        //"sLoadingRecords": "载入中...",
        //"sInfoThousands": ",",
        "oPaginate" : {
            "sFirst" : "首页",
            "sPrevious" : "上一页",
            "sNext" : "下一页",
            "sLast" : "末页"
        }
        //"oAria": {
        //    "sSortAscending": ": 以升序排列此列",
        //    "sSortDescending": ": 以降序排列此列"
        //}
    },
    /**
     *
     * @param nRow 当前行内容
     * @param aaData 当前数据对象
     * @param iDisplayIndex 当前行索引，从0开始
     * @param iDisplayIndexOfAadata 当前对象所在对象数组的索引，从0开始
     * @returns {*}
     */
    "fnRowCallback" : function(nRow, aaData, iDisplayIndex, iDisplayIndexOfAadata) {

        //修改第一列为多选框内容
        var firstTDHtml = '<label>firstTDHtml</label>';
        $('td:eq(0)', nRow).html(firstTDHtml);

        //修改第二列为序号
        var secondTDHtml = iDisplayIndex+1;
        $('td:eq(1)', nRow).html(secondTDHtml);

        return nRow;
    },
    "fnDrawCallback" : function(oSettings) {
        // jAlert( 'DataTables 重绘了' );
    },
    "fnFooterCallback" : function(nFoot, aData, iStart, iEnd, aiDisplay) {
        // jAlert("FooterCallback");
    },
});
```

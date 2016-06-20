title: Python 写csv文件Excel打开乱码解决方案
date: 2016-04-24 18:37:32
tags: [Python,Linux]
categories: Python笔记
---
### 需求背景
最近为公司开发了一套邮件日报程序，邮件一般就是表格，图片，然后就是附件。附件一般都是默认写到txt文件里，但是PM希望邮件里的附件能直接用Excel这种软件打开，最开始想保存为Excel，但是一想Excel的文件体积会多出好多倍，csv文件默认也是使用Excel打开的，但是根本还是文本文件，体积小，保存也方便，于是最终决定使用csv模块来保存文件。

### Python写csv文件
Python提供了内置模块读写csv文件，这里我只用到了写，读这里就不做介绍了，也不难,主要是解决乱码问题。
```
def save2csv(file_name=None, header=None, data=None):
    """
    保存成CSV格式文件,方便Excel直接打开
    :param file_name: 保存的文件名
    :param header: 表头,每一列的名字
    :param data: 具体填充数据
    :return:
    """
    if file_name is None or isinstance(file_name, basestring) is False:
        raise Exception('保存CSV文件名不能为空,并且必须为字符串类型')

    if file_name.endswith('.csv') is False:
        file_name += '.csv'

    file_obj = open(file_name, 'wb')
    file_obj.write(codecs.BOM_UTF8)     # 防止乱码
    writer = csv.writer(file_obj)

    if data is None or isinstance(data, (tuple, list)) is False:
        raise Exception('保存CSV文件失败,数据为空或者不是数据类型')

    if header is not None and isinstance(header, (tuple, list)) is True:
        writer.writerow(header)

    for row in data:
        writer.writerow(row)
```
**注意:**有三句话就是为了防止乱码的
```
    file_obj = open(file_name, 'wb')
    file_obj.write(codecs.BOM_UTF8)     # 防止乱码
    writer = csv.writer(file_obj)
```
在文件头部写入`codecs.BOM_UTF8`就能防止乱码了,文件都是utf-8编码格式的。

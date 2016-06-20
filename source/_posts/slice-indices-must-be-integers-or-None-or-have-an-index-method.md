title: slice indices must be integers or None or have an __index__ method
date: 2016-02-24 20:30:51
tags: [Python]
categories: Python笔记
---
写了个脚本读取指定文件的某一行到某一行，然后利用行数据做一些匹配提取，在终端中调用方式为：
```bash
python run.py file_name beg_line end_line
```
代码如下：
```Python
if __name__ == '__main__':
    file_name = sys.argv[1]
    beg_line = sys.argv[2]
    end_line = sys.argv[3]

    with open(file_name, 'rb') as f:
        for val in f.readlines()[beg_line:end_line]:
            print val.strip()
```
当我在终端中输入：
```bash
python run.py 12 20
```
时，报错：
> TypeError: slice indices must be integers or None or have an __index__ method

解决办法，python作为一个解释型语言，对类型没有强制检查，我们虽然输入的是两个数字，但是实际上**beg_line**,**end_line**是`str`类型,所以传入切片的类型不对，就会报上面的错。

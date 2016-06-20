title: MySQL 创建用户以及附权限
date: 2016-04-23 12:50:52
tags: [MySQL]
categories: Linux使用
---
有时候为了测试，需要单独建一些MySQL用户来限定这部分用户的权限，并且需要限定哪些ip可以访问到我的数据库，具体步骤为,在装有MysQL的机器上执行：
1. 创建用户，一般为了避免暴露自己的密码习惯，可以用Linux自带的生成密码命令

```
➜✗ md5pass
$1$6Fy9BTnB$Ruw50OR1oiUCwP73abnlD0
```
**备注:**`6Fy9BTnB`为用户名，`Ruw50OR1oiUCwP73abnlD0`为密码
然后执行MySQL命令，语法为:
```
mysql> CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```
假设你机器IP为`192.168.1.101`,则对应的命令为:
```
CREATE USER '6Fy9BTnB'@'Ruw50OR1oiUCwP73abnlD0' IDENTIFIED BY '192.168.1.101';
```

2. 给MySQL用户附权限

```
mysql> grant 权限1,权限2,...权限n on 数据库名称.表名称 to 用户名@用户地址 identified by 'password';
```
例如:
```
mysql>grant select,insert,update,delete,create,drop on order.oerder_info to 6Fy9BTnB@192.168.1.101 identified by 'Ruw50OR1oiUCwP73abnlD0';
```
只有来自192.168.1.101的用户6Fy9BTnB才能有对应的访问权限，其他的机器是无法访问的。
这里还有个快捷的操作，如果要附所有表的的所有操作权限，不可能一个一个去写，命令如下:
```
mysql>grant all privileges on order.oerder_info to 6Fy9BTnB@192.168.1.101 identified by 'Ruw50OR1oiUCwP73abnlD0';




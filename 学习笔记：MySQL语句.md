## 连接登录MySQL：
```
mysql -u [用户名] -p [密码]
```
>[用户名]：登录时的用户名，可填'root'登录root账户。<br>
>[密码]：账户密码，没有可不填。<br>

## 创建数据库
```
create database [数据库名];
```
>[数据库名]：为创建的数据库提供一个名称。<br>

## 删除数据库
```
drop database [数据库名];
```
>[数据库名]：填入要删除的数据库的名称。

## 显示全部数据库：
```
show databases;
```
>此处注意'databases'是复数形式（结尾的's'）。

## 连接一个数据库：
```
use database [数据库名];
```
>[数据库名]：填入指定的数据库名，可在该数据库内进行操作。<br>

## 显示数据库内的全部表
 ```
 show tables;
 ```
>此处的'tables'要求复数形式（注意's'）。<br>

## 连接一个表
```
 use [表名] from [数据库名];
```
>[表名]：要选择的表名。<br>
>[数据库名]：所选择的表所选的数据库。<br>


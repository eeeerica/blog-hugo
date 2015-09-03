---
date: 2015-06-06T11:10:08+08:00
description: ""
tags:
- development
- mysql
title: MySQL调整参数:open_files_limit & max_connections & table_open_cache
topics:
- Development
---



## 起因
非root用户运行MySQL，当MySQL配置比较高时，MySQL运行中生效的参数值与配置的值不一样,所以具体分析一下MySQL是怎么调整这些参数值的。
所以这篇文章的目的是为了说明在系统资源不够的情况下，MySQL 是怎么调整者三个参数的
### 说明
此文涉及到3个参数open_files_limit & max_connections & table_open_cache，这3个参数与系统相关的资源是最多能同时打开的文件（ulimit -n 查看）实际即文件描述符（fd）。

**系统参数与文件描述符的关系**
- max_connection & fd : 每一个MySQL connection都需要一个文件描述符
- table_open_cache &  fd 打开一张表至少需要一个文件描述符，如打开MyISAM需要两个fd




## MySQL调整参数的方式
1. 根据配置(配置的3个参数值或默认值)计算`request_open_files(需要的文件描述符)`
2. 获取有效的系统的限制值`effective_open_files`
3. 根据`effective_open_files`调整`request_open_files`
3. 根据调整后的`request_open_files`,计算实际生效的参数值(show variables 查看到的3个参数值)

### 计算`request_open_files`
`request_open_files`有三个计算公式
```
    // 最大连接数+同时打开的表的最大数量+其他（各种日志等等）
      limit_1= max_connections + table_cache_size * 2 + 10;
    
     //假设平均每个连接打开的表的数量（2-4）
     //源码中是这么写的：
     //We are trying to allocate no less than  
     //	max_connections*5 file handles 
      limit_2= max_connections * 5;
    
      //mysql 默认的最低值是5000
      limit_3= open_files_limit ? open_files_limit : 5000;

所以open_files_limit期待的最低 
    request_open_files= max(limit_1, limit_2,limit_3);
```
### 计算`effective_open_files`:    
MySQL 的思路：
在**有限值**的的范围内MySQL **尽量**将`effective_open_files`的值设大


![ddd](media/2.png)
### 修正request_open_files

`requested_open_files`= min(`effective_open_files`, `request_open_files`);


### 重新计算参数值

### 修正open_files_limit
`open_files_limit` = `effective_open_files`

### 修正max_connections  
  max_connections 根据 request_open_files 来做修正。

```
limit = requested_open_files - 10 - TABLE_OPEN_CACHE_MIN * 2;
```

- 如果配置的max_connections值大于limit，则将 max_connections 的值修正为limit
- 其他情况下 max_connections 保留配置值  
  
### 修正table_cache_size
table_cache_size 会根据 request_open_files 来做修正

```
// mysql table_cache_size 最小值，400
limit1 = TABLE_OPEN_CACHE_MIN 
// 根据 requested_open_files 计算
limit2 = (requested_open_files - 10 - max_connections) / 2
limit = max(limit1,limt2);
```
- 如果配置的 table_cache_size 值大于limit，则将 table_cache_size 的值修正为limit
- 其他情况下 table_cache_size 保留配置值

## 举例

**以下用例全部在非 root 用户下运行**

---
- 系统资源不够且无法调整
```
参数设置：

//mysql
	max_connections = 1000
//ulimit -n 
	1024

生效的值：

open_files_limit = 1024
max_connections = 1024 - 10 - 800 = 214
table_open_cache = ( 1024 - 10 - 214) / 2 = 400
```

---
- 系统资源不够可以调整
```
参数设置：

//mysql
	max_connections = 1000
//ulimit -S -n 
	1000
//ulimit -H -n 
	65535

生效的值：

open_files_limit = 65535
max_connections = 1000
table_open_cache = ( 1024 - 10 - 214) / 2 = 400
```

---
mysql 修改 open_files_limit
```
参数设置：

//mysql
	max_connections = 1000
	max_connections = 1000
//ulimit -n 
	65535

生效的值：

open_files_limit = 65535
max_connections = 1000
table_open_cache = 2000
```

## 其它
淘宝数据库内核月报中说道的相关内容：

[MySQL · 答疑解惑 · open file limits](http://mysql.taobao.org//monthly/2015/08/07/)
这篇主要讲的是，MySQL 在执行哪些操作时会执行打开文件的操作

## 附注
[1] . 非root用户，只能设置Hard limit之内的值，root用户可以设置hard limit的值
[2] .[参考文章][1]


 [1]: http://www.sudops.com/mysql-open-file-limit-max_connections-table-open-cache.htmlz

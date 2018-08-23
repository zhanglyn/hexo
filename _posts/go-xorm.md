---
title: 使用xorm-cmd反向生成数据库结构
date: 2018-08-23 15:13:02
tags: go-xorm

---

## 使用xorm工具，根据数据库自动生成 go 代码

### 关于 xorm

xorm是一个简单而强大的Go语言ORM库. 通过它可以使数据库操作非常简便。我在项目中经常使用，它的特性如下：

> - 支持Struct和数据库表之间的灵活映射，并支持自动同步表结构
> - 事务支持
> - 支持原始SQL语句和ORM操作的混合执行
> - 使用连写来简化调用
> - 支持使用Id, In, Where, Limit, Join, Having, Table, Sql, Cols等函数和结构体等方式作为条件
> - 支持级联加载Struct
> - 支持LRU缓存(支持memory, memcache, leveldb, redis缓存Store) 和 Redis缓存
> - 支持反转，即根据数据库自动生成xorm的结构体
> - 支持事件
> - 支持created, updated, deleted和version记录版本（即乐观锁）

想了解更多请移步：http://www.xorm.io/



### xorm 工具

xorm 是一组数据库操作命令的工具，包含如下命令：

> - **reverse** 反转一个数据库结构，生成代码
> - **shell** 通用的数据库操作客户端，可对数据库结构和数据操作
> - **dump** Dump数据库中所有结构和数据到标准输出
> - **source** 从标注输入中执行SQL文件
> - **driver** 列出所有支持的数据库驱动



首先下载该工具 ：

```
go get github.com/go-xorm/cmd/xorm
```

同时需要安装对应的 driver :

```
go get github.com/go-sql-driver/mysql  //MyMysql
go get github.com/ziutek/mymysql/godrv  //MyMysql
go get github.com/lib/pq  //Postgres
go get github.com/mattn/go-sqlite3  //SQLite
```

还需要下载 xorm ：

```
go get github.com/go-xorm/xorm
```

编译 `cmd/xorm` 会生成 xorm 工具， 假如环境变量。

这时候，执行 `xorm help reverse` 能获取帮助信息如下：

```
usage: xorm reverse [-s] driverName datasourceName tmplPath [generatedPath] [tableFilterReg]

according database's tables and columns to generate codes for Go, C++ and etc.

    -s                Generated one go file for every table
    driverName        Database driver name, now supported four: mysql mymysql sqlite3 postgres
    datasourceName    Database connection uri, for detail infomation please visit driver's project page
    tmplPath          Template dir for generated. the default templates dir has provide 1 template
    generatedPath     This parameter is optional, if blank, the default value is model, then will
                      generated all codes in model dir
    tableFilterReg    Table name filter regexp
```

例如：.\xorm.exe reverse  mysql root:123456@(127.0.0.1:3306)/source?charset=utf8 F:\gopro\src\github.com\go-xorm\cmd\xorm\templates\goxorm 

反向生成的model将会在github.com\go-xorm\cmd\xorm 目录下 

命令行添加-s后，数据库的表结构将会都存在于struct.go文件中

部分转载于：https://www.cnblogs.com/artong0416/p/7456674.html
---
title: PostgreSQL数据库教程
categories: 软件工程
---



## 环境配置



### 下载与登录

直接使用`homebrew`安装postgresql即可.

查看postgresql的所有可下载版本

```bash
brew search postgresql
```

然后直接用`brew install`下载.

下载之后, 添加环境变量:

```bash
echo 'export PATH="/opt/homebrew/opt/postgresql@15/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

然后, 启动服务:

```bash
brew services start postgresql@15
```

默认情况下, 如果你需要操作数据库, 需要有6个信息:

* 用户名 (默认创建的用户就是你电脑的Host)
* 密码 (你电脑Host的密码)
* 操作哪个数据库(默认创建的数据库叫`postgres`)
* 数据库Host(本地就是`localhost`)
* 数据库服务的端口 (postgresql默认是5432)

如果要登录数据库, 可以使用如下命令:

```bash
psql -h hostname \
		 -p portnumber \
     -U username \
     -d databasename
```



### 登录环境变量

每次命令的参数太多, 可以通过设置环境变量:

* Hostname: `PGHOST`
* Port: `PGPORT`
* Database: `PGDATABASE`
* User: `PGUSER`

更多的数据库环境变量信息可以参考: https://www.postgresql.org/docs/current/libpq-envars.html



## psql的配置和常用命令

### psql的增强版-pgcli

> https://github.com/dbcli/pgcli

在macos上用:

```bash
brew install pgcli
```

下载完成并配置完登录的环境变量后, 使用`pgcli`即可登录数据库.



### psql的常见命令

| 命令            | 含义                         |
| --------------- | ---------------------------- |
| `\l`            | 查看所有的数据库             |
| `\c + 数据库名` | 进入到某一个数据库           |
| `\d`            | 查看数据库中的表             |
| `\q`            | 退出psql                     |
| `<ctrl>l`       | 清空数据库屏幕               |
| `\dn`           | 列出当前Database所有的schema |
| `\dt`           | 列出当前所有表               |
| `\du`           | 查看所有user                 |
|                 |                              |





## PostgreSQL的逻辑存储



### Database

Database是数据库逻辑存储的最高一级, 可以通过下面的命令创建:

```sql
create database <Database_name>;
```

创建这个Database的人默认就是管理员.

这个数据库是从模板数据库`template1`克隆而来的, 可以使用`\l`查看所有数据库.

* 默认情况下, 如果不显式指定模板, 创建的database都是由`template1`克隆而来.



### Schema

Schema是Database下面一级的一个空间, 这个Schema可以存储不同的数据库对象, 例如表, 索引, 函数等.

如果创建了一个database, 那么默认会创建两个schema:

* `public`: 这个database中的所有用户都可以访问的schema, 默认创建的数据库对象就在`public`中.
* `information_schema`: 存储了当前database的一些元数据.

如果要创建一个schema, 可以使用:

```sql
create schema <Schema_name>;
```

* 如果要索引一个数据库中的某个对象, 可以用`<schema_name>.<dataobject_name>`.

* 如果要显示一个schema中的信息, 例如所有表格, 可以用`\dt schema_name.*`.
* 如果要在一个特定的schema下创建数据库对象(例如表格), 可以用`create table schema_name.table_name`.



## Create Table

创建表格的文法如下:

```sql
create table <table_name>
(
  column_name1 <data_type1> <inline_constraint>,
  column_name2 <data_type2> <inline_constraint>,
  column_name3 <data_type3> <inline_constraint>,
  ...,
  constraint <constraint_name>
    <constraints>,
    <constraints>,
    <constraints>,
  ...
);
```



## 字段的数据类型

### 整数

| 关键字    | 含义                 |
| --------- | -------------------- |
| `integer` | 32位有符号整数       |
| `serial`  | 32位的无符号自增整数 |
|           |                      |
|           |                      |



### 浮点数

| 关键字          | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| `decimal(p, s)` | 浮点数, `p`表示这个浮点数的所有数字占用的数位个数(包括小数), 取值范围是`[1, 1000]`, `s`表示小数所占的位数, 取值范围是`[0, p]`. |



### 字符串

| 关键字       | 含义                                                    |
| ------------ | ------------------------------------------------------- |
| `char(n)`    | 长度限制为`n`个字符的字符串, 如果不满`n`, 用空格补齐    |
| `varchar(n)` | 长度限制为`n`个字符的字符串, 如果不满`n`, 不用空格补齐. |
| `text`       | 没有长度限制的字符串, 用来存储长文本.                   |
| `varchar`    | 和`text`一样.                                           |



## NULL

注意: <font color=red>在SQL中, null不是一个数值, 而是表示这个字段还没有设置值.</font>

* 因此, 尽量不要将`null`和其他操作结合, 例如加减乘除.

在SQL中, 要用`is`关键字来判断一个字段是不是`null`, 而不是用`==`

```sql
select * from xxx where xxx is not null;
```



## 字段约束



### not null

这个约束表示一个字段不允许存在`null`值, 是一个inline-constraint, 定义的关键词就是`not null`.



### unique

如果一个/多个字段被设置成unique, 那么表中不允许有两条纪录有同样的字段值, 关键词是`unique`. 

* 但是<font color=red>即使这一列是`unique`, 这一列还是允许有多个`null`</font>.

`unique`可以作为inline-constraint, 也可以定义在constraint-block中, 如果定义在constraint-block中, 那么`unique`可以限制某几个字段的组合是唯一的, 例如:

```sql
# 限制first_name和last_name的组合唯一.
unique(first_name, last_name)
```



### primary key(主键)

主键的关键字是`primary key`.

如果一个/多个字段被设置成primary key, 那么有如下含义:

* 组成主键的所有字段都是`not null`.
* 组成主键的所有字段有`unique`约束.
* 主键可以唯一确定表中的所有其他字段.

一个表中只允许有一个主键.

如果只有一个字段是主键, 那么主键约束可以作为inline-constraint.

如果有多个字段是主键, 那么主键约束可以定义在constraint-block中, 语法如下:

```sql
primary key(column1, column2)
```



### foreign key(外键)

外键的关键字是`foreign key`, 外键主要是用来管理表与表之间的integration的.

一个表中的一个字段, 可以作为外键链接到另外一个表的主键/unique的字段, 如果被关联的表中某些记录发生了变化, 那么外键所在的表也需要变化, 才能保证数据的一致性.

* 一般来说, 一个外键只有一个字段, 但是一个表可以有多个外键.

定义外键需要在constraint-block中定义, 定义语法如下:

```sql
foreign key (column) references table_name (column)
	on update [update_policy]
	on delete [delete_policy]
```

其中的`update_policy`和`delete_policy`指的是, 如果修改了外键链接的字段, 或者删除了外键链接的表的某一行, 那么外键所在的表应该怎么变化, 其中一共有5种策略:

* `no action`: 外键所在的表不动, 这个是默认策略.
* `cascade`: 外键所在的表也删除对应记录, 或者更新对应字段.
* `restrict`: 直接报错.
* `set null`: 如果父表删除记录, 那么子表匹配行全部设成`null`, 父表更新字段, 子表对应字段设置成`null`.
* `set default`: 和`set null`原理一样, 只是不是设置成`null`, 而是设置成一个默认值.



## 三大范式

* 第一范式: 表格中的每一个字段都必须不可再分.
  * 例如你有一个字段叫班级, 字段值为高三一班就不可以, 就需要分成年级是高三, 班级是一班.
* 第二范式: 表格中的主键可以确定每一个其他字段的值, 但是<font color=red>表格中主键的任意一个子集却不能确定任何一个字段的值.</font>
* 第三范式: 对于表中的非主键字段, 它们之间完全独立, 没有依赖关系, 并且非主键字段不能确定主键.



## Insert

Insert语句用来向已经创建的表中插入一行, 语法是:

```sql
insert into <table_name>(column1, column2, ...) values
	(value11, value12, ...),
	(value21, value22, ...),
	...
```

其中, 如果有某些列没有指定值, 那么默认是`null`, 如果这一列凑巧又设置成了`not null`, 那么就会报错.

* 注意: 在SQL语句中, 字符串使用的是`''`.
* 注意: 在SQL语句中, 转义字符不是用`\`, 而使用`'`, 例如如果一个字符串中包含`'`字符, 那么字面值可以是`''a'`, 这个就表示`'a`这个字符串, 第一个`'`是转义字符.



## Select

Select指令用于从表中查询数据.

如果要从一个表中查询所有数据, 可以使用:

```sql
select * from table_name;
```



## 面试题

> 索引失效的场景?

* 模糊查询, 用`like`时, 如果开头是`%`.
* 数据类型错误.
* 对索引的字段使用函数.
* 如果不限制索引列是`not null`会失效.
* 对索引列进行加减乘除等运算.
* 在复合索引中, 如果不是按照从索引列最左侧开始查找, 那么就会失效.
* 如果数据库认为全表扫描比使用索引更快, 那么索引就会失效.

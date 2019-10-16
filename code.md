## 第三章 SQL语言入门
### 简介
1. 分类
SQL命令一般分为DQL(数据查询语言; select)，DML(数据操纵语言; insert、update、delete)，DDL(数据定义语言; create等)
2. 语法结构
命令由`;`结束。可以有注释，注释相当于空白。

### DDL
1. 建表
语法：
```SQL
create table table_name (
  col01_name data_type,
  col02_name data_type,
  ...
);
```
例子：
```
\d 显示所有表格
\d score # 显示score表格的定义情况
```
建表的时候，可以指定主键，主键是表中行的唯一标识，不可重复：
```SQL
create table student (no int primary key, student_name varchar(40), age int);
```
2. 删除表格：`drop table table_name;`

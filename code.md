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
```SQL
create table score (
  student_name varchar(40),
  chineses_score int,
  math_score int,
  test_date date,
);
\d # 显示所有表格
\d score # 显示score表格的定义情况
```
建表的时候，可以指定主键，主键是表中行的唯一标识，不可重复：
```SQL
create table student (no int primary key, student_name varchar(40), age int);
```
2. 删除表格：`drop table table_name;`

### DML
1. 插入语句
```SQL
insert into student values(1, '张三', 14);
insert into student (no, age, student_name) values (2, 13, '李四');
insert into student (no, student_name) values (3, '王二'); # 缺失值会被置空
2. 更新语句
```SQL
update student set age = 15;
update student set age = 14 where no = 3;
update student set age=13, student_name='王明充' where no=3; # 同时更新多个值
```
3. 删除语句
```SQL
delete from student where no = 3;
delete from student; # 删除表中所有数据
```

### DQL
1. 单表查询语句
```SQL
select no, student_name, age from student;
select age+5 from student;
select no, 3+5 from student;
select 10*2, 3*5+2;
select * from student;
```
2. 过滤条件查询(where)
```SQL
select * from student where age >= 15;
```
3. 排序(order by)
```SQL
select * from student order by age;

# 排序子句"order by"应该在"where"子句之后：
select * from student where age >= 15 order by age DESC;DESC 反序

select * from student order by age, student_name;
select * from student order by age desc;
select * from student order by age desc, student_name;
```
4. 分组查询(group by)
```SQL
select age, count(*) from student group by age;
```
使用"group by"语句时，需要使用聚合函数，常用的聚合函数为"count"、"sum"等。
5. 关联表查询(join)
```SQL
create table class (no int primary key, class_name varchar(40));
insert into class (no, class_name) values
  (1, '初二（1）班'),
  (2, '初二（2）班'),
  (3, '初二（3）班')，
  (4, '初二（4）班');

create table student (no int primary key, student_name, varchar(40), age int, class_no int);
insert into student (no, student_name, age, class_no) values
  (1, '张三', 14, 1),
  (2, '吴二', 15, 1),
  (3, '李四', 13, 2),
  (4, '吴三', 15, 2),
  (5, '王二', 15, 3),
  (6, '李三', 14, 3),
  (7, '吴二', 15, 4),
  (8, '张四', 14, 4);

select student_name, class_name from student, class where student.class_no = class.no;
select student_name, class_name from student a, class b where a.class_no = b.no; # 别名
select student_name, class_name from student a, class b where a.class_no = b.no and a.age > 14;
```

### 其他SQL语句
1. insert into ... select
该语句可以把数据从一张表插入到另一张表中。
```SQL
create table student_bak (no int primary key, student_name varchar(40), age int, class_no int);
insert into student_bak select * from student;
```
2. union
```SQL
select * from student where no = 1 union select * from student_bak where no =2;
# union会把相同的记录合并成一条，若不想合并，使用union all：
select * from student where no = 1 union all select * from student_bak where no =1;
```
3. truncate table
truncate table的用途是清空表内容。执行起来比delete快， delete是把数据一条条的删除，而truncate是直接把原先表的内容丢弃了重建一张表。
`truncate table student_bak`

## psql工具的使用介绍
### psql的简单使用
直接输入`psql`，安装PostgreSQL的数据库时，会建立一个与初始化数据库时的操作系统用户同名的数据库用户，该用户是数据库的超级用户，免密。可通过修改`pg_hba.conf`文件来要求输入密码。
`psql -l`查看有哪些数据库。
```SQL
\l # 查看数据库
\d # 列出当前数据库中的所有表
create database testdb;
\c testdb; # 连接到testdb
```
```bash
psql -h <hostname|ip> -p <port> [database_name] [user_name]

# 也可以直接用环境变量指定
export PGDATABASE=database_name
export PGHOST=ip
export PGPORT=port
export PGUSER=user_name
```
### psql的常用命令
- \d
\d [ pattern ]
\d [ pattern ] +

1. 什么都不带，将列出当前数据库中的所有表
2. 跟一表名，显示该表的结构定义
3. "表名_pkey"，显示该表的索引信息
4. 跟通配符`*`或`?`，多表查询
5. \d+ 显示比\d更详细的信息，包括与列表关联的注释，以及表中出现的OID。
6. 匹配不同对象类型的\d命令
只显示匹配的表： \dt
只显示索引： \di
只显示序列： \ds
只显示视图： \dv
只显示函数： \df
7. 显示SQL已执行的时间，\timing on
8. 列出所有的schema： \dn
9. 显示所有的表空间： \db
> 实际上PostgreSQL中的表空间就是对应一个目录，放在这个表空建的表，就是把表的数据文件放到这个表空间下。
10. 列出数据库中所有角色或用户：\du或\dg
> PostgreSQL数据库中用户和角色不分的。
11. \dp或\z命令用于显示表的权限分配情况

- 指定字符集编译的命令 `\enconding gbk;`, `\enconding utf8;`
- \pset 设置输出的格式
\pset border 0: 无边框
\pset border 1: 边框在内部
\pset border 2: 内外都有边框
- \x 把表中每一行的每列数据都拆分为单行展示
- \i <filename> 执行存储在外部文件中的sql语句或命令
也可以使用`psql -f <filename>`来执行SQL脚本文件中的命令。
- \echo 输出一行信息
- \? 显示更多命令用法

### psql使用技巧和注意事项
1. 补全：<tab><tab>
2. 自动提交方面的技巧
在psql中事务是自动提交的。如果不想自动提交：
1> 运行`begin;`命令，然后执行DML语句，最后在执行`commit;`或`rollback;`命令
2> 关闭自动提交功能 `\set AUTOCOMMIT off` (9.5.5不起作用)
3. 获得psql中命令实际执行的SQL
1> 启动psql的命令行中加"-E"参数，就可以把psql中各种以"\\"开头的命令执行的实际SQL打印出来。(`psql -E postgres`)
2> 在已运行的psql中，可以使用`\set ECHO HIDDEN on | off`命令

## 第五章 数据类型
### 类型介绍
#### 类型的分类：
1. 布尔类型(boolean)
2. 数值类型(整数类型有2字节smallint、4字节int、8字节bigint; 十进制精确类型有numeric; 浮点类型有real和double precision; 8字节的货币类型money)
3. 字符类型(varchar(n), char(n), text)
4. 二进制数据类型(bytea)
5. 位串类型(位串就是一串1和0的字符串，有bit(n)、bit varying(n))
6. 日期和时间类型(date、time、timestamp，而time和timestamp又分是否包括时区的两种类型)
7. 枚举类型(枚举类型是一种包含一系列有序静态值集合的数据类型，等于某些编程语言中的enum类型。使用时要先用`create type`创建这个类型)
8. 几何类型(点point、直线line、线段lseg、路径path、多边形polygon、圆cycle等类型)
9. 网络地址类型(cidr、inet、macaddr)
10. 数组类型
11. 复合类型
12. xml类型
13. json类型
14. range类型
15. 对象标识符类型(oid类型、regproc类型、regclass类型等)
16. 伪类型(不可作为字段的数据类型，可用于声明一个函数的参数或者结果类型。有any、anyarray、anyelement、cstring、internal、language_handler、record、trigger、void、opaque)
17. 其他类型(UUID类型、pg_lsn类型)
#### 类型输入与转换
简单的类型：
```SQL
select 1, 1.1421, 'hello world';
```
复杂类型：使用“类别名”加上单引号括起来
```SQL
select bit '11110011';
select int '1' + int '2';
```
使用类型转换函数`CAST`和双冒号`::`进行类型转换：
```SQL
select cast('5' as int), cast('2014-07-17' as date);
select '5'::int, '2014-07-17'::date;
```
### 布尔类型
#### 布尔类型解释
boolean在SQL中可以用不带引号的TRUE、FALS表示，也可以用更多的表示真和假的带引号的字符表示，如'true'、'false'、'yes'、'no'等等。'unknown'(未知)状态用NULL表示
```SQL
create table t (id int, col1 boolean, col2 text);
insert into t values (1, true, 'true');
insert into t values (2, false, 'falser');
insert into t values (3, 'true', '''true''');
insert into t values (4, 'false', '''false''');
insert into t values (5, 't', '''t''');
insert into t values (6, 'f', '''f''');
insert into t values (7, 'y', '''y''');
insert into t values (8, 'n', '''n''');
insert into t values (9, 'yes', '''yes''');
insert into t values (10, 'no', '''no''');
insert into t values (11, '1', '''1''');
insert into t values (12, '0', '''0''');
select * from t;
select * from t where col1;
select * from t where not col2;
```
#### 布尔类型的操作符 and，or， not， is
布尔类型可以使用"IS"比较运算符：
```
expression is true
expression is not true
expression is false
expression is not false
expression is unknown
expression is not unknown
```
### 数值类型
#### 数值类型解释
| 类型名称         | 存储空间         |    范围           |
| :-------------: | :-------------: | :-------------: |
| smallint        | 2字节           | $-2^{15} \sim 2^{15}-1$ |
| int 或 integer  | 4字节           | $-2^{31} \sim 2^{31}-1$ |
| bigint          | 8字节           | $-2^{63} \sim 2^{63}-1$ |
| numeric 或 decimal | 变长         | 无限制                   |
| real            | 4字节           | 6位十进制数字精度        |
| double precision | 8字节          | 15位十进制数字精度       |
| serial          | 4字节           | $1 \sim 2^{31}-1$      |
| bigserial       | 8字节           | $1 \sim 3^{63}-1$      |
#### 整数类型
PostgreSQL中没有MySQL中的tinyint(1字节)、mediumint(3字节)、unsigned类型。
#### 精确的小数类型
精确的小数类型可用numeric、numeric(m,n)、numeric(m)表示。其中，numeric与decimal是等效的，两种类型都是SQL标准，可以存储最多1000位精度的数字，并且可以进行准确的计算。这个类型特别适合用于货币金额和其他要求精确计算的场合。不过运算比浮点数慢很多。
`numeric(precision, scale)`，其中，精度preci必须为正数，标度scale可以为零或正数。如果关心移植性，最好总是声明精度和标度。
```SQL
create table t1 (id1 numeric(3), id2 numeric(3,0), id3 numeric(3,2), id4 numeric);
insert into t1 values (3.1, 3.5, 3.123, 3.123);
```
超过标度，四舍五入；超过精度，报错。

#### 浮点数类型
数据类型real和double precision是不精确的、变精确的数字类型。浮点类型的特殊值：'Infinity', '-Infinity', 'NaN'。在SQL命令里把这些数值当作常量时，必须在他们周围放上单引号，像：`update table set x = 'Infinity'`。输入时，这些值是大小写无关的。
#### 序列类型
```SQL
create table t (
  id serial
);
```
相当于：
```SQL
create sequence t_id_seq;
create table t (
  id integer not null default nextval('t_id_seq')
);
alter sequence t_id_seq owned by t.id;
```
#### 货币类型
货币类型(money)可以存储固定小数的货币数目，与浮点数不同，它是完全保证精度的。其输出格式与参数`lc_monetary`的设置有关。
```SQL
select '12.34'::money;
show lc_monetary;
set lc_monetary = 'zh_CN.UTF-8';
# set lc_monetary = 'en_US.UTF-8';
```
#### 数学函数和操作符
| 操作符/函数 | 描述     |
| :-------------: | :-------------: |
| + | 加 |
| - | 减 |
| * | 乘 |
| / | 除 |
| % | 模（求余） |
| ^ | 幂 |
| \|/ | 平方根 |
| \|\|/ | 立方根 |
| ! | 阶乘 |
| !! | 阶乘（前缀操作符） |
| @ | 绝对值 |
| & | 二进制AND |
| \| | 二进制OR |
| # | 二进制XOR |
| ~ | 二进制NOT |
| << | 二进制左移 |
| >> | 二进制右移 |
| abs(x) | 绝对值 |
| cbrt(dp) | 立方根 |
| ceil(dp/numeric) | 不小于参数的最小整数 |
| degree(dp) | 把弧度转为角度 |
| exp(dp/numeric) | 自然指数 |
| floor(dp/numeric) | 不大于参数的最大整数 |
| ln(dp/numeric) | 自然对数 |
| log(dp/numeric) | 以10为底的对数 |
| log(b numeric, x numeric) | 以b为底的对数 |
| mod(y,x) | y/x的余数（模） |
| pi() | "π"常量 |
| power(a dp, b dp) | a的b次幂 |
| power(a numeric, b numeric) | a的b次幂 |
| radians(dp) | 把度数转为弧度 |
| random() | 0.0到1.0之间的随机数 |
| round(dp/numeric) | 圆整为最近的整数（四舍五入） |
| round(v numeric, s int) | 圆整为s位小数（四舍五入） |
| setseed(dp) | 为随后的random()调用设置种子（0.0到1.0之间） |
| sign(dp/numeric) | 参数符号（-1,0,+1） |
| sqrt(dp/numeric) | 平方根 |
| trunc(dp/numeric) | 截断（向零靠近） |
| trunc(dp/numeric) | 截断为s位小数 |
| width_bucket(op numeric, b1 numeric, b2 numeric, count int) | 返回一个桶，在一个有count个桶、上界为b1、下界为b2的等深柱图中，operand将被赋予的就是这个桶 |
| acos(x) | 反余弦 |
| asin(x) | 反正弦 |
| atan(x) | 反正切 |
| atan2(x,y) | x/y的反正切 |
| cos(x) | 余弦 |
| cot(x) | 余切 |
| sin(x) | 正弦 |
| tan(x) | 正切 |

### 字符串类型
varchar(n)和char(n)分别是character varying(n)和character(n)的别名，没有声明长度的character等于character(1)，没有声明长度的character varying接受任何长度的字符串。
#### 字符串函数和操作符
| 操作符/函数 | 描述    |
| :--------: | :-----: |
| string \|\| string | 字符串连接 |
| bit_length(string) | 字符串里二进制位的个数 |
| char_length(string) 或 character_length(string) | 字符串中的字符个数 |
| convert(string using conversion_name) | 使用指定的转换名字改变编码。转换可以通过create conversion定义。当然，系统里有一些预定义的转换名字 |
| lower(string) | 把字符串转化为小写 |
| octet_length(string) | 字符串中的字节数 |
| overlay(string placing string from int [for int]) | 替换子字符串：overlap('Txxxxas' placing 'hom' from 2 for 4) $\rightarrow$ Thomas |
| position(substring in string) | 指定的子字符串的位置 |
| substring(string [from int] [for int]) | 抽取子字符串 |
| substring(string from pattern) | 抽取匹配POSIX正则表达式的子字符串：substring('Thomas' from '...\$') $\rightarrow$ mas |
| substring(string from pattern for escape) | 抽取匹配SQL正则表达式的子字符串：substring('Thomas' from '%#"o_a"_' for '#') $\rightarrow$ oma |
| trim([leading \| trailing \| both] [characters] from string) | 从字符串string的开头/结尾/两边删除只包含characters中字符（默认是一个空白）最长的字符串： trim(both 'x' from 'xThimxx') $\rightarrow$ Tom |
| upper(string) | 把字符串转化为大写 |
| ascii(string) | 参数第一个字符的ASCII码 |
| btrim(string text [, characters text]) | 从string开头和结尾删除包含在参数characters中字符，直到遇到一个不是在characters中的字符串为止，参数characters的默认值为空格：btrim('aaosdbaaa', 'aa') $\rightarrow$ osdb |
| chr(int) | 给出ASCII码的字符 |
| convert(string text, [src_encoding name], dest_encoding name) | 把原来编码为src_encoding的字符串转化为dest_encoding编码（如果省略了src_encoding，将使用数据库编码） |
| decode(string text, type text) | 把早先用encode编码的string里面的二进制数据解码 |
| encode(data bytea, type text) | 把二进制数据编码为只包含ASCII形式的数据。支持的类型有：base64、hex、escape。 |
| initcap(string) | 把每个单词的第一个字母转为大写，其余保留小写。单词是一系列字母数字组成的字符时，用非字母数字分隔。 |
| length(string) | string中字符的数目 |
| lpad(string text, length int [, fill text]) | 通过填充字符fill（默认为空白），把string填充为length长度。如果string已经比length长，则将其尾部截断。 |
| ltrim(string text [, characters text]) | 从string的开头删除包含在参数characters中的字符，值到遇到一个不是在characters中的字符为止 |
| md5(string) | 计算string的MD5散列，以十六进制返回结果 |
| pg_client_encoding() | 当前客户端编码名称 |
| quote_ident(string) | 返回使用于SQL语句的标识符形式（使用适当的引号进行界定）。只有在必要的时候会添加引号（字符串包含非标识符字符或者会转换大小写的字符）。嵌入的引号会被恰当地写双份。 |
| quote_literal(string) | 返回适用于在SQL语句里当作文本的形式。嵌入的引号和反斜杠被恰当地写了双份：quote_literal('O\\'Reilly') $\rightarrow$ 'O"Reilly' |
| regexp_replace(string text, pattern text, replacement text [, flags text]) | 替换匹配POSIX正则表达式的子字符串 |
| repeat(string text, from text, to text) | 将string重复number次 |
| replace(string text, from text, to text) |  把string里出现的所有子字符串from替换成字字符串to |
| rpad(string text, length int [, fill text]) | 通过填充字符fill（默认为空白），把string填充为length长度。如果string已经比length长，则将其尾部截断。 |
| rtrim(string text [, characters text]) | 从string的结尾删除包含在参数characters中的字符，值到遇到一个不是在characters中的字符为止 |
| split_part(string text, delimiter text, field int) | 根据delimiter分隔string返回生成的第field个子字符串（1为基） |
| strpos(string, substring) | 指定的子字符串的位置。和position(substring in string)一样，不过参数顺序相反 |
| substr(string, from [, count]) | 抽取子字符串 |
| to_ascii(string text [, encoding(text)]) | 把string从其他编码转换为ASCII（仅支持LATIN1、LATIN2、LATIN9、WIN1250编码）|
| to hex(number int/bigint) | 把number转化成16进制表现形式 |
| translate(string text, from text, to text) | 把在string中包含的任何与from中字符串匹配的字符转化为对应的在to中的字符：translate('12345', '14', 'db') $\rightarrow$ d23b5 |

### 二进制数据类型
#### 二进制数据类型解释
PostgreSQL中的bytea对应MySQL和Oracle中的blob类型。Oracle的raw类型也可以使用这个类型取代。
#### 二进制数据类型转义表示
要转义一个字节值，通常需要把它的数值转换成对应的三位八进制数，并且加两个前导反斜杠。有些八进制数值可以加一个反斜杠直接转义，比如单引号和反斜杠本身。
#### 二进制数据类型的函数 P68-P69

### 位串类型
#### 解释
位串就是一串1和0的字符串。在PostgreSQL中可以直观显式地操作二进制位。包括bit(n)、bit varying(n)，没有长度的bit等效于bit(1)，没有长度的bit varying表示没有长度限制。
如果明确地把一个位串值转换成bit(n)， 那么它的右边将被截断，或者在右边补齐0到刚好为n位，而不会抛出错误。bit varying(n) 类似。
#### 使用
```SQL
create table test (a bit(3), b bit varying(5));
insert into test values (b'101', b'00');
insert into test values (b'11110', b'101'); # 超长报错
```
#### 操作符及函数
| 操作符 | 描述 |
| :-------------: | :-------------: |
| \|\| | 连接 |
| & | 位与 |
| \| | 位或 |
| # | 异或 |
| ~ | 位非 |
| << | 左移：b'1101' << 3 $\rightarrow$ 1000 |
| >> | 右移：b'1101' >> 3 $\rightarrow$ 0001 |
函数：length, bit_length, octet_length, position, substring, overlay, get_bit, set_bit。
可以在整数和bit之间， 十进制、十六进制、二进制之间的转换：
```SQL
select 'xff'::bit(8)::int;
select to_hex(255);
```
### 日期/时间类型
#### 解释
timestamp [ (p) ] [ without time zone ]
timestamp [ (p) ] with time zone
interval [ (p) ]
date
time [ (p) ] [ without time zone ]
time [ (p) ] with time zone
p为精度值，用以指明秒域中小数部分的位数。如果没有明确的默认精度。对于timestamp和interval类型，p的范围是0~6。
timestamp数值是以双精度浮点数的方式存储的。timestamp值是以2000-01-01午夜之前或之后的秒数存储的。
#### 日期输入
如果DateStyle参数默认为"MDY"，则表示被“月 - 日 - 年”解析；如果参数设置为"DMY"，则按照“日 - 月 - 年”解析；设置为"YMD"，按照“年 - 月 - 日”解析。
```SQL
create table t (col1 date);
insert into t values (date '12-10-2010');
show datestyle;
set datestyle="YMD"
```
| 例子 | 描述 |
| :-------------: | :-------------: |
| date'April 26, 2011' | 在任何datestyle输入模式下都无歧义 |
| date'2011-01-08' | ISO 8601格式（建议格式），任何方式下都是2011年1月8号， 而不会是2011年8月1日 |
| date'1/8/2011' | 有歧义，“MDY”：2011年1月8日，“DMY”：2011年8月1日 |
| date'1/18/2011' | “MDY”：2011年1月18日，其他模式下被拒绝 |
| date'03/04/11' | “MDY”：2011年3月4日，“DMY”：2011年4月3日， “YMD”：2003年4月11日 |
| date'2011-Apr-08', date'Apr-08-2011', date'08-Apr-2011' | 2011年4月8日 |
| date'19980405' | ISO 8601格式，1998年4月5日 |
| date'110405' | ISO 8601格式， 2011年4月5日 |
| date'2011.062' | 2011年的第62天，即2011年3月3日 |
| date'J2455678' | 儒略日，即从公元前4713年1月1日起到今天过去的天数，多为天文学家使用。2455678天，即2011年4月26日 |
| date'April 26,202BC' | 公元前4月26日 |
#### 时间输入
time被认为是time without time zone的类型，这样即使字符串中有时区，也会被忽略：
```SQL
select time '04:05:06';
select time '04:05:06 PST';
select time with time zone '04:05:06 PST'
```
时间字符串可以用冒号作为分隔符，即输入格式为"hh:mm:ss"，也可以不用分隔符。
| 例子 | 描述 |
| :-------------: | :-------------: |
| time '22:55:06.789', time '22:55:06', time '22:55', time '225506' | ISO 8601 |
| time with time zone '22:55:06.789+8', '22:55:06+08:00', '225506+08' | ISO 8601 |
| time with time zone '22:55:06 CCT' | 缩写的时区 |
| select time with time zone '2003-04-12 04:05:06 Asia/Chongqing';| 用名字声明时区 |
注意，最好不要用时区缩写来表示时区。PostgreSQL中的时区缩写可以查询PostgreSQL中的时区缩写可以查询pg_timezone_abbrevs。
#### 特殊值
epoch, infinity, -infinity, now, today, tomorrow, yesterday, allballs
#### 函数和操作符
日期、时间和interval类型之间可以进行加减乘除运算。P75-76，表5-18。
函数：
| 函数 | 描述 |
| :-------------: | :-------------: |
| age(timestamp, timestamp) | 参数相减后的“符号化”结果 |
| age(timestamp) | 从current_date减去参数后的结果 |
| clock_timestamp | 实时时钟的当前时间戳 |
| current_date | 当前日期 |
| current_time | 当前时间 |
| current_timestamp | 当前事务开始时的时间戳 |
| date_part(text, timestamp/interval) | 获取子域（等效于extract）：date_part('hour', timestamp '2001-02-16 20:38:40') → 20 |
| date_trunc(text, timestamp) | 截断成指定的精度：date_trunc('hour', timestamp '2001-02-16 20:38:40') → 2001-02-16-20:00:00 |
| extract(field from timestamp/interval) | 获取子域 |
| isfinite(timestamp/interval) | 测试是否为有穷时间戳/隔 |
| justify_days(interval) | 按每月30天调整时间间隔：justify_days(interval '30 days') → 1 month |
| justify_hours(interval) | 按每天24小时调整时间间隔：justify_hours(interval '24 hours') → 1 day |
| justify_interval(interval) | 使用justify_days和justify_hours调整时间间隔的同时进行正负号调整：justify_interval(interval '1 mon - 1 hour') → 29 days 23:00:00 |
| localtime | 当日时间 |
| localtimestamp | 当前事务开始的时间戳 |
| now() | 当前事务开始时的时间戳 |
| statement_timestamp() | 实时时钟的当时时间戳 |
| timeofday() | 与clock_timestamp相同，但结果是一个text字符串 |
| transaction_timestamp() | 当前事务开始时的时间戳 |
除了这些函数以外，还支持SQL的overlaps操作符：
(start1, end1) overlaps (start2, end2)
(start1, length1) overlaps (start2, length2)
这个表达式在两个时间域重叠的时候生成真值。
#### 时间函数
current_time 和 current_timestamp 返回带有时区的值，localtime 和 localtimestamp 返回不带时区的值。这四个函数都可以接受一个精度参数，该精度会导致结果的秒数域四舍五入到指定的小数位。如果没有精度参数，将给予所能得到的全部精度。这些函数全部都是按照当前事务的开始时刻返回结果的，所以它们的值在事务运行的整个期间内都不会改变。例如在`bigin;`，`end;`事务中，这些函数的调用值都保持相同。
statement_timestamp()，clock_timestamp()，timeofday()这些函数在事务中返回实时时间值。
所有日期/时间类型还接受特殊的文本值now，用于声明当前的日期和时间（乃当前事务的开始时刻）。因此，下面三个都返回相同的结果：
```SQL
select current_timestamp;
select now();
select timestamp with time zone 'now';
```
```SQL
extract(field from source) # return double precision
```
field 的值：century、year、decade、millennium、quarter、month、week、dow[求日期是星期几，0为星期天]、day[本月的第几天]、doy[本年的第几天]、hour、minute、second、epoch[对于date和timestamp，是自1970-01-01 00:00:00以来的秒数（可为负）；对interval，它是时间间隔的总秒数]、milliseconds[秒域乘以1000]、microseconds、timezone、timezone_hour、timezone_minute。

### 枚举类型
#### 枚举类型的使用
在PostgreSQL中，使用枚举类型需要先使用create type创建一个枚举类型。
```SQL
create type week as enum ('Sun', 'Mon', 'Tues', 'Wed', 'Thur', 'Fri', 'Sat');
create table duty (person text, weekday week);
insert into duty (person, weekday) values
  ('张三', 'Sun'),
  ('李四', 'Mon'),
  ('王二', 'Tues'),
  ('赵五', 'Wed');
select * from duty where weekday = 'Sun';
# 输入的字符不在枚举类型之间，则会报错：
select * from duty where weekday = 'Sun.';
# 使用“\dT”命令查看枚举类型的定义：
\dT+ week
# 直接查询表pg_enum也可以看到枚举类型的定义：
select * from pg_enum;
```
#### 枚举类型的说明
在枚举类型中，值得顺序是创建枚举类型时定义的顺序。所有比较标准的运算符及其他相关的聚集函数都可支持枚举类型：
```SQL
select min(weekday), max(weekday) from duty;
select * from duty where weekday = (select max(weekday) from duty);
```
一个枚举值占4字节。一个枚举值的文本标签长度由namedatalen设置并编译到PostgreSQL中，且是以标准编译方式进行的，也就意味着至少是63字节。枚举类型的值是大小写敏感的。
#### 枚举类型的函数
| 函数 | 描述 |
| :-------------: | :-------------: |
| enum_first(anyenum) | 返回枚举类型的第一个值：enum_first('Mon'::week) → Sun |
| enum_last(anyenum) | 返回枚举类型的最后一个值 |
| enum_range(anyenum) | 以一个有序的数组形式返回输入枚举类型的所有值 |
| enum_range(anyenum, anyenum) | 以一个有序的数组返回在给定的两个枚举值之间的范围。若第一个为空，从第一个开始；若第二个为空，到最后一个结束：enum_range(null, 'Wed'::week) → {Sun, Mon, Tues, Wed} |
除了两个参数形式的enum_range外，其余这些函数会忽略传递给它们的具体值，使用null加上类型转换也将得到相同的结果。

### 几何类型
#### 几何类型概况
| 类型名称 | 描述 | 表现形式 |
| :----: | :----: | :----: |
| point | 平面中的点 | (x,y) |
| line | 直线 | ((x1,y1),(x2,y2)) |
| lseg | 线段 | ((x1,y1),(x2,y2)) |
| box | 矩形 | ((x1,y1),(x2,y2)) |
| path | 闭合路径 | ((x1,y1),...) |
| path | 开放路劲 | [(x1,y1),...] |
| polygon | 多边形 | ((x1,y1),...) |
| circle | 圆 | <(x,y),r> |
#### 几何类型的输入
`类型名称 '表现形式'` or `'表现形式'::类型名称`
```SQL
select '1,1'::point;
select '(1,1)'::point;
select lseg '1,1,2,2';
select lseg '(1,1),(2,2)';
select lseg '((1,1),(2,2))';
select lseg '[(1,1),(2,2)]';
select path '(1,1),(2,2)'; -- 默认为闭合
select box '1,1,2,2';
select box '(1,1),(2,2)';
select box '((1,1),(2,2))';
select circle '1,1,5';
select circle '((1,1),5)';
select circle '<(1,1),5>';
```
#### 几何类型的操作符
1. 平移运算符`+`、`-`及缩放/旋转运算符`*`、`/`
这四个运算符都是二元运算符，运算符左值可以是`point`、`box`、`path`、`circle`，运算符的右值只能是`point`：
```SQL
-- 点与点的加减乘除相当于两个复数之间的加减乘除
select point '(1,2)' + point '(10,20)'; -- (11,22)
select point '(1,2)' - point '(10,20)'; -- (-9,-18)
select point '(1,2)' * point '(10,20)'; -- (-30,40)
select point '(1,2)' / point '(10,20)'; -- (0.1,0)
-- 矩形与点之间的运算：
select box '((0,0),(1,1))' + point '(2,2)'; -- (3,3),(2,2)
select box '((0,0),(1,1))' - point '(2,2)'; -- (-1,-1),(-2,-2)
select box '((0,0),(1,1))' * point '(2,2)'; -- (0,4),(0,0)
-- 路径与点之间的运算：
select path '(0,0),(1,1),(2,2)' + point '(10,20)'; -- (10,20),(11,21),(12,22)
select path '(0,0),(1,1),(2,2)' - point '(10,20)'; -- (-10,-20),(-9,-19),(-8,-18)
-- 圆与点之间的运算：
select circle '((0,0),1)' + point '10,20'; -- <(10,20),1>
select circle '((0,0),1)' - point '10,20'; -- <(-10,-20),1>
-- 对于乘法，如果乘数的y值为0，比如point 'x,0'，则相当于集合对象缩放x 倍：
select circle '((0,0),1)' * point '(3,0)'; -- <(0,0),3>
select circle '((1,1),1)' * point '(3,0)'; -- <(3,3),3>
-- 如果乘数为point '0,1'，则相当于几何图像逆时针旋转90度，如果乘数为point '0,-1'，则表示顺时针旋转90度：
select circle '((1,1),1)' * point '(0,1)'; -- <(-1,1),1>
```
2. 运算符 `#`
对于两个线段，计算出交点；对于两个矩形，计算出相交的矩形；对于路径或多边形，计算出顶点数。
```SQL
select lseg '(0,0),(2,2)' # lseg '(0,2),(2,0)'; -- (1,1)
-- 如果两个线段没有相交，则返回空。
-- 两个矩形：
select box '(0,0),(2,2)' # box '(1,0),(3,1)'; -- (2,1),(1,0)
-- 路径或多边形
select # path '(1,1),(2,2),(3,3)'; -- 3
select # polygon '(1,1),(2,2),(3,3)'; -- 3
```
3. 运算符 `@-@`
此为一元运算符，参数只能为`lseg`、`path`。一般用于计算几何对象的长度：
```SQL
select @-@ lseg '(0,0),(1,1)'; -- 1.41421235623731
select @-@ path '(0,0),(2,2)'; -- 5.65685424949238
select @-@ path '(0,0),(1,1),(2,2)'; -- 5.65685424949238
select @-@ path '[(0,0),(1,1),(0,1)]'; -- 2.41421235623731
select @-@ path  '((0,0),(1,1),(0,1))'; -- 3.41421235623731
```
4. 运算符 `@@`
一元运算，用于计算中心点：
```SQL
select @@ circle '<(1,1),2>';
select @@ box '(0,0),(1,1)'; -- (0.5,0.5)
select @@ lseg '(0,0),(1,1)'
```
5. 运算符 `##`
二元运算符，用于计算两个几何对象上离得最近的点：
```SQL
select point '(0,0)' ## lseg '(2,0),(0,2)'; -- (1,1)
select point '(0,0)' ## box '(1,1),(2,2)'; -- (1,1)
select lseg '(1,0),(0,1.5)' ## lseg '((2,0),(0,2))'; -- (0,1.5)
```
6. 运算符 `<->`
二元运算符，用于计算两个几何对象之间的距离：
```SQL
select lseg '(0,1),(1,0)' <-> lseg '(0,2),(2,0)';
select circle '((0,0),1)' <-> circle '((3,0),1)'; -- 1
-- 对于两个矩形，实际上是中心点之间的距离：
select box '((0,0),(1,1))' <-> box '((2,0),(4,1))'; -- 2.5
```
7. 运算符 `&&`
二元运算符，用于计算两个几何对象之间是否重叠，只要有一个共同点，则为真：
```SQL
select box '((0,0),(1,1))' &&  box '((1,1),(2,2))'; -- t
select box '((0,0),(1,1))' && box '((2,2),(3,3))'; -- f
select circle '((0,0),1)' && circle '((1,1),1)'; -- t
select polygon '(0,0),(2,2),(0,2)' && polygon '(0,1),(1,1),(2,0)'; -- t
```
8. 判断两个对象相对位置的运算符
判断左右：
`<<`: 是否严格在左
`>>`: 是否严格在右
`&<`: 没有延展到右边
`&>`: 没有延展到左边
判断上下：
`<<|`: 严格在下
`|>>`: 严格在上
`&<|`: 没有延展到上面
`|&>`: 没有延展到下面
`<^`: 在下面（允许接触）
`>^`: 在上面（允许接触）
其他：
`?#`: 是否相交
`?-`: 是否水平或水平对齐
`?|`: 是否竖直或数值对齐
`?-|`: 两个对象是否垂直
`?||`: 两个对象是否平行
`@>`: 是否包含
`<@`: 包含或在其上
```SQL
select box '((0,0),(1,1))' << box '((1.1,1.1),(2,2))'; -- t
select polygon '(0,0),(0,1),(1,0)' << polygon '(0,1.1),(1.1,1),(1.1,0)'; -- f
select polygon '(0,0),(0,1),(1,0)' << polygon '(1.1,0),(1.1,1),(2,0)'; -- t
select circle '((0,0),1)' << circle '((1,1),1)'; -- f
select circle '((0,0),1)' << circle '((3,3),1)'; -- t
```
9. 判断两个几何对象是否相同的运算符 `~=`
```SQL
select polygon '((0,0),(1,1))' ~= polygon '((1,1),(0,0))'; -- t
select polygon '(0,0),(1,1),(1,0)' polygon '(1,1),(0,0),(1,0)'; -- t
select box '(0,0),(1,1)' ~= box '(1,1),(0,0)'; -- t
```
#### 几何类型的函数
| 函数 | 描述 |
| :----: | :----: |
| area(object) | 面积 |
| center(object) | 中心 |
| diameter(circle) | 直径 |
| height(box) | 高度 |
| width(box) | 宽度 |
| isclosed(path) | 是否闭合 |
| isopen(path) | 是否开放 |
| length(object) | 长度 |
| npoints(path/polygon) | 点数 |
| pclose(path) | 把路径转成闭合路径 |
| popen(path) | 把路径转成开放路径 |
| radius(circle) | 半径 |

转换函数：box(circle)、box(point,point)、box(polygon)、 circle(box)、circle(point, double precision)、circle(polygon)、lseg(box)、lseg(point,point)、path(polygon)、point(double precision, double precision)、point(box)、point(circle)、point(lseg)、point(polygon)、polygon(box)、polygon(circle)、polygon(npts,circle)、polygon(path)。

### 网络地址类型
PostgreSQL提供专门数据类型存储IPv4、IPv6和MAC地址。
| 类型 | 存储空间 | 描述 |
| :----: | :----: | :----: |
| cidr | 7或19字节 | IPv4或IPv6的网络地址 |
| inet | 7或19字节 | IPv4或IPv6的网络地址或主机地址 |
| macaddr | 6 字节 | 以太网MAC地址 |
> refer to P98-P103

### 复合类型
在PostgreSQL中可以如C语言中的结构体一样定义一个符合类型。
#### 复合类型的定义：
```SQL
create type complex as (
  r double precision,
  i double precision
);
create type person as (
  name text,
  age integer,
  sex boolean
);
-- 目前不能声明约束（比如 not null）
create table capacitance_test_data (
  test_time timestamp,
  voltage complex,
  current complex
);
-- 可使用此类型作为函数参数：
create function complex_multi(complex, complex) returns complex
  as $$ select row($1.r*$2.r - $1.i$2.i, $1.r*$2.i - $1.i*$2.r)::complex as language sql;
```
#### 复合类型的输入
`( val1, val2, ...)` 单引号加圆括号的形式。在此格式中，可以在任何字段值周围放上双引号，如果值本身包含逗号或圆括弧，则必须用双引号括起。要让一个字段值是null，那么在列表里它的位置上就不要写任何字符。要一个空字符，则写一对双引号。也可以用row表达式语法来构造复合类型值。表达式超过一个字段，关键字row可省略。
```SQL
create type person as (
  name text,
  age integer,
  sex boolean
);
create table author (
  id int,
  person_info person,
  book text
);
insert into author values (1, '("张三", 29, true)', '张三的自传');
insert into author values (2, row('李四',29, true), '自传');
insert into author values (3, ("王五", 28, true), '自传');
```
#### 访问复合类型
访问复合类型字段的一个域就如在C语言中访问结构体中的一个成员一样，即写出一个点和域的名字就可以了。
```SQL
select person_info.name from author; -- error
select (person_info).name from  author;
select (author.person_info).name from author;
select (my_func(...)).field from ...
```
#### 修改复合类型
```SQL
-- 更新整个字段：
insert into author values (('张三', 29, true), '自传');
update author set person_info = row('李四', 39, true) where id = 1;
update author set person_info = ('王二', 49, true) where id = 2;
-- 更新眸子子域：
update author set person_info.name = '王二二' where id =2;
update author set person_info.age = (person_info).age + 1 where id = 2;
-- 不能在set后面出现的字段名周围加上圆括号，但需要在等号右边的表达式中引用同一个字段则需加上圆括号，否则会报错。在insert也可以指定复合字段的子域，未指定子域用null填充：
insert into author (id, person_info.name, person_info.age) values (10, '张三', 29);
```
#### 复合类型的输入与输出
P106

### XML类型
要使用xml数据类型，在编译PostgreSQL源码时必须使用`--with-libxm`参数。
#### xml类型的输入
格式： xml '<tag>hello world</tag>' 或 '<tag>hello world</tag>'::xml类型
xml中存储的xml数据有两种：
- 由xml标准定义的“documents”。
- 由xml标准定义的“content”片段。
“content”片段可以有多个顶级元素或“character”节点，但“documents”只能有一个顶级元素。可以使用“xmlvalue is document”来判断一个特定的xml值是一个“documents”还是“content”片段。
PostgreSQL的xmloption参数用来指定输入的数据是“documents”还是“content”片段，默认情况下此值为“content”片段。修改成“document”将不能输入有多个顶级元素的内容：
```SQL
show xmloption;
select xml '<a>a</a><b>b</b>';
set xmloption to document;
select xml '<a>a</a><b>b</b>'; -- error
```
也可以由函数xmlparse通过字符串数据来产生xml类型的数据，使用xmlparse函数是SQL标准中将字符串转换成XML的唯一方式。语法如下：
xmlparse ( ( document | content) value)
其中参数“document”和“content”表示指定XML数据的类型：
```SQL
select xmlparse (document '<?xml version="1.0"?><person><name>John</name><sex>F</sex></person>');
select xmlparse (content '<person><name>John</name><sex>F</sex></person>');
```
#### 字符集的问题
提交输入到xml类型的字符串中的编码声明会被忽略掉，同时内容的字符集会被认为是当前数据库服务器的字符集。
正确处理XML字符集的方式是，先将XML数据的字符串在当前客户端中编码成当前客户端的字符集，然后再发送到服务端，这样会被转换成服务器的字符集存储。当查询xml类型的值时，数据又会被转换成客户端字符集，所以客户端收到的xml数据的字符集就是客户端的字符集。
通常xml数据都是用UTF-8编码格式处理的，因此把PostgreSQL数据库服务器端编码也设置成UTF-8将是一种比较好的选择。
#### xml类型的函数
| 函数 | 描述 | 例子 | 结果 |
| :----: | :----: | :----: | :----: |
| xmlcomment(text) | 创建一个包含XML注释的特定文本内容的值。文本中不能包含“--”或不能以“-”结束。如果参数为空，结果也为空。 | xmlcomment('hello') | \<!--hello--> |
| xmlconcat(xml[, ...]) | 把XML值列表拼接成XML“content”片段。忽略列表中的空值，只有当参数都为空时结果才是空 | xmlconcat('\<a/>', '\<os>linux\</os>') | \<a/>\<os>linux\<os/> |
| xmlelement(name, name [, xmlattributes (value [as attname][, ...])][, content, ...]) | 生成一个带有给定名称、属性和内容的xml元素 | xmlelement(name linux, xmlattributes('2.6.18' as version)) | \<linux version="2.6.18"/> |
| xmlforest(content [as name][, ...]) | 把指定的名称和内容元素生成为一个XML“森林” | xmlforest('2.6' as linux, 5.0 as vers) | \<linux>2.6\</linux>\<vers>5.0\</vers> |
| xmlpi(name target[, content]) | 创建一条XML处理指令。内容不能包含字符序列“?>” | xmlpi(name php, 'echo "hello world";') | \<php echo "hello world";?> |
| xmlroot(xml, version text \| no value[, standalone yes\|no\|no value]) | 更改root节点的属性。如果指定version，它将替换root节点的version值，如果指定一个standalone，它替换根节点的standalone值 | xmlroot(xmlparse(document '\<?xml version="1.1" standalone="no"?>\<content>abc\</content>'), version '1.0', standalone yes) |
| xmlagg(xml) | 聚合函数，把多行的xml数据聚合成一项 | create table test (i int, d xml); insert into test values (1, '\<a>a\</a>'); insert into test values (2, '\</b>'); select xmlagg(d) from test; | \<a>a\</a>\<b/> |
PostgreSQL提供了xpath函数来计算XPath1.0表达式的结果。XPath是W3C的一个标准，它最主要的目的是在XML1.0或XML1.1文档节点树中定位节点。目前有XPath1.0和XPath2.0两个版本。其中，XPath1.0是1990年成为W3C标准的，而XPath2.0标准的确立是在2007年。
xpath函数定义如下：
xapth(xpath, xml[, nsarray])
此函数的第一个参数是XPath1.0表达式，第二个参数是xml类型的值，第三参数是命名空间的数组映射。这个数组应该是一个两维数组，第二维的长度等于2，即包含2个元素。
示例：
```SQL
select xpath('/my/text()', '<my:a xmlns:my="http://example.com">test</my:a>', array [array['my', 'http://example.com']]);
select xpath('//mydefns:b/text()', '<a xmlns="http://example.com"><b>test</b></a>', array[array['mydefns', 'http://example.com']]);
```
PostgreSQL还提供了把数据库中的内容导出成XML数据的一些函数：
- table_to_xmlschema(tbl regclass, nulls boolean, tableforest boolean, targetns text)
- query_to_xmlschema(query text, nulls boolean, tableforest boolean, targetns text)
- cursor_to_xmlschema(cursor refcursor, nulls boolean, tableforest boolean, targetns text)
- table_to_xml_and_xmlschema(tbl regclass, nulls boolean, tableforest boolean, targetns text)
- query_to_xml_and_xmlschema(query text, nulls boolean, tableforest boolean, targetns text)
- schema_to_xml(schema name, nulls boolean, tableforest boolean, targetns text)
- schema_to_xmlschema(schema name, nulls boolean, tableforest boolean, targetns text)
- schema_to_xml_and_xmlschema(schema name, nulls boolean, tableforest boolean, targetns text)
- database_to_xml(nulls boolean, tableforest boolean, targetns text)
- database_to_xmlschema(nulls boolean, tableforest boolean, targetns text)
- database_to_xml_and_xmlschema(nulls boolean, tableforest boolean, targetns text)
```SQL
create table test01(id int, note text);
insert into test01 select seq, repeat(seq::text, 2) from generate_series(1,5) as t(seq);
select table_to_xmlschema('test01'::regclass, true, true, 'tangspace');
-- table_to_xmlschema把表的定义转成了xml格式
select query_to_xmlschema('select * from test01', true, true, 'tangspace');
-- query_to_xmlschema把查询结果中行的定义转成了xml
select table_to_xml_and_xmlschema('test01'::regclass, true, true, 'tangspace');
-- table_to_xml_and_xmlschema与table_to_xmlschema唯一不同的地方在于把表中的数据也导入到xml中了。
select schema_to_xml('public', true, true, 'tangspace');
-- schema_to_xml把schema中的数据导成了xml格式。
```

### JSON类型
JSON：JavaScript Object Notation
#### JSON类型简介
JSON类型是把输入的数据原封不动地存放到数据库中，使用的时候需要重新解析数据，而JSONB类型是在存放的时候就把JSON解析成二进制格式了，使用的时候就不需要再次解析了，所以JSONB在使用时性能会更高。此外，JSONB支持在其上建索引。
JSONB类型不会保留多余的空格，不会保留key的顺序，也不会保留重复的key。
可以使用\uXXXX形式的转义，从而忽视数据库的字符集编码。
#### JSON类型的输入与输出
```SQL
select '9'::json, '"osdba"'::json, 'true'::json, 'null'::json;
select json '9', json '"osdba"', json 'true', json 'null';
-- 使用jsonb的类型也一样
select '[9, true, "osdba", null]'::json, jsonb '[9, true, "osdba", null]';
select json '{"name": "osdba", "age": 40, "sex": true, "money": 250.12}';
-- 对于输入带小数点的情况：
select json '{"p": 1.6735777674525e-27}'; -- {"p": 1.6735777674525e-27}
select jsonb '{"p": 1.6735777674525e-27}'; -- {"p": 0.0000000000000000000000000016735777674525}
```
#### JSON类型的操作符
| 操作符 | 右操作符类型 | 描述 | 例子 | 结果 |
| :---: | :---: | :---: | :---: | :---: |
| -> | int | 取JSON数组的元素 | '[1,2,3]'::json->1 | 2 |
| -> | text | 通过key取JSON中的子对象 | '{"a":1, "b":2}'::json->'a' | 1 |
| ->> | int | 取JSON数组的元素，但返回的是一个text类型 | '[1,2,3]'::json->>1 | '2' |
| ->> | text | 通过key取JSON中的子对象，但返回是一个text类型 | '{"a":1, "b":2}'::json->>'a' | '1' |
| #> | text[] | 通过指定路径取JSON中的对象 | '{"a":{"b":{"c":1}}}'::json#>'{a,b}' | {"c":1} |
| #>> | text[] | 同上，但返回的是一个text类型 | '{"a":{"b":{"c":1}}}'::json#>>'{a,b,c}' | '1' |
仅可用于JSONB的操作符：
| 操作符 | 描述 |     
| :---: | :---: |
| = | 两个JSONB对象的内容是否相同 |
| @> | 左边的JSONB对象是否包含右边的JSONB对象 |
| <@ | 左边的JSONB对象是否包含于右边的JSONB对象中 |
| ? | 指定的字符串是否存在于JSON对象中的key或字符串类型元素中。注意JSON中的元素需要时字符串类型 |
| ?\| | 右值是一个数组，指定此数组中的任意一个元素是否存在于JSON对象的字符串类型的key或元素中。注意JSON中的元素需要时字符串类型 |
| ?& | 右值是一个数组，指定此数组中的任意一个元素是否存在于JSON对象的字符串类型的key或元素中。注意JSON中的元素需要时字符串类型 |
#### JSON类型的函数
P118~P121
#### JSON 类型的索引
因为JSON类型没有提供相关的比较函数。所以在JSON类型的列上无法直接建索引，但可以在JSON类型的列上建函数索引。
JSONB类型的列上可以直接建索引。除了可以建BTree索引以外，JSONB还支持建GIN索引，GIN索引可以高效地从JSONB内部的key/value对中搜索数据。
在JSONB上创建GIN索引的方式有两种：
- 使用默认的jsonb_ops操作符创建
- 使用jsonb_path_ops操作符创建

create index idx_name on table_name using gin (index_col);
create index idx_name on table_name using gin (index_col jsonb_path_ops);
关于GIN索引，jsonb_ops的与jsonb_path_ops的区别为：在json_ops的GIN索引中，JSONB数据中的每个key和value都是作为一个单独的索引项的，而jsonb_path_ops则只为每个value创建了一个索引项。
```SQL
create table jtest01 (
  id int,
  jdoc json
);
create or replace function random_string(integer) returns text as
$BODY$
select array_to_string(
  array (
    select substring(
      '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz'
      from (ceil(random()*62))::int for 1
    )
    from generate_series(1, $1)
  ),
  ''
)
$BODY$
language sql volatile;
insert into jtest01 select t.seq, ('{"a":{"a1":"a1a1", "a2":"a2a2"}, "name":"'||random_string(10)||'", "b":"bbbbb"}'):: json from generate_series(1,100000) as t(seq);
-- 用函数json_extract_path_text建立一个函数索引：
create index on jtest01 using btree (json_extract_path_text(jdoc,'name'));
analyze jtest01;
-- 看看查询没有走索引的执行计划：
explain analyze verbose select * from jtest01 where jdoc->>'name'='lnBtcJLR85'; -- Seq Scan
-- 看看走了此函数索引的执行计划：
explain analyze verbose select * from jtest01 where json_extract_path_text(jdoc, 'name') = 'lnBtcJLR85'; -- Index Scan
-- 在JSONB类型上建GIN索引：
create table jtest02 (
  id int,
  jdoc jsonb
);
create table jtest03 (
  id int,
  jdoc jsonb
);
insert into jtest02 select id, jdoc::jsonb from jtest01;
insert into jtest03 select * from jtest02;
create index idx_jtest02_jdoc on jtest02 using gin (jdoc);
create index idx_jtest03_jdoc on jtest03 using gin (jdoc jsonb_path_ops);
analyze jtest02;
analyze jtest03;
select * from jtest02 where jdoc @> '{"name":"lnBtcJLR85"}';
select * from jtest03 where jdoc @> '{"name":"lnBtcJLR85"}';
explain analyze verbose select * from jtest02 where jdoc @> '{"name":"lnBtcJLR85"}'; -- Bitmap Heap Scan
explain analyze verbose select * from jtest03 where jdoc @> '{"name":"lnBtcJLR85"}'; -- Bitmap Heap Scan
-- 这两个SQL都走到了索引上，查看索引大小
select pg_indexes_size('jtest02');
select pg_indexes_size('jtest03');
```

### Range类型
#### Rangele类型简介
用于表示范围，如一个整数的范围、一个时间的范围，而范围底下的基本类型（如整数、时间）则被称之为Range类型的subtype。
假设我们有一个需求，某个IP地址库记录了每个地区的IP地址范围，现在需要查询客户的IP地址在哪个地区。该IP地址库的定义如下：
```SQL
create table ipdb1 (
  ip_begin inet,
  ip_end inet,
  area text,
  sp text);
```
如果要查询的是IP地址115.195.180.105在哪个地区：
```SQL
select * from ipdb1 where ip_begin <= '115.195.180.105'::inet and ip_end >= '115.195.180.105'::inet;
```
因为表上没有索引，所以会进行全表扫描，故而很慢。这时可以在ip_begin和ip_end上建索引：
```SQL
create index idx_ipdb_ip_start on ipdb1(ip_begin);
create index idx_ipdb_ip_end on ipdb1(ip_end);
```
在PostgreSQL中，上面的SQL可以使用到这两个索引，但都是先分别扫描两个索引建位图，然后通过位图进行and操作：
```SQL
explain analyze verbose select * from ipdb1 where ip_begin <= '115.195.180.105'::inet and ip_end >= '115.195.180.105'::inet; -- Bitmap Heap Scan
```
使用Range类型，通过创建空间索引的方式来执行：
首先，创建类似的IP地址库表：
```SQL
create type inetrange as range (subtype = inet);
create table ipdb2 (
  ip_range inetrange,
  area text,
  sp text);
insert into ipdb2 select ('[' || ip_begin || ',' || ip_end ']') ::inetrange, area, sp from ipdb1;
```
然后，创建gist索引：
```SQL
create index id_pdb2_ip_range on ipdb2 using gist (ip_range);
select * from ipdb2 where ip_range @> '115.195.180.105':inet;
explain analyze verbose select * from ipdb2 where ip_range @> '115.195.180.105'::inet;
```
#### 创建Range类型
内置：int4range, int8range, numrange, tsrange(无时区的时间戳范围类型), tstzrange, daterange.
创建语法：
```SQL
create type name as range (
  subtype = type -- 指定子类型
  [, subtype_opclass = subtype_operator_class ] -- 指定子类的操作符
  [, collation = collation ] -- 指定排序规则
  [, canonical = canonical_function ] -- 创建一个稀疏的Range类型，而非连续的Range类型
  [, subtype_diff = subtype_diff_function ] -- 定义子类型的差别函数
)
-- 示例：
create type floatrange as range (
  subtype = float8,
  subtype_diff = float8mi
);
```
#### Range类型的输入与输出
'[value1, value2]', '[value1, value2)', '(value1, value2]', '(value1, value2)', 'empty'。如果是稀疏型的range，其内部的存储格式为“[value1, value2)”.
```SQL
select '(0,6)'::int4range; -- [1,6)
select '[0,6]'::int4range; -- [0,7)
-- int4range总是把输入格式转换成“'[value1, value2)'”格式。
-- 对于连续型的range，内部存储则是精确存储的
select '[0,6]'::numrange; -- [0,6]
select '[0,6)'::numrange; -- [0,6)
-- Range类型还可以表示极值的区间：
select '[1,)'::int4range; -- [1,)
select '[,1)'::int4range; -- (,1)
-- 使用Range类型的构造函数输入Range类型的值，Range类型的构造函数名称与类型名称相同：
select int4range(1,10,'[)'); -- [1,10)
select int4range(1,10,'()'); -- [2,10)
select int4range(1,10); -- [1,10)
```
#### Range类型的操作符
=, <>, <, >, <=, >=, @>(包含), <@(被包含), &&(重叠), <<(严格在左), >>(严格在右), &<(没有扩展到右边), &>(没有扩展到左边), -|-(连接在一起), +(union), \*(intersection), -(difference)
#### Range类型的函数
- lower(anyrange), 获得范围的起始值
lower(int4range '(11,22)') -- 12
lower(int4range '[,22)') is null -- t
lower(int4range 'empty') is null --t
- upper(anyrange), 获得范围的结束值
upper(int4range '[11,22)') -- 22
- isempty(anyrange), 是否是空范围
isempty(int4range '(,)') -- f
isempty(int4range '(1,1)') -- t
- lower_inc(anyrange), 起始值是否在范围内
lower_inc(int4range '[1,1)') -- f
- upper_inc(anyrange), 结束值是否在范围内
upper_inc(int4range '[1,2)') -- f
- lower_inf(anyrange), 起始值是否是一个无穷值
- upper_inf(anyrange), 结束值是否是一个无穷值
#### Range类型的索引和约束
在Range的列上可以创建GiST和SP-GiST索引：
```SQL
create index index_name on table_name using gist (range_column);
```
在SQL查询语句中，可以使用=, &&, <@, @>, <<, >>, -|-, &<, &>来执行索引。
在Range类型上也可以建Btree索引，但Btree索引是使用比较运算符的，通常只有在对Range的值进行排序时使用。
在Range的列上也可以建立约束，让其范围总是不重叠：
```SQL
create table rtest01 (
  idrange int4range,
  exclude using gist (idrange with &&)
);
insert into rtest01 values (int4range '[1,5)');
insert into rtest01 values (int4range '[4,5)'); -- error
```
如果是一个有两列的表，第一列是普通类型，第二列是Range类型， 若要让第一列值相等的行其第二列的值也不重叠，则需要使用另一个扩展模块btree_gist来实现了：
首先，安装btree_gist：
```bash
cd source_code_path/contrib/btree_gist
make
make install
```
使用示例：
```SQL
create extension btree_gist;
create table rtest02 (
  id int,
  idrange int4range,
  exclude using gist (id with =, idrange with &&)
);
insert into rtest02 values (1, int4range '[1,5)');
insert into rtest02 values (2, int4range '[2,5)');
insert into rtest02 values (1, int4range '[2,5)'); -- error
```

### 数组类型
#### 数组类型的声明
```SQL
create table testtab04(id int, col1 int[], col2 int[10], col3 text[][]);
```
在目前的PostgreSQL中，如果在定义数组类型中填一个数组长度的数字，并不会限制数组的长度；定义时指定数组的维度也是没有意义的，数组的维度是根据实际插入的数据来确定的。
#### 如果输入数组值
```SQL
create table testtab05 (id int, col1 int[], col2 text[], col3 box[]);
insert into testtab05 values (1, '{1,2,3}', '{how, how many, "who, what.", "It''s ok", "{os\"dba}"}', '{((1,1), (2,2)); ((3,3), (4,4)); ((1,2), (7,9))}');
-- 在PostgreSQL中，每个类型都定义的一个分隔号：
select typname, typdelim, from pg_type where typname in ('int4', 'int8', 'bool', 'char', 'box'); -- ,,,,;
-- 一个简单的数组构造器由关键字array、一个左方括号（[）、一个或多个元素值得表达式（用逗号分隔）、一个右方括号组成：
insert into testtab05 values (2, array[1,2,3], array['how', 'how many', 'who, what.', 'It''s ok', '{os"dba}'], array['((1,1),(2,2))'::box,box '(3,3),(4,4)','1,2,7,9'::box]);
-- 多维数组：
create table testtab06 (id int, col1 text[][]);
insert into testtab06 values (1, array[['os', 'dba'], ['dba', 'os']]);
-- 向多维数组中插入值时，各个维度的元素个数必须相同
insert into testtab06 values (2, '{{a,b}, {c,d,e}}'); -- error
insert into testtab06 values (2, '{{a,b,null}, {c,d,e}}');
-- PostgreSQL数据库中数组的下标是从1开始的，但也可以指定下标的开始值：
create table test02 (id int[]);
insert into test02 values ('[2:4]={1,2,3}');
select id[2], id[3], id[4] from test02;
```
#### 访问数组
```SQL
create table testtab07 (id int, col1 text[]);
insert into testtab07 values (1, '{aa, bb, cc, dd}');
select * from testtab07;
select id, col1[1] from testtab07;
select id, col1[1:2] from testtab07;
create table testtab08 (id int, col1 int[][]);
insert into testtab08 values (1, '{{1,2,3}, {4,5,6}, {7,8,9}}');
select id, col1[1][1], col1[1][2], col1[2][1], col1[2][2] from testtab08;
select id, col1[1:1] from testtab08; -- {{1,2,3}}
select id, col1[1:3][1:1] from testtab08; -- 等价于：
select id, col1[3][1:1] from testtab08;
-- PostgreSQL中规定，只要出现一个冒号，其他的单个下标将表示从1开始的一个切片，下标的数值表示该切片的结束值，“col1[3][1:2]”中的“col[3]”实际上表示的是“col[1:3]”，同理：
select id, col1[1:2][2] from testtab08;
```
#### 修改数组
```SQL
update testtab08 set col1='{{10,11,12}, {13,14,15}, {16,17,18}}' where id=1;
update testtab08 set col1[2][1]=100 where id=1;
-- 不能直接修改多维数组中某一维的值
```
#### 数组的操作符
=, <>, <, >, <=, >=, @>, <@, &&, ||([不]同维度的数组与数组连接; 元素类型必须相同), ||(元素与数组之间的连接; 元素与数组的元素类型必须相同)
#### 数组的函数
P139~P142

### 伪类型（Pseudo-Type）
是PostgreSQL中不能作为字段的数据类型，但它可以用于声明一个函数的参数或者结果类型：
any、anyelement、anyarray、anynoarray、anyenum、anyrange、cstring、internal、language_handler、fdw_handler、record、trigger、void、opaque。

### 其他类型
#### UUID类型
UUID（Universally Unique Identifiers）定义在“RFC 4122”和“ISO/IEC 9834-8:2005”中。它是一个128bit的数字。
#### pg_lsn类型
表示LSN（Log Sequence Number）的一种数据类型。LSN表示WAL日志的位置。
```SQL
\d pg_stat_replication View "pg_catalog.pg_stat_replication"
\d pg_replication_slots
```
在数据库内部，LSN是一个64bit的大整数，其输出的格式类似如下形式：
16/B374D848


























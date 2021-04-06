> 写在前面
> 本文是我之前对SQLite的学习分享PPT的一次整理，文末附带PPT与源码的GitHub下载地址

![嵌入式关系型数据库SQLite](http://upload-images.jianshu.io/upload_images/2048485-164b30f9739cfd40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 存储类型
> 五种存储类型：blob、text、real、integer以及null

- blob
二进制对象，最大长度可自定义，默认为1000,000,000 字节
- text
字符数据，支持UTF-8、UTF-16，字符串最大长度可被设置，默认为1000000000 字节
- real
![8字节浮点数范围](http://latex.codecogs.com/svg.latex?\{{-2^{63}\\,2^{63}-1}\})
- integer
1、2、3、4、6、8字节
- null
空值

![存储类型](http://upload-images.jianshu.io/upload_images/2048485-5f5d44207804b13d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

SQLite不存在数据类型的限制，只有**存储类型**的区别。比如，向一个text存储类型的字段插入浮点数值，SQLite将进行自动转换，SQLite内置函数typeof()根据值返回对应存储类型，比如`select typeof(‘3.14’);`将返回类型text

### 类型的相互转换
||blob|text|real|integer|null|
|:--|:--|:--|:--|:--|:--|
|blob|--|需要时加上\000终止符|转换为文本，atoi|转换为文本，atoi|--|
|text|原文|--|atoi|atoi|--|
|real|浮点数ASCII码|浮点数ASCII码|--|浮点型转整型|--|
|integer|整数ASCII码|整数ASCII码|整型转浮点型|--|--|
|null|null指针|null指针|0.0|0|--|

### 亲缘类型
> 为了和其他DBMS以及SQL标准兼容，在CREATE TABLE中指定列类型，SQLite提出列相似性（Column Affinity）的概念，即列的属性或叫亲缘类

|声明类型|亲缘类|转换规则|
|:--|:--|:--|
|INT、INTEGER、TINYINT、SMALLINT、MEDIUMINT、BIGINT、UNSIGNED BIG INT、INT2、INT8|INTEGER|如果类型字符串中包含"INT"，那么该字段的亲缘类型是INTEGER|
|CHARACTER(20)、VARCHAR(255)、VARYING、CHARACTER(255)、NCHAR(55)、NATIVE、CHARACTER(70)、NVARCHAR(100)、TEXT、CLOB|TEXT|如果类型字符串中包含"CHAR"、"CLOB"或"TEXT"，那么该字段的亲缘类型是TEXT，如VARCHAR|
|BLOB|NONE|如果类型字符串中包含"BLOB"，那么该字段的亲缘类型是NONE|
|REAL、DOUBLE、DOUBLE PRECISION、FLOAT|REAL|如果类型字符串中包含"REAL"、"FLOAT"或"DOUBLE"，那么该字段的亲缘类型是REAL|
|NUMERIC、DECIMAL(10,5)、BOOLEAN、DATE、DATETIME|NUMERIC|其余情况下，字段的亲缘类型为NUMERIC|

## 事务性
命令：begin、commit、rollback

### 隐式自成事务
默认情况下，SQLite单条SQL语句自成事务（自动提交）
![操作A自成事务](http://upload-images.jianshu.io/upload_images/2048485-b4a860e73ead2cc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 显式事务
由begin和commit之间的一组操作组成一项事务
事务性保证了一项操作或一组操作，要么100%完成，要么全部没有执行
```shell
// 示例：
begin；
DELETE FROM test WHERE id = 5;
rollback;
DELETE FROM test WHERE goal = 88;
commit;
```
![操作A没有完成，操作B完成](http://upload-images.jianshu.io/upload_images/2048485-1661b0899dc0281d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 事务的冲突解决
命令：replace、ignore、fail、abort、rollback
![事务的冲突解决](http://upload-images.jianshu.io/upload_images/2048485-00a57611434c89d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

|冲突解决|说明|
|:--|:--|
|replace|违反的记录被删除，以新记录代替之|
|ignore|违反的记录保持原貌，其它记录继续执行|
|fail|终止命令，违反之前执行的操作得到保存|
|abort|终止命令，恢复违反之前执行的修改|
|rollback|终止命令和事务，回滚整个事务|

语法：
1. 语句级（可覆盖对象级的冲突解决手段）
insert/update/create  or [resolution] table/index [tbl_name/idx_name] ……
[resolution]： replace、ignore、fail、abort、rollback

2. 对象级（定义表格时）
create table/index [tbl_name/idx_name] ([field_name] [format] [constraint] on conflict [resolution]);
[constraint]：unique、not null……

```shell
// 示例
create temp table cast (name text unique on conflict rollback);
insert into cast values (‘Jerry’);
insert into cast values (‘Bean’);
insert into cast values (‘Android 4.1’);

begin;
insert into cast values (‘Jerry’);
:uniqueness constraint failed
commit;
:cannot commit – no transaction is active

begin;
insert or replace into cast values (‘Jerry’);
commit;
```

## 锁机制
五种锁状态：unlocked、shared、reserved、pending、exclusive

|锁状态|说明|
|:--|:--|
|未加锁-unlocked|未对数据库进行访问（读写）之前|
|共享锁-shared|对数据库进行读操作|
|预留锁-reserved|对数据库进行写（缓存）操作|
|未决锁-pending|等待其它共享锁关闭|
|排它锁-exclusive|将缓存中的操作提交到数据库|

![红色箭头代表你，你想访问数据库DB](http://upload-images.jianshu.io/upload_images/2048485-729b803e1c22aed6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![红色箭头代表你，你受锁机制的约束](http://upload-images.jianshu.io/upload_images/2048485-f4da01e5dcf7f9a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![死锁原因](http://upload-images.jianshu.io/upload_images/2048485-a6cc51bcc96e87b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

死锁原因：拥有共享锁与未决锁的连接都不想放弃控制

### 预防死锁
三种事务类型：deferred、immediate、exclusive
SQLite根据不同的事务类型，以不同的锁状态启动事务（而不是以未加锁状态启动）

事务类型在begin命令中指定：
begin [deferred | immediate | exclusive] transaction;

|事务类型|说明|
|:--|:--|
|Deferred|直到必须使用时才获取锁|
|Immediate|在begin执行时试图获取预留锁|
|Exclusive|试图获取排它锁|

基本准则，多人对同一数据库进行操作时，必须共同遵守本守则：
使用begin immediate | exclusive transaction;而不是begin deferred transaction;或单纯的begin;

## SQLite不支持ANSI SQL92的部分特性
- 右外连接与全外连接
支持左外连接 LEFT OUTER JOIN
不支持右外连接 RIGHT OUTER JOIN 和全外连接 FULL OUTER JOIN
- 大部分的修改表操作
支持 RENAME TABLE 和 ADD COLUMN
不支持 DROP COLUMN、ALTER COLUMN、ADD CONSTRAINT
- 触发器
支持 FOR EACH ROW 触发器
不支持 FOR EACH STATEMENT 触发器
- 授权与撤销
不支持 GRANT 和 REVOKE，它们对于嵌入式数据库引擎无意义
SQLite 读取和写入一个普通磁盘文件
唯一的访问许可对于底层操作系统的一般文件有效

## 工作模式
||main db|temp db|创建方式|存储位置|
|:--:|:--|:--|:--|:--|
|磁盘文件|默认|create temp table|“path/db_name”|磁盘|
|驻留内存|默认|create temp table|“:memory:” or “”|RAM|

使用命令sqlite3创建一个数据库文件
- 默认存在main数据库对象，所有默认操作都是在main数据库对象中进行
- 若执行了CREATE TEMP TABLE命令，该数据库文件将连接一个temp数据库对象
- 若执行ATTACH命令，该数据库文件连接一个外部的数据库对象

## 配置
SQLite没有配置文件
所有的配置参数都是靠编译指令PRAGMA或编译时参数定义来实现
运行信息、schema、版本、 文件格式、内存使用和调试等可供配置
编译指令包括临时性和永久性两种
1）PRAGMA
2）编译时参数（宏）

### PRAGMA
1）连接缓冲区大小
PRAGMA cache_size;
PRAGMA default_cache_size;

2）数据库信息获取
PRAGMA database_list;
PRAGMA foreign_key_list(table-name); 
PRAGMA index_info(index-name);  
PRAGMA index_list(table-name);  
PRAGMA table_info(table-name);

3）写同步
PRAGMA synchronous; // 影响非常大的优化设置

4）临时存储器
PRAGMA temp_store;
PRAGMA temp_store_directory;

5）页大小、编码和自动清理
PRAGMA page_size;
PRAGMA encoding; 
PRAGMA auto_vacuum; // 开启后手动vacuum命令将不起作用

6）调试
PRAGMA integrity_check; // 完整检查，无误返回ok

### 宏
1）SQLITE_TEMP_STORE
等于0时，临时文件总是存储在磁盘上，而不会考虑PRAGMA temp_store指令的设置。
等于1时，临时文件缺省存储在磁盘上，但该值可以被PRAGMA temp_store指令覆盖。
等于2时，临时文件缺省存储在内存中，但该值可以被PRAGMA temp_store指令覆盖。
等于3时，临时文件总是存储在内存中，而不会考虑PRAGMA temp_store指令的设置。

![SQLite工作模式配置](http://upload-images.jianshu.io/upload_images/2048485-e296ce9ad78ccdcd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2）SQLITE_DEFAULT_TEMP_CACHE_SIZE
用于指定临时表和索引在占用多少Cache Page时才需要被刷新到磁盘文件
该参数的缺省值为500页

> 谨慎使用temp_store_directory修改临时文件存储目录！！！
对于Unix/Linux/OSX来说，默认可路径是/var/tmp, /usr/tmp, /tmp以及当前目录 current-directory中第一个可写的目录。对于 WINDOWS NT，默认路径由WINDOWS决定，通常是C:\Documents and Settings\user-name\Local Settings\Temp\。SQLite创建的临时文件在打开后会被立即删除（unlink）, 这样当SQLite进程退出时，操作系统就可以自动删除这些文件。所以正常状态下，使用ls或dir命令是无法看到这些临时文件的。

## C API

### 两个重要的对象

![两个重要的对象](http://upload-images.jianshu.io/upload_images/2048485-7bdef439a578ae55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- database connection
数据库连接对象 sqlite3，即数据库名
- prepared statement
预声明对象 sqlite3_stmt，简单理解就是指向 SQL 命令的一个结构

### 六个接口

![六个接口](http://upload-images.jianshu.io/upload_images/2048485-8c685614bd62508d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（1）使用sqlite3_open()连接一个数据库，一个连接访问多个数据库使用attach功能。
（2）使用sqlite3_prepare()创建prepared statement对象。
（3）调用sqlite3_step()执行prepared statement一次或多次。
（4）查询时，在两个sqlite3_step()调用之间调用sqlite3_column()解压结果。
（5）使用sqlite3_finalize()销毁prepared statement对象。
（6）调用sqlite3_close()关闭数据库连接。

### 封装接口

![封装接口](http://upload-images.jianshu.io/upload_images/2048485-a6d6db7c23f14061.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

sqlite3_exec()：适合执行不返回结果集的查询，它将结果交给callback函数处理
sqlite3_get_table()：适合执行返回结果集的查询，它将结果存储在堆内存中而不是交给callback函数处理

### 多次查询

![多次查询](http://upload-images.jianshu.io/upload_images/2048485-cf998750ae5a783e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![多次查询](http://upload-images.jianshu.io/upload_images/2048485-3dd0783847e53338.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

sqlite3_exec()：执行不返回结果集的查询
sqlite3_get_table()：执行返回结果集的查询

## 性能

测试目的

（1）了解SQLite文件模式与内存模式的插、查、删、递归速度
（2）了解SQLite文件模式与内存模式的操作稳定性

测试项目

（1）SQLite文件模式与内存模式的插、查、删、递归速度测试
（2）SQLite文件模式与内存模式的操作稳定性测试


### 常规操作性能测试

![测试样例](http://upload-images.jianshu.io/upload_images/2048485-cd8ab764151201d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

测试项目：
文件模式插入100万条记录
内存模式插入100万条记录
文件模式递归100万条记录
内存模式递归100万条记录
文件模式查询1000  条记录
内存模式查询1000  条记录
文件模式删除100万条记录
内存模式删除100万条记录

测试版本：
SQLite V3.7.13

![结果](http://upload-images.jianshu.io/upload_images/2048485-98abe1b4770a00d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 稳定性测试
以文件模式分别测试插入、查询、删除与递归在记录量为
10万、20万……100万的时间性能，
得到大致的时间性能曲线。

以内存模式分别测试插入、删除与递归在记录量为
100万、200万……1000万的时间性能各三次，
得到大致的时间性能稳定性曲线。

![内存插入](http://upload-images.jianshu.io/upload_images/2048485-d2ce711dd03b9492.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![内存删除](http://upload-images.jianshu.io/upload_images/2048485-20c79b091f5d40ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![内存递归](http://upload-images.jianshu.io/upload_images/2048485-13a7c250d5740b95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![文件插入](http://upload-images.jianshu.io/upload_images/2048485-c67a8a47eb756013.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![文件删除](http://upload-images.jianshu.io/upload_images/2048485-030201cfc2280b5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![文件递归](http://upload-images.jianshu.io/upload_images/2048485-dd81294751b3ab96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 老规矩-PPT及测试代码下载地址
[GitHub地址](https://github.com/hayukleung/sqlite-study)

## 关于SQLite的学习资料

1.[官网文档](http://www.sqlite.org/index.html)
2.[重点推荐这本书《SQLite权威指南》](http://www.apress.com/)
3.[SQLite中文社区](http://www.sqlite.com.cn/)
4.[SQLite维基](http://www.sqlite.org/cvstrac/wiki)
5.[博客园](http://www.cnblogs.com/stephen-liu74/archive/2012/03/09/2328757.html)
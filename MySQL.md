## MySQL

## 连接数据库

命令行连接！

```sql
mysql -uroot -proot -- 连接数据库

update mysq1.user set authentication_string=password('123456') where user=' root' and Host ='localhost'; --修改用户密码
flush privileges; --刷新权限
---------------------------------------------------------------------
show database; --查看所有的数据库
use 数据库名 -- 切换数据库

show tables; --查看数据库中所有的表
describe student; -- 显示数据库中的表的信息

create database 数据库名 -- 创建一个数据库

exit; --退出连接

--  单行注释
/*
	sql的多行注释
*/
```

1) DDL(Data Definition Language)数据定义语言
		用来定义数据库对象：数据库，表，列等。关键字：create, drop,alter 等
2) DML(Data Manipulation Language)数据操作语言
		用来对数据库中表的数据进行增删改。关键字：insert, delete, update 等
3) DQL(Data Query Language)数据查询语言
		用来查询数据库中表的记录(数据)。关键字：select, where 等
4) DCL(Data Control Language)数据控制语言(了解)
		用来定义数据库的访问权限和安全级别，及创建用户。关键字：GRANT， REVOKE 等



## 操作数据库

### 1.1、操作数据库

操作数据库>操作数据库中的表>操作数据库中的表中的数据

1、创建数据库

```sql
CREATE DATABASE [IF NOT EXISTS] westos;
```

2、删除数据库

```sql
DROP DATABASE [IF EXISTS] westos;
```

3、使用数据库

```sql
-- 如果字段名是数据库的关键字 加上``,就不会高亮显示了
USE `school`;
```

4、查看数据库

```sql
SHOW DATABASES  -- 查看所有数据库
```



### 1.2、数据库的数据类型

>数值

- tinyint        十分小的数据   1个字节
- smallint      较小的数据       2个字节
- mediumint  中等大小的数据  3个字节
- **int                  标准的整数         4个字节 常用的** 对应java中的int
- bigint            较大的数据          8个字节  对应于java中的long
- float              浮点数                  4个字节    float
- double          浮点数                  8个字节
- decimal          字符串的浮点数    金融计算的时候一般使用decimal    对应java的decimal



>字符串

- char        字符串固定大小   0~255
- **varchar   可变字符串          0~65535**     常用的   对应java String
- tinytext     微型文本            2^8-1
- **text            文本串               2^16-1**  保存大文本



>日期

- date  YYYY-MM-DD, 日期格式
- time   HH:mm:ss 时间格式
- **datetime    YYYY-MM-DD  HH:mm:ss  最常用的时间格式**
- **timestamp   时间戳 ， 1970.1.1到现在的毫秒数！也较为常见**
- year 年分表示

>null

- 没有值，未知
- ==注意，不要使用NULL进行运算，结果为NULL==



### 1.3、数据库的字段属性

==Unsigned==

- 无符号整数
- 如果声明了该列就不能在这输入负数



==zerofill：==

- 0填充的
- 不足的位数，使用0来填充， int(3)  5,     --- 005



==默认：==

- 设置默认的值！
- sex，默认值为 男，如果不指定该列的值，则会有默认值！



常用命令

```sql
SHOW CREATE DATABASE school  -- 查看创建数据库语句
SHOW CREATE TABLE student  -- 查看创建student表的语句
DESC student -- 查看表结构
```



### 1.4、数据表的类型

```sql
-- 关于数据库引擎
INNODB  -- 默认使用
MYISAM -- 早些年使用
```



表锁：如果两条sql去查询同一个表，会先把表给锁定，另外的sql操作需要等待，这就是表锁

行锁：只锁定行

全文索引：就是在数据中也有该关键字，就是类似于在文章中找到该关键字



|                    | MYISAM |        INNODB         |
| :----------------- | :----: | :-------------------: |
| 事务支持           | 不支持 |         支持          |
| 数据行锁定         | 不支持 |         支持          |
| 外键约束           | 不支持 |         支持          |
| 全文索引           |  支持  |        不支持         |
| 表空间的大小(内存) |  较小  | 较大，越为MYISAM的2倍 |

常规操作：

- MYISAM 节约空间，速度较快
- INNODB 安全性高，支持事务的处理，支持外键可以多表多用户操作



>在物理空间存在的位置

所有的数据库文件都存在data目录下，一个文件夹对应一个数据库

本质还是文件的存储！

MySQL引擎在物理文件上的区别

- innoDB  在数据库表中只有一个 *.frm文件是属于它的，以及上级目录下的ibdata1文件
- MYISAM 对应的文件
  - *.frm 表结构的定义文件
  - *.MYD 数据文件(data)
  - *.MYI  索引文件(index)



>修改表（DDL）

```sql
-- 修改表名 ALTER TABLE 旧表名 RENAME AS 新表名
ALTER TABLE teacher RENAME AS teacher1

-- 增加字段 ALTER TABLE 表名 ADD 字段名 列属性
ALTER TABLE teacher1 ADD get INT(11)

-- 修改表的字段 （重命名、修改类型和约束）
-- ALTER TABLE 表名 MODIFY 字段名 列属性[]
ALTER TABLE teacher1 MODIFY get VARCHAR(11)  -- 修改约束
-- ALTER TABLE 表名 CHANGE 旧字段名  新字段名 列属性[]
ALTER TABLE teacher1 CHANGE get age INT(1)  -- 修改字段名同时修改约束

-- 删除表的字段
-- ALTER TABLE 表名 DROP 字段名
ALTER TABLE teacher1 DROP age
```



>删除表（DDL）

```sql
-- DROP TABLE [IF EXISTS] 表名
DROP TABLE IF EXISTS teacher1
```

==所有的创建和删除操作尽量加上判断，以免报错==



注意点：

- 所有字段名 最好用``引起来
- 注释 -- 或者/**/
- sql关键字对大小写不敏感



## 3、MySQL数据管理

### 3.1、外键（了解即可）

>方式一、在创建表的时候，增加约束

```sql
CREATE TABLE `grade`(
	`gradeid` INT(10) NOT NULL AUTO_INCREMENT COMMENT '年级id',
	`gradename` VARCHAR(50) NOT NULL COMMENT '年级名称',
	PRIMARY KEY (`gradeid`)

)ENGINE=INNODB DEFAULT CHARSET=utf8


-- 学生表的gradeid字段 要去引用年级表的gradeid字段
-- 定义一个外键key
-- 给这个外键添加约束 （执行引用） references 引用
CREATE TABLE IF NOT EXISTS `student`(
	`id` INT(4) NOT NULL AUTO_INCREMENT COMMENT '学号',
	`name` VARCHAR(30) NOT NULL DEFAULT '匿名' COMMENT '姓名',
	`pwd` VARCHAR(20) NOT NULL DEFAULT '123456' COMMENT '密码',
	`sex` VARCHAR(3) NOT NULL DEFAULT '女' COMMENT '性别',
	`birthday` DATETIME DEFAULT NULL COMMENT '出生日期',
	`address` VARCHAR(100) DEFAULT NULL COMMENT '家庭住址',
	`gradeid` INT(10) NOT NULL,
	`email` VARCHAR(50) DEFAULT NULL COMMENT '邮箱',
	PRIMARY KEY(`id`),
	KEY `FK_gradeid` (`gradeid`), -- 定义一个外键key
	CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade`(`gradeid`)
)ENGINE=INNODB DEFAULT CHARSET=utf8
```



删除有外键关系的表的时候，要先删除引用别人的表（从表），再删除被引用的表（主表）



>方式二：创建表成功后，添加外键约束

```sql
CREATE TABLE `grade`(
	`gradeid` INT(10) NOT NULL AUTO_INCREMENT COMMENT '年级id',
	`gradename` VARCHAR(50) NOT NULL COMMENT '年级名称',
	PRIMARY KEY (`gradeid`)

)ENGINE=INNODB DEFAULT CHARSET=utf8

CREATE TABLE IF NOT EXISTS `student`(
	`id` INT(4) NOT NULL AUTO_INCREMENT COMMENT '学号',
	`name` VARCHAR(30) NOT NULL DEFAULT '匿名' COMMENT '姓名',
	`pwd` VARCHAR(20) NOT NULL DEFAULT '123456' COMMENT '密码',
	`sex` VARCHAR(3) NOT NULL DEFAULT '女' COMMENT '性别',
	`birthday` DATETIME DEFAULT NULL COMMENT '出生日期',
	`address` VARCHAR(100) DEFAULT NULL COMMENT '家庭住址',
	`gradeid` INT(10) NOT NULL,
	`email` VARCHAR(50) DEFAULT NULL COMMENT '邮箱',
	PRIMARY KEY(`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8

-- 在创建表时没有外键关系
-- ALTER TABLE 待添加外键的表名 ADD CONSTRAINT 约束名 FOREIGN KEY （作为外键的列） REFERENCES 那个表(哪个字段)
ALTER TABLE `student`
ADD CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES 	`grade`(`gradeid`);

```

以上的操作都是物理级别的外键，数据库级别的外键，我们不建议使用！（避免数据库过多造成困扰）



==最佳实践==

- 数据库就是单纯的表，只用来存数据，只有行列和主键
- 我们想使用多张表额数据，想使用外键，一般用程序去实现

### 3.2、DML语言

**数据库意义：**数据存储，数据管理

DML语言：数据操作语言

- insert
- update
- delete

### 添加

```sql
INSERT INTO `grade`(`gradename`) VALUES('大四');

-- 插入多个值
INSERT INTO `grade`(`gradename`) 
VALUES('大一'),('大二');

INSERT INTO `student`(`name`,`pwd`,`sex`,`gradeid`) VALUES('张三','aaaaaaaaaaaaaa','男',0)


INSERT INTO `student`(`name`,`pwd`,`sex`,`gradeid`) 
VALUES('李四','aaaaaaaaaaaaaa','男',0),('王五','aaaaaaaaaaaaaa','男',0)
```



### 修改

>update    表名    set 原来的值=新值 [ 条件表达式]

```sql
-- 语法：
-- update 表名 set column_name=value, [column_name=value,...] where [条件]
-- 按条件修改一个值
UPDATE `student` SET `name`='小绿' WHERE id=1;

-- 如果不加条件，直接将整个表给改掉了
UPDATE `student` SET `name`='小红';

-- 按条件修改多个值,多个属性之间用','隔开
UPDATE `student` SET `name`='小青',`pwd`='123456',`sex`='女',`email`='243561524@qq.com' WHERE id = 2;
```

| 操作符     | 含义     | 范围 | 结果 |
| ---------- | -------- | ---- | ---- |
| =          | 等于     |      |      |
| <> 或者 != | 不等于   |      |      |
| >          | 大于     |      |      |
| <          | 小于     |      |      |
| >=         | 大于等于 |      |      |
| <=         | 小于等于 |      |      |
| between    | 闭合区间 | []   |      |
| and        | 与       |      |      |
| or         | 或       |      |      |

语法：`update 表名 set column_name=value, [column_name=value,...] where [条件]`

注意：

- column_name 是表的列，尽量带上``
- 条件，筛选的条件，如果没有指定，则会修改所有的值
- value，是一个具体的值，也可以是一个变量, 这个变量用的也不多一般是时间

```sql
UPDATE `student` SET `birthday`=CURRENT_DATE WHERE id=2
```



### 3.5、删除

>delete 命令

语法：`delete from 表名 [where 条件]`

```sql
-- 删除数据(避免这样写，会全部删除)
DELETE FROM `student`;
-- 删除指定的数据
DELETE FROM `student` WHERE id = 1;
```

>TRUNCATE 命令

作用：完全清空一个数据库表，表的结构和索引约束都不会变！

```sql
-- 清空student表
TRUNCATE `student`
```

- 相同点：都能删除数据，都不会删除表结构
- 不同点：
  - TRUNCATE 重新设置 自增列 计数器会归零
  - TRUNCATE 不会影响事务

```sql
-- 测试delete 和truncate 区别

CREATE TABLE `test`(
	`id` INT(4) NOT NULL AUTO_INCREMENT,
	`coll` VARCHAR(20) NOT NULL,
	PRIMARY KEY (`id`)

)ENGINE=INNODB DEFAULT CHARSET=utf8


INSERT INTO `test`(`coll`) VALUES('1'),('2'),('3')

DELETE FROM `test` -- 不会影响自增

TRUNCATE TABLE	`test` -- 自增会归零
```

了解即可：`delete删除的问题` 重启数据库，现象

- InnoDB  自增列会从1从新开始（因为数据存在内存中，断电即失）
- MyISAM 继续从上一个自增量开始 （因为它存在文件中，不会丢失）

## DQL查询数据

### 1、DQL

（Data Query Language：数据库查询语言）

- 所有的查询操作都用它 select
- 简单的查询，复杂的查询它都能做
- ==数据库最核心的语言，最重要的语句==
- 使用频率最高的语句

### 2、指定查询字段

```sql
-- 函数 concat(a,b)连接两个字符串
SELECT CONCAT('姓名：',`StudentName`) AS 新名字 FROM `student`
```



select 完整语法：

```sql
SELECT
	[ALL | DISTINCT | DISTINCTROw ]
        [HIGH_PRIORITY]
        [STRAIGHT_JOIN]
        [sQL_SMALL_RESULT] [sQL_BIG_RESULT] [sQL_BUFFER_RESULT]
        [sQL_CACHE | sQL_NO_CACHE] [sQL_CALC_FOUND_ROws]
  	select_expr [， select_expr ...]
  	[FROM tab1e_references
    	[PARTITION partition_list]
    [WHERE where_condition]
    [GROUP BY {col_name / expr | position}
    [Asc | DESc]，... [wITH ROLLUP]]
    [HAVING where_condition]
    [ORDER BY {co1_name | expr | position}
    [AsC | DESc]，...]
    [LIMIT {[offset,] row_count / row_count OFFSET offset}]
    [PROCEDURE procedure_name(argument_list)]
    [INTO OUTFILE 'file_name'
     	[CHARACTER SET charset_name]
     	export_options
     | INTO DUMPFILE 'file_name'
     | INTO var_name [， var_name]]
     [FOR UPDATE / LOCK IN SHARE MODE]]
```



>去重 distinct

作用：去除select查询出来的结果中重复的数据，重复的数据只显示一个

```sql
SELECT DISTINCT `studentno` FROM `student`  -- 去除重复的数据  distinct
```



>数据库的列（表达式）

```sql
SELECT VERSION()  -- 查看mysql版本
SELECT 100*3-1 AS 计算结果 -- 用来计算（表达式）
SELECT @@auto_increment_increment  -- 查询自增的步长 (变量)

-- 学员考试成绩+1分查看
SELECT `studentno`,`studentresult`+1 AS 提分后 FROM `result`
```

==数据库中的表达式：文本值，列，Null，函数，计算表达式，系统变量……==



>模糊查询：比较运算符

| 运算符      | 语法                         | 描述                                         |
| ----------- | ---------------------------- | -------------------------------------------- |
| IS NULL     | a is null                    | 如果操作符为null，返回true                   |
| is not null | a is not null                | 如果操作符不为null，返回true                 |
| between     | a between b and c   [闭区间] | 若a在b和c之间, 返回true                      |
| **like**    | a like b                     | SQL匹配，如果a匹配b，则返回true              |
| **in**      | a in (a1,a2,a3,……)           | 假设a是a1,或者a2....其中的某一个值，返回true |



```sql
-- =======================模糊查询=============================
-- 查询姓小的同学
-- like结合 %(代表0到人一个字符) _(表示一个字符)
SELECT `studentno`,`studentname` FROM `student`
WHERE `studentname` LIKE '小%'

-- 查询小后面只有一个字同学
SELECT `studentno`,`studentname` FROM `student`
WHERE `studentname` LIKE '小_'


-- 查询名字中有宏的人
SELECT `studentno`,`studentname` FROM `student`
WHERE `studentname` LIKE '%宏%'


-- ================in（具体的一个或者多个值）=======================
-- 查询1001,1002,1003号学员
SELECT `studentno`,`studentname` FROM `student`
WHERE `studentno` IN (1001,1002,1003)

-- 查询在北京的学生
SELECT `studentno`,`studentname` FROM `student`
WHERE `address` IN ('北京朝阳','北京海淀')




-- ==========null not null===================
-- 查询地址为null或者''的学生
SELECT `studentno`,`studentname` FROM `student`
WHERE `address` IS NULL OR `address`='';

-- 查询出生日期不为null的同学  不为空
SELECT `studentno`,`studentname` FROM `student`
WHERE `borndate` IS NOT NULL

-- 查询出生日期为null的同学  为空
SELECT `studentno`,`studentname` FROM `student`
WHERE `borndate` IS  NULL
```



### 连表查询

>连表查询的关键词是  JOIN

![image-20210809161725579](.\typora-user-images\image-20210809161725579.png)

![image-20210809162036277](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210809162036277.png)

```sql
-- 使用join关键词后，可以跟on（判断连接的条件） where表示的是等值查询
-- 内连接 
SELECT s.studentno,studentname,subjectno,studentresult
FROM student AS s
INNER JOIN result AS r
WHERE s.studentno = r.studentno

-- right join
SELECT s.studentno,studentname,subjectno,studentresult
FROM student AS s
RIGHT JOIN result AS r
on s.studentno = r.studentno


-- left join
SELECT s.studentno,studentname,subjectno,studentresult
FROM student AS s
LEFT JOIN result AS r
on s.studentno = r.studentno

-- 查询缺考的同学
SELECT s.studentno,studentname,subjectno,studentresult
FROM student AS s
LEFT JOIN result AS r
on s.studentno = r.studentno
where studentresult is null
```

| 操作       | 描述                                       |
| ---------- | ------------------------------------------ |
| inner join | 如果表中至少有一个匹配，就返回行           |
| left join  | 会从左表中返回所有的值，即使右表中没有匹配 |
| right join | 会从右表中返回所有的行，即使左表中没有匹配 |

如何分左右，最接近from的表就是左表

```sql
-- 多表
-- 查询学生的学号，姓名，科目名称，考试成绩
SELECT s.studentno,studentname,subjectName,studentresult
FROM student AS s
RIGHT JOIN result AS r
ON r.studentno = s.studentno
INNER JOIN `subject` sub
WHERE r.subjectno=sub.subjectno

-- 我要查询哪些数据select ...
-- 从那几个表中查FROM表XXX Join连接的表on交叉条件
-- 假设存在一种多张表查询，慢慢来，先查询两张表然后再慢慢增加
```



### 自连接

自己的表和自己的表连接，**解题核心：一张表拆为两张一样的表即可**（了解即可）

```sql
-- 创表的语句
CREATE TABLE `category`( 
`categoryid` INT(3) NOT NULL COMMENT '主题id',
 `pid` INT(3) NOT NULL COMMENT '父id', 
 `categoryname` VARCHAR(10) NOT NULL COMMENT '主题名字', 
 PRIMARY KEY (`categoryid`) 
 )ENGINE=INNODB DEFAULT CHARSET=utf8 AUTO_INCREMENT=9

INSERT INTO `category` (`categoryid`, `pid`, `categoryname`) VALUES ('2', '1', '信息技术');
INSERT INTO `category` (`categoryid`, `pid`, `categoryname`) VALUES ('3', '1', '软件开发');
INSERT INTO `category` (`categoryid`, `PId`, `categoryname`) VALUES ('5', '1', '美术设计');
INSERT INTO `category` (`categoryid`, `pid`, `categorynamE`) VALUES ('4', '3', '数据库'); 
INSERT INTO `category` (`CATEgoryid`, `pid`, `categoryname`) VALUES ('8', '2', '办公信息');
INSERT INTO `category` (`categoryid`, `pid`, `CAtegoryname`) VALUES ('6', '3', 'web开发'); 
INSERT INTO `category` (`categoryid`, `pid`, `categoryname`) VALUES ('7', '5', 'ps技术');
```



**拆表：**

父类

| categoryid | categoryname |
| ---------- | ------------ |
| 2          | 信息技术     |
| 3          | 软件开发     |
| 5          | 美术设计     |

子类

| pid  | categoryid | categoryname |
| ---- | ---------- | ------------ |
| 3    | 4          | 数据库       |
| 3    | 6          | web开发      |
| 5    | 7          | ps技术       |
| 2    | 8          | 办公信息     |

操作：查询父类对应的子类关系，查询出来的结果如下

| 父类     | 子类     |
| -------- | -------- |
| 信息技术 | 办公信息 |
| 软件开发 | 数据库   |
| 软件开发 | web开发  |
| 美术设计 | ps技术   |

```sql
-- 查询父子信息
SELECT a.`categoryName` AS '父栏目',b.`categoryName` AS '子栏目'
FROM `category` AS a,`category` AS b
WHERE a.`categoryid` = b.pid
```



### 分页与排序

>排序

```sql
SELECT s.studentno,studentname,subjectName,studentresult
FROM student AS s
RIGHT JOIN result AS r
ON r.studentno = s.studentno
INNER JOIN `subject` sub
WHERE r.subjectno=sub.subjectno
order by studentresult asc[desc]
```



>分页

```sql
SELECT s.studentno,studentname,subjectName,studentresult
FROM student AS s
RIGHT JOIN result AS r
ON r.studentno = s.studentno
INNER JOIN `subject` sub
WHERE r.subjectno=sub.subjectno
order by studentresult asc[desc]
limit 0,5
-- 第一页 limit 0,5     (1-1)*5, 5
-- 第二页 limit 5,5      (2-1）*5, 5
-- 第N页  limit (N-1)*5,5 (n-1) * pagesize，pagesize
--【pagesize:页面大小】
-- 第一页7imit o, 5

--【pagesize:页面大小】
-- 【(n-1)* pagesize:起始值】
-- 【n :当前页】
-- 【数据总数/页面大小=总页数】 总页数

```

语法： `limit(查询起始下标，pageSize)`



### 子查询

本质就是:`在where语句中再嵌套一个值`

```sql
-- 查询课程为高等数学-2 且分数不小于80的同学的学号和姓名
SELECT s.`studentno`,`studentname`
FROM `student` s
INNER JOIN `result` r
ON s.`studentno`=r.`studentno`
INNER JOIN `subject` sub
ON sub.`subjectno`=r.`subjectno`
WHERE `studentresult`>=80 AND `subjectname`='高等数学-2'

-- 分数不小于80分的学生的学号和姓名
SELECT s.`studentno`,`studentname`
FROM `student` s
INNER JOIN `result` r
ON s.`studentno`=r.`studentno`
WHERE `studentresult`>=80

-- 在此基础上增加一个科目，高等数学-2
SELECT s.`studentno`,`studentname`
FROM `student` s
INNER JOIN `result` r
ON s.`studentno`=r.`studentno`
WHERE `studentresult`>=80 AND `subjectno`=(
SELECT subjectno FROM `subject` WHERE `subjectname`='高等数学-2'
)

-- 在改造
SELECT `studentno``studentname` FROM student WHERE `studentno` IN(
	SELECT studentno FROM `result` WHERE `studentresult`>=80 AND `subjectno`=(
		SELECT subjectno FROM `subject` WHERE `subjectname`='高等数学-2'
	)
)
-- 练习：查询 c语言-1 前5名同学的成绩的信息(学号，姓名，分数)，使用子查询
SELECT s.`studentno`,`studentname`,`studentresult`
FROM `student` s
INNER JOIN `result` r
ON r.`studentno`=s.`studentno`
WHERE `subjectno`=(
SELECT `subjectno` FROM `subject` WHERE `subjectname`= 'c语言-1'
)
```



## MySQL函数

官网：https://dev.mysql.com/doc/refman/5.7/en/built-in-function-reference.html

### 常用函数

```sql
-- 数学运算
SELECT ABS(-8) -- 绝对值
SELECT CEILING(9.4) -- 向上取整
SELECT FLOOR(9.4) -- 向下取整
SELECT RAND() -- 返回一个0~1的随机数
SELECT SIGN(-12) -- 判断一个数的符号 0-1 负数返回-1，正数返回1

-- 字符串函数
SELECT CHAR_LENGTH('我爱你中国') -- 字符串的长度
SELECT CONCAT('我','爱','你中国')  -- 拼接字符串
SELECT INSERT('我爱你中国',1,5,'是真的') -- 插入，如插入的字符串的位置有原字符就会替换，索引从1开始
SELECT LOWER('woaiNIZHONGGUO') -- 小写
SELECT UPPER('woainizhongguo') -- 大写
SELECT REPLACE('坚持就能成功','坚持','努力') -- 替换指定的字符串
SELECT INSTR('woainizhongguo','g')-- 返回第一次出现的字符位置
SELECT SUBSTR('我爱你中国',1,4) -- 返回指定的字符串（原字符串，截取的位置，截取的长度）
SELECT REVERSE('清晨我上马') -- 反转字符串

-- 查询姓小的同学，并把小改为大
SELECT REPLACE(studentname,'小','大') FROM 
student WHERE studentname LIKE '小%'

-- 时间和日期函数（记住）
SELECT CURRENT_DATE() -- 获取当前的日期
SELECT CURDATE() -- 获取当前的日期
SELECT LOCALTIME() -- 本地时间
SELECT SYSDATE() -- 系统时间
SELECT NOW() -- 获取当前的时间

SELECT YEAR(NOW())  -- 获取年
SELECT MONTH(NOW()) -- 获取月
SELECT DAY(NOW())   -- 获取日
SELECT HOUR(NOW())  -- 获取时
SELECT MINUTE(NOW()) -- 获取分
SELECT SECOND(NOW()) -- 获取秒

-- 系统
SELECT SYSTEM_USER() -- 获取系统当前用户
SELECT USER() -- 获取系统当前用户
SELECT VERSION() -- 获取mysql版本
```



### 分组

使用group by 后不能使用where，如果要过滤条件使用having

```sql
-- 查询不同课程的平均分，最高分，最低分，且平均分大于80
-- 核心：（根据不同的课程进行分组）
SELECT `subjectname`,AVG(`studentresult`) AS 平均分,MAX(`studentresult`) AS 最高分,MIN(`studentresult`) AS 最低分
FROM `result` r
INNER JOIN `subject` sub
ON sub.`subjectno`=r.`subjectno`
GROUP BY subjectname -- 通过什么字段分组
HAVING 平均分>80 AND 最高分 < 95 
```





### 聚合函数



```sql
-- ===========聚合函数=========
-- 都能统计表中的数据
SELECT COUNT(`studentno`) FROM student -- count(字段)，会忽略所有的null值,没有主键中这个最快
SELECT COUNT(*) FROM `student` -- count(*) 不会忽略null值， 本质是在计算行数
SELECT COUNT(1) FROM `student` -- COUNT(1)  不会忽略null值， 本质是在计算行数

SELECT SUM(`studentresult`) AS 总和 FROM `result` 
SELECT AVG(`studentresult`) AS 平均分 FROM `result` 
SELECT MAX(`studentresult`) AS 最高分 FROM `result` 
SELECT MIN(`studentresult`) AS 最低分 FROM `result` 
```



### MD5加密

MD5加密是不可逆，但是md5具体的值是确定的

```sql
-- =========测试MD5 加密==============
CREATE TABLE `testmd5`(
	`id` INT(4) NOT NULL,
	`name` VARCHAR(20) NOT NULL,
	`pwd` VARCHAR(50) NOT NULL,
	PRIMARY KEY (`id`)

)ENGINE=INNODB DEFAULT CHARSET=utf8


-- 明文密码
INSERT INTO `testmd5` VALUES(1,'zhangsan','123456'),(2,'lisi','123456'),(3,'wangwu','123456')

-- md5 加密
UPDATE `testmd5` SET pwd=MD5(pwd) -- 加密全部的密码

UPDATE `testmd5` SET pwd=MD5(pwd) WHERE id != 1 

-- 插入的时候加密
INSERT INTO `testmd5` VALUES(4,'小明',MD5('123456'))
-- 如何校验：将用户传递进来的密码，进行md5加密，然后对比加密后的值
SELECT * FROM `testmd5` WHERE `name`='小明' AND pwd=MD5('123456')
```



## 事务

>事务原则：ACID原则 即 原子性 一致性 隔离性 持久性

参考博客连接： https://blog.csdn.net/dengjili/article/details/82468576

原子性：一组sql的执行要么同时成功要么同时失败

一致性：针对一个事务操作前与操作后的数据库状态是一致的

​		比如  A（800） B（200） A->B(200) 此时 A还有600 B有400 ，他们操作前和操作后都是1000

持久性： 表示事务结束后的数据不随着外界原因导致数据丢失，也就是，一顿操作后，如果没有提交事务，此时服务器宕机，重启后的				数据还是之前没有操作的数据，如果提交后宕机，那么重启后数据就变成提交后的数据了（事务一旦提交就不可逆了）

隔离性：针对多个用户同时操作，主要是排除其他事务对本次事务的影响，比如A和C同时在事务中给B转钱，这两个事务是不会影响的





### 事务隔离级别

脏读：一个事务读取到另一个事务未提交的数据

不可重复度：在一个事务内读取表中的某一行数据，多次读取结果不同。(这个不一定是错误，只是某些场合不对)

虚读（幻读）：是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。(一般是行影响，多了一行)



>执行事务的流程

```sql
-- =========测试事务=======

-- mysql默认事务是提交的
SET autocommit = 0 -- 设置为0 自动提交事务关闭
SET autocommit = 1 -- 设置为1 自动提交事务开启，默认值

-- 手动处理事务

SET autocommit = 0 -- 设置为0 自动提交事务关闭
-- 事务开启
START TRANSACTION -- 标记一个事务的开启，从这个之后的sql都在同一个事务中执行

-- 提交： 持久化（成功）
COMMIT
-- 回滚：回到原来的样子
ROLLBACK
-- 关闭事务
SET autocommit = 1 -- 设置为1 事务开启



-- 了解

SAVEPOINT 保存点名-- 设置一个事务的保存点
ROLLBACK TO SAVEPOINT 保存点名-- 回滚到保存点
RELEASE SAVEPOINT保存点名-- 撤销保存点
```



>模拟事务场景

```sql
CREATE DATABASE `shop` CHARACTER SET utf8 COLLATE utf8_general_ci -- COLLATE 字符集校对
USE shop
CREATE TABLE `account`(
	`id` INT(3) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(30) NOT NULL,
	`money` DECIMAL(9,2) NOT NULL, -- DECIMAL 参数表示有9位数，其中有两位是小数
	PRIMARY KEY(`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO `account`(`name`,`money`) VALUES('A',2000.00),('B',10000.00)

-- 模拟转账
SET autocommit=0 -- 关闭事务自动提交
START TRANSACTION -- 开启一个事务（以下更新为一组事务）
UPDATE `account` SET `money`=`money`-500 WHERE `name`='A'

UPDATE `account` SET `money`=`money`+500 WHERE `name`='B'

COMMIT; -- 事务成功则提交
ROLLBACK; -- 如果事务失败，则回滚
SET autocommit=1 -- 恢复自动提交
```



## 索引

>MySQL官方对索引的定义为：**索引（Index）是帮助MySQL高效获取数据的数据结构。**
>
>提取句子主干，就可以得到索引的本质：索引是数据结构。

### 索引的分类

sql -explain： https://blog.csdn.net/jiadajing267/article/details/81269067

>在一个表中，主键索引是只能有一个，唯一索引可以有多个

- 主键索引（PRIMARY KEY）
  - 唯一的标识，主键是不可重复的，只能有一个列作为主键
- 唯一索引（UNIQUE KEY）
  - 避免有多个重复的列出现，唯一索引是可以重复的，多个列都可以标识为唯一索引
- 常规索引 （KEY/INDEX）
  - 默认的，使用index 或者key 来设置
- 全文索引 （FULLTEXT）
  - 在特定的数据库引擎下才有 MYISAM
  - 快速定位数据





基础语法

```sql
-- ===========索引的使用=============
-- 1、在建表的使用给字段添加索引
-- 2、建表完成后，增加索引

-- 显示所有的索引信息
SHOW INDEX FROM student

-- 增加一个全文索引  索引名(列名)
ALTER TABLE student ADD FULLTEXT INDEX `studentname`(`studentname`)

-- explain 分析sql执行的状况
EXPLAIN SELECT * FROM student   -- 非全文索引

EXPLAIN SELECT * FROM student WHERE MATCH(`studentname`) AGAINST('小')
```



### 测试索引

```sql
CREATE TABLE `app_user` (
`id` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT,
`name` VARCHAR(50) DEFAULT '' COMMENT '用户昵称',
`email` VARCHAR(50) NOT NULL COMMENT '用户邮箱',
`phone` VARCHAR(20) DEFAULT '' COMMENT '手机号',
`gender` TINYINT(4) UNSIGNED DEFAULT '0' COMMENT '性别（0：男；1：女）',
`password` VARCHAR(100) NOT NULL COMMENT '密码',
`age` TINYINT(4) DEFAULT '0' COMMENT '年龄',
`create_time` DATETIME DEFAULT NULL,
`update_time` TIMESTAMP NOT NULL ,
PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4 COMMENT='app用户表'


DELIMITER $$ -- 写函数之前必须要写，标志
CREATE FUNCTION mock_data()
RETURNS INT
BEGIN
	DECLARE num INT DEFAULT 1000000;
	DECLARE i INT DEFAULT 0;
	
	WHILE i<num DO
		INSERT INTO app_user(`name`,`email`,`phone`,`gender`,`password`,`age`,`create_time`,`update_time`)
		VALUES(CONCAT('用户',i),'24736743@qq.com',CONCAT('18',FLOOR(RAND()*((999999999-100000000)+100000000))),FLOOR(RAND()*2),UUID(),FLOOR(RAND()*100),SYSDATE(),SYSDATE());
		SET i=i+1;
	END WHILE;
	RETURN i;

END;
SELECT mock_data()

SELECT * FROM `app_user` WHERE `name`='用户99999'

EXPLAIN SELECT * FROM `app_user` WHERE `name`='用户99999'

-- 索引名命名规则：id_表名_字段名
-- CREATE [unique|primary] INDEX 索引名 on 表名(字段名)
CREATE INDEX id_app_user_name ON `app_user`(`name`)

SELECT * FROM `app_user` WHERE `name`='用户99999'

EXPLAIN SELECT * FROM `app_user` WHERE `name`='用户99999'
```

添加索引后搜索的行数

![image-20210811173151980](.\typora-user-images\image-20210811173151980.png)

没有添加索引搜索的行数

![image-20210811173304805](.\typora-user-images\image-20210811173304805.png)

==索引在小数据量的时候，用处不大，但是在大数据的时候，区别非常明显==





### 索引原则

- 索引不是越多越好
- 不要对经常变动的数据加索引
- 小数据量的表不需要加索引
- 索引一般加在常用来查询的字段上！



>索引的数据结构

Hash类型的索引

Btree：Innodb的默认数据结构



## 权限管理与备份

>sql yog 可视化管理

![image-20210811202759984](.\typora-user-images\image-20210811202759984.png)

>SQL 命令操作

用户表：mysql下的User    mysql.user

```sql
-- 创建用户 CREATE USER 用户名 IDENTIFIED BY 密码
CREATE USER rlxiang IDENTIFIED BY '123456'

-- 修改密码（修改当前用户的密码）
SET PASSWORD = PASSWORD('123456') 

-- 修改指定用户的密码
SET PASSWORD FOR rlxiang = PASSWORD('123456')


-- 给用户重命名 RENAME USER 原来用户名 TO 新的用户名
RENAME USER rlxiang TO rlxiang2

-- 用户授权 GRANT ALL PRIVILEGES(所有权限) ON *.* TO rlxiang2
-- ALL PRIVILEGES 除了给别人授权，其他的事都能干
GRANT ALL PRIVILEGES ON *.* TO rlxiang2

-- 查看权限
SHOW GRANTS FOR rlxiang2 -- 查看指定用户的权限
SHOW GRANTS FOR root@localhost 

-- GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD '*81F5E21E35407D884A6CD4A731AEBFB6AF209E1B' WITH GRANT OPTION 
-- 撤销权限 revoke 那些权限，在那个库撤销，给谁撤销
REVOKE ALL PRIVILEGES ON *.* FROM rlxiang2

-- 删除用户
DROP USER rlxiang2
```



### MySQL备份

为什么要备份：

- 保证重要的数据不丢失
- 数据转移

MySQL数据库备份的方式

- 直接拷贝物理文件

- 在sqlyog这种可视化工具中手动导出

  - 在想要导出的表或者库中，右键，选择备份或导出

    ![image-20210811205420319](.\typora-user-images\image-20210811205420319.png)

- 使用命令行导出 mysqldump  命令行使用

  ```bash
  # mysqldump -h 主机 -u用户名 -p密码 数据库表名 >物理磁盘位置/文件名
  mysqldump -h1ocalhost -uroot -p123456 schoo1 student >D:/a.sq1
  
  # dump多张表
  # mysqldump -h 主机 -u用户名 -p密码 表1 表1 表1 >物理磁盘位置/文件名          
  mysqldump -h1ocalhost -uroot -p123456 schoo1 student >D:/b.sq1
  
  # dump数据库
  # mysqldump -h 主机 -u用户名 -p密码 数据库 >物理磁盘位置/文件名          
  mysqldump -h1ocalhost -uroot -p123456 schoo1  >D:/c.sq1
  
  
  # 导入
  # 最好在登陆的状态下导入，先切换到指定的数据库
  # 然后 source，如果是直接导入数据库，就直接在后面跟上备份文件的地址即可
  # 如果是导入一张表，就先切换到指定的数据库中，然后source 备份文件地址
  source d:/a.sql
  
  # 如果没有登录，则使用下面的语法
  mysql -u用户名 -p密码 库名<备份文件地址
  ```

  假设要备份数据库，防止数据丢失，直接把sql给别人。

## 三大范式

**第一范式（1NF）**

原子性：保证每一列都是不可再分的

**第二范式（2NF）**

前提:满足第一范式

每张表只描述一件事情

**第三范式**

前提:满足第一范式和第二范式

第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。



**规范性和性能的问题**

关联查询的表不得超过三张表

- 考虑商业化的需求和目标，(成本，用户体验!）数据库的性能更加重要
- 在规范性能的问题的时候，需要适当的考虑一下规范性!
- 故意给某些表增加一些冗余的字段。(从多表查询中变为单表查询)
- 故意增加一些计算列(从大数据量降低为小数据量的查询:索引)



## JDBC

### 数据库驱动

每种类型的数据库都有其对应的数据库驱动

![image-20210811212042070](.\typora-user-images\image-20210811212042070.png)



### JDBC

SUN 公司为了简化开发人员的操作（对数据库的统一操作），提供了一个（Java操作数据库的）规范，俗称JDBC

这些规范的实现由具体的厂商实现，对开发人员来说，我们只需要掌握JDBC接口的操作即可

![image-20210811212437054](.\typora-user-images\image-20210811212437054.png)

![image-20210811213726004](.\typora-user-images\image-20210811213726004.png)

### sql注入

sql 存在漏洞，会被公职导致数据泄露，SQL会被拼接 or

PreparedStatement对象可以防止SQL注入，并且效率更好！
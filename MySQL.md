# MySQL笔记

## 1.基础操作

```mysql
SHOW DATABASES;
CREATE DATABASE MyDemo;
DROP DATABASE MyDemo;
SHOW ENGINES;
SHOW VARIABLES LIKE 'have%';
USE MyDemo;
CREATE TABLE userl (id INT PRIMARY KEY,
                  name VARCHAR(20) NOT NULL,
                  sex BOOLEAN,
                  money FLOAT DEFAULT 0,
                  );
CREATE TABLE grade (stu_id INT,
                   course_id INT,
                   grade FLOAT,
                   PRIMARY KEY(stu_id,course_id)
                   );
CREATE TABLE gradevalid (id INT PRIMARY AUTO_INCREMENT,
                        stu_id INT UNIQUE,
                        course_id INT,
                        CONSTRAINT c_fk FOREIGN KEY(stu_id,course_id)
                          REFERENCES grade(stu_id,course_id)
                        );
DESC userl;
/*修改默认字符集*/
ALTER TABLE name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

SHOW CREATE TABLE userl \G;
ALTER TABLE userl RENAME student;
ALTER TABLE student MODIFY name VARCHAR(30);
ALTER TABLE student MODIFY id sno INT;
ALTER TABLE student ADD phone VARCHAR(20) NOT NULL AFTER sex;
ALTER TABLE grade ADD num INT(8) UNIQUE FIRST;
ALTER TABLE student DROP money;
ALTER TABLE student MODIFY name VARCHAR(30) FIRST;
ALTER TABLE student MODIFY sex BOOLEAN AFTER phone;
ALTER TABLE student ENGINE=MyISAM;
/*
   删除外键之前要先SHOW CREATE TABLE查看外键别名
*/
ALTER TABLE gradevalid DROP FOREIGN KEY c_fk;
/*
   删除表之前需要删除关联（外键）
*/
DROP TABLE gradevalid;
```

## 2.索引

```mysql
//3种创建索引的方式
CREATE TABLE indexdemo(attribute1 TYPE [CONSTRAINT],
                      attribute2 TYPE [CONSTRAINT],
                      [UNIQUE|FULLTEXT|SPATIAL] INDEX|KEY [NAME](attribute [ASC|DESC])
                      );
CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX NAME ON indexdemo2 （attribute[length] [ASC|DESC]);
ALTER TABLE indexdemo3 ADD [UNIQUE|FULLTEXT|SPATIAL] INDEX NAME (attribute[length] [ASC|DESC]);
//删除索引
DROP INDEX NAME ON indexdemo;
```

## 3.视图

```mysql
/*创建*/
CREATE [ALGORITHM={UNDEFINED|MERGE|TEMPTABLE}] VIEW NAME[(attributes)] AS 
SELECT SENTENCES
[WITH[CASCADED|LOCAL] CHECK OPTION]

CREATE ALGORITHM=MERGE VIEW worker_view(name,department,sex,age,address) 
FROM worker,department WHERE worker.d_id=department.d_id
WITH LOCAL CHECK OPTION
/*查看*/
DESCRIBE worker_view;
SHOW TABLE STATUS LIKE 'worker_view';
SHOW CREATE VIEW worker_view\G;
SELECT * FROM infomation_schema.views;//查看所有
/*修改*/
CREATE OR REPLACE VIEW [ALGORITHM={UNDEFINED|MERGE|TEMPTABLE}]
VIEW NAME[(attributes)] AS SELECT SENTENCES
[WITH[CASCADED|LOCAL] CHECK OPTION];//修改内容

ALTER [ALGORITHM={UNDEFINED|MERGE|TEMPTABLE}]
VIEW NAME[(attributes)] AS SELECT SENTENCES
[WITH[CASCADED|LOCAL] CHECK OPTION];//修改内容

//TODO 重命名view好像是不可以的
/*更新*/
 UPDATE department_view SET name='name',function='function',address='address';
 //前提是view涵盖了基本表的内容（未涵盖的有默认值）并且是选定的单条（有限数）
 /*
   视图中包含如下不能update
   SUM(),COUNT(),MAX(),MIN()等函数
   UNION,UNION ALL,DISTINCT,GROUP BY,HAVING等关键字
   常量视图，比如select 'Aric' as name;
   SELECT包含子查询
   不可更新的view导出的view
   ALGORITHM为TEMPTABLE类型
   view对应的表存在没有默认值的列
   不满足WITH LOCAL|CASCADED CHECK OPTION
 */
 
 /*删除*/
 DROP VIEW [IF EXISTS] VIEWNAME [RESTRICT|CASCADE];
 /*user需要拥有drop权限才可以使用，如下查看
 SELECT Drop_priv FROM mysql.user WHERE user='用户名'*/
```

## 4.触发器

```mysql
CREATE TRIGGER NAME BEFORE|AFTER trigger_event ON tablename
FOR EACH ROW 
BEGIN 
SENTENCES;
END

CREATE TRIGGER dept_trig AFTER DELETE ON department FOR EACH ROW
BEGIN
   INSERT INTO trigger_time VALUES('21:01:01');
   INSERT INTO trigger_time VALUES('22:01:01');
END
//查看触发器
SHOW TRIGGERS \G;
SELECT * FROM infomation_schema.triggers;
SELECT * FROM infomation_schema.triggers WHERE TRIGGER_NAME='NAME';
DROP TRIGGER NAME;
//修改：drop然后重新创建
```

##     5.查询

### 5.1基本查询语句格式

```mysql
SELECT attributes FROM tabel|view [WHERE sentence] 
[GROUP BY attribute [HAVING sentence]][WITH ROLLUP]
[ORDER BY attribute [ASC|DESC]];
```

### 5.2 带IN/BETWEEN/LIKE/AND/OR关键字的查询

```mysql
SELECT * FROM employee WHERE d_id [NOT] IN(1001,1004);
SELECT * FROM employee WHERE age [NOT] BETWEEN 14 AND 25;
/*
   %代表任意长度字符串;
   _表示单个字符
   没有通配符时LIKE和'='相同，但是=无法查询带有通配符的字符串
 */
SELECT * FROM employee WHERE name [NOT] LIKE 'Aric'|'b%k'|'b_k';
SELECT * FROM employee WHERE info IS [NOT] NULL;
SELECT * FROM employee WHERE d_id=1001 AND sex LIKE '男'；
/*
   AND优先级大于OR
 */
SELECT * FROM employee WHERE sex='女' OR
num IN(1,3,4)AND age=25;
```

### 5.3对查询结果的处理(distinct,order by,group by,limit)

```mysql
SELECT DISTINCT d_id FROM employee;  //消除重复
SELECT * FROM employee ORDER BY age DESC;  //排序
/*
   GROUP BY关键字需要搭配GROUP_CONCAT(),COUNT(),SUM(),AVG(),MAX(),MIN()等函数一起使用
   单独的GROUP BY只显示分组情况，即每组的一条记录
 */
 SELECT * FROM employee GROUP BY sex;
 //查询sex和name并分组显示name字段
 SELECT sex, GROUP_CONCAT(name) FROM employee GROUP BY sex;
 SELECT sex, COUNT(sex) FROM employee GROUP BY sex;//对sex分组并统计记录数
 /*
    多个字段分组，先按前面的分组，如果重复再依次按后面的字段继续分组
 */
 SELECT * FROM employee GROUP BY d_id,sex;
 /*
    ROLLUP关键字会在所有记录最后加上一条记录，为所有记录的总和或集合
 */
 SELECT sex,COUNT(sex)FROM employee GROUP BY sex WITH ROLLUP;
 /*
    LIMIT 记录数 ;LIMIT 初始位置(0开始)，记录数
 */
 SELECT * FROM employee LIMIT 0,2;
```

### 5.4使用集合函数

```mysql
SELECT COUNT(*) FROM employee;
SELECT num,SUM(score) FROM grade WHERE num=1001;
SELECT AVG(age) FROM employee;
SELECT course,AVG(score) FROM grade GROUP BY course;
/*
   MAX(),MIN()不仅仅适用于数值类型，也适用于字符类型
*/
SELECT MAX(age) FROM employee;
SELECT MIN(name) FROM employee;
```

### 5.5连接查询

```mysql
 /*
   内连接只显示关联表选定字段都相同的记录
*/
 SELECT num,name,employee.d_id,age,sex,d_name,function FROM employee,department
 WHERE employee.d_id=department.d_id
 /*
   LEFT JOIN显示table1的全部记录，table2不存在条目则填充null
   RIGHT JOIN显示table2的全部记录，table1不存在条目则填充null
*/
SELECT attributes FROM table1 LEFT|RIGHT JOIN table2 
ON table1.attribute=table2.attribute;
```

### 5.6子查询

```mysql
(IN,NOT IN,ANY,ALL,EXISTS,NOT EXISTS,'=','!=','>','<'...)
SELECT * FROM employee WHERE d_id IN(SELECT d_id FROM department);
SELECT id,name,score FROM computer_stu WHERE score>=
         (SELECT score FROM scholarship WHERE level=1);
/*
   <>与!=等价;!>与<=等价;!<与>=等价
*/
SELECT * FROM employee
           WHERE EXISTS
                (SELECT d_name FROM department
                     WHERE d_id=1003);//内层返回的是Boolean值
SELECT * FROM computer_stu 
           WHERE score>=ANY
                (SELECT score FROM scholarship);
SELECT * FROM computer_stu
           WHERR score<=ALL
                (SELECT score FROM scholarship);
```

### 5.7合并查询结果

```mysql
SELECT SENTENCE1
   UNION|UNION ALL
SELECT SENTENCE2
   UNION|UNION ALL
SELECT SENTENCEn...
//UNION消除重复项而UNIONALL则不会
```

### 5.8表和字段取别名

```mysql
SELECT * FROM department d WHERE d.d_id=1001;
SELECT d_id AS department_id, d_name AS department_name FROM department;
```

### 5.9正则表达式

```mysql
/*属性名 REGEXP '匹配方式'*/
  ^L       //以L开头
  B$       //以B结尾
  ^L...y$  //L开头y结尾并且中间有三个任意字符
  [abc]    //包含a,b,c中任意一个
  [0-9a-z] [a-zA-Z] //指定集合区间
  [^a-k]   //匹配不包含指定区间内容
  'xdd'      //查询包含'xdd'的记录
  'xdd|pdd|abb'//查询包含任意一个的记录
  a*c      //c之前有0或者多个a的记录
  a+c      //c之前至少有一个a的记录
  'ab{3}'  //出现过'ab'三次的记录
  'ab{1,3}'//出现'ab'至少一次最多三次的记录 
  SELECT * FROM info WHERE name REGEXP 'AB{1,3}';
```

## 6. INSERT , UPDATE and DELETE

### 6.1 INSERT

```mysql
INSERT INTO tablename [attributes] VALUES (attribute1),(attribute2)...(attributeN);
INSERT INTO tablename [attributeS] 
        SELECT attributes FROM tablename2 WHERE sentences;
```

### 6.2 UPDATE

```mysql
UPDATE tablename SET
                 attribute1=value1,attribute2=value2 
                 ...attributeN=valueN
       WHERE sentences;

UPDATE product SET function='护理头发',address='上海市黄浦区'
    WHERE id>=1009 AND id<=1011;
```

### 6.3  DELETE

```mysql
DELETE FORM tablename [WHERE sentences];
```

## 7.MySQL运算符

```java
//算术运算符
+, -, *,  /等价于DIV,  x1%x2等价于MOD(x1,x2)
//比较运算符
=等价于<=>,但是只有<=>可以判断空值
<>等价于!=，均不能判断空值
>, >=, <, <=, IS (NOT) NULL, BETWEEN AND, IN, LIKE, REGEXP(a是否满足正则表达式b)
//逻辑运算符
&&等价于AND, ||等价于OR, !等价于NOT, XOR(异或)
//位运算符
&, |, ~, ^(异或), <<, >>
//优先级
与其他语言相同。实际使用时更多用括号
```

## 8.MySQL函数

### 8.1 数学函数

```JAVA
ABS(x) 绝对值
CEIL(x) 和 CEILING(x) 大于或等于x的最小整数
FLOOR(x) 小于或等于x的最大整数
RAND() 和 RAND(x) 0-1的随机数 RAND(x1)=RAND(x1)
SIGN(x) 返回符号
PI() 圆周率
TRUNCATE(x,y) 保留x到y位小数
ROUND(x) x最近的整数
ROUND(x,y) x到y位小数的值（四舍五入）
SQRT(x) 平方根 MOD(x,y) x/y的余数
POW(x,y) 或 POWER(x,y) x的y次方
EXP(x) e的x次方
```

### 8.2字符串函数

```java
CHAR_LENGTH(s) 字符串字符数
LENGTH(s) 字符串长度
CONCAT(s1,s2,...) 合并字符串
CONCAT_WS(x,s1,s2,...) 合并并用x隔开
INSERT(s1,x,len,s2) s1中x(0开始)位置开始长度为len的字符串替换为s2
UPPER(s) 和 UCASE(s) 转大写 
LOWER(s) 和 LCASE(s) 转小写
LEFT(s,n) s的前n个字符  RIGHT(s,n) s的后n个字符
LPAD(s1,len,s2) s2填充到s1头部使长度到len, 如果不够则重复s2，超出则截断s2
RPAD(s1,len,s2) s2填充到s1尾部使长度到len, 如果不够则重复s2，超出则截断s2
LTRIM(s) 去掉s头部的空格 RTRIM(s) 去尾部空格 TRIM(s) 去掉s前后空格
TRIM(s1 FROM s) 去掉s头尾的s1（可能是多个连续的s1）
REPEAT(s,n) s重复n次
SPACE(n) 返回n个空格
REPLACE(s,s1,s2) s2替换s中的s1
STRCMP(s1,s2) 比较字符串s1,s2的大小并返回（1,0，-1）
SUBSTRING(s,n,len) 从s的第n个位置开始获取长度为len的字符串
MID(s,n,len) 同SUBSTRING
LOCATE(s1,s) , POSITION(s1 IN s) , INSTR(s,s1) 从s中获取s1开始的位置
REVERSE(s) 字符串s逆序
ELT(n,s1,s2,...) 返回第n个字符串sn
FIELD(s,s1,s2,...) 返回第一个与s匹配的字符串的位置（从1开始）
FIND_IN_SET(s1,s2) :{
    FIND_IN_SET('like','i,like,bei,jing')=2,
}
MAKE_SET(x,s1,s2,...)按X的二进制数从右到左数来取字符串：{
   	MAKE_SET(11,'a','b','c','d') 11=1011 所以取a,b,d  
}
```

### 8.3日期与时间函数

```java
//获取日期时间季度星期的函数略
TIME_TO_SEC(t) SEC_TO_TIME(t) 时间和秒钟转换
DATEDIFF(d1,d2) d1,d2之间隔的天数
ADDDATE(d,n) 加天数 SUBDATE(d,n) 减天数 ADDTIME(t,n) 加秒 SUBTIME(t,n) 减秒
ADDDATE(d,INTERVAL expr type) 和 DATE_ADD(d,INTERVAL expr type) \
SUBDATE(d,INTERVAL expr type)
ADDDATE(dt,INTERVAL '1 1' YEAR_MONTH)
```

![img](file:///C:\Users\Seattle\Documents\Tencent Files\835228063\Image\Group\V3_444}3I[~7[3`_TC~D2S5.jpg)

```java
DATE_FORMAT(d,f) 按f格式化d
TIME_FORMAT(t,f) --
```

### 8.4条件判断函数

```java
IF(expr,v1,v2) expr?v1:v2
IFNULL(v1,v2)  v1?v1:v2
CASE [WHEN expr1 THEN v1 ...WHEN exprN THEN vn][ELSE vn+1] END
CASE expr [WHEN e1 THEN v1...][ELSE vn] END
```

### 8.5系统信息函数

```mysql
VERSION(), CONNECTION_ID(),DATABASE(),SCHEMA()
USER(),SYSTEM_USER(),SESSION_USER(),CURRENT_USER(),CURRENT_USER()
CHARSET(str),COLLATION(str)
LAST_INSERT_ID()
```

### 8.6加密函数

```mysql
PASSWORD(str) 不可逆
MD5(str) 
ENCODE(str,pswd_str)用pswd_str加密str
DECODE(crypt_str,pswd_str)
```

### 8.7其他

```java
FORMAT(x,n) 保留x的n位小数
ASCII(s) s第一个字符的ASCII码
BIN(x) 返回x的二进制码
HEX(x) 十六进制
OCT(x) 八进制
CONV(x,f1,f2) x从f1进制变成f2进制
INET_ATON('IP') INET_NTOA(n) IP地址和数字转换
GET_LOCT(name,time) 成功1 超时0 错误NULL
RELEASE_LOCK(name)  成功1 超时0 错误NULL
FREE_LOCK(name) 判断是否使用名为name的锁 使用0,未使用1
BENCHMARK(count,expr) 重复expr count次然后返回执行时间
CONVERT(s USING cs) s字符集改为cs
CAST(x AS type) 将字段x变为type类型 同 CONVERT(x,type)
```


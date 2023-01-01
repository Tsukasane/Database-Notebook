# 数据库笔记（BNU MOOC版）

## Chapter 3 PostgreSQL数据定义与操作

[TOC]

### 3.1 SQL与PostgreSQL

- PostgreSQL（PG）结构化查询语言
- SQL字面含义是“查询语言”，但其功能包括数据定义、查询、修改和保护等许多内容。
- SQL语言是大小写不敏感的。建议SQL保留字用大写，一般所有对象名用小写
- 查询（SELECT）语句中其它子句都可以不出现，但==至少要有一个SELECT子句。==
- 执行过程不是真实的，实际中数据库管理系统会为了尽可能快地获得等价查询结果而采取完全不同的执行步骤。



### 3.2 数据定义

* 定义表的属性时必须指明数据类型

* 插入元组可以直接插入常量元组，可以插入查询结果，可以插入表。插入元组的属性值必须在属性域中，==属性值空null也在属性域中==

* PG中使用==单引号==做字符串常量的标识，如果要查询带单引号的字符串，需要在其单引号后再加一个单引号，或者在 ’ 符号前加一个转义符号 \ ，并在整个字符串前加一个 E 字母。

* WHERE 和 HAVING虽然后面都跟着一个条件表达式，但是它们不可以互换使用。没有FROM就没有WHERE，没有GROUP BY 就没有HAVING

* $*$ 和 $@$ 可以得到表中的所有列

* 一些SQL语句举例

  ```sql
  //检索数据
  FROM
  ```

  ```sql
  //数据查询
  SELECT
  
  //查询工资为4000或为8000的所有考官
  SELECT * FROM examiner WHERE ersalary IN(4000,8000);
  ```

  <!--IN 后面跟的是列举的集合，不是区间-->

  ```sql
  //数据定义
  CREATE	
  DROP
  ALTER
  
  //---------------------------------------------------------------------------------------
  
  //创建表
  CREATE TABLE
  
  //修改表的结构
  ALTER TABLE
  
  //把表examiner的名字改为erexamine
  ALTER TABLE examiner RENAME TO erexamine;
  //向examiner表增加属性，属性名为er_entrance，数据类型为日期型
  ALTER TABLE examiner ADD COLUMN er_entrance DATE;
  //将erage的数据类型由SMALLINT型改为INT型
  ALTER TABLE examiner ALTER COLUMN erage TYPE INT;
  //删除examiner中的erage列
  ALTER TABLE examiner DROP COLUMN erage;
  
  ```

  ```sql
  //数据操控
  UPDATE
  INSERT
  DELETE
  
  //---------------------------------------------------------------------------------------
  
  //使examiner表中所有大于30岁考官的年龄+1
  UPDATE examiner SET erage=erage+1
  WHERE erage>30;
  
  
  //向表中插入一个常量元组
  INSERT INTO examinee(eedepa, eeage, eeid, eesex, eeename) VALUES ('历史学院', 20, '218811011016', '男', '张强');
  INSERT INTO examinee VALUES ('历史学院', 20, '218811011016', '男', '张强');
  ```

  ```sql
  //数据控制
  GRANT
  DELETE
  ```

  

### 3.3 单表查询

* 投影：投影是指选取表中某些列的列值；广义投影是指在选取属性列时，允许进行适当运算。

* ORDER BY子句，按照后跟属性的出现先后决定优先级排序。升序用ASC，排序列为空值的行最后显示，降序用DESC，排序列为空值的行最先显示（空最大）。默认为升序
* 投影结果中可能出现所有列值均相等的重复行，DBMS采取惰性原则，非特别要求（DISTINCT）时==保留重复==。在关系代数向SQL语句转化的时候，需要注意加上DISTINCT
* HAVING子句的条件只对GROUP BY子句形成的分组起作用
* 关系代数中的一元运算有选择运算σ，投影运算Π，更名运算ρ三个。选择运算σ的作用相当于SQL语句中的WHERE，投影运算Π相当于SQL语句中的SELECT，更名运算ρ相当于SQL语句中的AS
* ==聚集函数不能进行复合运算==，聚集中SUM和AVG的输入必须是数值类型，其他聚集可以有非数值类型的输入



### 3.4 联接查询

* 外联接都要首先计算内联接，然后再在内联接的结果中加上相应的表中的悬浮行。e.g.**右外联接**就是把右侧表中的悬浮行补上空值后加入结果表，这些行中**来自左侧表的属性赋为空值null**
* 关键词顺序 e.g. NATURAL LEFT OUTER JOIN，==OUTER可以省略==，LEFT，RIGHT，FULL默认包含了OUTER
* 内联接INNER，外联接OUTER，默认为INNER



### 3.5 嵌套查询

* PG提供嵌套子查询机制，一个SELECT-FROM-WHERE语句是一个查询块
* ==不相关嵌套查询==：如果内层子查询不依赖于外层查询，称为不相关嵌套查询，可**由内向外逐层**处理。内层子查询的结果用于外层查询。
* ==相关嵌套查询==：这种情况下，对外层查询表中的**每一行**，根据它与内层子查询相关的列值处理内层查询，直至外层查询处理完为止。

#### 表式嵌套

* 出现位置：WITH子句，FROM子句后

* 查询块可以出现在另外一个查询中表名可以出现的任何地方（子块返回一个表）

* **WITH子句** 定义一个临时表，在SELECT子句之前给出

  ```sql
  //查询学生中平均分>=80的人数
  WITH avgach(eeid, avgchieve) AS
  					(SELECT eeid, AVG(achieve)
             FROM eeexam
             GROUP BY eeid)
  SELECT COUNT(*)
  FROM avgach
  WHERE avgachieve>=80;
  ```

* **FROM子句** 中的AS可以省略不写，下面AS的括号表示可选，不是要加括号

  ```sql
  //查询学生中平均分>=80的人数
  SELECT COUNT(*)
  FROM (SELECT eeid, AVG(achieve)
        FROM eeexam
       	GROUP BY eeid) (AS) avgach(eeid, avgachieve)
  WHERE avgachieve>=80;
  ```

#### 集合式嵌套

* 出现位置：WHERE子句，HAVING子句后
* HAVING只对GROUP BY中的属性做限制，如果没有GROUP BY，则用WHERE即可。
* 当SELECT后面同时出现聚集和不聚集的属性，==不聚集的属性一定要出现在GROUP BY子句中==。
* 查询块也可以出现在集合能够出现的任何的地方
* **IN关键词**，判断是否属于子集合
* **EXISTS关键词**，如果内层查询的结果为空，则返回true，否则返回false
* **比较运算符**，
  * <>ALL：和所有的都不相等，没有出现
  * <>SOME（ANY）：和部分（某个）不相等，一般用来看两个集合有没有共同元素
  * =SOME（ANY）：和部分（某个）相等，一般用来看有没有交集
  * =ALL：和所有相等，一般用来看两个集合是否相等

* **WHERE子句**

  ```sql
  //查询218811011013号考生报考的试卷号和试卷名
  SELECT eid, ename
  FROM exampaper
  WHERE eid IN(SELECT eid
               FROM eeexam
               WHERE eeid='218811011013');
  ```

  ```sql
  //查询所有报考了0205000002号试卷的考生详细信息
  SELECT *
  FROM examinee
  WHERE EXISTS(SELECT *
               FROM eeexam
               WHERE examinee.eeid=eeid AND eid='0205000002');
  ```

* **HAVING子句**

  ```sql
  //查询有名叫刘诗诗的考生的学院的考生平均年龄
  SELECT eedepa, AVG(eeage)
  FROM examinee
  GROUP BY eedepa
  HAVING eedepa IN(SELECT eedepa
                   FROM examinee
                   WHERE eename='刘诗诗');
  ```

#### 标量式嵌套

* 出现位置：如果能确定查询块只返回单行单列的单个值，查询块可以出现在**单个属性名、单个表达式、单个常量，即单值表达式能够出现的任何地方。**
* 举例：SELECT子句，WHERE子句，ORDER BY子句、LIMIT子句（后跟数值，要求子查询结果为数值，限制显示条数）、OFFSET子句（后跟数值，要求子查询结果为数值，限制显示起始位置）后

* **WHERE子句**

  ```sql
  //查询与218811011028号考生同院系的考生的报考号、姓名
  SELECT eeid, eename
  FROM examinee
  WHERE eedepa = (SELECT eedepa
                  FROM examinee
                  WHERE eeid='218811011028');
  ```

* **SELECT子句** 和 **GROUP BY**子句

  ```sql
  //查询每个院系的平均分
  SELECT (SELECT eedepa
          FROM examinee
          WHERE examinee.eeid=eeexam.eeid),
          AVG(achieve)
  FROM eeexam
  GROUP BY (SELECT eedepa
            FROM examinee
            WHERE examinee.eeid=eeexam.eeid);
  ```

  <!--要查的两个属性分布在两个表中，选一个为主表（eeexam），另一个为子表（examinee），主表和子表之间通过eeid属性相等建立联系-->

* **LIMIT子句** 和 **OFFSET子句** 

  ```sql
  //对examinee表按eeid升序，从考生人数1/4处开始，列出1/4的考生信息
  SELECT *
  FROM examinee
  ORDER BY eeid
  LIMIT (SELECT COUNT(*)/4
         FROM examinee)
  OFFSET (SELECT COUNT(*)/4
          FROM examinee);
  ```

  <!--ORDER BY默认ASC-->

* **SELECT子句** 和 **ORDER BY子句**

  ```sql
  //查询每个院每个考生得分，列出院名、考生名和分数，按院名升序排列，同院按分数升序排列
  SELECT (SELECT eedepa
          FROM examinee
          WHERE examinee.eeid=eeexam.eeid),
         (SELECT eename
          FROM examinee
          WHERE examinee.eeid=eeexam.eeid),
          achieve
  FROM eeexam
  ORDER BY (SELECT eedepa
            FROM examinee
            WHERE examinee.eeid=eeexam.eeid),
            achieve ASC;
  ```

  <!--如果多个属性都需要用子查询得到，每个属性需要一个单独的子查询块-->
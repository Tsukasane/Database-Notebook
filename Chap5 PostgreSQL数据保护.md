# 数据库笔记（BNU MOOC版）

## Chapter 5 PostgreSQL数据保护

[TOC]

### 5.1 数据保护

#### 数据安全性

**1、数据保护保密性**：指仅允许经授权地读数据

* 数据值的保密
* 数据存在性的保密

**2、数据完整性**：指数据的可信度。通常保护完整性就是指==仅允许合法授权的数据修改==

* 数据值的完整性

* 数据来源的完整性

**3、数据可用性**：是指对数据的期望访问能力。保护数据可用性通常指减少数据库系统停工时间，保持数据持续可访问。

#### 数据保护

* 安全策略：允许什么，禁止什么的说明

* 安全机制：实施策略
* **访问控制矩阵**：描述系统访问策略的最简单模型，矩阵的行对应用户或角色，列对应数据库对象，矩阵中的元素表示相应用户对相应数据库对象的访问权限，访问控制矩阵通常依据用户在系统应用中担任的角色确定。
* PG中的==授权和收权语句==可以赋予或撤销用户相应的访问权限，数据库管理系统确保只有获得授权的用户有资格访问数据库对象，令所有未授权人员无法接近，从而保护数据保密性和完整性。



### 5.2 视图

#### 视图和表的异同

* 视图和表都是关系，都可在查询中直接应用

* DB中存储表的模式定义和数据；但==只存储视图的定义，不存视图的数据==，视图数据在**使用视图时临时计算**。

#### 视图的使用机制

* **数据的逻辑独立性**：视图定义中引用的表成为基表，**当基表中的数据发生变化时，相应的视图数据也随之改变**。视图可以把基表结构的细节封装起来，表可以随应用进化而变化，但视图以及基于视图的应用程序，可以尽可能少地受表变化的影响，
* **数据存在性的保密性保护**：用户只能查询和修改视图中见得到的数据。

#### 相关SQL语句

* 创建视图

  ```sql
  //基于表创建视图
  CREATE VIEW avgachieve(eeid, average) AS
  	SELECT eeid, AVG(achieve)
  	FROM eeexam
  	GROUP BY eeid;
  ```

  <!--组成视图的属性列名，可以全部省略或全部指定，视图属性名省略时，取SELECT结果关系属性名。-->

  ```sql
  //使用视图创建视图
  //首先基于表创建视图eeexamv1
  CREATE VIEW eeexamv1 AS
  	SELECT examinee.eeid, examinee.eedepa, 
  				 exampaper.ename, eeexam.achieve
  	FROM examinee, exampaper, eeexam
  	WHERE examinee.eeid=eeexam.eeid 
  				AND eeexam.eid=exampaper.eid;
  				
  //接着使用eeexamv1创建视图eeexamv2
  CREATE VIEW eeexamv2 AS
  	SELECT eeid, ename, achieve
  	FROM eeexamv1
  	WHERE eedepa='历史学院';
  ```

  <!--用到SELECT结果关系属性名时，视图中的属性若无重名可以不再对应原表-->

* 修改视图

  **PG只允许对可更新视图进行修改操作**，可更新视图需要满足如下条件：

  - 视图是从单个关系只使用投影、选择操作导出
  - SELECT子句中只包含属性名，不包含其它表达式、聚集、DISTINCT声明等，查询中没有GROUP BY或HAVING子句，WHERE子句子查询不出现基表名。并且对视图的更新操作符合一般更新语句的规则，比如插入时主键不能为空，执行插入操作的视图的投影列需包含基础关系的键
  - 比如，对视图进行INSERT操作，实际是对基表的对应位置进行了对应属性的插入（元组的其他位置为空）；如果在视图定义的末尾包含**WITH CHECK OPTION**，数据库管理系统自动检查对视图的更新应满足视图定义中**WHERE**的条件，如果不满足此插入操作便会被拒绝执行。



### 5.3 访问控制

* 给定用户拥有在给定数据库对象上的给定操作权限
  * 用户
  * 数据对象
  * 操作权限

#### 用户

* PG使用角色来统一管理用户，分登陆角色和组角色。登陆角色就是具有登陆权限的角色，相当于用户。组角色就是作为组使用的角色，一般不应当具有LOGIN属性。

* 默认情况下，新建立的数据库总是包含一个超级用户角色，并且默认这个角色名是postgres

* 创建角色

  ```sql
  //创建组角色yanni
  CREATE ROLE yanni;
  
  //创建具有登陆权限的组角色yuxiaotong
  CREATE ROLE yuxiaotong LOGIN;
  
  //创建可创建数据库的组角色masu
  CREATE ROLE masu CREATEDB;
  
  //创建可创建角色的组角色lichen
  CREATE ROLE lichen CREATEROLE;
  
  //创建具有口令的组角色wangxi
  CREATE ROLE wangxi PASSWORD ‘654321’;
  
  //创建数据库超级用户角色nini
  CREATE ROLE nini SUPERUSER;
  ```

  <!--只有超级用户才有权创建超级用户-->

* 变更角色

  ```sql
  //修改组角色名yanni为newyanni
  ALTER ROLE yanni RENAME TO newyanni;
  
  //给组角色yangchen添加创建角色和数据库的这个权限
  ALTER ROLE yangchen CREATEROLE CREATEDB;
  
  //收回组角色yangchen创建角色和数据库的权限
  ALTER ROLE yangchen NOCREATEROLE NOCREATEDB;
  ```

  为了与SQL标准兼容，PostgreSQL也有用户管理命令，该角色管理的命令很相似。

  ```sql
  //创建用户lini
  CREATE USER nini;
  和 CREATE ROLE nini LOGIN; 等价
  
  //删除组角色yangni
  DROP ROLE yangni;
  
  //删除用户wangni
  DROP USER wangni;
  ```

* **组角色**权限

  ```sql
  CREATE ROLE Alice LOGIN INHERT
  CREATE ROLE Bob NOINHERIT
  CREATE ROLE Rose NOINHERIT
  GRANT Bob TO Alice
  GRANT Rose TO Bob
  ```

  * 在以角色Alice连接之后，该数据库会话将立刻拥有直接赋予Alice的权限加上任何赋予Bob的权限，因为Alice“继承”Bob的权限。不过属于Rose的权限不可用，因为即使Alice是Rose的一个间接成员，但该成员关系是通过Bob过来的，而该组有NOINHERIT属性
  * 在SET ROLE Bob之后，该会话将只拥有那些已经赋予Bob的权力，而不包含那些已经赋予Alice的权限。在CREATE ROLE Rose之后，该会话将只能使用已赋予Rose的权限，而不包括已赋予Alice或Bob的权限

* PG提供**GRANT语句**和**REVOKE语句**来给角色**授予或撤销数据库操作权限**

  ```sql
  GRANT UPDATE(erid)，SELECT
  ON TABLE examiner
  TO uYing;
  ```

  <!--当这个GRANT语句包含WITH GRANT OPTION选项的时候，在此语句获得权限的角色可以将所获得权限授予其他角色-->

  ```sql
  REVOKE SELECT
  ON TABLE examiner
  FROM uYing
  ```

  

### 5.4 完整性约束

* DBMS无法保证数据始终与其对应的现实世界状态一致，但可以保证数据始终与系统中明确定义的约束一致。这种约束通常称为完整性约束。

#### 分类

* 主键约束（实体完整性）

  * 不重复，不为空。使用PRIMARY KEY关键词
  * 单属性构成的主键——属性级/元组级约束，多个属性构成的主键——元组级约束，即在所有属性声明的后面 PRIMARY KEY(eeid)。
  * 只有对关系进行插入或修改时，系统才检查主键约束。删除时不检查主键约束。

* 外键约束（引用完整性）：

  * REFERENCES后跟着另一个表中的属性

    ```sql
    CREATE TABLE eeexam{
    	eeid CHAR(9) REFERENCES examinee(eeid),
    	achieve SMALLINT
    };
    ```

  * 引用表插入元组或对外键列进行修改时，或被引用表进行删除或修改时，系统自动检查是否违背外键约束。

  * **违背外键约束**，有以下几种**处理策略：**

    * NO ACTION-拒绝相应操作；
    * RESTRICT-不允许事物检查推迟到晚些时候；
    * CASCADE-级联，删除被引用行的属性值或分别把被引用行的属性值更新为引用属性的新数值；
    * 设为空值或者默认值

* 非空约束

  NOT NULL

* 唯一值约束

  UNIQUE <候选键>，==与主键约束不同，可为空值==

* CHECK约束

  * CHECK(P)子句指定一个谓词P，关系中的每一个元组都必须满足谓词P
  * PostgreSQL目前仅支持属性级（插入元组或修改此属性值时）和元组级CHECK约束（涉及多个属性时使用元组级，插入元组或修改一行中任意属性时），目前尚不允许CHECK约束出现子查询
  * 新值与约束相违背时拒绝更新，修改时不满足条件则拒绝执行

#### 给约束命名

* 可以给约束起名便于更改，如果没有为约束命名，PG会自动命名

* 声明一个命名约束

  * 使用CONSTRAINT <约束名> UNIQUE(属性) 的格式。

  * 或者

    ```sql
    ALTER TABLE department ADD CONSTRAINT dun UNIQUE(dloca);
    ALTER TABLE department DROP CONSTRAINT dun;
    ```

    

### 5.5 触发器

* 触发器Trigger是用户定义在关系表上的由时间驱动调用函数的机制。比CHECK约束更灵活。

#### 格式

```sql
//声明触发器函数
CREATE FUNCTION function_name() 
RETURNS TRIGGER AS $<name>$
DECLARE 变量声明; //DECLARE部分可选
BEGIN
	函数执行代码;
END;
$<name>$ LANGUAGE plpgsql; //对函数编写语言的说明
```

```sql
//创建触发器
CREATE TRIGGER name {BEFORE|AFTER} INSERT ON examinee
FOR EACH {ROW|STATEMENT}
[WHEN (condition)]
EXECUTE PROCEDURE function_name();
```

* WHEN子句（可选）的condition是个布尔表达式，指明**触发条件**：触发器被激活时，只有触发条件为真触发器函数才执行；否则触发器函数不执行；如果没有WHEN子句，则触发器函数在触发器激活后立即执行

#### 行级触发器和语句级触发器

* 触发器函数必须返回一个NULL或者一个元组类型的变量
* 行级触发器的触发器函数为触发语句影响的每一行执行一次；语句级触发器的触发器函数为每条触发语句执行一次
  * 假设在examiner表上创建了一个AFTER UPDATE触发器，如果表examiner有10000行，执行如下这样一个语句UPDATE examiner SET erage=erage+1
  * 如果该触发器为语句级触发器，那么执行完该语句后，触发动作只发生一次，如果是行级触发器，触发动作将执行10000次

* 行级触发器函数，可以使用NEW或者OLD进行引用；语句级触发器函数，不能使用NEW或者OLD进行引用
* ==行级AFTER触发器==的值总是==被忽略==，可以返回NULL；行级BEFORE触发器的返回值不同，对触发器操作的影响也不同，如果返回NULL则忽略该触发器的行级别操作，其后的触发器也不会被执行，如果非NULL则返回的行将成为被插入或者更新的行。
* 通常，用行级BEFORE触发器检查或修改将要插入或者更新的数据。

#### 激活触发器的顺序

​	  执行该表上**语句级BEFORE**触发器

—>执行该表上**行级BEFORE**触发器

—>执行激活触发器的**SQL语句**

—>执行该表上的**行级AFTER**触发器

—>执行该表上**语句级AFTER**触发器

<!--行级AFTER触发器在修改由触发SQL语句影响的每一行记录之后触发。行级BEFORE触发器在对特定行进行操作之前触发-->

#### 删除触发器

```sql
DROP TRIGGER <触发器名> ON <表名>
```



### 5.6 事务

* 事务是对数据库进行操作的程序单位。

#### ACID特性

* ==原子性==**（事务中包含的所有操作要么都做要么都不做）**
* ==一致性==（事务单独成功执行，使数据库**从一个一致性状态转换到另一个一致性状态**）
* ==隔离性==**（一个事务的执行不能被其他事务干扰）**
* ==持久性==（一个事务**一旦被提交，它对数据库中数据的改变就应该是持久性的**，接下来的其他操作或故障不应该对其结果有影响）

#### 事务声明

* PG用BEGIN和COMMIT（或ROLLBACK）将数据库访问操作指令序列包围以声明一个事务。如果没有显示的BEGIN命令，PG把每个SQL语句当作一个事务来看待。

#### 事务隔离级别

1. **读已提交 READ COMMITED** ==（默认）==：在这个隔离级别事务中的语句，看到的是其开始执行时瞬间数据库的一个快照，在同一个事务里两个相邻的SELECT命令可能看到不同的快照，因为其他事务会在第一个SELECT执行期间提交，读已提交提供的这种部分隔离对于许多应用而言就足够，并且这个模式速度快，使用简答。

2. **可重复读 REPEATABLE READ**：如果一个事务需要连续做若干个命令，而这几个命令必须看到完全相同的数据库视图，可以选择可重复读隔离级别，同一个事务内部后面的==SELECT命令==总是看到同样的数据。

   ```sql
   //设置隔离级别为REPEATABLE READ
   SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
   ```

3. **可串行化 SERIALIZABLE**：看到的是该事务开始时的快照，执行最严格的隔离级别。

* ==可重复度和可串行化==隔离级别事务中的语句，看到的是该事务开始时的快照，而不是该事务内部当前查询开始时的快照，这一点==和读已提交==不一样。REPEATABLE READ和SERIALIZABLE隔离级别的事务里面的SELECT命令总是看到相同的数据状态。可串行化级别提供最严格的事务隔离，就像==（实际不是）==事务是一个接着一个那样串行执行似的。



### 5.7 加密

* 数据被称为**明文**，用某种方法伪装数据以隐藏它的内容的过程称为**加密**。加密所用的方法称作**加密算法**，数据被加密后的结果被称为**密文**，把密文还原为明文的过程称为**解密**，解密所用的算法称为**解密算法**。
* 加密体系中**最核心**的是用于加密解密的==算法和密钥==。
* 现代加密体系中==算法通常是公开的==，==密钥是保密的==并且需要向可信权威机构申请，**安全性完全取决于密钥的保密性**。

#### **加密算法的分类**
加密算法一般可分为加密解密密钥相同的**对称加密**、加密解密密钥不同的**非对称加密**、**单向加密**三种。

* **对称加密**体系当中代表性的算法有DES算法、三重DES、AES算法等等。
* **非对称加密**体系当中的代表性算法有RSA算法、DSA算法。
* **单向加密**体系当中的代表性算法有MD5、SHA算法等等。

#### pgcrypto扩展包

* 在PG中使用加密技术，首先需要通过**CREATE EXTENSIOIN pgcrypto**创建pgcrypto扩展包。
* 该扩展包包含了当前主流加密解密算法的函数定义。
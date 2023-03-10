# 数据库笔记（BNU MOOC版）

## Chapter 6 数据库设计：实体-联系方法

[TOC]

### 6.1 数据库设计方法和生命周期

#### 数据库设计方法分类

* 数据库设计方法主要包括==实体-联系方法==和==属性-联系方法==两种

  【宏观 把握关键】实体-联系方法以**实体**为中心，着重于一个关系模式基本对应一个实体或联系，即关系模式与实体或联系之间基本是一一对应的。

  【微观 进行优化】属性-联系方法以**属性**为中心，着重于属性之间的**依赖关系**。把需要数据库保存的所有属性放在一张关系表中，进而通过属性之间的联系来优化这个关系模式。==属性-联系方法没有概念设计阶段==。

#### 实体-联系设计方法的主要过程

1. 分析数据需求，有了数据需求，就可以进行概念设计，即设计概念模式，概念模式与具体DBMS无关，通常使用==实体–联系图==表示，也叫==E-R图==。
2. 在概念模式的基础上，进行逻辑设计，即将==概念模式转换成==相应的逻辑模式，获得符合选定DBMS数据模型的逻辑结构，比如==关系模式==。
3. 物理设计，在需要的时候进行一些参数的设置和选择，比如索引机制、块大小，由于关系数据库系统的物理数据独立性，在逻辑模式确定后伴随着物理设计，就可以进行应用程序的设计
4. 物理设计完成后，就可以开始实现数据库，用DDL/DPL定义数据库结构、把数据入库、编制与调试应用程序并进行数据库试运行，如果试运行中**功能指标和性能指标**都达到了设计目标，数据库系统就可以正式投入运行。
5. 在运行过程当中，还要经常对数据库进行==维护==
   * 定期对数据库和事务日志进行==备份==，保证发生故障时，能利用这些备份，尽快将数据库恢复到一个一致的状态
   * 当应用环境、用户、完整性约束等出现变化时往往需要根据实际情况调整原有==安全策略==、完善原有==安全机制==
   * 利用DBMS提供的系统性能监测工具监督系统运行，必要时调整某些参数，改进系统性能
   * 针对数据库性能随着数据库运行逐渐下降问题，必要时重组数据库，回收垃圾，减少指针链，提高系统性能
   * 针对应用需求的变化，适当调整数据库模式也叫重构数据库
6. 如果实际应用发生了根本性的变化，重构数据库的代价就会非常大，那么这个时候，我们就应该终止现有的数据库应用系统生命周期，重新建立新的数据库系统，开始新的生命周期
   



### 6.2 E-R模型的基本元素

#### E-R模型的基本元素

* E-R图主要包含实体和联系，以及它们各自的属性。
* 实体【方框】和实体集统称为实体。实体之间不是孤立的，总是存在一些联系。
* 联系【菱形框】就是一个或多个实体之间的关联关系。同类联系组成的集合称为联系集。习惯上，把联系和联系集统称为联系。
* 属性域：属性【椭圆】可能取值的范围。
* 主键（标识符）要加下划线，指能够并且用以区分一个实体集中不同实体的最小的属性集，组成主键的属性称为标识属性。



### 6.3 基本E-R图设计

#### 联系元数

* 联系关联的实体个数称为该联系的元数或度数
  1. 同类实体集内部实体与实体之间的联系，称为一元联系。（e.g.一个人会喜欢另一个人，人与人之间的“喜欢”是一个一元联系。）
  2. 两个不同实体集中实体之间的联系，称为二元联系。
  3. 三个不同实体集之间的联系称为三元联系。
  4. 如果实体集E1中，每个实体可以与实体集E2中任意个（零个或多个）实体之间具有联系，并且E2中每个实体至多和E1中一个实体有联系，那么我们就把E1对E2的联系称为“一对多联系”。

#### 映射基数

* 如果实体集E1和E2之间有二元联系，则参与该联系的实体数目称为映射基数。（一对一 / 一对多 / 多对多）注：这里的“多”指==0个或多个==，即任意个。
  * 一对一实例：一个学生只有个身份证编号。
  * 一对多实例：一个班级有多个学生。
  * 多对多实例：多对多就是双向一对多，一个学生可以选择多门课，一门课也有多名学生。

#### 属性分类

* 复合属性：可以按组成分解成一系列子属性。

* 单值属性：同一个实体在这个属性上只能取一个值。

* 多值属性：同一个实体的某些属性可能取多个值。如，一个考官可能有多个联系方式。可以用多个单值属性代替单个多值属性。

* 派生属性：能从其他属性值推导出值的属性。

#### 完全参与和部分参与

* 如果实体集S中的每个实体都参与联系集L的至少一个联系，称实体集S“完全参与”【双线】联系集L。

* 如果实体集S中只有部分实体参与联系集L的联系，称实体集S“部分参与”【单线】联系集L。



### 6.4 基本E-R图转换为关系模式

1. 一个实体转化成一个关系模式
   * 实体的属性-表的列
   * 实体的主键-表的主键

2. 一个联系转化成一个关系模式
   * 如果==联系M:N==，主键是所有参与联系的==各实体主键的并集==加上联系自己特有的属性；
   * 如果==联系1:N==，主键是==多端实体主键==；
   * 如果==联系1:1==，主键是==任一端实体主键==。

3. 主键相同的关系模式可以**合并（关系的精简）**
   * 如果联系是1:N，那么==联系表可合并到多端实体的表==
   * 如果联系是1:1，那么==联系表可与任一端实体对应的表合并在一起==

* 由联系转换来的表的主键与任一端实体主键相同。❌
* e.g. 借还书数据库
  * 图书（<u>ISBN</u>, 书名，作者，出版社，出版时间，图书状态）
  * 读者（<u>读者号</u>，姓名，类型）
  * 借阅（<u>读者号，ISBN，借书时间</u>）<!--下划线是一整条-->


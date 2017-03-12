# mysql 

### SQL 编程

#### SQL编程的三个阶段：
* `面向过程话的SQL编程阶段`。 这个阶段SQL程序员刚开始使用数据库，此时他们没有多少处理关系模型的经验和基于集合的思想。在这一阶段，经常会有滥用各种工具，比如游标，临时表，动态SQL语句等的情况，而它们通常意识不到他们的破坏。
* `面向集合的SQL编程阶段` 。 这个阶段SQL程序员开始意识到SQL编程与面向过程面相对象编程的不同之处，知道运用SQL编程需要更多的东西，慢慢的意识到SQL编程与面向过程和对象编程的不同之处，知道运用SQL编程需要更多的东西，慢慢发现SQL不再是妨碍编程的令人讨要的东西，而是建立在机遇关系模型集合理论的强大基础上产物，从这一阶段开始，程序员开始相信那些说游标，临时表，动态SQL有害而永远不应该使用的专家。
* `融合的SQL编程阶段`，这个阶段SQL程序员已尽固有了丰富的知识并对SQL有了深入理解。他们对自己的代码非常自信，但这并不意味着他们会停止钻研更深入的知识以及提高关键性的技术。在这一阶段，SQL程序员不再迷恋所谓的专家，他们可能意识到即使是有表，也并不是在所有的恶情况下都是无用和有害的。这个阶段的程序员已经具备了判断什么时候使用纯静态的SQL编程方法不能完成某些任务的能力，尽管纯静态的SQL编程是一种非常典型的方法，但是它只在大部分情况下适用。有时候，使用临时表可以显著的改善性能，使用动态SQL可以解决复杂的问题。适当的使用游标可以提高程序运行的效率，使用C，C++这样的过程语言可以带来更大的灵活性。而且不易与关系模型发生冲突。

### 数据库的应用类型
一般来说，可将数据库的应用类型分为OLTP（online transaction processing,联机事务处理）和OLAP（online analysis processing , 联机分析处理）。其中OLTP针对常见的日常数据库，OLAP则是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。

#### OLTP
oltp的基本特征是可以立即将顾客的原始数据传送到计算中心进行处理，并在很短的时间内给出处理结果，这个过程的最大优点是可以即时地处理输入的数据，及时的回答，因此OLTP又被称为实时系统，OLTP数据库旨在使事务应用程序仅完成对所需数据的写入，以便尽快处理单个事务。

#### OLAP
olap为一个或多个多维数据集，每个数据集都由多位数据集管理员组织和设计，以适应用户检索和分析数据的方式，从而更易于创建和使用所需的数据透视图和数据透视表。olap的主要特点是直接仿照用户的多角度思考模式，预先为用户创建多维的数据模型。维度，指的是分析的角度。比如以销售为例，时间周期是一个维度，地理位置是一个维度，产品类别是一个维度，分销渠道是一个维度，客户群类别等等。一旦多维度数据模型建立完成，用户可以快速地从各个分析灵活性。这也是olap这些年被关注的根本原因。
* 维：用户观察数据的角度。
* 维的层次：用户观察的某一个角度的具体细分，比如时间维度，可以进一步细分为月，季度，年
olap的基本多维分析有钻取，切片，切块，旋转
* 钻取：改变为的层次，变换分析的粒度。


#### OLTP VS OLAP
![oltp vs olap](https://i.imgsafe.org/50cd37f5df.png)

#### MySQL 存储引擎

* InnoDB存储引擎；支持事务，主要针对oltp应用。其特点是行锁设计，支持外键。默认读取操作不会产生锁。5.5.8开始为默认的存储引擎。InnoDB存储引擎将数据放入到一个逻辑的表空间中，对应一个独立的idb文件。
* MyISAM :不支持事务，表锁设计，支持全文索引，主要真多olap应用。

### 数据类型

* `show create table test\G;` 查看test表的具体信息。
* set sql_mode='&&&' 设置变量值对于当前session中。
* check sql_mode global value and session value : select @@global.sql_mode\G;  select @@global.sql_mode\G;
* timestamp and datetime all indicate 'yyyy-MM-dd hh:mi:ss' ,but timestamp is the million seconds between 1970-01-01 and now , and it can set a default value , but datetime can not set a default value . and if you update the timestamp record , the time will automaticly update to now . 
* select IF(1 > 2,1,2) from dual; IF sentence , good stuff . 
* 排序规则也是可以设置的，不同的排序比较算法，得出的顺序是不一样的

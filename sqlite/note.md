#SQLite学习笔记

[TOC]

##杂记
- sqlite 的关键字 如select insert integer real 之类的不区分大小写 但名称 常量区分
- 数据库清理 sqlite3 test.db vacuum
- 数字常量直接写数字
- 如果要用16进制常量 要写成
	- x'01'
	- x'0fff'

_ _ _


##创建表:

```sql
create [temp] table table_name (column_definitions [, constraints]);
```
- temp关键字声明临时表, 连接断开后该表会被自动销毁
- table_name 表名是必须的 且不能和其他关键字重复
- column_definitions是由逗号分隔的字段列表组成 每个字段定义包括一个名称, 一个域(类型)和一个逗号分隔的字段约束
	- sqlite有五种本地类型:integer real text blob null约束用来控制什么样的值可以储存在表中或字段中

- - -


##修改表:

```sql
alter table table_name {rename to new_table_name | add column column_definitions};
```
- rename to 改表名
- add column 添加列

_ _ _


##查询表:
###关系操作
select语句提供混合, 比较和过滤数据的"关系操作", 这些关系操作分为三种类型:

- 基本类型:
        Restriction                 限制
        Projection                  投影
        Cartesian Product           笛卡尔积
        Union                       联合
        Difference                  差
        Rename                      重命名

- 附加操作
        Intersection                交
        Natureal Join               自然连接
        Assign                      赋值
- 扩展操作
        Generalized Projection      广义投影
        Left Outer Join             左外连接
        Right Outer Join            右外连接    sqlite不支持
        Full Outer Join             全外连接    sqlite不支持

所有这些操作都定义在关系(也就是通常说的表). 它们将一个或多个关系视为输入, 然后产生另一个关系作为输出,
允许选择操作(一种关系)的输出作为另一个select语句的输入:
```sql
    select name from (select name, type_id from (select * from foods));
```
___
###select命令与操作管道(64页有张图)
```sql
select [distinct] heading               (distinct用于去掉重复的行)
from tables
where predicate                         (predicate谓词，用来描述或判定客体
                                        性质、特征或者客体之间关系的词项)
group by columns
having predicate
order by columns
limit count, offset;
```

    流程是  n张表--from tables-->R1--where predicate-->R2--group by columns-->R3--having predicate-->R4--heading-->R5--distincate-->R6--order by columns-->R7--limit count, offset-->Result
___
###过滤
如果select是sql中最复杂的命令, where就是select最复杂的子句. where 子句的主语是行的集合 sqlite 对from子句产生的每一行应用where子句如: select * from dogs where color='purple' and grin='toothy'; sqlite会获取表dogs(主语)的所有行, 应用where形成逻辑判断, 如果为真,将会包含在结果集中.
- 值 值代表真实世界的某种数据
- 操作符 操作符使用一个或多个值作为输入并产生一个值作为输出 操作符可以形成序列, 一个的输出作为后一个的输入
- 二元操作符  (68页有张列表)
    | 操作符  		  |类型     	 | 作用   |
    |:--------------|:----------|:-------|
    |&#124;&#124;	|String		|连接	    |
    |*				|Arithmetic	|乘		|
    |/            	|Arithmetic |除		|
    |%            	|Arithmetic |模      |
    |+              |Arithmetic |加      |
    |-              |Arithmetic |减      |
    |<<             |Bitwise    |左移    |
    |>>             |Bitwise    |右移    |
    |&              |Logical    |与      |
    |&#124;			|Logical   	|或 		|
    |<              |Relational |小于		|
    |<=             |Relational |小于等于  |
    |>              |Relational |大于     |
    |>=             |Relational |大于等于  |
    |=              |Relational |等于     |
    |==             |Relational |等于     |
    |<>             |Relational |不等于   |
    |!=             |Relational |不等于   |
    |IN             |Logical    |In      |
    |AND            |Logical    |与       |
    |OR             |Logical    |或       |
    |IS             |Logical 	|等于     |
    |LIKE           |Relational |字符串匹配 |
    |GLOB           |Relational |文件名匹配 |

- like 和 glob
    - like 用于字符串模式匹配 模式中的%可以匹配0到多个字符, _匹配一个字符
    ```sql
    select * from foods where name like 'J%';   匹配所有J开头的食物
    select * from foods where name like 'J_';   匹配所有J开头名字长度为2的食物
    select * from foods
    where name like '%ac%P%'
    and name not like '%Sch%'; 					可利用not否定某些模式
    ```
    - glob操作符像UNIX/Linux文件通配语法
    ```sql
    select * from foods where name glob 'Pine*';
    ```
___

###限定和排序
limit和offset可以限制结果集的起始偏移和大小
```sql
select * from food_types order by id limit 1 offset 1;  
返回food_types表第二行的记录
```
order by子句使得记录集在返回之前按一个或多个字段的值进行排序, 每个字段项都可能配合排序方向--asc默认的升序 desc降序
select * from foods where name like 'B%' order by type_id desc, name limit 10;
注意 上面的order by 子句对两个字段值进行排序, 当第一个字段出现重复就需要按后续二, 三 ...字段排序
如上例type_id出现重复时, 就会按name排序
___
###函数(Function)和聚合(Aggregate)
- sqlite提供了很多内置函数和聚合, 可以用在各种子句中,当然也包括where子句

- 函数名不区分大小写 可以接受字段值作为参数, 如:
    ```sql
    select upper('hello newman'), length('hello newman'), abs(-12);
    ```
    ```sql
    select id, upper(name), length(name) from foods where length(name) < 5;
    ```
- 聚合是一类特殊函数, 它从一组记录中计算聚合值. 标准的聚合函数有sum(), avg(), count(), min(), max() 例如:
    ```sql
    select cout(*) from foods where type_id=1;
    count
    47
    ```
    count()聚合返回关系中所有行的数目, 看到聚合就应该想到"对表中每一行做某种运行"

    聚合不仅可以聚合字段, 也可有聚合任何表达式, 包括函数, 例如要获取所有食物名称长度平均值, 可以在length(name)上应用聚合avg
    ```sql
    select avg(length(name)) from foods;
    ```
___

###分组(Grouping)
- 聚合的主要部分就是分组, 也就是说, 聚合不只能计算结果集的聚合值. 还可有吧结果集分为多个组, 然后计算每个组的聚合值
    ```sql
    select type_id from foods group by type_id;
    ```
    上例中有15个不同的食物类型, 因此group by将所有的行依type_id分到15个组中, select接受所有的组, 提取共有的type_id将其
    放到单独的行, 因此结果有15行, 使用group by时select子句对每组单独应用聚合, 而不是对整个结果惊喜聚合.
    ```sql
    select type_id, count(*) from foods group by type_id; 对前面的例子应用count聚合, 获取美国type_id组的记录数目
    type_id         count(*)
    1               47
    2               15
    3               23
    15              32
    如果不用上面的句子
    就要用15条select分别获取结果
    select count(*) from foods where type_id=1;
    select count(*) from foods where type_id=2;
                    ......
    select count(*) from foods where type_id=15;
    ```

- 分组的另一个关键点having子句
    group by 没有在select子句处理钱过滤这些组, having是一个可以应用到group by的断言子句, 它从group by组过滤
    组的方式与where子句从from子句中过滤行的方式相同. 唯一的不同, where子句的预测是针对单个行, 而having断言是针对聚合
    如前例 这次只关注食品数小于20的食品类型
    ```sql
    select type_id, count(*) 
    from foods group by type_id having count(*) < 20;
    ```
    断言count(*) < 20应用到
___

###去掉重复
distinct处理select的结果并过滤掉其中重复的行
```sql
select distinct type_id from foods;
```
___
###多表连接(join)
连接是多表(关系)数据工作的关键, 它是select命令的第一个操作, 连接操作的结果作为输入
供select语句的其他部分(过滤)处理.
例如:
foods表有个字段是type_id, 该字段的值都与food_types表中的id字段对应, 这样两个表之间就存在关联
id是food_types表的主键, foods表的外键, 于是可以将foods表和food_types表连接起来形成新关系
这样可以提供更多信息----为foods表的每个食品提供food_type.name
```sql
select foods.name, food_types.name
from foods, food_types
where foods.type_id=food_types.id limit 10;

name                    name
-------------------------------------------
Bagels                  Bakery
Bagels, raisin          Bakery
        ......
Carrot Cake             Bakery
Chips Ahoy Cookie       Bakery
```

sqlite 支持6种连接, 其中内连接是最普遍的

- 内连接
```sql
select * from foods inner join food_types on foods.id = food_types.id;
//返回同时满足连接条件的行
```

- 自然连接
    和内连接的另一种形式, 通过表中共有的字段名称来将两个表连接起来, 不用添加连接条件

- 交叉连接
    交叉连接就是不指定任何连接条件, 最基础, 最根本的连接 称为交叉连接, 笛卡尔积,或交叉乘积, 它是强制的几乎无意义的连接
    即将第一个表中的所有行和第二个表中的所有行联合起来.(通常是无意义的)

- 外连接
    四种连接中的其余三种都是外连接, 内连接根据给定关系选择表中的行. 外连接选择内连接的所有行外加一些关系之外的行.
    三种外连接分别是左外连接, 右外连接和全外连接
    sqlite不支持右外连接和全外连接, 但右外连接可以用左外连接代替, 全外连接可以用复合查询代替

- 左外连接, 左外连接操作SQL命令中的"左表"
    ```sql
    select *
    from foods left outer join foods_episodes
    on foods.id=foods_episodes.food_id;
    ```
    foods 是其中的左表, 左外连接试图将foods中的所有行与
    foods_episodes的所有行进行连接(foods.id=foods_episodes.food_id)的匹配,
    如果foods表中有的食品的id没有在foods_episodes表中出现,
    foods表中未能匹配的行仍然会在结果集中出现
    foods_episodes中没有提供相应的行的话则会以null补充

- 右外连接, 和左外连接一样, 只是以右边的表为主, 所以需要右外连接的情况,都可以用左外连接代替
- 全外连接, 是左外连接和右外连接的结合

- 连接的语法偏好
    可以用隐喻把内连接写成:
    ```sql
    select * from foods, food_types
    where foods.id=food_types.food_id;
    ```
    这是不推荐的过时写法, sqlite官方推荐的王道写法是用链接语句显示表达
    ```sql
    select * from foods inner join food_types
    where foods.id=food_types.food_id;

    select * from foods left outer join food_types
    where foods.id=food_types.food_id;

    select * from foods cross join food_types
    where foods.id=food_types.food_id;
    ```
___
###名称和别名

- 名称
        连接表时, 如果两个表有同样名称的字段, 就会产生歧义, 可以在字段名前面加上表名来消除
        table_name.name

- 别名
    如果表名太长,又频繁出现 可以在from子句中对表重命名, 然后在其他地方直接使用
    ```sql
    select f.name t.name 
    from foods f, food_types t
    where f.type_id = t.id
    limit 10;
    ```

    通过别名 使得表自己和自己连接成为可能

    ```sql
    select f.name as food, e1.name, e1.season, e2.name, e2.season
    from episodes e1, foods_episodes fe1, foods f,
    episodes e2, foods_episodes fe2
    where
        --Get foods in season 4
        (e1.id = fe1.episodes_id and e1.season = 4)
        and fe1.food_id = f.id
        --Link foods with all other episodes
        and (fe1.food_id = fe2.food_id)
        --Link with their respective episodes and filter out e1's season
        and (fe2.episodes_id = e2.id and e2.season != e1.season)
    oder by f.name;
    ```
结果集中别名语法:select base-name [[as] alias] 也有人倾向于保留as 使得别名更清晰
___

###子查询
子查询就是嵌套的select 通过in 操作符来指定输入结果集
```sql
select 1 in (1,2,3);
1
select 2 in (1,2,3);
2
select count(*) 
from foods
where type_id in (1,2);
select count(*)
from foods
where type_id in(
select id
from food_types
where name='Bakery' or name='Cerea1');
```
___

###复合查询
与子查询相反, 用三种特殊的关系操作符来处理多个查询结果
三种操作分别为union, intersect, except
分别对应联合 交叉和差集
复合查询有一些要求和需要注意的地方
    - 涉及的关系的自动数目必须相同
    - 只能有一个order by子句, 并且处在复合查询的最末尾, 对联合查询结果进行排序
    - 复合查询中的关系从左向右处理

- 联合操作
	输入两个关系 A 和 B, 将两者联合成一个只包含A和B中非重复字段的单一关系.
    默认情况 联合会消除重复, 如果想保留重复数据, 要使用union all
    下面的例子是找出foods表中最高频率和最低频率的食物
    ```sql
    select f.*, top_foods.count from foods f
    inner join
      (select food_id, count(food_id) as count from foods_episodes
         group by food_id
         order by count(food_id) desc limit 1) top_foods
      on f.id=top_foods.food_id
    union
    select f.*, bottom_foods.count from foods f
    inner join
      (select food_id, count(food_id) as count from foods_episodes
         group by food_id
         order by count(food_id) limit 1) bottom_foods
      on f.id=bottom_foods.food_id
    order by top_foods.count desc;
    ```

- 交叉操作
	输入两个关系A 和 B, 选择那些既在A也在B的行, 下面的例子使用intersect找出
    episodes介于3~5之间的处于前10位的食品
    ```sql
    select f.* from foods f
    inner join
      (select food_id, count(food_id) as count
         from foods_episodes
         group by food_id
         order by count(food_id) desc limit 10) top_foods
      on f.id=top_foods.food_id
    intersect
    select f.* from foods f
      inner join foods_episodes fe on f.id = fe.food_id
  inner join episodes e on fe.episode_id = e.id
  where e.season between 3 and 5
order by f.name;
```

- 差集操作
输入两个关系A 和 B, 找出所有在A但不在B的行, 把前例的intersect改成
except可以找出episodes介于3~5而不在前10位的食品
___
###条件结果
case表达式允许在select语句中处理各种情况, 它有两种形式:

- 接收静态值并列出各种情况下的case返回值
    ```sql
    case value
        when x then value_x
        when y then value_y
        when z then value_z
        else default_value
    end

    select name || case type_id
                     when 7  then ' is a drink'
                     when 8  then ' is a fruit'
                     when 9  then ' is junkfood'
                     when 13 then ' is seafood'
                     else null
                   end description
    from foods
    where description is not null
    order by name
    limit 10;
    ```

- 第二种case形式允许when条件中有表达式
    ```sql
    case
    when condition1 then value1
    when condition2 then value2
    when condition3 then value3
    else default_value
    end

    select name, (select
    case
         when count(*) > 4 then 'Very High'
         when count(*) = 4 then 'High'
         when count(*) in (2,3) then 'Moderate'
         else 'Low'
       end
    from foods_episodes
    where food_id=f.id) frequency
    from foods f
    where frequency like '%High';
    ```
___
###处理sqlite中的 null
sqlite中一些null的关键点

1. sqlite中的null不是值,只是缺失信息的占位符
2. null 不是真也不是假 也不是零, 只是它本身
3. sqlite的逻辑运算采用三值逻辑 true false 和null 关系表见121页
4. 可以用 is null或者 is not null检查null是否存在, 不能用null=null因为null不等于任何值甚至不等于其他null
5. 最重要一条, 由于2) 将null与其他值比较基本都会返回false
```sql
    select *
    from mytable
    where myvalue = null;
    --通常不会返回任何结果 因为null不等于任何值

    --但作为变通 可以用is 操作符
    select null is null; 会返回1
    select null = null; 则什么都不返回
```

6. coalesce 函数 将一组值作为输入 返回第一个非null
```sql
    select coalesce(null, 7, null, 4);
    返回7
```

7. nullif 函数输入两个参数, 如果两个参数相同 返回null否则返回第一个参数
```sql
    select nullif(1, 1);
    null
    select nullif(1, 2);
    1
```
---

##插入记录
使用insert命令想表中插入记录. insert在单表上工作, 一次插入一条记录
```sql
insert into table (column_list) values(value_list);
```
###插入单行
```sql
insert into foods (name, type_id) values('Cinnamon Bobka', 1);
```
如果在insert语句中给所有字段提供值, 可以省略字段列表
```sql
insert into foods values(null, 1, 'Blueberry Bobka');
```
注意此处给主键传入null, 利用其自动递增的性质
___
###插入一组行
子查询可以用在insert语句中, 作为值列表的一部分.
指定子查询为值列表时, 实际上是在插入一组行
```sql
insert into foods
values (null,
		(select id from food_types where name='Bakery')
        'Blackberry Bobka');
这里就不用硬编码type_id了
```
```sql
insert into foods
select last_insert_rowid()+1, type_id, name
from foods
where name='Chocolate Bobka';
本例使用select完全取代了值列表 因为与要插入的字段完全匹配
这里last_insert_rowid()也是不好的用法, 因为如果上次没插入
会返回0, 应该用null来让主键自增
```
___
###插入多行
使用select形式的insert可以一次插入多行, 只要字段匹配, insert可以插入结果集的所有行
```sql
create table foods2 (id int, type_id int, name text);
insert into foods2 select * from foods;
```
```sql
create temp table  list as
select
	f.name food,
    t.name name,
	(select count(episode_id)
    	from foods_episode
        where food_id=f.id) episodes
from foods f, food_type t
where f.type_id=t.id;
注意这种方式复制的表, 源表的约束并未一起复制
```
___
##更新记录
update命令用于更新表中的记录, 该命令可以修改一个表中一行或多行的一个或多个字段.
```sql
update table set update_list where predicate;
```
update_list是一个或多个"字段赋值"的列表, 其格式为column_name=value,
where 和后面的谓词对要修改的行进行过滤

```sql
update foods set name='CHOCOLATE BOBKA'
where name='Chocolate Bobka'
```
update语句非常简单直接, 但需要注意约束, 比如unique就可以使得update被终止
___
##删除记录
最简单的sql命令
```sql
delete from table where predicate;
```
就像update少了set部分, 只需要用where子句的谓词来过滤删除条件

##数据完整性
- 数据完整性用于定义和保护表内部或表之间的数据关系. 一般有四种完整性:
	- 域完整性,  域完整性控制字段内的值
	- 实体完整性, 实体完整性控制表中的行
	- 引用完整性, 引用完整性控制表之间的行----就是外键关系
	- 用户定义完整性, 用户定义完整性可以包罗万象.
- 数据完整性是通过约束实现的, 他们可以和字段定义或表定义关联起来
	- 字段级约束包括: not null, unique, primary key, foreign key, check, collate
	- 表一级约束: primary key
___
###实体完整性
关系理论要求数据库所有的域都是唯一的, 可以定位的. 域要可寻址
它相应的行必须是可寻址的. 行必须在某种方式上是唯一的, 这就是主键的功能.
主键由至少带有unique约束的一个或一组字段组成.

- **惟一性约束**
	unique约束要求一个或一组字段的所有值均不同. 如果试图插入或更新一个已经存在的值, 数据库会引发一个约束非法, 并终止操作.
    ```sql
    CREATE TABLE contacts ( id INTEGER PRIMARY KEY,
                            name TEXT NOT NULL COLLATE NOCASE,
                            phone TEXT NOT NULL DEFAULT 'UNKNOWN',
                            UNIQUE (name,phone) );
    注意这段代码定义的惟一性约束, 是对name和phone的联合的

    insert into contacts (name, phone) values('Jerry', 'UNKNOWN');

    insert into contacts (name) values('Jerry');
    SQL error:columns name, phone 
    ```
- **主键约束**
	SQLite中, 定义一个表时总会指定一个主键, 无论自己是否定义. 它是一个64-bit整型数, 有两个别名_rowid_和oid, 它默认按增序自动生成.如果在integer primary key 后加上autoincrement, SQLite采用不同的主键生成算法,基本上阻止了rowid回收(详细去看131页)
    - 有一点要注意, 和惟一性约束一样, 主键约束可以定义在多个字段中,主键也不一定需要是整型, 如果主键是整型, SQLite会把它处理成,rowid的别名如果设置其他类型作为主键, SQLite仍会在内部维护rowid.
    ```sql
    create table pkey(x text, y text, primary key(x, y));
    本例的主键从技术上讲横跨两个字符串字段, 由于sqlite
    总是自己维护rowid. 很多数据库专家建议使用实际字段作为主键
    ```
___
###域完整性
最简单的域完整性定义就是字段值的类型定义
SQLite的类型检查有争议, 11章再讨论
先讨论简单内容, 默认值, not null约束, check约束, 排序规则

- 默认值 default只是一个约束, 它保证该字段有值, 并在需要时出现. 它符合域完整性, 因为它为处理字段中的null提供了一种策略--在insert时语句没有为字段指定值, SQLite会为它赋值null, 如果定义表时有指定default, 则使用这个值.
    - 预定义保留字, 用于即时生成日期和时间.
    - current_date, 生成(YYYY-MM-DD)的当前日期
    - current_time, 生成(HH:MM:SS)的当前时间
    - current_timestamp, 生成(YYYY-MM-DD HH:MM:SS)
    ```sql
    create table times(id int,
        date not null default current_date,
        time not null default current_time,
        timestamp not null default current_timestamp);
    ```
- NOT NULL约束
insert 语句向声明为not null的字段插入null会引发一个约束非法, 所以经常将not null约束和default配合使用来保证insert语句总能安全使用, 又能排除null.

- check约束
check约束允许通过定义表达式来测试要插入或更新的字段值, 如果该值不满足设定的表达式, 数据库会报约束违反错误, 通过它可以自定义unique或not null以外的数据完整性以满足特定应用.如果发现check约束不能完全表达的内容, 可以试试触发器.
```sql
create table contacts
(id integer primary key,
name text not null,
phone text not null default 'UNKNOWN',
unique(name, phone)
check(length(phone) >= 7));
本例中, 任何试图插入或者更新语句中phone的值如果小于七个字符, 都会触发违反约束
```
```sql
create table foo(
x integer,
y integer check(y > x),
z integer check(z > abs(y))
);
也可以在一条语句中多个check 使用语句中其他符号名
```
___
###外键约束
外键约束用于确保表间关系完整性, 它确保了一个表中的关键值必须从另一个表中引用, 且该数据必须在另一个表中实际存在.
```sql
create table food_types(
id integer primary key,
name text
);
create table foods(
id integer primary key,
type_id integer references food_types(id)
on delete restrict
deferrable initially deferred,
name text);
```
```sql
create table table_name
(column_definition references foreign_table(column_name)
    on {delete|update} integrity_action
    [not] deferrable [initially {deferred|immediate},]
);
```
外键语法结构
    1. 告知SQLite该字段使用了何表的何字段
    2. 完整性约束 在更新或删除时, 触发何种动作
        - restrict 完全阻止 触发终止事务
        - set null 如果父值被删除或不存在了, 剩余子值改为null
        - set default 如果父值被删除或不存在了, 剩余子值改为默认值
        - cascade 更新父值时, 更新所有匹配的子值, 删除父值时删除所有子值, 注意删除可能出现未定义的结果
        - no action 使用一种松弛的办法, 不干涉操作执行, 只是观察变化, 在整个语句(如果约束定义为deferred的事务也一样)的结尾报出错误.
    3. deferrable 子句, 控制定义的约束是立即执行还是延迟到整个事务结束
___
###排序规则
排序规则涉及文本值如何比较, 有些排序规则可能是不区分大小写的, 另一些大小写敏感.
关键词collate定义字段的排序规则:
    1. 默认按二进制比较使用memcmp
    2. nocase非大小写敏感
    3. reverse和二进制排序规则相反
```sql
create table contacts
(id integer primary key,
name text not null collate nocase,
phone text not null default 'UNKNOWN',
unique(name, phone)
check(length(phone) >= 7));
```
___
###存储类
SQLite有5个原始数据类型, 被称为储存类, 是类型和数据类型的同义词
| 名称 | 说明 |
|---:---|---:---|
|integer|整数值的最大范围是8字节 但SQLite实际是根据其大小自动控制所占的字节|
|real|实数是10进制, 8字节长度的浮点数|
|text|文本是字符数据SQLite支持包括UTF-8和UTF-16的字符编码, 字符串默认最大长度1,000,000,000字节|
|blob|二进制大对象默认最大长度1,000,000,000字节|
|null|null表示无值, SQLite具有null的完全支持|

SQLite函数typeof()根据值的表示法返回其存储类,
```sql
select typeof(3.14), typeof('3.14'), typeof(314), typeof(x'3142'), typeof(null);
real 	text 	integer		blob	null
```
**SQLite中一个字段可以包含不同存储类的值, 这也是SQLite处理数据与其他数据库最大的不同**:
```sql
create table domain(x);
insert into domain values (3.142);
insert into domain values ('3.142');
insert into domain values (3142);
insert into domain values (x'3142');
insert into domain values (null);
select rowid, x, typeof(x) from domain;

rowid		x		typeof(x)
1			3.142	real
2			3.142	text
3			3142	integer
4			1B		blob
5			null	null
```
这种机制会造成一些麻烦, 比如比较时, 细节可见115页, 更多细节见11章
___

###视图
视图即虚拟表, 也称派生表, 因它们的内容都派生子其他表的查询结果, 基本表的内容是持久的而视图的内容是在使用时动态产生的
```sql
create view name as select-stmt;
视图的名称有name指定, 由select-stmt定义. 最终生成的视图看起来像名为name的基本表
```
```sql
总是编写这种多表查询让人厌烦
select f.name, ft.name, e.name
from foods f
inner join food_types ft on f.type_id=ft.id
inner join foods_episodes fe on f.id=fe.food_id
inner join episodes e on fe.episode_id=e.id;

可以创建视图来简化查询
create view details as
select f.name as fd, ft.name as tp, e.name as ep, e.season as ssn
from foods f
inner join food_types ft on f.type_id=ft.id
inner join foods_episodes fe on f.id=fe.food_id
inner join episodes e on fe.episode_id=e.id;

像查询基本表一样查询视图
select fd as Food, ep as Episode
from details where ssn=7 and tp like 'Drinks';

删除视图
drop view name;
```
通常关系数据库支持可更新的视图, 可以向其insert或者update, 修改会应用到底层表中, 但SQLite并不支持可更新视图, 不过可以用触发器达到类似的目的.
___

###索引
索引是一种用来加速查询的结构.
```sql
select * from foods where name='JujyFruit';
```
当数据库搜索匹配的行时, 执行这种查询的默认方式是顺序扫描, 如果频繁进行这种查询, 且foods表非常大, 使用索引就很有必要了.
```sql
create index [unique] index_name on table_name(columns);
```
columns是一个字段或以逗号分隔的多个字段
如果使用关键字unique将会在索引上添加约束, 索引中的所有值必须是唯一的
unique关键字覆盖index的所有字段, 不管是联合值还是单个值都必须惟一
```sql
drop index index_name
删除索引
```

- 排序规则
    索引中每个字段都有相关排序规则, 如要在foods.name上创建大小写不敏感的索引可以如下:
    ```sql
    create index foods_name_idx on foods(name collate nocase);
    ```
- 使用索引
    对下列情况, SQLite会使用单个字段索引:
    ```sql
    column {=|>|>=|<=|<} expression
    expression {=|>|>=|<=|<} column
    column in  (expression-list)
    column in  (subquery)
    ```
    多字段索引有更复杂的条件
    多字段索引从左往右使用字段, 直到where子句中没有有效条件
    ```sql
    create table foo(a, b, c, d);
    create index foo_idx on foo(a, b, c, d);
    select * from foo where a=1 and b=2 and d=4;
    上面这句只有第一和第二个条件使用索引, 第三个条件因为索引字段没有连续使用而简单的放弃了
    ```
    不等式条件也会终止索引的从左往右使用
    ```sql
    select * from foo where a>1 and b=2 and c=3 and d=4;
    由于a>1用了不等号, 它之后的字段被认为是不合格的
    ```
- 索引使用的注意事项:
    1. 当使用索引, 一定要有理由确保可以获得性能提升. 选择好的索引是非常重要的, 分布凌乱的索引会造成反效果.
    2. 而且索引会增加数据库的大小, 对表中所有字段创建索引, 可能会使得表的大小翻倍.
    3. 索引的维护是有代价的, 进行insert update delete时, 除了修改表, 也需要修改对应的索引. 虽然索引可以加快查询速度, 却会降低其他操作的速度.
___
###触发器
# sql题目收集题目（保持状态）



https://blog.csdn.net/dragonbranches/article/details/82759005
sql聚合函数相关  


（1）**查出每个用户第一单的消费金额**。链接：https://www.nowcoder.com/questionTerminal/b61f123aeaad48dd8afa1937f528e5e6
来源：牛客网



用户分析是电商数据分析中重要的模块，在对用户特征深度理解和用户需求充分挖掘基础上，进行全生命周期的运营管理（拉新—>活跃—>留存—>价值提升—>忠诚），请尝试回答以下3个问题： 

  ① 用户第一单购买的行为往往反映了用户对平台的信任度和消费能力。现在数据库中有一张用户交易表order，其中有userid（用户ID）、amount（消费金额）、paytime（支付时间），请写出对应的SQL语句，**查出每个用户第一单的消费金额**。

② 当你发现本月的支付用户数环比上月大幅下跌（超30%），你会如何去探查背后的原因？请描述你的思路和其中涉及的关键指标 

  ③ 为了更好的理解用户，我们通常会基于用户的特征对用户进行分类，便于更加精细化的理解用户，设计产品和运营玩法，请你设计对应的聚类方法，包括重点的用户特征的选择及聚类算法并说明其基本原理和步骤



```sql
select userid,amount from (select userid,amount,min(paytime) group  by userid )
```





```
select a.* from
order a join (
  ``select userid, min(paytime) min_time
  ``from order group by userid
  ``) b
on a.userid = b.userid
and a.paytime = b.min_time;
```

<br>

(2)[176. 第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

编写一个 SQL 查询，获取 Employee 表中第二高的薪水（Salary） 。

+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
例如上述 Employee 表，SQL查询应该返回 200 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 null。

```sql
select ifnull(select max(Salary) SecondHighestSalary
from employee
where
salary<(select max(salary) from employee),null) as SecondHighestSalary   
# 小于最大的，剩下的最大的就是第二大的，嵌套 ,外层再加一个判断，如果是空的化返回一个null
```

或者分页<br>

```sql
select IFNULL((select distinct(Salary) 
from Employee
order by Salary desc
limit 1,1),null) as SecondHighestSalary
```



#### （3）[177. 第N高的薪水](https://leetcode-cn.com/problems/nth-highest-salary/)

这下有思路了，分页是个不错的操作啊，

SQl也是可以写函数的，写好后怎么用呢？我查查

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    set N = N-1;  # 声明一个东西用set，赋值用 :=   海象算子？
    IF N<0 THEN
    RETURN NULL;
    ELSE
    RETURN (
        # Write your MySQL query statement below.
        ###########
        select ifnull(
            (select Salary from Employee group by Salary order by Salary limit N,1) 
        ,null) as getNthHighestSalary
        ###########
    );
    END IF;
END

# 先执行这个，后调用函数，往后再单独调用，
select getNthHighestSalary(4)    # 这样调用就可以

# 分页是limit n,1   (第一个n是从第n个顺序开始，然后往后去1个的意思，注意sql的也是从下标0开始)

# 删除自定义函数
DROP FUNCTION [IF EXISTS] function_name
```

#### （4）[178. 分数排名](https://leetcode-cn.com/problems/rank-scores/)

排序相关，这个csdn的帖子也讲得很好的了。https://blog.csdn.net/weixin_42228338/article/details/83417411<br>

编写一个 SQL 查询来实现分数排名。如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。<font color=red><b>(也就是说这是连续排名)</b></font>

种类：

+ <font color=red>连续的排名</font>不空缺（1，2，3，4）下面的解法，或者mysql8.0
  + 可以使用窗口函（需要版本支持）
  + **其中需要注意的是：over（partition by class order by property） 按照property排序进行累计，order by是个默认的开窗函数，按照class分区。这里面是按照class来进行不同类别的排序的**
    + <font color=blue>dense_rank() over(patition by something order by other......) </font>这种是**连续的排序，1，1，1，1，2，2，3，3，3这样，适合这种。**不跳跃1，2，3名 （dense 稠密的 ，排序就是连续的排序）
  + 不使用窗口函数
    + 使用声明中间变量来存储
+ <font color=red>跳跃的排名</font>（1，1，3）这种是默认的排序，默认就是跳跃的。
  + <font color=blue>rank()  over(patition by something order by other......) </font> 这个是**跳跃排序，相同的排1，1，然后下一个就是3**
+ <font color=red>普通的序号</font>【1，1，1，2，3】-》（1，2，3，1，1） 
  + 序号函数：<font color=blue>row_number()  over(patition by something order by other......)</font>，**相同的值里面排序，多个同为100的值排1，2，3（按一定的顺序排）**

mysql中变量不用事前申明，在用的时候直接用“@变量名”使用就可以了。
第一种用法：set @num=1; 或set @num:=1; //这里要使用变量来保存数据，直接使用@num变量
第二种用法：select @num:=1; 或 select @num:=字段名 from 表名 where ……
注意上面两种赋值符号，使用set时可以用“=”或“：=”，但是使用select时必须用“：=赋值”

第三种用法：select 字段名1,字段名2 into @变量1,@变量2 from 表名 where ......

在函数或存储过程或触发器中，在不能使用set的时候推荐第三种，因为第二种会在执行时返回查询结果，这在函数或触发器中会报 “Not allowed to return a result set from a function”错误。而第三种则不会报错

### ①①不使用窗口函数，<font color=red>连续的排名</font>，相同分数排名相同(自我连接self join)

常规使用声明中间变量来处理的情况mysql8.0以下：

https://www.bbsmax.com/A/l1dyPRr05e/

6	1
6	1
4	2
3	3
2	4
1	5
1	5

```python
set @rank = 0;
set @prescore = 0;

SELECT *,case 
	when @prescore=score then @rank
	when @prescore:=score then @rank:=@rank+1   # 第二个的时候这种算when是算没有判断条件的，赋值结束直接进入第二个then
	end as RANK
FROM `test` order by score desc
```

或者

```python
SET @rank=0;
 
SET @PREScore=0;
 
SELECT Score,RANk,sss FROM(
 
SELECT Score,IF(@PREScore=Score,@rank,@rank:=@rank+1) AS RANK,
 
@PREScore:=Score  as  sss
 
FROM test
ORDER BY Score DESC)S;  
```

再或者使用这种取巧的方法，比前面有多少个比自己大的。

````
select s1.score,(select count(*)+1 from 
  			(select distinct score from Scores) s2 where s2.score>s1.score) as Rank 
from Scores s1 order by Rank  
#上面这个和下下面这个是等价的，这个就是自连的例子
# 如果需要只看独立的还可以group by Rank 加上
# 这种叫做相关子查询，所以先执行外层的那个别名
SELECT id2 ,(select count(distinct only.id2)+1 from hello only  where a.id2>only.id2) as rank FROM `Scores` as a order by rank 

````



## ①②mysql8.0或者以上支持窗口函数

```sql
select score as "Score" , dense_rank() over (order by score desc) as "Rank" from Scores
# 这种需要mysql8.0 才支持窗口函数  ,连续的排名
```

------



## ②①不连续的,<font color=red>跳跃的排位</font>则需要中间变量来存储一下排位值。1，1，1，4这样前面会占用排名

这种复杂一点点，多一个变量用来记录连续的排位，

1	1
1	1
2	3
3	4
4	5
6	6
6	6

```sql
SET @RANK=0;
SET @PREScore=0;
SET @INCRANK=1;
SELECT Score,RANK FROM(
SELECT Score,@RANK:=IF(@PREScore=Score,@RANK,@INCRANK) AS RANK,
@PREScore:=Score,
@INCRANK:=@INCRANK+1
FROM scores
ORDER BY Score DESC
)S;
```

### ②②窗口函数

```sql
# 跳跃的排名 mysql8.0 或者以上才支持
select score,rank() over(patition by class order by score) order by score desc
```

------



## ③①普通的相同值下进行序号的(相同的值下进行行号排列的)

逻辑改一下就可以，可行的。

```sql
# 1.
SET @RANK=0;
SELECT score,@RANK:=@RANK+1
FROM test
ORDER BY score DESC;  # 但是这种不带分类的，只是直接从头到尾标序号

包含分类的，需要设置多一个type来做分类
# 2.上面版本的增强，加上一个类型的判断
set @rank = 0;
set @type = null;
select *,case
		when @type=id then  @rank:=@rank+1
		when (@type:=id) is not null then @rank:=1   # 一般的赋值都是放到了第二个when里面来的，第一个是判断是同一类
		end as RANK from test2 order by id,date
```

### ③②对应着的是窗口函数 row_number() 

```sql
select score,row_number() over(patition by class order by score) order by score desc
```

<br>

#### （5）[180. 连续出现的数字](https://leetcode-cn.com/problems/consecutive-numbers/)

编写一个 SQL 查询，查找所有至少连续出现三次的数字。

+----+-----+
| Id | Num |
+----+-----+
| 1  |  1  |
| 2  |  1  |
| 3  |  1  |
| 4  |  2  |
| 5  |  1  |
| 6  |  2  |
| 7  |  2  |
+----+-----+
例如，给定上面的 Logs 表， 1 是唯一连续出现至少三次的数字。

+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+

解答：

```sql
# 这种可以通过连续的 自连接 来进行查找
select l1.num from Logs l1,Logs l2,Logs l3 where
 l1.id = l2.id-1 and 
 l2.id = l3.id -1 and 
 l1.num = l2.num and 
 l2.num = l3.num 
 
 
 # 这里面使用id 递增作为连续的标准，其实抛砖引玉的化，换做其他的字段作为连续的也需要会做，比如连续三天的，日期字段作为连续的（使用日期的时候间隔使用 date_sub(date,interval - 1 day/month/year) 这种mysql自带的日期处理函数才是比较准确的），不过这个需要一般连续次数比较少，然后与 求最大连续登陆天数还是有点不同的，
 
```

## 另一种还是使用设置变量的，这种的效率应该会高一点，比自交的方式

而且这种方式可以分类变形求得每个用户最大连续数字，越来越接近最大连续登陆天数， 某用户，这些才是我们需要多去思考的。 

```sql
select distinct Num as ConsecutiveNums
from (
  select Num, 
    case 
      when @prev = Num then @count := @count + 1
      when (@prev := Num) is not null then @count := 1  # 这儿是先把Num赋值给@prev ，然后再进行判断是否为空
    end as CNT
  from Logs, (select @prev := null,@count := null) as t  # 这里面的（select @prev :=null...去掉也没问题，这两个其实最终查询的时候并没有用到，可以直接放到前面去赋初值操作
) as temp
where temp.CNT >= 3
```

后面的都是自己衍生出来的用来举一反三的，<br>

这种是变形后直接得到各个id的最大

```sql
set @prev = null; # 存放前面的另一个字段，用来作为判断是否连续用的。
set @count = null;   # 放前面来赋初值操作,相比较前面的那个放后面新生成两个为null的列
select ID,max(CNT) AS '最大连续' # DISTINCT ID # distinct real_id as ConsecutiveNums
from (
  select id ,
    case 
      when @prev = id then @count := @count + 1
      when (@prev := id) is not null then @count := 1   # 只要出现连续的id变化，那就把count重新赋初值变为1，然后继续，所以查询的结果必须是按 需要的顺序进行排序，然后才能开始计数。
    end as CNT  # 这个字段其实是用来计数连续的
  from test2 as t
) as temp 
#where temp.CNT >= 2
group by id
```

<br>

### （6）自交和这种计数的思考对于解决找到用户最大连续登陆天数，或者说得到<font color=red>用户连续登陆天数</font>

自交的方法适合连续登录天数比较少（但是一般比较多才有意义）

<img src=".\static\登录用户简化表格.jpg" alt="登录用户简化表格" style="zoom:80%;" />

<b>首先是mysql中常用的处理时间的函数</b>

+ DATE_ADD(date,interval 13 day) 函数 # 向日期增加指定天数的日子

+ DATE_SUB()   # 与上同，这个是减

+ DATEDIFF(date1,date2)  #得到date1 与date2相差的天数 ，前面减去后面，值有正，有负

+ DATE_FORMAT() 函数用于以不同的格式显示日期/时间数据

  ```
  select  
  DATE_FORMAT(NOW(),'%b %d %Y %h:%i %p'),
  DATE_FORMAT(NOW(),'%m-%d-%Y'),
  DATE_FORMAT(NOW(),'%d %b %y'),
  DATE_FORMAT(NOW(),'%d %b %Y %T:%f')
  ```

+  now()   # 返回当前的日期和时间

+ EXTRACT(year/month/day, date)  # 从日期中提取出这些东西，和YEAR(),MONTH(),DAY() 一样

  <br>

### 这个的思路也是

+ 先根据条件按一定的顺序进行排列，(与这个等价的排序，mysql 8.0  row_number() over(partition by userid order by loginDate))   # date要转化成按Y-M-D的格式
+ 然后再把连续的进行计数它们最大重复次数
+ 然后得出它们的最大重复次数，这个就是它们按照一天间隔的连续天数了，很是巧妙。

但是遇到这种最大连续登录的，出现了id，date，排序rank （自己生成）这几个关键字段，这种order by date是不可以的，因为中间可能有缺少的日子，逻辑不够完善。

有一种方法就是先row_number(等价的处理)

<img src="static\各个用户最长连续登陆天数.jpg" alt="各个用户最长连续登陆天数" style="zoom:80%;" />

```sql
# 有一个很巧妙的方法，就是row_number
set @rank = 0;
set @type = null;
select id,max(num) "最大连续登陆天数" from (
select id,date,tempdate,count(tempdate) as num from (
select *,DATE_FORMAT(date_sub(date,INTERVAL RANK DAY) ,"%Y-%m-%d")as tempdate  from (
select id,date,case
		when @type=id then  @rank:=@rank+1
		when (@type:=id) is not null then @rank:=1   # 一般的赋值都是放到了第二个when里面来的，第一个是判断是同一类
		end as RANK
		from test2 order by id,date  )as t 
		) as y group by tempdate,id
		) as x group by id order by max(num)
```

之后再来考虑优化是否可用的问题，优化， 要么是时间换空间，要么是空间换时间，一般都是空间换时间。这种优化方式。



（7）[181. 超过经理收入的员工](https://leetcode-cn.com/problems/employees-earning-more-than-their-managers/)

这个比较简单， 自己表内的字段和自己其他字段进行对比，可以考虑自连接

```sql
Employee 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。

+----+-------+--------+-----------+
| Id | Name  | Salary | ManagerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+
给定 Employee 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。在上面的表格中，Joe 是唯一一个收入超过他的经理的员工。

+----------+
| Employee |
+----------+
| Joe      |
+----------+

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/employees-earning-more-than-their-managers
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

很简单的一种解法。



```sql
# Write your MySQL query statement below
select e1.Name as Employee from Employee e1 ,Employee e2 
    where e1.ManagerId = e2.Id and e1.Salary > e2.Salary
```

发散思维,这种虽然感觉可能有点儿不符合题意

```sql
# Write your MySQL query statement below
SELECT 
    Name Employee
FROM
    Employee AS a
WHERE
    Salary > (SELECT 
            Salary
        FROM
            Employee
        WHERE
            Id = a.Managerid)
```



(8)[182. 查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)

这个也是拓宽思维用的

```sql
182. 查找重复的电子邮箱
编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。

示例：

+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
根据以上输入，你的查询应返回以下结果：

+---------+
| Email   |
+---------+
| a@b.com |
+---------+
说明：所有电子邮箱都是小写字母。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/duplicate-emails
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

答案：

第一种

```sql
# Write your MySQL query statement belo
select distinct p1.Email from Person p1 ,Person p2 where
    p1.Id != p2.Id and 
        p1.Email = P2.Email
```

第二种,种种没有链接表，速度更快

```sql
#还有就是常规的计数了，group by的
select Email from (
select Email,count(Email) from Person group by Email
    ) where count(Email)>2 
```



（9）找到重不购物的用户

```sql
某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

Customers 表：

+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
Orders 表：

+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
例如给定上述表格，你的查询应返回：

+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/customers-who-never-order
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

答案很简单

```sql
# 一中就是嵌套查询不包含
select distinct Name Customers from Customers where Id not in (select distinct CustomerId from Orders )

# 连接表进行查询也可以，判断null 就可以
select dis

```



（10）七日留存率 mysql 查询次日留存 三日留存 七日留存

https://blog.csdn.net/daliangliangliangge/article/details/102743004 

这个例子就很好，

这种才更加的现实。这些东西是实际使用的指标。留存率就是  比如三日留存率，三天连续登陆的人的人数/三天前就登陆过的人的总人数 * 100% 加上一个百分号这样就可以了。  这种理解是对的，几天后选出

留存率=新增用户中登录用户数/新增用户数*100%（一般统计周期为天）

新增用户数：在某个时间段（一般为第一整天）新登录应用的用户数；只出现一次日期的登陆

登录用户数：登录应用后至当前时间，至少登录过一次的用户数；

第N日留存：指的是新增用户日之后的第N日依然登录的用户占新增用户的比例

第1日留存率（即“次留”）：（当天新增的用户中，新增日之后的第1天还登录的用户数）/第一天新增总用户数；

第2日留存率：（当天新增的用户中，新增日之后的第2天还登录的用户数）/第一天新增总用户数；

第3日留存率：（当天新增的用户中，新增日之后的第3天还登录的用户数）/第一天新增总用户数；

第7日留存率：（当天新增的用户中，新增日之后的第7天还登录的用户数）/第一天新增总用户数；

第30日留存率：（当天新增的用户中，新增日之后的第30天还登录的用户数）/第一天新增总用户数

```sql
-- member-id为用户唯一标识
-- create_time为用户登陆时间，一个用户可以存在多次登陆记录
-- 以2019-09-01为基准，计算一天的1到30日留存数
SELECT retain_day, COUNT(member_id)
FROM (
-- 用户在当天可以有多次登陆行为，所以需要distinct
	SELECT DISTINCT member_id, DATE_FORMAT(create_time, '%Y-%m-%d') AS date1
		, DATEDIFF(create_time, '2019-09-01') AS retain_day
	FROM bss_memb_active
	WHERE DATE_FORMAT(create_time, '%Y-%m-%d') BETWEEN '2019-09-01' AND '2019-09-30'
-- 需要去除不在基准日的数据
		AND member_id IN (
			SELECT DISTINCT member_id
			FROM bss_memb_active
			WHERE DATE_FORMAT(create_time, '%Y-%m-%d') = '2019-09-01'
		)
) aa
GROUP BY retain_day  # 多日留存率对比的。

```


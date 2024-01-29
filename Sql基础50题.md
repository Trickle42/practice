# [Leetcode 高频SQL50题](https://leetcode.cn/studyplan/sql-free-50/)


## 12.21
### [1.1](https://leetcode.cn/problems/find-customer-referee/description/?envType=study-plan-v2&envId=sql-free-50)
**示例：**

<img width="209" alt="image" src="https://github.com/Trickle42/practice/assets/67224782/b1b65fdd-94b3-40b3-b2ae-78b56267e25a">

**解答：**
```sql
SELECT name FROM Customer WHERE referee_id <> 2 OR referee_id is NULL;
```
> 1. sql存在三种真值：TRUE、FALSE、UNKNOWN；NULL在逻辑式的左右边都会使得式子变为UNKNOWN
> 2. <>表示不等于；

## 12.22
### [2.1](https://leetcode.cn/problems/rising-temperature/?envType=study-plan-v2&envId=sql-free-50)
**题目**

找到前一天温度比当天温度低的日子

**示例：**
<img width="245" alt="image" src="https://github.com/Trickle42/practice/assets/67224782/3736eeab-f9ec-4cd7-b077-8d1cd1241f73">

**解答：**
1. Cross Join
```sql
select a.ID, a.date
from weather as a cross join weather as b 
     on datediff(a.date, b.date) = 1
where a.temp > b.temp;
```

> Cross Join：这意味着对于表中的每一行，都会与表中的所有其他行组合，创建一个笛卡尔积，On可以限制筛选条件
> 
> datediff函数：用于计算两个日期之间的差异

2. 窗口函数-Lag
```sql
select id
from
    (select 
        id,
        temperature,
        recordDate,
        lag(recordDate,1,0) over(order by recordDate) as last_date,
        lag(temperature,1,0) over(order by recordDate) as last_temperature
    from Weather) a
where temperature > last_temperature and datediff(recordDate, last_date) = 1
```
> Lag：LAG (expression, offset, default) OVER (PARTITION BY partition_expression ORDER BY sort_expression)
> expression: 要获取的值或表达式。
> 
> offset: 指定要获取的前几行的偏移量，如果不指定，默认为1，表示前一行。
> 
> default: 可选参数，如果指定的偏移行不存在（例如，对于第一行），则使用此默认值。


## 12.22
**3.1**
> 自己的做法
```sql
SELECT a.machine_id, ROUND(AVG(b.timestamp-a.timestamp),3) AS processing_time
FROM (SELECT *  FROM Activity WHERE activity_type = 'start') AS a
LEFT JOIN (SELECT *  FROM Activity WHERE activity_type = 'end')AS b
on a.machine_id = b.machine_id AND a.process_id = b.process_id
GROUP BY a.machine_id ORDER BY a.machine_id desc;
```

> 更好的做法
```sql
select 
a1.machine_id,#在JOIN的情况下，摘取连接键的时候必须明确是哪个表的。
round(avg(a2.timestamp -a1.timestamp ),3) as processing_time 
from  Activity as a1 left join Activity as a2 on 
a1.machine_id=a2.machine_id and 
a1.process_id=a2.process_id and 
a1.activity_type ='start' and 
a2.activity_type ='end' 
group by machine_id;
```
**注意点**
1. 平均函数叫AVG
2. ROUND(expression,number_of_decimals) 保留小数位数的函数
3. JOIN的on条件可以直接进行筛选。
4. 在JOIN的情况下，选取连接键的时候必须明确是哪个表的。

**3.2 LEFT JOIN与CROSS JOIN连用**

学生们参加各科测试的次数

<img width="208" alt="image" src="https://github.com/Trickle42/practice/assets/67224782/6cc26e73-217d-49cf-a789-df63c2bf3b40">
<img width="359" alt="image" src="https://github.com/Trickle42/practice/assets/67224782/4a02941f-734b-4bca-ad63-188944605c98">

```sql
SELECT 
    s.student_id, s.student_name, sub.subject_name, IFNULL(grouped.attended_exams, 0) AS attended_exams
FROM 
    Students s
CROSS JOIN 
    Subjects sub
LEFT JOIN (
    SELECT student_id, subject_name, COUNT(*) AS attended_exams
    FROM Examinations
    GROUP BY student_id, subject_name
) grouped 
ON s.student_id = grouped.student_id AND sub.subject_name = grouped.subject_name
ORDER BY s.student_id, sub.subject_name;
```

**注意点**

0.问题是如何产生并处理null值。必须用到left join，因为cross和inner都不产生。
1. LEFT join 和 INNER JOIN的区别 ： LEFT JOIN 返回左表中的所有行，以及右表中连接条件匹配的行。
如果连接条件在右表中没有匹配的行，那么将在结果中显示右表的列，但其他部分将包含NULL。

但INNER JOIN只返回两边条件都允许的行

2. IFNULL(expression, if null value):left join后会产生null，必须修正
3. 子查询必须要有一个别名，否则会报错
"Every derived table must have its own alias" ，这是一个 SQL 错误消息，通常发生在查询中使用了子查询（Derived Table），但没有为子查询指定别名（Alias）。

**3.3 一张表不同列相连接 INNER JOIN**

编写一个解决方案，找出至少有五个直接下属的经理。

<img width="271" alt="image" src="https://github.com/Trickle42/practice/assets/67224782/61830da3-446f-4029-bc29-7cb0bdebe73a">

**注意点**

0. 需要一步步进行拆解
1. 找到每一个经理的下属员工数量进行罗列,需要分立两个表，通过JOIN（INNER JOIN）操作，回
```sql
select Manager.Name, count(Report.Id) as cnt
from
Employee as Manager join Employee as Report
on Manager.Id = Report.ManagerId # 会复制manager.ID以其在managerID出现的次数
group by Manager.Id
```
<img width="461" alt="image" src="https://github.com/Trickle42/practice/assets/67224782/6dd4564e-c0e5-4f05-b0ec-650b586c3f70">

2. Having 子句操作进行操作
```sql
select Manager.Name
from
Employee as Manager join Employee as Report
on Manager.Id = Report.ManagerId
group by Manager.Id
having count(Report.Id) >= 5
```

**
## 12.31
**4.1**
> Register a
inner join
(select count(distinct user_id) as fenmu from Users)b on 1=1

这里的 1=1 是一个始终为真的条件，它实际上是一个占位符。在一些动态生成SQL语句的情况下，它可以作为条件的起点。在这个查询中，它意味着对主查询的每一行都与子查询的每一行进行连接，因为无论什么时候，1总是等于1

因为count(distinct user_id)只有一个数据，所以可以依次全部连接

>  ROUND(SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 END) / COUNT(*) * 100, 2) AS poor_query_percentage
>  round(avg(rating<3)*100,2) as poor_query_percentage

两者等价，用逻辑表达式做虚拟变量求和更为方便。

## 1.2
> DATE_FORMAT(trans_date, '%Y-%m') AS month
> 
> 2018-12-18 -> 2018-12

DATE_FORMAT(date, format) ：用于以不同的格式显示日期/时间数据。date 参数是合法的日期，format 规定日期/时间的输出格式。

> 找到每一个客户的首个订单的信息
```sql
where (customer_id, order_date) in (
    select customer_id, min(order_date)
    from delivery
    group by customer_id)
```
要求顾客id和订单日期同时对应，以防遇到某一订单日期对于不同顾客可能是首次，或是非首次的情况。

where in+ 聚合函数的用法值得借鉴

> 报告在首次登录的第二天再次登录的玩家的 比率
```sql
select IFNULL(round(count(distinct(Result.player_id)) / count(distinct(Activity.player_id)), 2), 0) as fraction
from (
  select Activity.player_id as player_id
  from (
    select player_id, DATE_ADD(MIN(event_date), INTERVAL 1 DAY) as second_date
    from Activity
    group by player_id
  ) as Expected INNER JOIN Activity
  where Activity.event_date = Expected.second_date and Activity.player_id = Expected.player_id
) as Result CROSS JOIN Activity
```
> DATE_ADD(date, INTERVAL value unit)
> 
> -- 在当前日期上增加 3 天
SELECT DATE_ADD(CURDATE(), INTERVAL 3 DAY) AS new_date;
> CROSS JOIN方便运算两张不同表的数据

>  查询最近30日的信息
>
> HAVING activity_date BETWEEN DATE_ADD('2019-07-27',INTERVAL -29 day) and '2019-07-27';
>
> HAVING datediff('2019-07-27', activity_date) between 0 and 29

>  查询仅在2019年春季出现的数据，即如果在其他时间也出现，则不会被筛选
> 
>  HAVING MIN(sale_date) >= '2019-01-01' AND MAX(sale_date) <= '2019-03-31'
>
>  having count(sale_date between '2019-01-01' and '2019-03-31' or null) = count(*)

>  JOIN的妙用，自连接，在遇到两列的值有相同的地方，同时又要根据两列做不同的筛选时
```sql
select a.employee_id as 'employee_id', a.name as 'name', count(b.employee_id) as 'reports_count', round(avg(b.age), 0) as 'average_age'
from Employees a inner join Employees b
on a.employee_id = b.reports_to
group by a.employee_id
order by a.employee_id
```


## 1.26

**数字连续三次出现**

1. 运用自连接
```sql
SELECT distinct l1.num AS ConsecutiveNums 
FROM
Logs l1 INNER JOIN
Logs l2 ON l1.id = l2.id-1 INNER JOIN
Logs l3 ON l2.id = l3.id-1
WHERE l1.Num = l2.Num
    AND l2.Num = l3.Num
```

2. 运用lead和lag窗口函数构建子查询
```sql
SELECT distinct Num AS ConsecutiveNums
FROM (
    SELECT id, Num,
           LEAD(Num) OVER (ORDER BY id) AS nextNum,
           LAG(Num) OVER (ORDER BY id) AS prevNum
    FROM Logs
) AS subquery
WHERE Num = prevNum AND Num = nextNum;
```
3. UNION ALL 在分类汇总中的妙用&SUM+条件语句可以快捷筛选符合条件的数量

场景： 查询每个工资类别的银行账户数量。 工资类别如下：

"Low Salary"：所有工资 严格低于 20000 美元。

"Average Salary"： 包含 范围内的所有工资 [$20000, $50000] 。

"High Salary"：所有工资 严格大于 50000 美元。

结果表 必须 包含所有三个类别。 如果某个类别中没有帐户，则报告 0 。

分析： 如果单纯用CASE WHEN + COUNT，则如果没有筛选到某个类别，则不满足需求要求返回0，可以用UNION进行连接

```sql
SELECT 
    'Low Salary' AS category, 
    SUM(income < 20000) AS accounts_count
FROM Accounts
UNION ALL
SELECT 
    'Average Salary' AS category, 
    SUM(income BETWEEN 20000 AND 50000) AS accounts_count
FROM Accounts
UNION ALL
SELECT 
    'High Salary' AS category, 
    SUM(income > 50000) AS accounts_count
FROM Accounts;
```

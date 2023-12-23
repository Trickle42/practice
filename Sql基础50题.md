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
4. 在JOIN的情况下，摘取连接键的时候必须明确是哪个表的。

**3.2**

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


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


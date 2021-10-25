<!--
date: 2021-10-25T22:34:12+08:00
lastmod: 2021-10-25T22:34:12+08:00
-->
## 627. Swap Salary

### 题目

https://leetcode.com/problems/swap-salary/description/

```
Salary table:
+----+------+-----+--------+
| id | name | sex | salary |
+----+------+-----+--------+
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
+----+------+-----+--------+

Result table:
+----+------+-----+--------+
| id | name | sex | salary |
+----+------+-----+--------+
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
+----+------+-----+--------+
(1, A) and (3, C) were changed from 'm' to 'f'.
(2, B) and (4, D) were changed from 'f' to 'm'.
```

sex字段是枚举类型，只有m和f两种值，现在要求只能用一个简单的update语句来将表中数据的sex字段的值对调，即m变为f，f变为m；不允许使用select之类的语句。
<!--more-->
### 答案

有两种思路，一种是常规的，也是实践项目中常用的`case when`写法，个人更推荐这种：

```sql
UPDATE salary
SET sex = (CASE WHEN sex = "m" 
                THEN "f" 
                ELSE "m"
           END);
```

一种是使用异或来实现，两个相等的数异或的结果为 0，而 0 与任何一个数异或的结果为这个数。这种做法需要先把sex字段的值转换为ASCII码再异或，最后再转为字符：

```sql
UPDATE salary
SET sex = CHAR ( ASCII(sex) ^ ASCII( 'm' ) ^ ASCII( 'f' ) );
```

## 620. Not Boring Movies

### 题目

https://leetcode.com/problems/not-boring-movies/description/

```
Cinema table:
+----+------------+-------------+--------+
| id | movie      | description | rating |
+----+------------+-------------+--------+
| 1  | War        | great 3D    | 8.9    |
| 2  | Science    | fiction     | 8.5    |
| 3  | irish      | boring      | 6.2    |
| 4  | Ice song   | Fantacy     | 8.6    |
| 5  | House card | Interesting | 9.1    |
+----+------------+-------------+--------+

Result table:
+----+------------+-------------+--------+
| id | movie      | description | rating |
+----+------------+-------------+--------+
| 5  | House card | Interesting | 9.1    |
| 1  | War        | great 3D    | 8.9    |
+----+------------+-------------+--------+

Write an SQL query to report the movies with an odd-numbered ID and a description that is not "boring".

Return the result table in descending order by rating.
```

获取奇数id、描述不为无聊且按照等级倒序排序。

### 答案

```sql
select * from Cinema where id % 2 = 1 and description != 'boring' order by rating desc;
```

## 184. Department Highest Salary

### 题目

https://leetcode.com/problems/department-highest-salary/description/

```
Employee table:
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Jim   | 90000  | 1            |
| 3  | Henry | 80000  | 2            |
| 4  | Sam   | 60000  | 2            |
| 5  | Max   | 90000  | 1            |
+----+-------+--------+--------------+

Department table:
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+

Write a SQL query to find employees who have the highest salary in each of the departments. For the above tables, your SQL query should return the following rows (order of rows does not matter).
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Jim      | 90000  |
| Sales      | Henry    | 80000  |
+------------+----------+--------+
```

查找每个部门中收入最高者的信息。

### 答案

需要先创建临时表，再查找每个部门中收入最高者的信息。

```sql
SELECT
    D.NAME Department,
    E.NAME Employee,
    E.Salary
FROM
    Employee E,
    Department D,
    ( SELECT DepartmentId, MAX( Salary ) Salary 
     FROM Employee 
     GROUP BY DepartmentId ) M
WHERE
    E.DepartmentId = D.Id
    AND E.DepartmentId = M.DepartmentId
    AND E.Salary = M.Salary;
```

## 176. Second Highest Salary

### 题目

https://leetcode.com/problems/second-highest-salary/description/

```
Employee table:
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+

For example, given the above Employee table, the query should return 200 as the second highest salary. If there is no second highest salary, then the query should return null.
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

找到第二高的工资，如果找不到则返回null。

### 答案

`max()`在找不到最大值时会返回null，不建议使用`limit 1,1`，如果表中存在重复的第一高的工资，会得到错误的答案。

```sql
SELECT max(Salary)
FROM Employee
WHERE Salary < (SELECT max(Salary) FROM Employee);
```

## 177. Nth Highest Salary

### 题目

https://leetcode.com/problems/nth-highest-salary/

```
Employee table:
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+

For example, given the above Employee table, the nth highest salary where n = 2 is 200. If there is no nth highest salary, then the query should return null.
+------------------------+
| getNthHighestSalary(2) |
+------------------------+
| 200                    |
+------------------------+
```

求第N高的工资，找不到就返回null。

### 答案

这题明确了第n高的工资是n*100的数值，也就是说不存在重复的数据，需要用`limit`来做。

```sql
CREATE FUNCTION getNthHighestSalary ( N INT ) RETURNS INT BEGIN

SET N = N - 1;
RETURN ( 
    SELECT ( 
        SELECT DISTINCT Salary 
        FROM Employee 
        ORDER BY Salary DESC 
        LIMIT N, 1 
    ) 
);

END
```

## 178. Rank Scores

### 题目

https://leetcode.com/problems/rank-scores/description/

```
Scores table:
+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+

For example, given the above Scores table, your query should generate the following report (order by highest score):
+-------+---------+
| score | Rank    |
+-------+---------+
| 4.00  | 1       |
| 4.00  | 1       |
| 3.85  | 2       |
| 3.65  | 3       |
| 3.65  | 3       |
| 3.50  | 4       |
+-------+---------+
```

按分数排行，同样分数并列排名，且不会导致之后的名次产生空挡，即下一个排名比并列排名的名次大1。

### 答案

两种思路，一种是直接使用窗口函数`dense_rank() over ()`，一种是使用自连接和统计函数。

窗口函数：

```sql
select score 'Score', dense_rank() over(order by score desc) 'Rank' from Scores order by score desc;
```

自连接：

```sql
SELECT
    S1.score 'Score',
    COUNT( DISTINCT S2.score ) 'Rank'
FROM
    Scores S1
    INNER JOIN Scores S2
    ON S1.score <= S2.score
GROUP BY
    S1.id, S1.score
ORDER BY
    S1.score DESC;
```

### SQL Schema

```
DROP TABLE
IF
    EXISTS Scores;
CREATE TABLE Scores ( Id INT, Score DECIMAL ( 3, 2 ) );
INSERT INTO Scores ( Id, Score )
VALUES
    ( 1, 4.1 ),
    ( 2, 4.1 ),
    ( 3, 4.2 ),
    ( 4, 4.2 ),
    ( 5, 4.3 ),
    ( 6, 4.3 );
```

## 180. Consecutive Numbers

### 题目

https://leetcode.com/problems/consecutive-numbers/description/

```
Logs table:
+----+-----+
| Id | Num |
+----+-----+
| 1  | 1   |
| 2  | 1   |
| 3  | 1   |
| 4  | 2   |
| 5  | 1   |
| 6  | 2   |
| 7  | 2   |
+----+-----+

Result table:
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
1 is the only number that appears consecutively for at least three times.
```

Id是自增序列，查询连续出现3次以上的Num。

### 答案

需要自连接2次来查询：

```sql
select distinct a.Num ConsecutiveNums
from LOGS a, LOGS b, LOGS c
where a.Id = b.Id - 1 and b.Id = c.Id - 1
and a.Num = b.Num and b.Num = c.Num;
```

### SQL Schema

```
DROP TABLE
IF
    EXISTS LOGS;
CREATE TABLE LOGS ( Id INT, Num INT );
INSERT INTO LOGS ( Id, Num )
VALUES
    ( 1, 1 ),
    ( 2, 1 ),
    ( 3, 1 ),
    ( 4, 2 ),
    ( 5, 1 ),
    ( 6, 2 ),
    ( 7, 2 );
```

## 626. Exchange Seats

https://leetcode.com/problems/exchange-seats/description/

### 题目

```
table seat:
+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Abbot   |
|    2    | Doris   |
|    3    | Emerson |
|    4    | Green   |
|    5    | Jeames  |
+---------+---------+

Change seats for the adjacent students, result table:
+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Doris   |
|    2    | Abbot   |
|    3    | Green   |
|    4    | Emerson |
|    5    | Jeames  |
+---------+---------+
```

交换相邻两个人的座位，id是自增序列。

### 答案

这个可以通过三个不同的查询来代表不同的情况，然后用union联合起来，最终再通过id排序。

```sql
select a.id, b.student
from seat a
left join seat b on a.id + 1 = b.id and a.id % 2 = 1
where b.student is not null
union
select a.id, b.student
from seat a
left join seat b on a.id - 1 = b.id and a.id % 2 = 0
where b.student is not null
union
select id, student
from seat
where id = (select max(id) from seat) and id % 2 = 1
order by id;
```

### SQL Schema

```
DROP TABLE
IF
    EXISTS seat;
CREATE TABLE seat ( id INT, student VARCHAR ( 255 ) );
INSERT INTO seat ( id, student )
VALUES
    ( '1', 'Abbot' ),
    ( '2', 'Doris' ),
    ( '3', 'Emerson' ),
    ( '4', 'Green' ),
    ( '5', 'Jeames' );
```

## 参考链接

* [SQL 练习](http://www.cyc2018.xyz/%E6%95%B0%E6%8D%AE%E5%BA%93/SQL%20%E7%BB%83%E4%B9%A0.html#_595-big-countries)
* [Simple query which handles the NULL situation](https://leetcode.com/problems/second-highest-salary/discuss/52957/Simple-query-which-handles-the-NULL-situation)
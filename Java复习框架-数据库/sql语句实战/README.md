### [Leetcode595大的国家](https://leetcode-cn.com/problems/big-countries/description/?utm_source=LCUS&utm_medium=ip_redirect&utm_campaign=transfer2china)

如果一个国家的面积超过 300 万平方公里，或者人口超过 2500 万，那么这个国家就是大国家。

```sql
# Write your MySQL query statement below
select name,population,area 
from World 
where area>3000000 or population>25000000;
```

### [Leetcode627 变更性别](https://leetcode-cn.com/problems/swap-salary/description/)

将男变成女，女变成男。

两个相等的数异或的结果为 0，而 0 与任何一个数异或的结果为这个数。

sex 字段只有两个取值：'f' 和 'm'，并且有以下规律：

```shell
'f' ^ ('m' ^ 'f') = 'm' ^ ('f' ^ 'f') = 'm'
'm' ^ ('m' ^ 'f') = 'f' ^ ('m' ^ 'm') = 'f'
```

```sql
# Write your MySQL query statement below\
update salary
set sex = CHAR( ASCII(sex) ^ ASCII('m') ^ ASCII('f') );

```

### [Leetcode620 有趣的电影](https://leetcode-cn.com/problems/not-boring-movies/description/)

需要编写一个 SQL查询，找出所有影片描述为**非** `boring` (不无聊) 的并且 **id 为奇数** 的影片，结果请按等级 `rating` 排列。

作为该电影院的信息部主管，您需要编写一个 SQL查询，找出所有影片描述为非 boring (不无聊) 的并且 id 为奇数 的影片，结果请按等级 rating 排列。

```sql
# Write your MySQL query statement below
select *
from cinema
where id%2=1 and description!='boring'
order by rating desc;
```

### [Leetcode596 超过5名学生的课](https://leetcode-cn.com/problems/classes-more-than-5-students/description/)

有一个`courses` 表 ，有: **student (学生)** 和 **class (课程)**。

请列出所有超过或等于5名学生的课。

> 解题思路：对 class 列进行分组之后，再使用 count 汇总函数统计每个分组的记录个数，之后使用 HAVING 进行筛选。HAVING 针对分组进行筛选，而 WHERE 针对每个记录（行）进行筛选。

```sql
# Write your MySQL query statement below
select class 
from courses 
group by class
having count(distinct student)>=5;
```

### [Leetcode182查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/description/)

编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。

> 解题思路：对Email进行分组，如果并使用count进行计数统计，结果大于等于2的表示email重复。

```sql
# Write your MySQL query statement below
select Email
from Person
group by Email
having count(Id)>=2;
```

### [Leetcode196 删除重复的电子邮箱](https://leetcode-cn.com/problems/delete-duplicate-emails/description/?utm_source=LCUS&utm_medium=ip_redirect&utm_campaign=transfer2china)

编写一个 SQL 查询，来删除 `Person` 表中所有重复的电子邮箱，重复的邮箱里只保留 **Id** *最小* 的那个。

```sql
# Write your MySQL query statement below
delete p1
from Person p1,Person p2
where p1.Email=p2.Email and p1.Id >p2.Id;
```

### [Leetcode175组合两个表](https://leetcode-cn.com/problems/combine-two-tables/description/)

编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：

FirstName, LastName, City, State

> 考察左连接

```sql
# Write your MySQL query statement below
select FirstName,LastName,City,State
from Person p
left join address a
on p.PersonId=a.PersonId

```

### [Leetcode181 超过经理收入的员工](https://leetcode-cn.com/problems/employees-earning-more-than-their-managers/description/)

`Employee` 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。

给定 `Employee` 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。在上面的表格中，Joe 是唯一一个收入超过他的经理的员工。

```sql
# Write your MySQL query statement below
select Name as Employee
from Employee as a
where Salary > (select Salary from Employee where Id=a.ManagerId)

# Write your MySQL query statement below
select e1.name as Employee
from Employee e1 
inner join Employee e2
on e1.ManagerId = e2.Id
and e1.Salary > e2.Salary;
```

### Leetcode183 从不订购的客户

某网站包含两个表，`Customers` 表和 `Orders` 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

> 考察左连接

```java
# Write your MySQL query statement below
select C.Name as Customers
from Customers C
left join Orders O
on C.Id=O.CustomerId
where O.CustomerId is null
```

### [Leetcode184 部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/description/)

`Employee` 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。

`Department` 表包含公司所有部门的信息。

编写一个 SQL 查询，找出每个部门工资最高的员工。对于上述表，您的 SQL 查询应返回以下行（行的顺序无关紧要）。

```sql
# Write your MySQL query statement below
select D.Name as Department, E.Name as Employee, E.Salary as Salary
from Department D,Employee E
where E.DepartmentId=D.Id
and 
(e.Salary,e.DepartmentId) in (select max(Salary),DepartmentId from Employee group by DepartmentId);

```

### [Leetcode176 第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

编写一个 SQL 查询，获取 `Employee` 表中第二高的薪水（Salary） 。

例如上述 `Employee` 表，SQL查询应该返回 `200` 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 `null`。

> 解题思路：最外层用一个select防止不给出值。
>
> 里面用order by 排序之后用limit

```sql
# Write your MySQL query statement below 
# 对其排序
select(
    select DISTINCT Salary as SecondHighestSalary 
    from Employee
    order by Salary DESC
    limit 1,1
)SecondHighestSalary
```

### [Leetcode177 第N高的薪水](https://leetcode-cn.com/problems/nth-highest-salary/)

编写一个 SQL 查询，获取 `Employee` 表中第 *n* 高的薪水（Salary）。

例如上述 `Employee` 表，*n = 2* 时，应返回第二高的薪水 `200`。如果不存在第 *n* 高的薪水，那么查询应返回 `null`。

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

### [Leetcode178 分数排名](https://leetcode-cn.com/problems/rank-scores/description/)

编写一个 SQL 查询来实现分数排名。

如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。

思路： 

- 1.从两张相同的表scores分别命名为s1，s2。 

- 2.s1中的score与s2中的score比较大小。意思是在输出s1.score的前提下，有多少个s2.score大于等于它。比如当s1.salary=3.65的时候，s2.salary中[4.00,4.00,3.85,3.65,3.65]有5个成绩大于等于他，但是利用count(distinct s2.score)去重可得s1.salary3.65的rank为3 
- 3.group by [s1.id](http://s1.id/) 不然的话只会有一条数据 
- 4.最后根据s1.score排序desc

```sql
select s1.score,count(distinct s2.score) as rank
from scores as s1,scores as s2
where s1.score<=s2.score
group by s1.id
order by s1.score desc;
```

```sql
# Write your MySQL query statement below
select Score,dense_rank() over(order by  Score desc) as 'Rank'
from Scores;

```

### [Leetcode180 连续出现的数字](https://leetcode-cn.com/problems/consecutive-numbers/description/)

编写一个 SQL 查询，查找所有至少连续出现三次的数字。

返回的结果表中的数据可以按 **任意顺序** 排列。

> 连续的三个数字

```sql
# Write your MySQL query statement below
select DISTINCT l1.Num as ConsecutiveNums 
from Logs l1, Logs l2,Logs l3
where l1.id = l2.id-1 and l2.id=l3.id-1 and l1.num=l2.num and l2.num=l3.num;

```

### [Leetcode626 换座位](https://leetcode-cn.com/problems/exchange-seats/description/)

小美是一所中学的信息科技老师，她有一张 seat 座位表，平时用来储存学生名字和与他们相对应的座位 id。

其中纵列的 id 是连续递增的

小美想改变相邻俩学生的座位。

你能不能帮她写一个 SQL query 来输出小美想要的结果呢？

> 解题思路利用rank() over这个
>
> 并且异或将奇数位不变 偶数位全部减少2

```sql
# Write your MySQL query statement below
select rank() over(order by (id-1)^1) as id,student from seat;
```


# SQL Notes

Personal Notes and references on SQL. Taken from various places on the internet. (keep updating)

## Table of Contents
* [Window Functions](#Window-functions)
* [Subquery](#Subquery)
* [CTEs](#Ctes)
* [Self Join](#Self-Join)
* [Union All](#Union-All)
* [Others](#others)


## Window Functions

* `GROUP BY` will compress the result, `PARTITION BY` will not

Here is an example [LeetCode_1303_Easy](https://leetcode.com/problems/find-the-team-size/).


```text
Table: Employee
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| employee_id   | int     |
| team_id       | int     |
+---------------+---------+
employee_id is the primary key for this table.
Each row of this table contains the ID of each employee and their respective team.

Write an SQL query to find the team size of each of the employees.

Return result table in any order.


Input: 
Employee Table:
+-------------+------------+
| employee_id | team_id    |
+-------------+------------+
|     1       |     8      |
|     2       |     8      |
|     3       |     8      |
|     4       |     7      |
|     5       |     9      |
|     6       |     9      |
+-------------+------------+
Output: 
+-------------+------------+
| employee_id | team_size  |
+-------------+------------+
|     1       |     3      |
|     2       |     3      |
|     3       |     3      |
|     4       |     1      |
|     5       |     2      |
|     6       |     2      |
+-------------+------------+
Explanation: 
Employees with Id 1,2,3 are part of a team with team_id = 8.
Employee with Id 4 is part of a team with team_id = 7.
Employees with Id 5,6 are part of a team with team_id = 9.
```

```SQL
SELECT 
    employee_id,
    COUNT(employee_id) OVER (partition by team_id) AS team_size

FROM Employee
```

* All window functions are to expand a column and then filter on the new table
 
Here is an example [LeetCode_1126_Medium](https://leetcode.com/problems/active-businesses/).

```text
Table: Events

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| business_id   | int     |
| event_type    | varchar |
| occurences    | int     | 
+---------------+---------+
(business_id, event_type) is the primary key of this table.
Each row in the table logs the info that an event of some type occurred at some business for a number of times.
 

The average activity for a particular event_type is the average occurences across all companies that have this event.

An active business is a business that has more than one event_type such that their occurences is strictly greater than the average activity for that event.

Write an SQL query to find all active businesses.

Return the result table in any order.

The query result format is in the following example.

 

Example 1:

Input: 
Events table:
+-------------+------------+------------+
| business_id | event_type | occurences |
+-------------+------------+------------+
| 1           | reviews    | 7          |
| 3           | reviews    | 3          |
| 1           | ads        | 11         |
| 2           | ads        | 7          |
| 3           | ads        | 6          |
| 1           | page views | 3          |
| 2           | page views | 12         |
+-------------+------------+------------+
Output: 
+-------------+
| business_id |
+-------------+
| 1           |
+-------------+
Explanation:  
The average activity for each event can be calculated as follows:
- 'reviews': (7+3)/2 = 5
- 'ads': (11+7+6)/3 = 8
- 'page views': (3+12)/2 = 7.5
The business with id=1 has 7 'reviews' events (more than 5) and 11 'ads' events (more than 8), so it is an active business.
```

```SQL
SELECT 
    business_id
FROM
(SELECT 
    a.*,
    AVG(occurences) OVER(partition by event_type) AS avgo
FROM 
    Events a) tmp
WHERE occurences > avgo
GROUP BY business_id
HAVING COUNT(event_type) > 1
```

* Adding **rows # preceding** can creat the moving window

Here is an example [LeetCode_1321_Medium](https://leetcode.com/problems/restaurant-growth/).

```SQL
SELECT
    *
FROM
    (SELECT
        visited_on,
        SUM(amount) OVER(ORDER BY visited_on rows 6 preceding) AS amount,
        ROUND(AVG(amount) OVER(ORDER BY visited_on rows 6 preceding),2) AS average_amount
    FROM
        (SELECT
            visited_on,
            SUM(amount) AS amount
        FROM
            Customer
        GROUP BY 
            visited_on) t1
     ) t2
WHERE
    datediff(visited_on, (SELECT MIN(visited_on) FROM Customer)) >= 6
    
```

## Subquery
* In sorting problems, use subqueries instead if not allowed to use window functions
```SQL
SELECT first_name as employee_name, department , salary 
FROM employee  
WHERE salary in (select max(salary) from employee group by department);
```
```SQL
WITH ranked_salary AS(
SELECT 
    department, first_name, salary,
    dense_rank() over(PARTITION BY department ORDER BY salary DESC) AS rnk
FROM employee
)
SELECT department, first_name AS employee_name, salary
FROM ranked_salary
WHERE rnk = 1
```


## CTEs
* A recursive CTE is a CTE that has a subquery which refers to the CTE name itself. The following illustrates the syntax of a recursive CTE

```SQL
WITH RECURSIVE cte_name AS (
    initial_query  -- anchor member
    UNION ALL
    recursive_query -- recursive member that references to the CTE name
)
SELECT * FROM cte_name;
```
Here is an example [StrataScratch:Reviews of Categories](https://platform.stratascratch.com/coding/10049-reviews-of-categories?tabname=discussion&code_type=3/).



## Self Join

* Sometimes using `WHERE IN` **subquery** can get the same results

Here is an example [LeetCode_641_Medium](https://leetcode.com/problems/second-degree-follower/).

```text
Table: Follow

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| followee    | varchar |
| follower    | varchar |
+-------------+---------+
(followee, follower) is the primary key column for this table.
Each row of this table indicates that the user follower follows the user followee on a social network.
There will not be a user following themself.

A second-degree follower is a user who:

follows at least one user, and
is followed by at least one user.
Write an SQL query to report the second-degree users and the number of their followers.

Return the result table ordered by follower in alphabetical order.

Input: 
Follow table:
+----------+----------+
| followee | follower |
+----------+----------+
| Alice    | Bob      |
| Bob      | Cena     |
| Bob      | Donald   |
| Donald   | Edward   |
+----------+----------+
Output: 
+----------+-----+
| follower | num |
+----------+-----+
| Bob      | 2   |
| Donald   | 1   |
+----------+-----+
Explanation: 
User Bob has 2 followers. Bob is a second-degree follower because he follows Alice, so we include him in the result table.
User Donald has 1 follower. Donald is a second-degree follower because he follows Bob, so we include him in the result table.
User Alice has 1 follower. Alice is not a second-degree follower because she does not follow anyone, so we don not include her in the result table.
```
Solution 1 using `WHERE IN` **subquery**:

```SQL
SELECT 
    followee AS follower,
    COUNT(DISTINCT follower) AS num
FROM 
    Follow
WHERE 
    followee in
    (SELECT
        DISTINCT follower
    FROM
        Follow)
GROUP BY 1
```

Solution 2 using `Self Join`:

```SQL
SELECT
    a.followee as follower,
    COUNT(DISTINCT a.follower) AS num
FROM
    Follow a, Follow b
WHERE
    a.followee = b.follower
GROUP BY 1
```

* `SELF JOIN` can find the relationship between layers of nesting。

Here is an example [LeetCode_1270_Medium](https://leetcode.com/problems/all-people-report-to-the-given-manager/).

```text
Table: Employees

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| employee_id   | int     |
| employee_name | varchar |
| manager_id    | int     |
+---------------+---------+
employee_id is the primary key for this table.
Each row of this table indicates that the employee with ID employee_id and name employee_name reports his work to his/her direct manager with manager_id
The head of the company is the employee with employee_id = 1.
 

Write an SQL query to find employee_id of all employees that directly or indirectly report their work to the head of the company.

The indirect relation between managers will not exceed three managers as the company is small.

Return the result table in any order.

The query result format is in the following example.

 

Example 1:

Input: 
Employees table:
+-------------+---------------+------------+
| employee_id | employee_name | manager_id |
+-------------+---------------+------------+
| 1           | Boss          | 1          |
| 3           | Alice         | 3          |
| 2           | Bob           | 1          |
| 4           | Daniel        | 2          |
| 7           | Luis          | 4          |
| 8           | Jhon          | 3          |
| 9           | Angela        | 8          |
| 77          | Robert        | 1          |
+-------------+---------------+------------+
Output: 
+-------------+
| employee_id |
+-------------+
| 2           |
| 77          |
| 4           |
| 7           |
+-------------+
Explanation: 
The head of the company is the employee with employee_id 1.
The employees with employee_id 2 and 77 report their work directly to the head of the company.
The employee with employee_id 4 reports their work indirectly to the head of the company 4 --> 2 --> 1. 
The employee with employee_id 7 reports their work indirectly to the head of the company 7 --> 4 --> 2 --> 1.
The employees with employee_id 3, 8, and 9 do not report their work to the head of the company directly or indirectly. 
```

```SQL
SELECT DISTINCT a.employee_id
FROM Employees a
JOIN Employees b
ON a.manager_id = b.employee_id
JOIN Employees c
ON b.manager_id = c.employee_id
WHERE a.employee_id <> 1
AND c.manager_id = 1
```

## Union All

* Using `UNION ALL` and `LEFT JOIN` can realize full **Outer Join** in MySQL

Here is an example [LeetCode_1205_Medium](https://leetcode.com/problems/monthly-transactions-ii/).

```text
Table: Transactions

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| id             | int     |
| country        | varchar |
| state          | enum    |
| amount         | int     |
| trans_date     | date    |
+----------------+---------+
id is the primary key of this table.
The table has information about incoming transactions.
The state column is an enum of type ["approved", "declined"].
Table: Chargebacks

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| trans_id       | int     |
| trans_date     | date    |
+----------------+---------+
Chargebacks contains basic information regarding incoming chargebacks from some transactions placed in Transactions table.
trans_id is a foreign key to the id column of Transactions table.
Each chargeback corresponds to a transaction made previously even if they were not approved.
 

Write an SQL query to find for each month and country: the number of approved transactions and their total amount, the number of chargebacks, and their total amount.

Note: In your query, given the month and country, ignore rows with all zeros.

Return the result table in any order.

The query result format is in the following example.

 

Example 1:

Input: 
Transactions table:
+-----+---------+----------+--------+------------+
| id  | country | state    | amount | trans_date |
+-----+---------+----------+--------+------------+
| 101 | US      | approved | 1000   | 2019-05-18 |
| 102 | US      | declined | 2000   | 2019-05-19 |
| 103 | US      | approved | 3000   | 2019-06-10 |
| 104 | US      | declined | 4000   | 2019-06-13 |
| 105 | US      | approved | 5000   | 2019-06-15 |
+-----+---------+----------+--------+------------+
Chargebacks table:
+----------+------------+
| trans_id | trans_date |
+----------+------------+
| 102      | 2019-05-29 |
| 101      | 2019-06-30 |
| 105      | 2019-09-18 |
+----------+------------+
Output: 
+---------+---------+----------------+-----------------+------------------+-------------------+
| month   | country | approved_count | approved_amount | chargeback_count | chargeback_amount |
+---------+---------+----------------+-----------------+------------------+-------------------+
| 2019-05 | US      | 1              | 1000            | 1                | 2000              |
| 2019-06 | US      | 2              | 8000            | 1                | 1000              |
| 2019-09 | US      | 0              | 0               | 1                | 5000              |
+---------+---------+----------------+-----------------+------------------+-------------------+
```

```SQL
WITH cte AS
(
    SELECT * FROM Transactions
    UNION ALL
    (SELECT
        t1.trans_id,
        country,
        "chargeback" AS state,
        amount,
        t1.trans_date
    FROM
        Chargebacks t1
    LEFT JOIN
        Transactions t2
    ON 
        t1.trans_id = t2.id)
)
      
      
SELECT 
    DATE_FORMAT(trans_date, "%Y-%m") AS month,
    country,
    SUM(state = 'approved') AS approved_count,
    SUM(IF(state = 'approved', amount, 0)) AS approved_amount,
    SUM(state = 'chargeback') AS chargeback_count,
    SUM(IF(state = 'chargeback', amount, 0)) AS chargeback_amount
FROM
    cte
GROUP BY 1,2
HAVING approved_count <> 0 or chargeback_count <> 0
```

## Others
* Use `DATE_FORMAT`(trans_date,'%Y-%m')to extract month from year-month-day
* When the timestamp column contains too much information, use `DATE_TRUNC`(‘[interval]’, time_column) to round the timestamp to the interval you want
* Calculate the result under some circumstances using `SUM(IF(state = 'approved',amount,0))`
* Get max value based on case when statement using expression like `MAX(CASE WHEN ... THEN ... END`
    
    Here is an example [StrataScratch:_Users By Average Session Time](https://platform.stratascratch.com/coding/10352-users-by-avg-session-time?code_type=3/).
* Use `LAG()` function to calculate month-over-month percentage difference
    Syntax of Lag function:
    
    ```SQL
    LAG (scalar_expression [,offset] [,default])  
    OVER ( [ partition_by_clause ] order_by_clause )
    ```
    Here is an example [StrataScratch:_Monthly Percentage Difference](https://platform.stratascratch.com/coding/10319-monthly-percentage-difference?code_type=3/).

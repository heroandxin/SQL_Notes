# SQL Notes

Personal Notes and references on SQL. Taken from various places on the internet. (keep updating)

## Table of Contents
* [Window Functions](#window-functions)


## Window Functions

* `group by` will compress the result, `partition by` will not

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

## Self Join

* Sometimes using `where in` **subquery** can get the same results

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
Solution 1 using `where in` **subquery**:

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

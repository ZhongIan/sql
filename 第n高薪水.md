
# leetcode

https://leetcode.com/problems/nth-highest-salary/

編寫SQL查詢以從Employee表中獲得第n高薪水。

    +----+--------+
    | Id | Salary |
    +----+--------+
    | 1  | 100    |
    | 2  | 200    |
    | 3  | 300    |
    +----+--------+

例如，給定上述Employee表中，其中第N = 2高的薪水為200。如果沒有那麼查詢應該返回null。

# 答案


```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    SET N = N - 1;
    RETURN(
        # Write your MySQL query statement below.
        SELECT IFNULL(
            (
                SELECT DISTINCT Salary FROM Employee
                ORDER BY Salary DESC
                LIMIT 1 OFFSET N
            ),NULL 
        )AS getNthHighestSalary
    );
END
```








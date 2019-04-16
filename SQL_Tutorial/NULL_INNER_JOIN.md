
https://sqlzoo.net/wiki/Using_Null

# NULL_INNER_JOIN

關鍵字

    NULL, 
    INNER JOIN, LEFT JOIN, RIGHT JOIN
    COALESCE(a,'NULL') 等於 IFNULL(a,'NULL') {Mysql}
        如果沒有值(NULL)，則用'NULL'替代

# 表格

**teacher老師**

    id
    dept部門
    name名稱
    phone電話
    mobile

**dept部門**

    id
    name部門名稱


# 1.列出其部門為NULL的教師

不能使用 = 用 IS

```sql
SELECT name FROM teacher
WHERE dept IS NULL
```

# 2.INNER JOIN 僅保留兩邊都有的

錯過了沒有部門的老師

和沒有老師的部門

```sql
SELECT teacher.name, dept.name
FROM teacher 
    INNER JOIN dept ON (teacher.dept=dept.id)
```

# 3.LEFT JOIN 保留左邊(LEFT)有的

```sql
SELECT teacher.name, dept.name
FROM teacher 
    LEFT JOIN dept ON (teacher.dept=dept.id)
```

# 4.RIGHT JOIN 保留右邊(RIGHT)有的

```sql
SELECT teacher.name, dept.name
FROM teacher 
    RIGHT JOIN dept ON (teacher.dept=dept.id)
```

# 5.顯示教師姓名和手機號碼或'07986 444 2266'

使用COALESCE打印手機號碼。如果沒有給出號碼，請使用號碼'07986 444 2266'

```sql
SELECT name, COALESCE(mobile, '07986 444 2266') AS mobile 
FROM teacher
```

# 6.使用COALESCE功能和LEFT JOIN打印教師姓名和部門名稱。在沒有部門的地方使用字符串'None'

```sql
SELECT teacher.name,  COALESCE(dept.name,  'None')
FROM teacher 
    LEFT JOIN dept ON (teacher.dept=dept.id)
```

# 7.使用COUNT顯示教師人數和手機數量

```sql
SELECT COUNT(name), COUNT(mobile)
FROM teacher
```

# 8.使用COUNT和GROUP BY dept.name顯示每個部門和員工人數。使用RIGHT JOIN確保列出工程部門

```sql
SELECT dept.name, COUNT(teacher.name)
FROM teacher
    RIGHT JOIN dept ON (teacher.dept=dept.id)
GROUP BY dept.name
```

# 9.如果教師在第1或第2 部門，則使用CASE顯示每位教師的姓名，然後顯示“Sci ”，否則顯示“Art”

```sql
SELECT teacher.name, 
    CASE WHEN dept.id IN(1,2) 
        THEN 'Sci' ELSE 'Art' 
    END
FROM teacher
    LEFT JOIN dept ON (teacher.dept=dept.id)

```

# 10.如果教師在1或2部門，則使用CASE顯示每個教師的姓名，然後顯示'Sci'，如果教師部門是3，則顯示'Art'，否則顯示'None'。

```sql
SELECT teacher.name,  
    CASE WHEN dept.id IN(1,2) THEN 'Sci'
        WHEN dept.id = 3 THEN 'Art' ELSE 'None' 
    END
FROM teacher
LEFT JOIN dept ON (teacher.dept=dept.id)

```



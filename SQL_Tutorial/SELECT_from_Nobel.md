
https://sqlzoo.net/wiki/SELECT_from_Nobel_Tutorial/zh

# SELECT_from_Nobel Tutorial

# nobel表

nobel(yr年份, subject獎項, winner得獎者)

# 9.查看1980年獲獎者，但不包括化學獎(Chemistry)和醫學獎(Medicine)

```sql
SELECT *
FROM nobel
WHERE yr = 1980 
AND subject NOT IN('Chemistry','Medicine')
```

# 10.顯示早期的醫學獎(Medicine)得獎者（1910之前，不包括1910），及近年文學獎(Literature)得獎者（2004年以後，包括2004年）

```sql
SELECT yr, subject, winner
FROM nobel
WHERE (yr < 1910 AND subject = 'Medicine') OR
(yr >= 2004 AND subject = 'Literature')
```


# 11.查找PETERGRÜNBERG贏得的獎項的所有詳情

**非ASCII字符**

名字中有一個變音符號 Ü

https://en.wikipedia.org/wiki/%C3%9C#Keyboarding

```sql
SELECT * FROM nobel
WHERE winner = 'PETER GRÜNBERG'
```

# 12.查找尤金•奧尼爾EUGENE O'NEILL得獎的所有細節


**跳脫字符：單引號**

你不能把一個單引號直接的放在字符串中。

但您可連續使用兩個單引號在字符串中當作一個單引號。

```sql
SELECT * FROM nobel
WHERE winner = 'EUGENE O''NEILL'
```

# 13.列出爵士的獲獎者，年份，獎頁（爵士的名字以開始爵士）。先顯示最新獲獎者，然後同年再按名稱順序排列

```sql
SELECT winner, yr, subject FROM nobel
WHERE winner LIKE 'Sir%'
ORDER BY yr DESC, winner ASC
```

# 14.顯示1984年獲獎者和主題按主題和獲勝者名稱排序; 但最後列出化學和物理

subject IN ('Physics','Chemistry') 可以用作值 -> 它將是0或1

```sql
SELECT winner, subject #, subject IN ('Physics','Chemistry') 
FROM nobel
WHERE yr=1984
ORDER BY subject IN ('Physics','Chemistry') ASC, subject, winner
```





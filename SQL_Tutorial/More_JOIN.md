
https://sqlzoo.net/wiki/More_JOIN_operations/zh

# More_JOIN

# 表格

**movie電影**
    
    id編號
    title電影名稱
    yr首影年份
    director導演
    budget製作費
    gross票房收入

**actor演員**

    id編號
    name姓名

**casting角色**

    movieid電影編號
    actorid演員編號
    ord角色次序

*角色次序代表第1主角是1, 第2主角是2...如此類推.*


# 7.列出電影北非諜影 'Casablanca'的演員名單

使用 movieid=11768

```sql
SELECT DISTINCT name
FROM casting JOIN actor ON(actorid=id)
WHERE movieid=11768
```

# 8.顯示電影異型'Alien' 的演員清單

**子查詢**

```sql
SELECT DISTINCT name
FROM casting JOIN actor ON(actorid=id)
WHERE movieid = (SELECT id FROM movie
WHERE title = 'Alien')
```

**多次JOIN**

```sql
SELECT DISTINCT name
FROM casting 
    JOIN actor ON(casting.actorid=actor.id)
    JOIN movie ON(casting.movieid=movie.id)
WHERE title = 'Alien'
```

# 9.列出演員夏里遜福 'Harrison Ford' 曾演出的電影

```sql
SELECT DISTINCT title
FROM casting 
    JOIN actor ON(casting.actorid=actor.id)
    JOIN movie ON(casting.movieid=movie.id)
WHERE name = 'Harrison Ford'
```

# 10.列出演員夏里遜福 'Harrison Ford' 曾演出的電影，但他不是第1主角

```sql
SELECT DISTINCT title
FROM casting 
    JOIN movie ON(movieid=id)
    JOIN actor ON(actorid=actor.id)
WHERE name = 'Harrison Ford'
AND ord != 1
```

# 11.列出1962年首影的電影及它的第1主角

```sql
SELECT movie.title, actor.name
FROM casting 
    JOIN movie ON(movieid=id)
    JOIN actor ON(actorid=actor.id)
WHERE movie.yr = 1962 AND casting.ord = 1
```

# 困難的題目

## 12.尊·特拉華達'John Travolta'最忙是哪一年? 顯示年份和該年的電影數目

```sql
SELECT yr,COUNT(title) 
FROM movie 
    JOIN casting ON movie.id=movieid
    JOIN actor   ON actorid=actor.id
where name='John Travolta'
GROUP BY yr
HAVING COUNT(title)=(
    SELECT MAX(c) 
    FROM (
        SELECT yr,COUNT(title) AS c 
        FROM movie 
            JOIN casting ON movie.id=movieid
            JOIN actor   ON actorid=actor.id
        where name='John Travolta'
        GROUP BY yr
    ) AS t
)
```

## 13.列出演員茱莉·安德絲'Julie Andrews'曾參與的電影名稱及其第1主角

```sql
SELECT title, name 
FROM casting 
    JOIN movie ON(movieid=movie.id)
    JOIN actor ON(actorid=actor.id)
WHERE movieid IN (
    SELECT movieid 
    FROM actor 
        JOIN casting ON(actorid=actor.id)
  WHERE actor.name = 'Julie Andrews'
)
AND ord = 1
```

## 14.列出按字母順序，列出哪一演員曾作30次第1主角

```sql
SELECT name 
FROM casting 
    JOIN actor ON(actorid=actor.id)
WHERE ord = 1 
GROUP BY name
HAVING COUNT(movieid) >= 30
ORDER BY name
```

## 15.列出1978年首影的電影名稱及角色數目，按此數目由多至少排列

```sql
SELECT title, COUNT(actorid) 
FROM casting 
    JOIN movie ON(movieid=movie.id)
WHERE yr = 1978
GROUP BY title
ORDER BY COUNT(actorid) DESC
```

## 16.列出曾與演員亞特·葛芬柯'Art Garfunkel'合作過的演員姓名

```sql
SELECT DISTINCT name 
FROM casting 
    JOIN movie ON(movieid=movie.id)
    JOIN actor ON(actorid=actor.id)
WHERE movieid IN (
    SELECT movieid 
    FROM actor 
        JOIN casting ON(actorid=actor.id)
    WHERE actor.name = 'Art Garfunkel'
)
AND actor.name != 'Art Garfunkel'
```





https://sqlzoo.net/wiki/SELECT_from_WORLD_Tutorial

# SELECT basics WORLD Tutorial

關鍵字

    LIKE % _ 
    ROUND
    LENGTH
    LEFT RIGHT

# 6.顯示name包含“United”字樣的國家

```sql
SELECT name
FROM world
WHERE name LIKE 'United %';
```

# 7.按人口顯示面積大或面積大的國家。顯示名稱，人口和麵面積

如果一個國家面積超過300萬平方公里，或者人口超過2.5億，那麼這個國家就很大

```sql
SELECT name, population, area
FROM world
WHERE area > 3000000 OR
population > 250000000;
```

# 8.XOR

* 澳大利亞面積很大但人口很少，應該包括在內。
* 印度尼西亞人口眾多但面積很小，應該包括在內。
* 中國人口眾多，面積大，應該被排除在外。
* 英國人口少，面積小，應排除在外。

```sql
SELECT name, population, area
FROM world
WHERE (area < 3000000 AND population > 250000000) OR
(area > 3000000 AND population < 250000000);

```

# 9.顯示name與population數以百萬計，並以十億為continent“南美”各國的國內生產總值。使用ROUND函數將值顯示為兩位小數。

```sql
SELECT name, 
       ROUND(population/1000000,2) AS population, 
       ROUND(gdp/1000000000,2) AS gdp
FROM world
WHERE continent = 'South America'

```

# 10.將萬億美元國家的人均GDP顯示以1000為單位。

1234567 萬億 -> 1235000

```sql
SELECT name, ROUND(gdp/population,-3) 
FROM world
WHERE gdp > 1000000000000 

```

# 11.顯示國家和首都字符數相同的國家和首都

```sql
SELECT name, capital
FROM world
WHERE LENGTH(name)=LENGTH(capital)
```

# 12.顯示國家和首都的首字母相同但全名不同的國家和首都

```sql
SELECT name, capital
FROM world
WHERE LEFT(name,1) = LEFT(capital,1)
AND name != capital
```

# 13.找到包含所有元音(aeiou)且名稱中沒有空格的國家

```sql
SELECT name
FROM world
WHERE name LIKE '%a%' 
AND name LIKE '%e%'
AND name LIKE '%i%'
AND name LIKE '%o%'
AND name LIKE '%u%'
AND name NOT LIKE '% %'
```
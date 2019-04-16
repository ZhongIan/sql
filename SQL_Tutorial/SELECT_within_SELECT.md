
https://sqlzoo.net/wiki/SELECT_within_SELECT_Tutorial/zh

# 子查詢

SELECT_within_SELECT Tutorial

# world表格

world表格(name國家名稱, continent洲份, area面積, population人口, gdp國內生產總值)

# 1.列出每個國家的名字 name，當中人口 population 是高於俄羅斯'Russia'的人口

```sql
SELECT name FROM world
WHERE population > (
    SELECT population FROM world
    WHERE name='Russia'
);
```

# 2.列出歐州每國家的人均GDP，當中人均GDP要高於英國'United Kingdom'的數值。

```sql
SELECT name FROM world
WHERE continent='Europe' 
AND gdp/population > (
    SELECT gdp/population FROM world
    WHERE name='United Kingdom'
);
```

# 3.在阿根廷Argentina 及 澳大利亞 Australia所在的洲份中，列出當中的國家名字 name 及洲分 continent 。按國字名字順序排序

```sql
SELECT name, continent FROM world
WHERE continent IN(
    SELECT continent FROM world 
    WHERE name IN('Argentina','Australia')
)
ORDER BY name;
```

# 4.哪一個國家的人口比加拿大Canada的多，但比波蘭Poland的少?列出國家名字name和人口population

```sql
SELECT name, population FROM world
WHERE population > (
    SELECT population FROM world WHERE name = 'Canada'
) AND population < (
    SELECT population FROM world WHERE name = 'Poland'
) ;
```


# 5.顯示歐洲的國家名稱name和每個國家的人口population。以德國的人口的百分比作人口顯示

使用函數 CONCAT 增加的百分比符號

```sql
SELECT name, CONCAT(ROUND(world.population/tmp.population*100,0), '%') AS pop_pencent
FROM world, (
    SELECT population FROM world WHERE name = 'Germany'
) AS tmp
WHERE world.continent = 'Europe';
```

# 6.哪些國家的GDP比Europe歐洲的全部國家都要高呢? [只需列出 name ] 

(有些國家的記錄中，GDP是NULL，沒有填入資料的。)

```sql
SELECT name FROM world
WHERE gdp > ALL(
  SELECT gdp FROM world
  WHERE continent = 'Europe' 
  AND gdp > 0
);
```

# 7.在每一個州中找出最大面積的國家，列出洲份 continent, 國家名字 name 及面積 area

 (有些國家的記錄中，AREA是NULL，沒有填入資料的)

 ```sql
SELECT continent, name, area 
FROM world AS x
WHERE area >= ALL(
    SELECT area FROM world y
    WHERE y.continent=x.continent
    AND area > 0
);

# 'OR' GROUP BY

SELECT continent, name, area
FROM world WHERE area IN (
    SELECT MAX(area) FROM world
    GROUP BY continent
);
 ```

 # 8.列出洲份名稱，和每個洲份中國家名字按子母順序是排首位的國家名
 
 (即每洲只有列一國)

```sql
SELECT continent, name
FROM world x
WHERE name <= ALL(
    SELECT name FROM world y
    WHERE y.continent=x.continent
);

# 'OR' GROUP BY

SELECT continent, name
FROM world WHERE name IN (
    SELECT MIN(name) FROM world
    GROUP BY continent
);
```

# 困難的題目(早前練習沒有包含的技巧)

## 9.找出洲份，當中全部國家都有少於或等於 25000000 人口. 在這些洲份中，列出國家名字name，continent 洲份和population人口。

```sql
SELECT name, continent, population
FROM world x
WHERE 25000000 >= ALL(
  SELECT population FROM world y
  WHERE y.continent=x.continent
  AND population > 0
)
```

## 10.有些國家的人口是同洲份的所有其他國的3倍或以上。列出 國家名字name 和 洲份 continent

```sql
SELECT name, continent
FROM world x
WHERE x.population >= ALL(
  SELECT y.population*3 FROM world y
  WHERE y.continent=x.continent
  AND x.population != y.population
)
```




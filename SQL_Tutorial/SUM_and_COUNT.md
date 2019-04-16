
https://sqlzoo.net/wiki/SUM_and_COUNT/zh

# SUM_and_COUNT

關鍵字

    SUM, Count, MAX, DISTINCT, ORDER BY
    GROUP BY ... HAVING ...

# world表格

world表格(name國家名稱, continent洲份, area面積, population人口, gdp國內生產總值)

# 1.展示世界的總人口

```sql
SELECT SUM(population)
FROM world
```

# 2.列出所有的洲份, 每個只有一次

```sql
SELECT distinct continent
FROM world
```

# 3.找出非洲(Africa)的GDP總和

```sql
SELECT SUM(gdp)
FROM world
WHERE continent = 'Africa'
```

# 4.有多少個國家具有至少百萬(1000000)的面積

```sql
SELECT COUNT(name)
FROM world
WHERE area > 1000000
```

# 5.('France','Germany','Spain')（“法國”，“德國”，“西班牙”）的總人口是多少

```sql
SELECT SUM(population)
FROM world
WHERE name IN('France','Germany','Spain')
```

# 使用 GROUP BY 和 HAVING.

## 6.對於每一個洲份，顯示洲份和國家的數量

```sql
SELECT continent, COUNT(name)
FROM world
GROUP BY continent
```

## 7.對於每一個洲份，顯示洲份和至少有1000萬人(10,000,000)口國家的數目

**執行順序**

1. from
2. where
3. group by
4. having
5. select
6. order by

```sql
SELECT continent, COUNT(name)
FROM world
WHERE  population >= 10000000
GROUP BY continent;
/*HAVING world.population >= 10000000*/

```
HAVING 以聚合運算為主 -> ex. COUNT(name)

## 8.列出有至少100百萬(1億)(100,000,000)人口的洲份

```sql
SELECT DISTINCT continent
FROM world
WHERE continent IN(
    SELECT continent 
    FROM world 
    GROUP BY continent
    HAVING SUM(population) >= 100000000
) 
```


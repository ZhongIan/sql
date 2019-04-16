
https://sqlzoo.net/wiki/SELECT_basics/zh

# SELECT basics

關鍵字

    SELECT, WHERE, IN, BETWEEN


# world表格

world表格(name國家名稱, continent洲份, area面積, population人口, gdp國內生產總值)

# 1.顯示德國 Germany 的人口

```sql
SELECT population FROM world
WHERE name = 'Germany';
```

# 2.查詢面積為 5,000,000 以上平方公里的國家,對每個國家顯示她的名字和人均國內生產總值(gdp/population)

```sql
SELECT name, gdp/population FROM world
WHERE area > 5000000;
```

# 3.顯示“Ireland 愛爾蘭”,“Iceland 冰島”,“Denmark 丹麥”的國家名稱和人口

```sql
SELECT name, population FROM world
WHERE name IN ('Ireland', 'Iceland', 'Denmark');
```

# 4.顯示面積為 200,000 及 250,000 之間的國家名稱和該國面積

```sql
SELECT name, area FROM world
WHERE area BETWEEN 200000 AND 250000
```


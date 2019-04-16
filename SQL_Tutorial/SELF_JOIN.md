
https://sqlzoo.net/wiki/Self_join

# SELF_JOIN

# 表格

    stops(id, name)
    route(num, company, pos, stop)

# 3.顯示ID和名稱為站上的'4''輕軌'服務

```sql
SELECT id,  name 
FROM stops
    JOIN route ON(id=stop)
WHERE company = 'LRT'
    AND num = 4
```

# 4.顯示的查詢給出了訪問倫敦路（149）或克雷格洛克哈特（53）的路線數量。運行查詢並註意鏈接這些停靠點的兩個服務的計數為2.

添加HAVING子句以限制輸出到兩個計數

```sql
SELECT company, num, COUNT(*)
FROM route 
WHERE stop=149 OR stop=53
GROUP BY company, num
HAVING COUNT(*) = 2
```

# 5.顯示從Craiglockhart（53）到London Road（149）的服務

```sql
SELECT a.company, a.num, a.stop, b.stop
FROM route a 
    JOIN route b ON
        (a.company=b.company AND a.num=b.num)
WHERE a.stop=53 AND b.stop=149
```

# 6.與前一個查詢類似，但通過名稱顯示“Craiglockhart”和“London Road”之間的服務

可以通過名稱而不是數字來引用stops

```sql
SELECT a.company, a.num, stopa.name, stopb.name
FROM route a 
    JOIN route b ON
        (a.company=b.company AND a.num=b.num)
    JOIN stops stopa ON (a.stop=stopa.id)
    JOIN stops stopb ON (b.stop=stopb.id)
WHERE stopa.name='Craiglockhart' AND stopb.name='London Road'
```

# 7.列出連接115和137（'Haymarket'和'Leith'）的所有服務

```sql
SELECT DISTINCT a.company, a.num
FROM route a 
    JOIN route b ON
        (a.company=b.company AND a.num=b.num)
WHERE a.stop=115 AND b.stop=137
```

# 8.列出連接'Craiglockhart'和'Tollcross' 站點的服務

```sql
SELECT a.company, a.num
FROM route a 
    JOIN route b ON
        (a.company=b.company AND a.num=b.num)
    JOIN stops stopa ON (a.stop=stopa.id)
    JOIN stops stopb ON (b.stop=stopb.id)
WHERE stopa.name='Craiglockhart' AND stopb.name='Tollcross'
```

**結果**

公司 | NUM
-|-
LRT | 10
LRT | 27
LRT | 45
LRT | 47



# 9.通過乘坐一輛公共汽車（LRT公司和“Craiglockhart”），顯示從“Craiglockhart”獲得一個明確的停靠列表(stops.name)、公司和公交車號碼

```sql
SELECT stopb.name, a.company, a.num
FROM route a 
    JOIN route b ON
        (a.company=b.company AND a.num=b.num)
    JOIN stops stopa ON (a.stop=stopa.id)
    JOIN stops stopb ON (b.stop=stopb.id)
WHERE stopa.name='Craiglockhart' AND a.company = 'LRT'
```

**結果**

名稱 | 公司 | NUM
-|-|-
Silverknowes | LRT	 | 0
Muirhouse | LRT | 10
新天堂 | LRT | 10
...| | 
Silverknowes | LRT | 27
克魯收費站 | LRT | 27
...| |

# 10.未能正確的理解

查找包含兩條巴士的路線，從Craiglockhart到Lochend。
顯示巴士號。和第一輛公共汽車的公司，轉移的停止的名稱，
和公共汽車號碼。和公司的第二輛公共汽車

```sql
SELECT a.num, a.company, stopb.name, b.num, b.company
FROM route a 
    JOIN route b ON
        (a.company=b.company AND a.num=b.num)
    JOIN stops stopa ON (a.stop=stopa.id)
    JOIN stops stopb ON (b.stop=stopb.id)
WHERE stopa.name='Craiglockhart' 
```


SELECT a.num, a.company, stopb.name, b.num, b.company
FROM route a JOIN route b ON
  (a.company=b.company)
  JOIN stops stopa ON (a.stop=stopa.id)
  JOIN stops stopb ON (b.stop=stopb.id)
WHERE stopa.name='Craiglockhart' AND stopb.name IN(
SELECT stopa.name
FROM route a JOIN route b ON
  (a.company=b.company)
  JOIN stops stopa ON (a.stop=stopa.id)
  JOIN stops stopb ON (b.stop=stopb.id)
WHERE stopb.name='Craiglockhart'
)
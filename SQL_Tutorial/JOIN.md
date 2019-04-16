
https://sqlzoo.net/wiki/The_JOIN_operation/zh

# JOIN

關鍵字

    JOIN ON(a=b),
    CASE WHEN condition THEN 1 ELSE 0 END

# 表格

**game(賽事)**
    
    id(編號), 
    mdate(日期), 
    stadium(場館), 
    team1(隊伍1), 
    team2(隊伍2)


**goal(入球)**
    
    matchid(賽事編號),
    teamid(隊伍編號), 
    player(入球球員), 
    gtime(入球時間


**eteam(歐洲隊伍)**

    id(編號), 
    teamname(隊名), 
    coach(教練)

# 1.列出 賽事編號matchid 和球員名 player ,該球員代表德國隊Germany入球的

要找出德國隊球員，要檢查: teamid = 'GER'

```sql
SELECT matchid, player FROM goal 
WHERE teamid = 'GER'
```

# 2.只顯示賽事1012的 id, stadium, team1, team2

由以上查詢，你可見Lars Bender's 於賽事 1012入球。

現在我們想知道此賽事的對賽隊伍是哪一隊。

留意在 goal 表格中的欄位 matchid ，是對應表格game的欄位id。

我們可以在表格 game中找出賽事1012的資料。

```sql
SELECT id,stadium,team1,team2
FROM game
WHERE id = 1012
```

# 3.顯示每一個德國入球的球員名，隊伍名，場館和日期

我們可以利用JOIN來同時進行以上兩個步驟(1. AND 2.)

```sql
SELECT goal.player, goal.teamid, game.stadium, game.mdate
FROM game JOIN goal ON (id=matchid)
WHERE teamid = 'GER'
```

# 4.列出球員名字叫Mario有入球的隊伍1 team1, 隊伍2 team2 和 球員名 player

列出球員名字叫Mario (player LIKE 'Mario%')

```sql
SELECT game.team1, game.team2, goal.player
FROM game JOIN goal ON (id=matchid)
WHERE player LIKE 'Mario%'
```

# 5.列出每場球賽中首10分鐘有入球的球員 player, 隊伍teamid, 教練coach, 入球時間gtime

列出每場球賽中首10分鐘 (gtime <= 10)

```sql
SELECT goal.player, goal.teamid, eteam.coach, goal.gtime
FROM goal JOIN eteam on teamid=id
WHERE goal.gtime <= 10
```

# 6.列出'Fernando Santos'作為隊伍1 team1 的教練的賽事日期，和隊伍名

要合拼JOIN 表格game 和表格 eteam，你可以使用
    
    game JOIN eteam ON (team1=eteam.id)
或
    
    game JOIN eteam ON (team2=eteam.id)

注意欄位id同時是表格game 和表格 eteam的欄位，

你要清楚指出eteam.id而不是只用id

```sql
SELECT game.mdate, eteam.teamname
FROM game JOIN eteam ON (team1=eteam.id)
WHERE eteam.coach = 'Fernando Santos'
```

# 7.列出場館 'National Stadium, Warsaw'的入球球員

```sql
SELECT goal.player
FROM goal JOIN game ON (game.id=goal.matchid)
WHERE game.stadium = 'National Stadium, Warsaw'
```

# 更困難的題目

## 8.列出全部賽事，射入德國龍門的球員名字

HINT

    德國可以在賽事中作team1 隊伍１（主）或team2隊伍２（客）
    你可以用DISTINCT來防止球員出現兩次以上

```sql
SELECT DISTINCT goal.player
FROM game JOIN goal ON (game.id=goal.matchid)
WHERE (
    game.team1!='GER' AND game.team2='GER'
    /* 德國在賽事中作team1 隊伍１（主） */
    AND goal.teamid!='GER' 
) OR (
    game.team1='GER' AND game.team2!='GER'
    /* 德國在賽事中作team2 */
    AND goal.teamid!='GER'
)

```

## 9.列出隊伍名稱 teamname 和該隊入球總數

```sql
SELECT eteam.teamname, COUNT(goal.player)
FROM eteam JOIN goal ON (eteam.id=goal.teamid)
GROUP BY eteam.teamname

```

## 10.列出場館名和在該場館的入球數字

```sql
SELECT game.stadium, COUNT(goal.player)
FROM game JOIN goal ON (game.id=goal.matchid)
GROUP BY game.stadium
```

## 11.每一場波蘭'POL'有參與的賽事中，列出賽事編號 matchid, 日期date 和入球數字

```sql
SELECT goal.matchid, ame.mdate, COUNT(goal.player)
FROM game JOIN goal ON (game.id=goal.matchid)
WHERE team1 = 'POL' OR team2 = 'POL'
GROUP BY goal.matchid, game.mdate
```

## 12.每一場德國'GER'有參與的賽事中，列出賽事編號 matchid, 日期date 和{德國}的入球數字。

```sql
ELECT goal.matchid, ame.mdate, COUNT(goal.player)
FROM game JOIN goal ON (game.id=goal.matchid)
WHERE (team1 = 'GER' OR team2 = 'GER') 
    AND teamid = 'GER'
    /* {德國}的入球 */
GROUP BY goal.matchid, game.mdate
```

## 13.列出每場比賽，每場比賽得分如表所示

mdate | TEAM1 | score1 | TEAM2 | score2
-|-|-|-|-
2012年7月1日 | ESP | 4 | ITA | 0
2012年6月10日 | ESP | 1 | ITA | 1
2012年6月10日 | IRL | 1 | CRO | 3
...|

在列表中註意每個目標的列表。

如果它是一個team1目標，那麼在score1中出現1，否則有一個0.

    CASE WHEN teamid=team1 THEN 1 ELSE 0 END

你可以使用此列來計算team1得分的數量。按mdate，matchid，team1和team2對結果進行排序。

```sql
SELECT mdate,
  team1,
  SUM(CASE WHEN teamid=team1 THEN 1 ELSE 0 END) AS score1,
  team2,
  SUM(CASE WHEN teamid=team2 THEN 1 ELSE 0 END) AS score2
FROM game JOIN goal ON matchid = id
GROUP BY mdate,matchid, team1, team2
```

# [MS SQL] SQL 挑戰 - 排列組合和階層目錄 (使用 CTE 遞迴)

[[MS SQL] SQL 挑戰 - 排列組合和階層目錄 (使用 CTE 遞迴)](https://ithelp.ithome.com.tw/articles/10202028)

# 組合

原始資料 `甲乙丙丁戊`，請展開原始資料的所有組合，並需符合下列兩個條件。

1. 結果不考慮順序，例如: `甲乙` 和 `乙甲` 算是同一個組合。
2. 結果需由小的順位開始展開，例如: `甲乙` 和 `乙甲` 需先展開 `甲乙`

## 解答1 - 排列組合

```sql

-- 宣告
DECLARE @Data TABLE
(
    Id INT,
    Txt NVARCHAR(10)
)

-- 資料
INSERT INTO @Data VALUES 
    (1,N'甲'),
    (2,N'乙'),
    (3,N'丙'),
    (4,N'丁'),
    (5,N'戊')   
;

WITH DLOOP (Id, Txt, R) AS 
(
    -- 初始化資料 甲乙丙丁戊
    SELECT Id, Txt, Txt AS R FROM @Data
    UNION ALL
    SELECT B.Id,  -- 每次回傳最後一位的Id
           B.Txt, -- 每次回傳最後一位的Txt (其實不必這個欄位)
           -- R為結果
           -- A.R (上一次遞迴的結果) + B.Txt (這次 Join 的項目)
           CAST (A.R+B.Txt AS NVARCHAR(10)) AS R 
    FROM DLOOP AS A
    -- Join 編號小於自己的項目
    INNER JOIN @Data AS B ON B.Id>A.Id
)

SELECT R FROM DLOOP
ORDER BY R
```

## 第一題分析

甲的遞迴

    第一次遞迴   第二次遞迴   第三次遞迴   第四次遞迴
    -----------|-----------|-----------|-----------
    　　甲乙　　　　甲乙丙　　　甲乙丙丁　　 甲乙丙丁戊
    　　甲丙　　　　甲乙丁　　　甲乙丙戊
    　　甲丁　　　　甲乙戊　　　甲乙丁戊
    　　甲戊　　　　甲丙丁　　　甲丙丁戊
    　　　　　　　　甲丙戊
    　　　　　　　　甲丁戊

乙的遞迴

    第一次遞迴   第二次遞迴   第三次遞迴
    -----------|-----------|-----------
    　　乙丙　　　　乙丙丁　　　乙丙丁戊
    　　乙丁　　　　乙丙戊
    　　乙戊　　　　乙丁戊


# 題目2 - 階層目錄

    Id: 目錄編號
    PatentId: 父層目錄編號
    Txt: 目錄文字


最上層目錄的 PatentId 為 0

    Id    PatentId     Txt
    ----|----------|----------
    1  |    0     |   1
    2  |    0     |   2
    3  |    1     |   1-1
    4  |    1     |   1-2
    5  |    2     |   2-1
    6  |    3     |   1-1-1

現在需要用 `-->` 串出該目錄到 `根目錄` 的所有階層，結果如下:

    Id    PatentId     Txt                R
    ----|----------|----------|-----------------------
    1  |    0     |   1         1        
    2  |    0     |   2         2
    3  |    1     |   1-1       1 --> 1-1  
    4  |    1     |   1-2       1 --> 1-2  
    5  |    2     |   2-1       2 --> 2-1  
    6  |    3     |   1-1-1     1 --> 1-1 --> 1-1-1    


## 解答2 - 階層目錄

```sql
-- 宣告
DECLARE @Data TABLE
(
    Id INT,
    PatentId INT,
    Txt NVARCHAR(1000)
)
-- 資料
INSERT INTO @Data VALUES
    (1, 0, '1'),
    (2, 0, '2'),
    (3, 1, '1-1'),
    (4, 1, '1-2'),
    (5, 2, '2-1'),
    (6, 3, '1-1-1');

WITH DLOOP AS
(
    -- 原始目錄
    SELECT Id, PatentId, Txt,
           Txt AS R, 
           PatentId AS PrevId,
           1 AS Dp   -- 設定第一層 Dp 為 1
    FROM @Data
    UNION ALL
    SELECT A.Id, A.PatentId, A.Txt,   -- 原始目錄的欄位
           -- R為結果
           -- B.Txt (為這次找到的上層目錄) + A.R (上一次遞迴的結果)
           CAST(B.Txt + ' --> ' + A.R AS NVARCHAR(1000)) AS R,
           -- PrevId 提供下一次遞迴要去 Join 的 PatentId
    	   B.PatentId AS PrevId,
           -- 目錄層數，最後用來找出最長的那個結果
    	   A.Dp + 1 AS Dp
    FROM DLOOP AS A
    -- 利用 PatentId 找出上層目錄 
    INNER JOIN @Data AS B ON B.Id=A.PrevId
), DResult AS 
(
    SELECT Id, PatentId, Txt, R
    FROM 
    (
        -- 利用原始目錄 Id 進行分組，並以 Dp 排序
    	SELECT *, ROW_NUMBER() OVER(PARTITION BY Id ORDER BY Dp DESC) AS G
    	FROM DLOOP
    ) AS A
    WHERE G=1   -- 找出最多層的那筆資料
)

SELECT * FROM DResult 

```

## 第二題分析

以 1-1-1 這個目錄來看

    第一次遞迴     第二次遞迴          第三次遞迴
    -----------|---------------|---------------------
    1-1-1        1-1 --> 1-1-1   1 --> 1-1 --> 1-1-1

最後結果會產生三筆資料，但其實我們只要最後一筆

    1-1-1
    1-1 --> 1-1-1
    1 --> 1-1 --> 1-1-1  <-只需要這筆資料

因此在遞迴中我新增了一個 `Dp` 欄位，紀錄了目錄的層數，最後就可以利用 `Dp` 找出最長的那筆資料。





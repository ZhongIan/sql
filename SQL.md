# SELECT[查詢]
```SQL
SELECT DISTINCT[去除重復] column, another_column, ... FUNC(ex:SUM(column)) AS new_name, ...
FROM mytable AS new_name

    INNER/LEFT/RIGHT/FULL JOIN another_table 
        ON mytable.id = another_table.id

    WHERE condition (column IS/IS NOT NULL)(column LIKE "A%")
        AND/OR another_condition
        AND/OR ...

    GROUP BY column
    HAVING group_condition

    ORDER BY column ASC/DESC
    LIMIT num_limit OFFSET num_offset
;

```
 表達式 | 描述
 --|--
 = |Case sensitive exact string comparison (notice the single equals) **col_name = "abc"**
 != or <> | Case sensitive exact string inequality comparison	**col_name != "abcd"**
 LIKE，NOT LIKE | Case insensitive exact string comparison	**col_name LIKE "ABC"**
 % | Used anywhere in a string to match a sequence of zero or more characters (only with LIKE or NOT LIKE)	**col_name LIKE "%AT%"**(matches "AT", "ATTIC", "CAT" or even "BATS")
 _ | Used anywhere in a string to match a single character (only with LIKE or NOT LIKE)	**col_name LIKE "AN_"**(matches "AND", but not "AN")
 IN(...)，NOT IN(...) | String exists in a list	**col_name IN ("A", "B", "C")**

ORDER BY column ASC/DESC 升序/降序

LIMIT num_limit OFFSET num_offset

> 通用聚合函數[GROUP BY]
> 
> 以下是我們將在示例中使用的一些常見集合函數：

功能 | 描述
--|--
COUNT（ * ），COUNT（列）| 如果未指定列名，則用於計算組中行數的常用函數。否則，請計算指定列中非空值組中行的數量。
MIN（列） | 查找組中所有行的指定列中的最小數值。
MAX（列） |　查找組中所有行的指定列中的最大數值。
AVG（專欄） | 查找組中所有行的指定列中的平均數值。
SUM（列） | 為組中的行查找指定列中所有數值的總和。

# INSERT[插入]
```sql
INSERT INTO mytable
(column, another_column, ...)
VALUES (value_or_expr, another_value_or_expr, ...),
        (value_or_expr_2, another_value_or_expr_2, ...),
        ...
;
```

# UPDATE[更新]
```sql
SELECT * FROM mytable
WHERE condition
;


UPDATE mytable
SET column = value_or_expr, 
    other_column = another_value_or_expr, 
    ...
WHERE condition
;
```

# DELETE[刪除]
```sql
SELECT * FROM mytable
WHERE condition;

DELETE FROM mytable
WHERE condition;

```

# CREATE[創建]
```sql
CREATE TABLE IF NOT EXISTS mytable (
    column DataType TableConstraint DEFAULT default_value,
    another_column DataType TableConstraint DEFAULT default_value,
    ...
);

```

 數據類型 | 描述
 -- |-- 
 INTEGER， BOOLEAN	| 整數數據類型可以存儲整數值，如數字或年齡的計數。在一些實現中，布爾值僅被表示為僅為0或1的整數值。
 FLOAT，DOUBLE，REAL |浮點數據類型可以存儲更精確的數值數據，如度量值或小數值。根據該值所需的浮點精度，可以使用不同的類型。
 CHARACTER(num_chars)，VARCHAR(num_chars)，TEXT	| 基於文本的數據類型可以將字符串和文本存儲在各種語言環境中。處理這些列時，各種類型之間的區別通常等於數據庫的效率。CHARACTER和VARCHAR（可變字符）類型都是用它們可以存儲的最大字符數指定的（較長的值可能會被截斷），所以用大表來存儲和查詢可能會更高效。
 DATE， DATETIME | SQL還可以存儲日期和時間戳以跟踪時間序列和事件數據。特別是在跨時區操縱數據時，處理它們可能會非常棘手。
 BLOB | 最後，SQL可以將二進制數據存儲在數據庫中的blob中。這些值對數據庫通常是不透明的，因此您通常必須使用正確的元數據來存儲它們以重新查詢它們。

## EX

```sql
CREATE TABLE movies (
    id INTEGER PRIMARY KEY,
    title TEXT,
    director TEXT,
    year INTEGER, 
    length_minutes INTEGER
);
```
### TableConstraint

 約束 | 描述
 --|--
 PRIMARY KEY | 這意味著此列中的值是唯一的，並且每個值可用於標識此表中的單個行。
 AUTOINCREMENT | 對於整數值，這意味著該值將自動填入並隨每行插入而遞增。不支持所有數據庫。
 UNIQUE | 這意味著此列中的值必須是唯一的，因此您不能在該列中插入具有相同值的另一行作為表中的另一行。與“PRIMARY KEY”的不同之處在於它不必是表格中行的鍵。
 NOT NULL | 這意味著插入的值不能是NULL。
 CHECK (expression)	| 這允許您運行更複雜的表達式來測試插入的值是否為值。例如，您可以檢查值是正數還是大於特定大小，或者以某個前綴等開頭。
 FOREIGN KEY | 這是一致性檢查，可確保此列中的每個值都與另一個表中列中的另一個值相對應。例如，如果有兩個表（一個按ID列出所有員工，另一個列出他們的工資信息），那麼`FOREIGN KEY`可以確保工資表中的每一行對應於主員工列表中的有效員工。


# ALTER[更改表格]

```sql
ALTER TABLE mytable
ADD column DataType OptionalTableConstraint 
    DEFAULT default_value;
```



# leetcode
https://leetcode.com/problems/swap-salary/

給定一個表salary，例如下面的表，其中m =男性，f =女性值。使用單個更新語句交換所有f和m值（即，將所有f值更改為m，反之亦然），並且沒有中間臨時表。

請注意，您必須編寫單個更新語句，不要為此問題編寫任何select語句。

    | id | name | sex | salary |
    |----|------|-----|--------|
    | 1  | A    | m   | 2500   |
    | 2  | B    | f   | 1500   |
    | 3  | C    | m   | 5500   |
    | 4  | D    | f   | 500    |

運行update語句後，上面的salary表應該如下

    | id | name | sex | salary |
    |----|------|-----|--------|
    | 1  | A    | f   | 2500   |
    | 2  | B    | m   | 1500   |
    | 3  | C    | f   | 5500   |
    | 4  | D    | m   | 500    |

# 解答

```sql
# mysql

UPDATE salary
SET sex = CASE sex
    WHEN 'm' THEN 'f'
    ELSE 'm'
    END
;
```
Using *UPDATE* and *CASE*...*WHEN*

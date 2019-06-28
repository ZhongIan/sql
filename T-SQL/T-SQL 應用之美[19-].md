
[T-SQL 應用之美](https://ithelp.ithome.com.tw/articles/10007971)

# 19-21.使用萬用字元：* % _ [] [^]

*: 所有，全部
    
    SELECT * FROM HumanResources.Employee 

    SELECT Emp.EmployeeID, Dep.*
    FROM  HumanResources.Employee Emp INNER JOIN
	HumanResources.EmployeeDepartmentHistory Dep ON 
	Emp.EmployeeID = Dep.EmployeeID

%: 至少 0 個任意字元

    -- 職稱是 xxx Engineer 的資料記錄
    SELECT Title 職稱 FROM HumanResources.Employee
    WHERE Title LIKE '%Engineer'

_: 單一個任意字元

    -- 職稱倒數末 4 個字元是 WC 開頭的資料
    SELECT Title 職稱 FROM HumanResources.Employee
    WHERE Title LIKE '%WC__'

[]: 相符的字元，包含一般的字元與萬用字

    -- 職稱是以字元 a～f 開頭的資料
    SELECT DISTINCT Title 職稱 FROM HumanResources.Employee
    WHERE Title LIKE '[a-f]%'

* 範圍（例如：[a-f]）
* 集合（例如：[abcdef]）

\[^]: 不相符的字元

    -- 職稱"不是"以字元 a～f 開頭的資料
    SELECT DISTINCT Title 職稱 FROM HumanResources.Employee
    WHERE Title LIKE '[^a-f]%'

# 24.[ESCAPE]查詢的資料包含萬用字元應如何處置？

```sql
USE tempdb
GO

CREATE TABLE TabTest(
	[地址] nvarchar(50) NOT NULL,
	[折扣] varchar(5) NOT NULL,
	[附註] nvarchar(50) NOT NULL)
GO

SET NOCOUNT ON

INSERT INTO TabTest VALUES('台北市忠孝東路一段10-1號', '30%', '* 此地址為辦公處所')
INSERT INTO TabTest VALUES('台中市東區逢甲大道110號3樓3-1室', '10%' ,'* 按一下 [匯入網際網路郵件及地址]')
INSERT INTO TabTest VALUES('台北市皇后大道123-1號', '12%', '再按一下 [下一步]')
INSERT INTO TabTest VALUES('高雄市美術館東路25號', '15%', '按一下 [匯入及匯出')
INSERT INTO TabTest VALUES('台南市南區港乾路712號2-1樓', '20%', '按一下 [下一步]')
INSERT INTO TabTest VALUES('花蓮市海連路12號', '30%', '按一下 [完成]')
```

假設要找出 TabTest 資料表中，「折扣」欄位的資料值為 30% 的資料，
就要使用 `ESCAPE` 關鍵字並指定適當的「逸出字元」：

```sql
SELECT * FROM TabTest
WHERE 折扣 LIKE '30!%' ESCAPE '!'
```

只要含有 - （減號）的任何記錄

```sql
SELECT * FROM TabTest
WHERE 地址 LIKE '%#-%' ESCAPE '#'
```

只要含有 * （星號）的任何記錄

```sql
SELECT * FROM TabTest
WHERE 附註 LIKE '@*%' ESCAPE '@' 
```

# 25.如何找出欄位值是 NULL 的資料

找出「產品編號」 325 以下的產品，且「顏色」欄位為 NULL 的資料

```sql
SELECT ProductID 產品編號, Color 顏色
FROM Production.Product
WHERE Color IS NULL
	AND ProductID < 325
ORDER BY ProductID
```

# 26.如何改變欄位值是 NULL 的資料顯示的結果

不要顯示為 NULL，而是改以其他文字來取代

ISNULL 

    ISNULL(陳述式1, 陳述式2)

```sql
SELECT ProductID 產品編號, ISNULL(Color, '未知') 顏色
FROM Production.Product
WHERE ProductID < 325
ORDER BY ProductID
```

當「陳述式1」的結果與「陳述式2」的結果相等時，就會傳回 NULL 值，否則就會傳回「陳述式1」的值

NULLIF

    NULLIF(陳述式1, 陳述式2)

```sql
SELECT ProductID 產品編號, MakeFlag 製造代碼, FinishedGoodsFlag 成品代碼,
   NULLIF(MakeFlag, FinishedGoodsFlag) 製造代碼與成品代碼是否相同
FROM Production.Product
WHERE ProductID < 325
ORDER BY ProductID
```

# 27.[LEFT]如何使用 T-SQL 取到小數點後第 n 位

```sql
-- 宣告變數
DECLARE @pi float

-- 設定初始值
SET @pi=3.1415926

-- 顯示原本的值
SELECT 原本的值 = @pi

-- 取到小數點後第 1 位
SELECT [取到小數點後第 1 位] = LEFT(@pi, CHARINDEX('.', @pi) + 1)
```

自訂函數

```sql
-- 建立使用者自訂函數
CREATE FUNCTION dbo.GetDecimal(@number float, @point int)
RETURNS float
AS
BEGIN
	RETURN LEFT(@number, CHARINDEX('.', @number) + @point)
END
GO

-- 測試看看
SELECT [取到小數點後第 3 位] = dbo.GetDecimal(3.1415926, 3)
```

# 28.如何取得文數字的資料中，所有的數字的值

* DATALENGTH
* SUBSTRING
* ASCII

```sql
-- 宣告變數
DECLARE @position int, @strInput nvarchar(100), @intASCII int, @strTemp nvarchar(1), @strOuput nvarchar(100)

-- 初始化變數
SET @position = 1
SET @strInput = '(02)2311-3731'
SET @strTemp = ''
SET @strOuput = ''

-- 當目前的字元位置小於或等於輸入的字串長度時
WHILE @position <= DATALENGTH(@strInput)
  BEGIN
	-- 取得某個位置的字元
	SET @strtemp = SUBSTRING(@strInput, @position, 1)

	-- 取得單一字元的 ASCII 之值
	SET @intASCII = ASCII(@strTemp)

	-- 判斷是否為數字：介於 48 ～ 57 之間
	IF (@intASCII >=48 and @intASCII <= 57) 
		BEGIN
			-- 將目前的數字附加到先前的數字之後
			SET @strOuput = @strOuput + @strTemp
		END

    -- 將目前的字元位置再加上 1，也就是往後移動 1 個字元
    SET @position = @position + 1
  END

PRINT N'轉換前的文字：' + @strInput
PRINT N'轉換後的文字：' + @strOuput
```

# 29.[EXECUTE]如何動態組出 T-SQL 指令（上）

找出「顏色」是黑色的產品

```sql
-- 宣告變數
DECLARE @SQLCommand nvarchar(200)
DECLARE @columnList nvarchar(50)
DECLARE @color varchar(10)

-- 定義變數
SET @columnList = N'ProductID 產品編號, Color 顏色'
SET @color = '''Black'''
-- 第 1 個單引號代表變數值的型別是 nvarchar 型別
-- 第 2 與第 3 個單引號最後會被視為只有一個單引號。

-- 動態組出 T-SQL
SET @SQLCommand = 'SELECT ' + @columnList + ' FROM Production.Product WHERE Color = ' + @color

-- 執行動態組出的 T-SQL
EXECUTE (@SQLCommand)
```

再多加上一個查詢的限制條件，就是「產品編號」要小於 325

```sql
USE AdventureWorks
GO

-- 宣告變數
DECLARE @SQLCommand nvarchar(200)
DECLARE @columnList nvarchar(50)
DECLARE @color varchar(10)
DECLARE @pID varchar(5)

-- 定義變數
SET @columnList = N'ProductID 產品編號, Color 顏色'
SET @color = '''Black'''
SET @pID = '325'

-- 動態組出 T-SQL
SET @SQLCommand = 'SELECT ' + @columnList + ' FROM Production.Product WHERE Color = ' + @color + ' AND ProductID < ' + @pID

-- 執行動態組出的 T-SQL
EXEC (@SQLCommand)
```

# 如何動態組出 T-SQL 指令（下）

`EXECUTE` 陳述式會讓駭客更容易進行 SQL 資料隱碼攻擊`（SQL Injection）`，所以這次要分享使用系統預存程序 `sp_executesql`，透過指定參數的方法動態組出 T-SQL 指令，來避免 SQL 資料隱碼的攻擊。

```sql
USE AdventureWorks
GO

-- 宣告變數
DECLARE @SQLCommand nvarchar(200)
DECLARE @columnList nvarchar(50)
DECLARE @parmColor varchar(10)

-- 定義變數
SET @columnList = N'ProductID 產品編號, Color 顏色'
SET @parmColor = 'Black'

-- 動態組出 T-SQL
SET @SQLCommand = 'SELECT ' + @columnList + ' FROM Production.Product WHERE Color = @color'

-- 執行動態組出的 T-SQL
EXECUTE sp_executesql @SQLCommand, N'@color varchar(10)', @color = @parmColor
```

## sp_executesql

系統預存程序 sp_executesql 一共需要傳入 3 個參數，分別說明如下：

### 第 1 個參數：

所要執行的 T-SQL 陳述式，其內容必須是 Unicode 字串，所以在宣告 `@SQLCommand` 時，必須指定為 n 開頭的資料型別，在這個範例我是使用 `nvarchar` 型別。
因為要把英文的欄位名稱改用中文來顯示，所以在定義變數 `@columnList` 時，要使用前置詞 `N` 來指定該變數的內容。

### 第 2 個參數：

它會包含第 1 個參數中，所有內嵌的參數定義。以本程式碼為例，`@color` 就是內嵌的參數定義。同樣的，這個參數也必須使用前置詞 `N `來指定該變數的內容，且每個參數的定義都是由參數名稱（在該程式碼內，就是 @color）和資料型別（在該程式碼中，就是 varchar(10)）所組成。

### 第 3 個參數：

指定第 2 個參數內的參數定義的值。以本程式碼為例，第二個參數指定為 `@color varchar(10)`，所以要把先前設定的 `@parmColor` 的值指派給 `@color`，這樣才能讓 `@parmColor` 所代表的 `Black` 傳遞給 `@color`。

## 再加上一個查詢的限制條件

```sql
USE AdventureWorks
GO

-- 宣告變數
DECLARE @SQLCommand nvarchar(200)
DECLARE @columnList nvarchar(50)
DECLARE @parmColor varchar(10)
DECLARE @parmPID int

-- 定義變數
SET @columnList = N'ProductID 產品編號, Color 顏色'
SET @parmColor = 'Black'
SET @parmPID = 325

-- 動態組出 T-SQL
SET @SQLCommand = 'SELECT ' + @columnList + ' FROM Production.Product WHERE Color = @color AND ProductID < @pID'

-- 執行動態組出的 T-SQL
EXECUTE sp_executesql @SQLCommand, N'@color varchar(10), @pID int',
						@color = @parmColor, @pID = @parmPID
```

## 不想使用 EXECUTE 或 sp_executesql 來動態組出 T-SQL

```sql
USE AdventureWorks
GO

-- 宣告變數
DECLARE @parmColor varchar(10)
DECLARE @parmPID int

-- 定義變數
SET @parmColor = 'Black'
SET @parmPID = 325

SELECT ProductID 產品編號, Color 顏色
FROM Production.Product WHERE Color = @parmColor AND ProductID < @parmPID
```





[T-SQL 應用之美](https://ithelp.ithome.com.tw/articles/10007971)

[SQL中Case的使用方法](http://sstsupman.blogspot.com/2012/10/sqlcase.html)

# 10.使用流程控制：BEGIN...END 與 RETURN

## 函數

    BEGIN...END 與 RETURN
    --呼叫自訂的函數
    SELECT dbo.fn_GetAge('19990818') AS 年齡

```sql
-- 建立自訂函數
CREATE FUNCTION dbo.fn_GetAge(@myDate datetime)
RETURNS int
AS
BEGIN
-- 宣告變數
DECLARE @age int, @day datetime
SET @day = getdate()
-- 以「年」為單位計算出年齡
SET @age = DATEDIFF(yy, @myDate, @day) -
		CASE WHEN @day < DATEADD(yy, DATEDIFF(yy, @myDate, @day), @myDate)
			THEN 1
			ELSE 0
		END
RETURN @age

END
GO

-- 呼叫自訂的函數
SELECT dbo.fn_GetAge('19990818') AS 年齡
-- 19 : 2019-1999=20 未到8月
```

## 預存程序

    不用 RETURN
    呼叫預存程序
    EXEC dbo.proc_GetAge2 '19990818'

```sql
-- 建立預存程序
CREATE PROCEDURE proc_GetAge1(@myDate datetime)
AS
BEGIN
-- 宣告變數
DECLARE @age int, @day datetime
SET @day = getdate()
-- 以「年」為單位計算出年齡
SET @age = DATEDIFF(yy, @myDate, @day) -
		CASE WHEN @day < DATEADD(yy, DATEDIFF(yy, @myDate, @day), @myDate)
			THEN 1
			ELSE 0
		END
-- 使用 SELECT 傳回結果
SELECT @age 

END
GO

-- 呼叫預存程序
EXEC dbo.proc_GetAge2 '19990818'
```

# 11.使用流程控制：WHILE、BREAK 與 CONTINUE

計算從 1 加到 100 的總和

```sql
-- 定義變數
DECLARE @count int, @sum int

-- 指定初始值
SET @count = 0
SET @sum = 0

-- 開始計算
WHILE (@count < 101)
	BEGIN
		SET @sum = @sum + @count
		SET @count = @count + 1
	END

-- 顯示結果
SELECT @sum 總和
```

通常我們會在 WHILE 迴圈中，使用 @@FETCH_STATUS 判斷是否要繼續從 CURSOR 中，抓資料出來

```sql
USE AdventureWorks
GO

-- 定義變數
DECLARE @Name nvarchar(50)

-- 定義 CURSOR
DECLARE myCursor CURSOR FOR
	SELECT [Name] FROM Sales.Store

-- 開啟 CURSOR
OPEN myCursor

-- 抓出資料
FETCH NEXT FROM myCursor
	INTO @Name

-- 檢查 @@FETCH_STATUS 來決定是否要繼續執行
WHILE (@@FETCH_STATUS = 0)
	BEGIN
		PRINT N'商店名稱：' + @Name
		FETCH NEXT FROM myCursor
			INTO @Name
	END

-- 關閉 CURSOR
CLOSE myCursor

-- 釋放 CURSOR
DEALLOCATE myCursor
```

把 `SELECT` 的查詢結果作為 `WHILE` 迴圈的條件，然後再使用 `BREAK` 與 `CONTINUE` 來決定是要離開迴圈或是繼續執行迴圈

```sql
USE AdventureWorks
GO

-- 建立一個測試用的資料表
SELECT [Name], ListPrice INTO myTable
	FROM Production.Product

-- 未調價前的平均價格
SELECT 調價前的平均價格 = AVG(ListPrice) FROM myTable

-- 如果平均單價小於 450，則提高 2 倍價格
WHILE (SELECT AVG(ListPrice) FROM myTable) < $450
	BEGIN
		UPDATE myTable
		SET ListPrice = ListPrice * 2

		-- 找出最高的價格
		SELECT 最高的價格 = MAX(ListPrice) FROM myTable

		-- 如果最高價格超過 800，就不再調價
		-- 如果沒有超過 800，還要繼續調價
		IF (SELECT MAX(ListPrice) FROM myTable) > $800
			BREAK
		ELSE
			CONTINUE
	END

SELECT 調價後的平均價格 = AVG(ListPrice) FROM myTable

-- 刪掉測試用的資料表
DROP TABLE myTable
```

# 12.使用流程控制：IF...ELSE

在建立資料表之前，通常都會檢查是否已經存在相同名稱的資料表

**OBJECT_ID**

    使用OBJECT_ID時N和U代表什麼/？
    IF OBJECT_ID（N'tempdb .. #test'，'U'）IS NOT NULL
    N'tempdb .. #test' : unicode（nvarchar）字符串
    'U'表示用戶定義的表

```sql
-- 檢查資料表是否已經存在，如果存在，就把它刪除掉
IF OBJECT_ID(N'myTable', N'U') IS NOT NULL
	DROP TABLE myTable
```

例如在性別欄位中，使用 M 代表男性，使用 F 代表女性。而在顯示這類欄位的資料時，就可以使用 IF...ELSE 來將代碼轉換成文字描述，這樣才能讓然人家清楚的知道資料所代表的意義是什麼：

```sql
USE AdventureWorks
GO

-- 定義變數
DECLARE @EmployeeID int, @Gender nchar(1), @ChineseGender nchar(2)

-- 定義 CURSOR
DECLARE myCursor CURSOR FOR
	SELECT EmployeeID, Gender FROM HumanResources.Employee

-- 開啟 CURSOR
OPEN myCursor

-- 抓出資料
FETCH NEXT FROM myCursor
	INTO @EmployeeID, @Gender

-- 檢查 @@FETCH_STATUS 來決定是否要繼續執行
WHILE (@@FETCH_STATUS = 0)
	BEGIN
		PRINT N'員工編號：' + CAST(@EmployeeID AS NCHAR(10))
		
		IF @Gender = 'M'
			SET @ChineseGender = N'男'
		ELSE
			SET @ChineseGender = N'女'
		
		PRINT N'性別：' + @ChineseGender
		PRINT REPLICATE('-', 20)

		FETCH NEXT FROM myCursor
			INTO @EmployeeID, @Gender
	END

-- 關閉 CURSOR
CLOSE myCursor

-- 釋放 CURSOR
DEALLOCATE myCursor
```

## 效能[IF EXISTS & SELECT COUNT(*)]

假設資料表中，當某個欄位的欄位值已經存在資料，就要更新這些資料，如果沒有相關的資料存在，就要新增一筆資料。通常我們會直覺的使用下面這種寫法：

```sql
-- 計算筆數
IF (SELECT COUNT(*) FROM myTable WHERE 欄位名稱='欄位值') > 0
    UPDATE myTable SET (...) WHERE 欄位名稱='欄位值'
ELSE
    INSERT INTO myTable VALUES (...)
```

因為我們只是要檢查是否存在任何相符的記錄，根本不需要知道有多少筆資料，所以應該改用 `IF EXISTS` 來取代 `SELECT COUNT(*)`。這麼一來，只要找到第一筆相符的資料時，`IF EXISTS` 就會馬上停止搜尋，也就是說，根本不需要再多花費其他額外的時間，就已經知道至少存在一筆相符的資料了

```sql
-- 檢查資料是否已經存在
IF EXISTS (SELECT * FROM myTable WHERE 欄位名稱='欄位值')
    UPDATE myTable SET (...) WHERE 欄位名稱='欄位值'
ELSE
    INSERT INTO myTable VALUES (...)
```

因為在執行 `SELECT` 或 `UPDATE` 指令時，都會進行 Table Scan 或 Index Scan，也就是說最多會進行 2 次 SCAN，最少 1 次 SCAN。在改用下面的寫法之後，可以確保最多只會執行 1 次 Table Scan 或 Index Scan：

```sql
-- 更新資料表
UPDATE myTable SET (...) WHERE 欄位名稱='欄位值'

-- 檢查更新的資料筆數是否為 0，是 0 的話，就表示根本沒相符的資料要更新
-- 所以要新增一筆新的資料
IF @@ROWCOUNT = 0
    INSERT INTO myTable VALUES (...)
```

# 13.使用流程控制：CASE...WHEN

[SQL中Case的使用方法](http://sstsupman.blogspot.com/2012/10/sqlcase.html)

這個範例是將代表性別與婚姻代碼的欄位，轉換成大家看的懂得文字描述：

**簡單的 CASE 功能**

    它將會與一組簡單的運算式比對，以判斷其結果
    <<單一值的比較>>

```sql
USE AdventureWorks
GO

SELECT 員工編號 = EmployeeID,
	性別 = CASE Gender
			WHEN 'M' THEN N'男'
			ELSE N'女'
		END,
	婚姻 = CASE MaritalStatus
			WHEN 'S' THEN N'單身'
			ELSE N'已婚'
		END
FROM HumanResources.Employee
```

改寫:

**搜尋的 CASE 功能**

    它將會與一組布林運算式(可以寫複雜的邏輯)來判斷其結果
    <<多種條件的判斷>> -> WHEN (cond1 AND/OR cond2 ...)

```sql
USE AdventureWorks
GO

SELECT 員工編號 = EmployeeID,
	性別 = CASE
			WHEN (Gender = 'M') THEN N'男'
			ELSE N'女'
		END,
	婚姻 = CASE 
			WHEN (MaritalStatus = 'S') THEN N'單身'
			ELSE N'已婚'
		END
FROM HumanResources.Employee
```

# 14.使用流程控制：GOTO

GOTO 語法

```sql
-- 定義一個名稱為 label 的標籤
label:
-- 在這裡面寫程式碼
...
...

-- 跳到名稱為 label 的標籤哪邊去執行
GOTO label

-- 定義一個名稱為 execution 的標籤
execution:
-- 在這裡面寫程式碼
...
...
```

把先前在「使用流程控制：WHILE、BREAK 與 CONTINUE」用過的例子：計算從 1 加到 100 的總和，再拿來用，但是這次要改用 GOTO：

```sql
-- 定義變數
DECLARE @count int, @sum int

-- 指定初始值
SET @count = 0
SET @sum = 0

-- 開始計算
StartCalculate:
IF (@count < 101)
	BEGIN
		SET @sum = @sum + @count
		SET @count = @count + 1
		GOTO StartCalculate
	END
ELSE
	GOTO StopCalculate

-- 顯示結果
StopCalculate:
SELECT @sum 總和
```

精簡

```sql
-- 定義變數
DECLARE @count int, @sum int

-- 指定初始值
SET @count = 0
SET @sum = 0

-- 開始計算
StartCalculate:
IF (@count < 101)
	BEGIN
		SET @sum = @sum + @count
		SET @count = @count + 1
		GOTO StartCalculate
	END

-- 顯示結果
SELECT @sum 總和
```

## 批次 GO 指令

GO 不是 T-SQL 的陳述式，它是 sqlcmd 、osql、isql 這幾個公用程式與 SQL Server Management Studio 或 SQL Server Enterprise Manager 程式碼編輯器都能夠辨識的一個命令，以便將目前的 T-SQL 陳述式以**「批次」方式傳送給 SQL Server 進行執行

```sql
PRINT '今天是 ' + CONVERT(char(10), getdate(), 101)
GO 10
```

GO 後面所接的那個參數，是指定要執行在 GO 之前的 T-SQL 批次的次數啊！

# 15.如何在 T-SQL 中宣告變數

在 T-SQL 中，需要使用到變數，不外乎下面幾種情況：

* 將參數傳給預存程序或是函式 # 10.
* 控制迴圈的執行 # 11.
* 在 IF 陳述式中，作為條件判斷的依據 # 12.
* 在 WHERE 陳述式中，作為篩選條件的依據

在 Microsoft SQL Server 裡，所有的變數都是區域變數，也就是說，所宣告的變數只能夠在同一個批次、預存程序或該變數所定義的區塊裡面，被用到。

## 宣告變數

```sql
-- 可以同時宣告多個變數，變數跟變數之間，用逗號來分隔
DECLARE @count int, @x int, @y nvarchar(10)

-- 檢查變數的初始值
SELECT [@count] = @count, [@x] = @x, [@y] = @y
-- [@count] : [欄位名稱]

-- 使用 SET 指派值
SET @count = 1

-- 使用 SELECT 指派值
SELECT @x = 0, @y = 'alexc'

-- 檢查變數的設定值
SELECT [@count] = @count, [@x] = @x, [@y] = @y
```

## WHERE [與 宣告變數] 陳述式中，作為篩選條件的依據

假設要從 AdventureWorks 資料庫的 HumanResources.Employee 資料表中，找出員工編號小於等於 10 的員工，可以使用下面的程式碼：

```sql
USE AdventureWorks
GO

DECLARE @EmpID int
SET @EmpID = 10

SELECT 員工編號 = EmployeeID,
	性別 = CASE Gender
			WHEN 'M' THEN N'男'
			WHEN 'F' THEN N'女'
		END,
	婚姻 = CASE MaritalStatus
			WHEN 'S' THEN N'單身'
			WHEN 'M' THEN N'已婚'
		END
FROM HumanResources.Employee
WHERE EmployeeID <= @EmpID
```

# 16.如何使用次序函數於查詢資料時，針對某個欄位進行排名(上)

[排名函數 row_nimber() rank() dense_rank()](http://tina0221.pixnet.net/blog/post/69775503-%E3%80%90sql%E3%80%91case%E5%87%BD%E6%95%B8%E3%80%81%E6%8E%92%E5%BA%8F%E5%87%BD%E6%95%B8%28row%E3%80%81rank%E3%80%81dense_rank%29)

[T-SQL DOC 次序函數](https://docs.microsoft.com/zh-tw/previous-versions/sql/sql-server-2005/ms189798(v=sql.90))

Microsoft SQL Server 2008 已經把*次序函數*改名成*排名函數*。

**「次序函數」來傳回資料分割（Partition）中，某個欄位的一個次序值，共有下列 4 種「次序函數」**：

* ROW_NUMBER

row_nimber()

    row_nimber() over (order by 欄位排序)：一欄位排序給序號

* RANK

rank() 

    rank() over (order by 欄位排序)：同上，遇到同值時排名相同，後面跳過(1、2、2、4)
* DENSE_RANK

dense_rank()

    dense_rank() over (order by 欄位排序):同上，後面不跳過而是接續(1、2、2、3)

* NTILE

NTILE()

    NTILE(4) OVER (ORDER BY 欄位排序) : 按照給定數字，等分排名 
    EX: ROW -> 9 NTILE(4)
    排序後給定序號 -> 1,1,1,2,2,3,3,4,4

先在 AdventureWorks 資料庫中，建立在「使用流程控制：BEGIN...END 與 RETURN」討論過的自訂計算年齡函數： # 10

```sql
USE AdventureWorks
GO

-- 建立自訂函數
CREATE FUNCTION dbo.fn_GetAge(@myDate datetime)
RETURNS int
AS
BEGIN

-- 宣告變數
DECLARE @age int, @day datetime

-- 以「年」為單位計算出年齡
SET @age = DATEDIFF(yy, @myDate, getdate()) -
		CASE WHEN @day < DATEADD(yy, DATEDIFF(yy, @myDate, @day), @myDate)
			THEN 1
			ELSE 0
		END
RETURN @age

END
GO
```

接著要從 HumanResources.Employee 資料表中，透過上面建立的 fn_GetAge 函數，依照年齡由小至大，產生排名的欄位：

```sql
SELECT 依照年齡排名 = ROW_NUMBER() OVER (ORDER BY dbo.fn_GetAge(BirthDate)),
	年齡 = dbo.fn_GetAge(BirthDate),
	員工編號 = EmployeeID,
	性別 = CASE Gender
			WHEN 'M' THEN N'男'
			WHEN 'F' THEN N'女'
		END,
	婚姻 = CASE MaritalStatus
			WHEN 'S' THEN N'單身'
			WHEN 'M' THEN N'已婚'
		END
FROM HumanResources.Employee
```

`PARTITION BY` 的欄位可以有多個，例如先依照性別分組，再依照婚姻來分組：

```sql
SELECT 依序以性別與婚姻分組的年齡排名 = ROW_NUMBER() OVER (
    PARTITION BY Gender, MaritalStatus 
    ORDER BY dbo.fn_GetAge(BirthDate)
    ),
	年齡 = dbo.fn_GetAge(BirthDate),
	員工編號 = EmployeeID,
	性別 = CASE Gender
			WHEN 'M' THEN N'男'
			WHEN 'F' THEN N'女'
		END,
	婚姻 = CASE MaritalStatus
			WHEN 'S' THEN N'單身'
			WHEN 'M' THEN N'已婚'
		END
FROM HumanResources.Employe
```

# 17.如何使用次序函數於查詢資料時，針對某個欄位進行排名（下）

如果要把結果分成特定數目的組，可以使用 `NTILE()` 函數，例如下面的程式碼會依照年齡來排名，將資料分成 58 組：

```sql
SELECT [依照年齡排名分成 58 組] = NTILE(58) OVER (
    ORDER BY dbo.fn_GetAge(BirthDate)
    ),
	年齡 = dbo.fn_GetAge(BirthDate),
	員工編號 = EmployeeID,
	性別 = CASE Gender
			WHEN 'M' THEN N'男'
			WHEN 'F' THEN N'女'
		END,
	婚姻 = CASE MaritalStatus
			WHEN 'S' THEN N'單身'
			WHEN 'M' THEN N'已婚'
		END
FROM HumanResources.Employee
```

`NTILE()` 函數最常用於找出某一組中的資料，例如依照年齡排名，把資料分成 58 組，然後要找出第 2 組中的資料：

```sql
SELECT 年齡 = dbo.fn_GetAge(BirthDate),
	員工編號 = EmployeeID,
	性別 = CASE Gender
			WHEN 'M' THEN N'男'
			WHEN 'F' THEN N'女'
		END,
	婚姻 = CASE MaritalStatus
			WHEN 'S' THEN N'單身'
			WHEN 'M' THEN N'已婚'
		END
FROM (
    SELECT 分組 = NTILE(58) OVER (ORDER BY dbo.fn_GetAge(BirthDate)),
	BirthDate, EmployeeID, Gender, MaritalStatus
	FROM HumanResources.Employee
    ) temp
WHERE 分組 = 2
```

# 18.如何使用次序函數刪除重複的記錄











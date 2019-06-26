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

```sql
-- 檢查資料表是否已經存在，如果存在，就把它刪除掉
IF OBJECT_ID(N'myTable', N'U') IS NOT NULL
	DROP TABLE myTable
```


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
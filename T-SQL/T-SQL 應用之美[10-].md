
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












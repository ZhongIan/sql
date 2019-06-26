
[T-SQL 應用之美](https://ithelp.ithome.com.tw/articles/10007971)

[DATEADD DATEEIFF()](https://dotblogs.com.tw/lastsecret/2010/10/04/18097)

`DATEDIFF`是算兩個日期間的間隔，傳回帶正負號的整數

    DATEDIFF ( datepart , startdate , enddate )

`DATEADD`是計算某日期加上一個數值，傳回的日期

    DATEADD (datepart , number , date )


# 1.使用 getdate() 函數找出本月第一天

`DATEADD` 與 `DATEEIFF()` 這兩個函數之後，就可以推算出本月第一天的日期，我們更進一步使用 `CAST()` 與 `CONVERT()` 函數轉換日期的格式。

`mm` 表月份

```sql
-- 精確到毫秒
-- DATEDIFF(mm, '', getdate()), '') 與 1900-1-1 差多少 mm
-- DATEADD 在從 1900-1-1 + 差多少 mm
SELECT DATEADD(mm, DATEDIFF(mm, '', getdate()), '') AS [本月第一天(精確到毫秒)]

-- 使用 CAST 轉換格式
-- CAST ( expression AS data_type )
SELECT CAST(DATEADD(mm, DATEDIFF(mm, '', getdate()), '') AS VARCHAR(20)) AS [本月第一天(僅到分)]

-- 使用 CONVERT 轉換日期輸出的格式
-- CONVERT ( data_type , expression [ , style ] )  
SELECT CONVERT(VARCHAR(20), DATEADD(mm, DATEDIFF(mm, '', getdate()), ''), 101) AS [本月第一天(僅到日)]
```

[CONVERT DOC](https://docs.microsoft.com/zh-tw/sql/t-sql/functions/cast-and-convert-transact-sql?view=sql-server-2017)

    CAST ( expression AS data_type )
    CONVERT ( data_type , expression [ , style ] ) 

以2000年2月12日這個特定的日期作為基準點，去推算出 2000年2月的第一天之日期

```sql
-- 精確到毫秒
SELECT DATEADD(mm, DATEDIFF(mm, '', '2000/2/12'), '') AS [2000年2月的第一天(精確到毫秒)]

-- 使用 CAST 轉換日期輸出的格式
SELECT CAST(DATEADD(mm, DATEDIFF(mm, '', '2000-02-12'), '') AS VARCHAR(20)) AS [2000年2月的第一天(僅到分)]

-- 使用 CONVERT 轉換日期輸出的格式
SELECT CONVERT(VARCHAR(20), DATEADD(mm, DATEDIFF(mm, '', '02-12-2000'), ''), 101) AS [2000年2月的第一天(僅到日)]
```

# 2.使用 T-SQL 找出特定日期之當年的第一天

```sql
-- 以電腦目前的日期和時間為基準點
-- 精確到毫秒
SELECT DATEADD(year, DATEDIFF(year, '', getdate()), '') AS [本年的第一天(精確到毫秒)]

-- 使用 CAST 轉換日期輸出的格式
SELECT CAST(DATEADD(year, DATEDIFF(year, '', getdate()), '') AS VARCHAR(20)) AS [本年的第一天(僅到分)]

-- 使用 CONVERT 轉換日期輸出的格式
SELECT CONVERT(VARCHAR(20), DATEADD(year, DATEDIFF(year, '', getdate()), ''), 101) AS [本年的第一天(僅到日)]
```

year 表年，可以寫成 yyyy 或是 yy

# 3.使用 T-SQL 找出特定日期之當季的第一天


```sql
-- 以電腦目前的日期和時間為基準點
SELECT getdate() AS 目前日期與時間

-- 精確到毫秒
SELECT DATEADD(quarter, DATEDIFF(quarter, '', getdate()), '') AS [本季的第一天(精確到毫秒)]

-- 使用 CAST 轉換日期輸出的格式
SELECT CAST(DATEADD(quarter, DATEDIFF(quarter, '', getdate()), '') AS VARCHAR(20)) AS [本季的第一天(僅到分)]

-- 使用 CONVERT 轉換日期輸出的格式
SELECT CONVERT(VARCHAR(20), DATEADD(quarter, DATEDIFF(quarter, '', getdate()), ''), 101) AS [本季的第一天(僅到日)]
```

quarter 表季，可以寫成 qq 或是 q

# 4.如何使用 T-SQL 找出特定日期該週的星期一日期為何

```sql
-- 以 2000 年 2 月 12 日為基準點
-- 精確到毫秒
SELECT DATEADD(week, DATEDIFF(week, '', '2000/2/12'), '') AS [2000年2月12日當週星期一的日期(精確到毫秒)]

-- 使用 CAST 轉換日期輸出的格式
SELECT CAST(DATEADD(week, DATEDIFF(week, '', '2000-02-12'), '') AS VARCHAR(20)) AS [2000年2月12日當週星期一的日期(僅到分)]

-- 使用 CONVERT 轉換日期輸出的格式
SELECT CONVERT(VARCHAR(20), DATEADD(week, DATEDIFF(week, '', '02-12-2000'), ''), 101) AS [2000年2月12日當週星期一的日期(僅到日)]
```

week表週，可以寫成 wk 或是 ww

```sql
-- 自訂一個 DATETIME 型別的變數
DECLARE @myDate as DATETIME
-- 找出2000年2月12日當週星期一的日期
SET @myDate = DATEADD(week, DATEDIFF(week, '', '2000/2/12'), '')
-- 星期一往後數的第 4 天是星期五
SELECT DATEADD(day, +4, @myDate) AS [2000年2月12日當週星期五的日期(精確到毫秒)]
```

# 5.透過 T-SQL 找出特定日期的上個月或前一年最後一天與倒數第 n 天之日期

```sql
-- 自訂一個 DATETIME 型別的變數
DECLARE @myDate AS DATETIME

-- 以 2008 年 3 月 19 日為基準點，找出 3 月第一天的日期
SELECT @myDate = DATEADD(mm, DATEDIFF(mm, '', '2008/3/19'), '')

-- 以「日」為單位，扣掉 1 天
SELECT DATEADD(day, -1, @myDate) AS [2008年3月19日的上個月最後一天(精確到毫秒)]
```

# 6.如何使用 T-SQL 找出特定日期該月的第一個星期一之日期

```sql
-- 自訂相關變數
DECLARE @myDate DATETIME,
	@days int,
	@firstSat DATETIME,
	@weekDiff int

-- 指定特定的日期給 @myDate
SET @myDate = '20190624'

-- 計算天數
SELECT @days = DAY(@myDate)
-- 上面那段程式碼同等於下面這段
-- SELECT DATEPART(dd, @myDate)
-- '20190624' -> 24

-- 取得當月第 6 天的日期
SELECT DATEADD(dd, 6 - @days, @myDate)
-- int: 6-@days + str: @myDate = 當月第 6 天

-- 與 1900 年 1 月 1 日相距多少週
SELECT @weekDiff = DATEDIFF(wk, '', @firstSat)

-- 將 1900 年 1 月 1 日加上相距的週數就是我們要的結果
SELECT DATEADD(wk, @weekDiff, '') [特定日期該月的第一個星期一]
```

```sql
-- 自訂一個 DATETIME 型別的變數
DECLARE @myDate DATETIME

-- 指定特定的日期給 @myDate
SET @myDate = '20080919'

-- 使用一道程式碼來計算我們要的結果
SELECT DATEADD(wk, DATEDIFF(wk, '', DATEADD(dd, 6 - DAY(@myDate), @myDate)), '') AS
	[特定日期該月的第一個星期一]
```

```sql
-- 自訂一個 DATETIME 型別的變數
DECLARE @myDate DATETIME

-- 指定特定的日期給 @myDate
SET @myDate = '20080919'

-- 找出某月第 3 個星期一的日期
SELECT DATEADD(wk, DATEDIFF(wk, '', DATEADD(dd, 6 - DAY(@myDate), @myDate)) + 2 , '') AS 
	[第 3 個星期一的日期]
```

# 7.如何使用 T-SQL 計算年齡

```sql
-- 自訂一個 DATETIME 型別的變數
DECLARE @myDate as DATETIME

-- 指定日期變數
SET @myDate = '19880907'

-- 以「年」為單位計算出年齡
SELECT DATEDIFF(yy, @myDate, getdate()) as 年齡
```

**虛歲，實歲**

```sql
-- 宣告變數
DECLARE @myDate DATETIME, @today DATETIME

-- 指定日期變數
SET @myDate = '20081231'
SET @today = '20090101'

-- 以「年」為單位計算出年齡
SELECT DATEDIFF(yy, @myDate, @today) 年齡
-- 差1年
```

```sql
-- 宣告變數
DECLARE @myDate DATETIME, @age int, @day DATETIME

-- 指定日期變數
SET @myDate = '19881207'
SET @day = getdate()

-- 以「年」為單位計算出年齡
SET @age = DATEDIFF(yy, @myDate, @day) -
		CASE WHEN @day < DATEADD(yy, DATEDIFF(yy, @myDate, @day), @myDate)
			THEN 1
			ELSE 0
		END

SELECT @age AS 年齡, DATEDIFF(yy, @myDate, @day) AS DA
-- 年齡: 30, DA: 31
-- @day < DATEADD(yy, DATEDIFF(yy, @myDate, @day), @myDate)
-- < : 虛歲(-1) ;  > : 實歲(-0)
```

# 8.如何找出某個日期區間內的資料

## 測試資料

```sql
USE tempdb
GO

CREATE TABLE TabTest(
	[出貨日期] [datetime] NOT NULL)
GO

SET NOCOUNT ON

INSERT INTO TabTest VALUES('2008-01-13 00:00:00.000')
INSERT INTO TabTest VALUES('2008-01-23 08:08:08.000')
INSERT INTO TabTest VALUES('2008-01-31 00:00:00.000')
INSERT INTO TabTest VALUES('2008-02-05 00:00:00.000')
INSERT INTO TabTest VALUES('2008-02-05 23:59:00.997')
INSERT INTO TabTest VALUES('2008-02-06 00:00:00.000')
INSERT INTO TabTest VALUES('2008-03-08 21:55:00.997')
INSERT INTO TabTest VALUES('2008-03-14 23:59:00.997')
INSERT INTO TabTest VALUES('2008-03-16 10:00:00.000')
INSERT INTO TabTest VALUES('2008-03-28 18:58:00.000')
```

```sql
SELECT * FROM TabTest
	WHERE 出貨日期 = '20080205'

-- '2008-02-05 00:00:00.000'
```

原來是因為出貨日期欄位的資料精確到「毫秒」

```sql
SELECT * FROM tabtest
	WHERE 出貨日期 BETWEEN '20080205 00:00:00.000' AND
                          '20080205 23:59:59.999'              
```

多出一筆 2008 年 2 月 6 日的資料

由於 Microsoft SQL Server 對於時間的精確度可以高達「**一千分之三秒**」，所以結束的時間不能用 23:59:59.999，要改用 **23:59:59.997**，這樣才能避免 SQL Server 把 20080205 23:59:59.999** 變成 20080206 00:00:00.000：

```sql
SELECT * FROM tabtest
	WHERE 出貨日期 >= DATEADD(day, DATEDIFF(day, '', '20080205'), '')
	AND   出貨日期 <  DATEADD(day, DATEDIFF(day, '', '20080205') + 1, '')
```

# 9.如何使用 CAST 與 CONVERT 格式化日期與時間資料

## CONVERT

    CONVERT ( data_type , expression [ , style ] ) 
    CONVERT ( 資料型別 [ (資料長度) ] , 運算式 [ , 日期格式樣式 ] )

```sql
-- 定義變數
DECLARE @myDate DATETIME
-- 指派一個日期給該變數
SET @myDate = '2008/09/09 08:25 AM'

-- 開始轉換 不同格式
PRINT CONVERT(CHAR(19), @myDate)
PRINT CONVERT(CHAR(19), @myDate, 0)
PRINT CONVERT(CHAR(8),  @myDate, 1)
PRINT CONVERT(CHAR(8),  @myDate, 2)
PRINT CONVERT(CHAR(8),  @myDate, 3)
PRINT CONVERT(CHAR(8),  @myDate, 4)
PRINT CONVERT(CHAR(8),  @myDate, 5)
PRINT CONVERT(CHAR(9),  @myDate, 6)
PRINT CONVERT(CHAR(10), @myDate, 7)
PRINT CONVERT(CHAR(8),  @myDate, 8)
PRINT CONVERT(CHAR(26), @myDate, 9)
PRINT CONVERT(CHAR(8),  @myDate, 10)
PRINT CONVERT(CHAR(8),  @myDate, 11)
PRINT CONVERT(CHAR(6),  @myDate, 12)
PRINT CONVERT(CHAR(24), @myDate, 13)
PRINT CONVERT(CHAR(12), @myDate, 14)
PRINT CONVERT(CHAR(19), @myDate, 20)
PRINT CONVERT(CHAR(23), @myDate, 21)
PRINT CONVERT(CHAR(20), @myDate, 22)
PRINT CONVERT(CHAR(10), @myDate, 23)
PRINT CONVERT(CHAR(8),  @myDate, 24)
PRINT CONVERT(CHAR(23), @myDate, 25)
PRINT CONVERT(CHAR(19), @myDate, 100)
PRINT CONVERT(CHAR(10), @myDate, 101)
PRINT CONVERT(CHAR(10), @myDate, 102)
PRINT CONVERT(CHAR(10), @myDate, 103)
PRINT CONVERT(CHAR(10), @myDate, 104)
PRINT CONVERT(CHAR(10), @myDate, 105)
PRINT CONVERT(CHAR(11), @myDate, 106)
PRINT CONVERT(CHAR(12), @myDate, 107)
PRINT CONVERT(CHAR(8),  @myDate, 108)
PRINT CONVERT(CHAR(26), @myDate, 109)
PRINT CONVERT(CHAR(10), @myDate, 110)
PRINT CONVERT(CHAR(10), @myDate, 111)
PRINT CONVERT(CHAR(8),  @myDate, 112)
PRINT CONVERT(CHAR(24), @myDate, 113)
PRINT CONVERT(CHAR(12), @myDate, 114)
PRINT CONVERT(CHAR(19), @myDate, 120)
PRINT CONVERT(CHAR(23), @myDate, 121)
PRINT CONVERT(CHAR(23), @myDate, 126)
PRINT CONVERT(CHAR(23), @myDate, 127)
PRINT CONVERT(CHAR(32), @myDate, 130)
PRINT CONVERT(CHAR(25), @myDate, 131)
```

## CAST ( expression AS data_type )

```sql
-- 定義變數
DECLARE @myDate DATETIME
-- 指派一個日期給該變數
SET @myDate = '2008/09/09 08:25 AM'

-- 開始轉換
PRINT '使用 CONVERT 轉換 ==> ' +
	CONVERT(CHAR(23), @myDate, 21)
PRINT '使用 CAST 轉換 =====> ' +
	CAST(DATEPART(YY, @myDate) AS CHAR(4)) + '-'
	+ RIGHT(CAST(100 + DATEPART(MM, @myDate) AS CHAR(3)), 2) + '-'
	+ RIGHT(CAST(100 + DATEPART(DD, @myDate) AS CHAR(3)), 2) + ' '
	+ RIGHT(CAST(100 + DATEPART(HH, @myDate) AS CHAR(3)), 2) + ':'
	+ RIGHT(CAST(100 + DATEPART(MI, @myDate) AS CHAR(3)), 2) + ':'
	+ RIGHT(CAST(100 + DATEPART(SS, @myDate) AS CHAR(3)), 2) + '.'
	+ RIGHT(CAST(1000+ DATEPART(MS, @myDate) AS CHAR(4)), 3)

-- CAST(100 + DATEPART(DD, @myDate) AS CHAR(3) 
-- 9 -> 09
```

## AM 或 PM 換成中文的上午或下午

```sql
DECLARE @myDate DATETIME
SET @myDate = '2008/09/09 08:25 AM'
SELECT CAST(DATEPART(YYYY, @myDate) AS CHAR(4)) + '/' 
	+ RIGHT(CAST(100 + DATEPART(MM, @myDate) AS CHAR(3)), 2) + '/'
	+ RIGHT(CAST(100 + DATEPART(DD, @myDate) AS CHAR(3)), 2) + ' '
	+ CASE WHEN DATEPART(HH, @myDate) < 13
		THEN RIGHT(CAST(100 + DATEPART(HH, @myDate) AS CHAR(3)), 2)
		ELSE CAST(DATEPART(HH, @myDate) - 12 AS CHAR(2))
	END + ':'
	+ RIGHT(CAST(100 + DATEPART(MI, @myDate) AS CHAR(3)), 2)
	+ CASE WHEN DATEPART(HH, @myDate) < 13
		THEN '上午'
		ELSE '下午'
	END
```

## 轉換西元日期成為民國年

```sql
DECLARE @myDate DATETIME
SET @myDate = '2008/09/09 08:25 AM'
SELECT CAST(YEAR(@myDate) - 1911 AS NVARCHAR(3)) + '年'
	+ CAST(Month(@myDate) AS NVARCHAR(2)) + '月'
	+ CAST(Day(@myDate) AS NVARCHAR(2)) + '日'

```











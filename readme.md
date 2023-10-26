## Check Constraints
    ``` bash
    CONSTRAINT CHK_Person CHECK (Age>=18 AND City='Sandnes')
    ```

## Computed
    ``` bash
    ALTER TABLE dbo.Products ADD RetailValue AS (QtyAvailable * UnitPrice * 1.5);
    ```
    Optionally, add the PERSISTED argument to physically store the computed values in the table:
    ``` bash
    ALTER TABLE dbo.Products ADD RetailValue AS (QtyAvailable * UnitPrice * 1.5) PERSISTED;
    ```

## Identity column
    ``` bash
    IDENTITY [ (seed , increment) ]
    ```

## Indexs
    1. Create a primary key with clustered index in a new table
    ``` bash
    -- Create table to add the clustered index
    CREATE TABLE Production.TransactionHistoryArchive1
    (
        CustomerID uniqueidentifier DEFAULT NEWSEQUENTIALID()
        , TransactionID int IDENTITY (1,1) NOT NULL
        , CONSTRAINT PK_TransactionHistoryArchive1_CustomerID PRIMARY KEY NONCLUSTERED (CustomerID)
    )
    ;

    -- Now add the clustered index
    CREATE CLUSTERED INDEX CIX_TransactionID ON Production.TransactionHistoryArchive1 (TransactionID);
    ```
    2. Create a nonclustered index on a table or view
    ``` bash
    CREATE INDEX index1 ON schema1.table1 (column1);
    ```
    3. Create a clustered index on a table and use a 3-part name for the table
    ``` bash
    CREATE CLUSTERED INDEX index1 ON database1.schema1.table1 (column1);
    ```
    4. Create a nonclustered index with a unique constraint and specify the sort order
    ``` bash
    CREATE UNIQUE INDEX index1 ON schema1.table1 (column1 DESC, column2 ASC, column3 DESC);
    ```

## Foreign key
    1. Create a foreign key in a new table
    ``` bash
    CREATE TABLE Sales.TempSalesReason 
        (
            TempID int NOT NULL, Name nvarchar(50)
            , CONSTRAINT PK_TempSales PRIMARY KEY NONCLUSTERED (TempID)
            , CONSTRAINT FK_TempSales_SalesReason FOREIGN KEY (TempID)
                REFERENCES Sales.SalesReason (SalesReasonID)
                ON DELETE CASCADE
                ON UPDATE CASCADE
        )
    ;
    ```
    2. Create a foreign key in an existing table
    ``` bash
    ALTER TABLE Sales.TempSalesReason
        ADD CONSTRAINT FK_TempSales_SalesReason FOREIGN KEY (TempID)
            REFERENCES Sales.SalesReason (SalesReasonID)
            ON DELETE CASCADE
            ON UPDATE CASCADE
    ;
    ```

## Select
    1. like thai and any first character
    ``` bash
    SELECT * FROM menu where descr like N'_ไก่%'  -- N = National -> Unicode
    ```
    2. Not
    ``` bash
    SELECT * FROM menu where descr like N'%[^ไก่]%'
    ```

## Aggregate function
    1. Example
    ``` bash
    select count(*) 
	from customer

    -- count(*) vs count(column_name)
    select count(*), count(fname), count(tel) from customer

    -- understand NULL
    select count(*) 
        from customer
        where tel is not null

    -- simple aggregate function
    select count(*), min(price), max(price), avg(price) from menu

    -- alias column name
    select count(*) as [# menu], 
        min(price) as [min price], 
        max(price) as [max price], 
        avg(price) as [avg. price]
    from menu
    ```
    2. Group By
    ``` bash
    select count(*) as [# menu]
	from menu
	group by categoryid

    -- column in group by
    select categoryid, count(*) as [# menu]
        from menu
        group by categoryid

    -- where vs having
    select categoryid, 
            count(*) as [# menu], 
            avg(price) as [avg. price]
        from menu
        where price > 100
        group by categoryid
    ```
    3. Having
    ``` bash
    -- where vs having
    select categoryid, 
            count(*) as [# menu], 
            avg(price) as [avg. price]
        from menu
        group by categoryid
        having avg(price) > 100

    select sum(NetPay), avg(NetPay) from orderhdr

    select sum(NetPay) 
        from orderhdr
        where year(intime) = 2002 and month(intime) = 1

    select year(intime), month(intime), sum(NetPay), avg(NetPay) 
        from orderhdr
        group by year(intime), month(intime)
        order by year(intime), month(intime)
    ```

## Collation sort thai
    ``` bash
    collate thai_ci_ai
    ```
## Date / Time
    ``` bash
    select getdate() -- GETDATE() วันและเวลาปัจจุบัน

    select CONVERT(date, getdate()) -- เอาเฉพาะส่วนของวันที่
    select CONVERT(time, getdate()) -- เอาเฉพาะส่วนที่เป็นเวลา

    select getdate(), 
        year(getdate()) as year, 
        month(getdate()) as month, 
        day(getdate()) as date

    -- int DATEPART
    -- see http://msdn.microsoft.com/en-us/library/ms174420.aspx
    select getdate() as now,
        datepart(year, getdate()) as year,
        datepart(month, getdate()) as month,
        datename(m, getdate()) as [month name],
        datepart(day, getdate()) as day,
        datepart(dw, getdate()) as [week day], -- sunday = 1, monday = 2
        datename(dw, getdate()) as [week day name],
        datepart(dy, getdate()) as [day of year],-- วันที่เท่าไรของปี
        datepart(quarter, getdate()) as quarter,
        datepart(hour, getdate()) as hour,
        datepart(minute, getdate()) as minute,
        datepart(second, getdate()) as second

    select getdate(), 
        dateadd(d, 5, getdate()),
        dateadd(d, -2, '2014-5-31')

    select datediff(day, '1995-7-28', getdate())

    select datediff(year, '1995-7-28', getdate()),
        datediff(month, '1995-7-28', getdate()),
        datediff(day, '1995-7-28', getdate())
    ```

    ``` bash
    -- date/time function

    select *
        from OrderHdr

    select *
        from OrderHdr 
        where intime = '2002-01-1'

    select *
        from OrderHdr 
        where year(intime) = 2002 and
            month(intime) = 1 and 
            day(intime) = 1

    -- แสดงการใช้ฟังก์ชัน convert เพื่อเอาเฉพาะส่วนของวันที่
    select *
        from OrderHdr 
        where convert(date, intime) = '2002-1-1'

    -- ยอดขายในวันที่ 2002-2-14 
    select convert(date, intime), sum(netpay)
        from OrderHdr 
        where convert(date, intime) = '2002-2-14'
        group by convert(date, intime)

    -- ลูกค้าเข้ามารับประทานอาหารในช่วงเวลาใด
    select datepart(hour, intime) as hour, 
        count(orderid) as [# customers]
        from OrderHdr
        where year(intime) = 2002
        group by datepart(hour, intime)
        order by hour

    -- ลูกค้าเข้ามารับประทานอาหารในช่วงเวลาใด
    -- แสดงการใช้ concat เพื่อสร้าง period
    select concat(datepart(hour, intime), ':00 - ', datepart(hour, intime), ':59') as period,
        count(orderid) as [# customers]
        from OrderHdr
        where year(intime) = 2002
        group by concat(datepart(hour, intime), ':00 - ', datepart(hour, intime), ':59')
        order by period

    -- เวลาเฉลี่ย (นาที) ที่ลูกค้าแต่ละโต๊ะใช้ แยกตามไตรมาส
    select datepart(quarter, intime) as quarter, avg(datediff(minute, intime, outtime)) as [avg. minutes]
        from OrderHdr
        where year(intime) = 2003
        group by datepart(quarter, intime)
        order by datepart(quarter, intime)

    -- วันใดในเดือน มกราคม 2002 มี NetPay สูงสุด
    select top (1) with ties convert(date, intime) as date, sum(netpay) as netpay
        from OrderHdr
        where year(intime) = 2002 and month(intime) = 1
        group by convert(date, intime)
        order by sum(netpay) desc

    -- วันใดในเดือน มกราคม 2002 มี NetPay ต่ำสุด
    select top (1) with ties convert(date, intime) as date, sum(netpay) as netpay
        from OrderHdr
        where year(intime) = 2002 and month(intime) = 1
        group by convert(date, intime)
        order by sum(netpay)

    -- top n percent
    select top (10) percent with ties convert(date, intime) as date, sum(netpay) as netpay
        from OrderHdr
        where year(intime) = 2002 and month(intime) = 1
        group by convert(date, intime)
        order by sum(netpay) desc
    ```

## Inner join
    ``` bash
    select MenuID, Descr, DescrTH, CategoryID
        from menu

    select * 
        from Category

    -- inner join 2 tables
    select MenuID, Menu.Descr, DescrTH, Menu.CategoryID, Category.Descr
        from Menu 
                inner join Category on menu.CategoryID = Category.CategoryID

    -- alias table name
    select MenuID, m.Descr, DescrTH, m.CategoryID, c.Descr
        from Menu m 
                inner join Category c on m.CategoryID = c.CategoryID

    -- inner join 3 tables
    select a.OrderID, b.MenuID, c.DescrTH
        from OrderHdr a 
                inner join OrderMenuDtl b on a.OrderID = b.OrderID
                inner join Menu c on b.MenuID = c.MenuID
        where convert(date, intime) = '2002-1-1'

    -- distinct
    select distinct b.MenuID, c.DescrTH
        from OrderHdr a 
                inner join OrderMenuDtl b on a.OrderID = b.OrderID
                inner join Menu c on b.MenuID = c.MenuID
        where convert(date, intime) = '2002-1-1'

    -- inner join + aggregate function
    select b.MenuID, c.DescrTH, sum(b.Qty) as [# orders]
        from OrderHdr a 
                inner join OrderMenuDtl b on a.OrderID = b.OrderID
                inner join Menu c on b.MenuID = c.MenuID
        where convert(date, intime) = '2002-1-1'
        group by b.MenuID, c.DescrTH
        order by sum(b.qty) desc
    ```

## Outer join
    ``` bash
    /*
    inner join เป็นการเชื่อมแถวระหว่างตาราง โดยผลลัพธ์ที่ได้จะเป็นแถวที่มีค่าของ common column ที่ตรงกัน 
    หรือ matching rows เช่น Menu.CategoryID = Category.CategoryID

    ส่วน outer join  จะเป็นการเชื่อมที่สามารถจะแสดงทั้งแถวที่มีค่า common column ตรงกัน 
    หรือ mathing rows และแถวที่ไม่ non-matching rows
    outer join จะแสดงแถวของตารางที่เชื่อมกันโดยจะแสดงทุกแถวตามเงื่อนไขที่กำหนดออกมา 
    แม้ว่าแถวนั้นจะไม่มีค่าที่ตรง  non-matching rows กับแถวในอีกตารางหนึ่งก็ตาม
    */

    select MenuID, DescrTH, b.CategoryID, a.Descr
        from Category a inner join Menu b
            on a.CategoryID = b.CategoryID

    select * from Category

    select MenuID, DescrTH, b.CategoryID, a.Descr
        from Category a left outer join Menu b
            on a.CategoryID = b.CategoryID

    select MenuID, DescrTH, b.CategoryID, a.Descr
        from Category a right outer join Menu b
            on a.CategoryID = b.CategoryID

    select MenuID, DescrTH, b.CategoryID, a.Descr
        from Menu b right outer join Category a
            on a.CategoryID = b.CategoryID
        where b.CategoryID is null
    ```

    ``` bash
    -- คุณสมบัติที่ต้องการของงานในตำแหน่งหน้าที่ A = {Word, Excel, PowerPoint, Photoshop}
    -- คุณสมบัติของผู้สมัคร B = {Excel, Photoshop}
    -- หาว่าผู้สมัครขาดคุณสมบัติใด A left outer join B where B.skill is NULL = {Word, PowerPoint}
    -- สร้างตารางเก็บคุณสมบัติที่ต้องการของงานในตำแหน่งหน้าที่
    CREATE TABLE [dbo].[ReqSkills](
        [skill] [nvarchar](50) NULL
    )
    GO

    insert into ReqSkills values
        ('Word'),
        ('Excel'),
        ('PowerPoint'),
        ('Photoshop')
    go

    -- สร้างตารางเก็บคุณสมบัติของผู้สมัคร
    CREATE TABLE [dbo].[ApplicantSkills](
        [skill] [nvarchar](50) NULL
    )
    GO
    insert into ApplicantSkills values
        ('Word'),
        ('PowerPoint')
    go

    select *
        from ReqSkills a left outer join ApplicantSkills b
            on a.skill = b.skill

    select *
        from ReqSkills a left outer join ApplicantSkills b
            on a.skill = b.skill
        where b.skill is null
    ```

## Cross Join
    ``` bash
    /*
    CROSS JOIN เป็นการเชื่อมตารางสองตารางเข้าด้วยกันโดยข้อมูลทุกแถวของตารางแรกมาเชื่อมกับข้อมูลทุกแถวในตารางที่สอง 
    หรืออาจมองว่าเป็นการจับคู่กันของทุกแถวของตารางทั้งสองก็ได้ (ในกรณีไม่มีการระบุเงื่อนไขภายใต้ WHERE) 
    ซึ่งการทำ cross join นั้นผลที่ได้จะอยู่ในรูปของผลคูณคาร์ทีเซียน (Cartesian product) 
    ของแถวที่เกิดจากตารางทั้งสอง 
    ดังนั้นหากตารางแรกมี 4 แถว และตารางที่สองมี 13 แถว การทำ cross join 
    จะได้ผลลัพธ์ทั้งสิ้น 4 x 13 = 52 แถว

    การประยุกต์ใช้ cross join นั้นจะเหมาะสำหรับปัญหาในลักษณะที่ต้องการนำค่าจากตาราง 2 ตารางมาใช้ร่วมกัน 
    แต่ตารางทั้ง 2 นั้นไม่มีคอลัมน์ร่วมที่จะใช้เชื่อมกันได้
    */

    select *
        from Ranks cross join Suits

    select *
        into Deck
        from Ranks cross join Suits

    -- หารายได้แยกตามเดือนของปี 2002
    select month(intime) as [month], 
            sum(Netpay) as revenue
        from OrderHdr
        where year(intime) = 2002
        group by month(intime)
        order by [month]

    -- ตาราง OrderHdr และ ตาราง Parameter ไม่มี common column เพื่อทำการ inner/outer join
    -- ตาราง parameter ใช้สำหรับเก็บค่าที่ใช้ร่วมกันในหลายตาราง เช่น ขนาดพื้นที่ร้าน อัตราภาษีมูลค่าเพิ่ม ที่ตั้งร้าน เบอร์โทรศัพท์ ฯลฯ
    -- หารายได้เฉลี่ยต่อพื้นที่ในหนึ่งเดือน
    select month(intime) as [month], 
            sum(Netpay) revenue,
            b.Area,
            sum(Netpay) / b.Area [revenue per sq.m]
        from OrderHdr a cross join Parameter b
        where year(intime) = 2002
        group by month(intime), b.Area
        order by [month]
    ```

## Union operator
    ``` bash
    select a.MenuID, b.DescrTH
        from OrderMenuDtl a inner join Menu b on a.MenuID = b.MenuID
        where OrderId =1
    union
    select a.MenuID, b.DescrTH
        from OrderMenuDtl a inner join Menu b on a.MenuID = b.MenuID
        where OrderId =2
    union
    select a.MenuID, b.DescrTH
        from OrderMenuDtl a inner join Menu b on a.MenuID = b.MenuID
        where OrderId =3

    select N'วันเสาร์อาทิตย์', sum(NetPay)
        from OrderHdr
        where datepart(weekday, intime) in (1, 7)
    union
    select N'วันธรรมดา', sum(NetPay)
        from OrderHdr
        where datepart(weekday, intime) not in (1, 7)
    union
    select N'รวม', sum(NetPay)
        from OrderHdr

    select N'1. วันเสาร์อาทิตย์' as [period], sum(NetPay)
        from OrderHdr
        where datepart(weekday, intime) in (1, 7)
    union
    select N'2. วันธรรมดา' as [period], sum(NetPay)
        from OrderHdr
        where datepart(weekday, intime) not in (1, 7)
    union
    select N'3. รวม' as [period], sum(NetPay)
        from OrderHdr
        order by period
    ```

## Intersect and Except operators
    ``` bash
    -- intersect
    -- เมนูที่ลูกค้าโต๊ะที่ (OrderID) 1, 2 และ 3 สั่งเหมือนกัน
    select a.MenuID, b.DescrTH
        from OrderMenuDtl a inner join menu b on a.MenuID = b.MenuID
        where OrderID = 1
    intersect
    select a.MenuID, b.DescrTH
        from OrderMenuDtl a inner join menu b on a.MenuID = b.MenuID
        where OrderID = 2
    intersect
    select a.MenuID, b.DescrTH
        from OrderMenuDtl a inner join menu b on a.MenuID = b.MenuID
        where OrderID = 3


    -- except
    -- ลูกค้าที่ไม่ได้เข้ามาใช้บริการในช่วงวันที่ 2002-1-1 ถึง 2002-1-7
    select CustomerID, FName
        from Customer
    except
    select a.CustomerID, b.FName
        from OrderHdr a inner join Customer b on a.CustomerID = b.CustomerID
        where convert(date, InTime) between '2002-1-1' and '2002-1-7'

    -- เมนูที่ไม่มีลูกค้ารายใดสั่งเลยในวันที่ 2002-1-1
    select MenuID, DescrTH
        from Menu
    except
    select b.MenuID, c.DescrTH
        from OrderHdr a inner join OrderMenuDtl b on a.OrderID = b.OrderID inner join
        Menu c on b.MenuID = c.MenuID
        where convert(date, intime) = '2002-1-1'
    ```

## DISTINCT
    ``` bash
    ;with cte as (
        select distinct publisher
            from games2016
    )
    select rank() over (order by publisher) as pubid, publisher
        from cte
    ```

## View
    ``` bash
    use Yummi2012
    go

    -- เลือกแถวข้อมูลตามเงื่อนไข
    -- จำกัดการเข้าถึงข้อมูลบางคอลัมน์ เช่น ต้นทุน
    create view vwMenuList
    as
    select MenuID, a.Descr, a.DescrTH, a.Price, b.Descr as [CategoryName]
        from menu a inner join Category b
            on a.CategoryID = b.CategoryID
    go

    -- ไม่ควรมี Order by ใน view
    -- หากบางคนต้องการเรียงตาม CategoryName ส่วนบางคนต้องการเรียงตาม Price หรือตามชื่อ Menu (Descr)
    select * from vwMenuList order by price
    select * from vwMenuList order by CategoryName, DescrTh, price

    create view vwRevenueByMonth
    as
    select year(intime) as year, month(intime) as month, sum(NetPay) as revenue
        from OrderHdr
        group by year(intime), month(intime)
    go

    select * from vwRevenueByMonth order by year, month

    select a.month, 
            a.revenue as [2002], 
            b.revenue as [2003], 
            b.revenue - a.revenue as [2003 - 2002], 
            (b.revenue - a.revenue) / a.revenue as [% change]
        from vwRevenueByMonth a 
            inner join vwRevenueByMonth b
            on a.month = b.month
        where a.year = 2002 and b.year = 2003
        order by month

    -- ใช้ฟังก์ชัน FORMAT() 
    select a.month, 
            format(a.revenue, 'n0') as [2002], 
            format(b.revenue, 'n2') as [2003], 
            b.revenue - a.revenue as [2003 - 2002], 
            format((b.revenue - a.revenue) / a.revenue, 'p2') as [% change]
        from vwRevenueByMonth a 
            inner join vwRevenueByMonth b
            on a.month = b.month
        where a.year = 2002 and b.year = 2003
        order by month
    ```

## Insert
    ``` bash
    select * from DineType

    insert into DineType values ('DT', 'Drive-thru')

    insert into DineType (DineTypeID, Descr) values ('CT', 'Catering')

    insert into DineType (DineTypeID) values ('XX')

    insert into DineType (DineTypeID, Descr) values ('TA', 'Type A'), ('TB', 'Type B')

    -- สร้างตาราง MonthlyRevenue จากข้อมูลของปี 2002
    select year(intime) as year, month(intime) as month, sum(NetPay) as revenue
        into MonthlyRevenue
        from OrderHdr
        where year(intime) = 2002
        group by year(intime), month(intime)

    select * from MonthlyRevenue

    insert into MonthlyRevenue(year, month, revenue)
        select year(intime) as year, month(intime) as month, sum(NetPay) as revenue
        from OrderHdr
        where year(intime) = 2003
        group by year(intime), month(intime)
    ```

## SELECT ... INTO and INSERT INTO SELECT
    ใช้ SELECT ... INTO เมื่อต้องการนำผลลัพธ์ที่ได้ไปสร้างเป็นตารางใหม่
    ใช้ INSERT INTO SELECT เพื่อนำผลลัพธ์ที่ได้จากคำสั่ง SELECT ไปเพิ่มในตารางที่มีก่อนหน้า

## Update
    ``` bash
    select * from menu
    a
    update menu
        set Descr = 'Special Red Wine'
        where Descr = 'Red Wine'

    update menu
        set DescrTH = N'ไวน์แดงพิเศษ',
            Price = 120
        where MenuID = '310S8'

    update Menu
        set price = price * 1.1

    update Menu
        set price = price / 1.1

    update Menu
        set price = price * 1.1
        where CategoryID = 'AP'

    update Menu
        set price = price * 1.2
        where CategoryID = 'BE'

    update Menu
        set price = price * 1.3
        where CategoryID = 'DI'

    update Menu
        set price = price *
            case
                when CategoryID = 'AP' then 1.1
                when CategoryID = 'BE' then 1.2
                when CategoryID = 'DI' then 1.3
                else 1
            end

    select * from OrderHdr
    select * from OrderMenuDtl

    select a.OrderID, sum(a.Qty * b.Price) as Total
        from OrderMenuDtl a
            inner join Menu b
                on a.MenuID = b.MenuID
        group by a.OrderID

    update OrderHdr
        set Total = 0

    create view vwTotalByOrder
    as
        select a.OrderID, sum(a.Qty * b.Price) as Total
            from OrderMenuDtl a
                inner join Menu b
                    on a.MenuID = b.MenuID
            group by a.OrderID
    go

    select * from vwTotalByOrder

    update a
        set a.Total = b.Total
        from OrderHdr a
            inner join vwTotalByOrder b
                on a.OrderID = b.OrderID
    ```

## Delete
    ``` bash
    select * 
    into TestCategory
    from Category

    select * from TestCategory

    delete TestCategory
        where CategoryID = 'AP'

    delete TestCategory

    -- REFERENCE constraint ระหว่าง OrderHdr กับ OrderMenuDtl
    select * from OrderHdr
        where OrderID in (1, 2, 3)

    select * from OrderMenuDtl
        where OrderID in (1, 2, 3)

    delete OrderMenuDtl
        where OrderID in (1, 2, 3)

    delete OrderHdr
        where OrderID in (1, 2, 3)

    select * from OrderHdr
        where OrderID in (1, 2, 3)

    select * from OrderMenuDtl
        where OrderID in (1, 2, 3)

    select * from OrderHdr
        where OrderID in (4, 5, 6)

    select * from OrderMenuDtl
        where OrderID in (4, 5, 6)

    delete OrderHdr
        where OrderID in (4, 5, 6)

    delete OrderMenuDtl
        where OrderID in (4, 5, 6)

    sp_changedbowner 'sa'

    -- แสดงรายการที่ลูค้าเป็นชาวอังกฤษ
    select OrderHdr.*
        FROM Customer INNER JOIN OrderHdr
        ON Customer.CustomerID = OrderHdr.CustomerID
        WHERE CountryID = 'UK'

    delete OrderHdr
        FROM Customer INNER JOIN OrderHdr
        ON Customer.CustomerID = OrderHdr.CustomerID
        WHERE CountryID = 'UK'
    ```

## Subquery
    ``` bash
    select * from menu

    select avg(price) from menu

    select * from menu
        where price > 93.421

    select * from menu
        where price > avg(price)

    select * from menu
        where price > (select avg(price) from menu)

    select MenuID, DescrTH, Price, (select avg(price) from menu) as [avg. price] from menu
        where price > (select avg(price) from menu)
    -- 2) หายอดขายแยกตามเดือนของปี 2002 พร้อมกับแสดงว่ายอดขายแต่ละเดือนเป็นกี่เปอร์เซ็นต์ของยอดขายทั้งปี
    select month(intime) month, 
            sum(NetPay) as revenue
        from OrderHdr
        where year(intime) = 2002
        group by month(intime)
        order by month

    select sum(NetPay)
        from OrderHdr
        where year(intime) = 2002

    select month(intime) month, 
            sum(NetPay) as revenue,
            100.0 * sum(NetPay) / (select sum(NetPay)
                                from OrderHdr
                                where year(intime) = 2002
                            ) as [% total]
        from OrderHdr
        where year(intime) = 2002
        group by month(intime)
        order by month

    -- 3) หายอดขายแยกตามเดือนและประเภทการรับประทานในปี 2002
    select month(intime) month, 
            sum(NetPay) as Revenue
        from OrderHdr
        where year(intime) = 2002 and DineTypeID = 'EI'
        group by month(intime)
        order by month

    select month(intime) month, 
            sum(NetPay) as Revenue
        from OrderHdr
        where year(intime) = 2002 and DineTypeID = 'TG'
        group by month(intime)
        order by month

    select ei.month, ei.Revenue as [Eat-in], tg.Revenue as [To go]
        from 
        (
            select month(intime) month, 
                    sum(NetPay) as Revenue
                from OrderHdr
                where year(intime) = 2002 and DineTypeID = 'EI'
                group by month(intime)
        ) ei
        inner join
        (
            select month(intime) month, 
                    sum(NetPay) as Revenue
                from OrderHdr
                where year(intime) = 2002 and DineTypeID = 'TG'
                group by month(intime)
        ) tg
        on ei.month = tg.month
        order by ei.month
    ```

## CASE...WHEN...THEN...END
    ``` bash
    -- CASE ... END 
    -- http://technet.microsoft.com/en-us/library/ms181765.aspx
    /* --------------------------------------------------------------- */
    -- Q: แสดงคำนำหน้าชื่อ
    -- Syntax 1: [column name] = case ... end 
    select * from Customer

    select CustomerID, FName, LName, Gender,
            title= case Gender
                        when 'F' then 'Ms.'
                        when 'M' then 'Mr.'
                    end
        from Customer

    select CustomerID, FName, LName, Gender,
            case 
                when Gender = 'F' then 'Ms.'
                when Gender = 'M' then 'Mr.'
            end as title
        from Customer

    /* --------------------------------------------------------------- */
    -- Q: แยกอาหารออกเป็น ปลา ผัก และอื่นๆ 
    select * from menu

    select MenuID, DescrTH,
        case
            when DescrTH like N'%ปลา%' then N'ปลา'
            when DescrTH like N'%ผัก%' then N'ผัก'
            else N'อื่นๆ'
        end as [menu type]
        from menu



    /* --------------------------------------------------------------- */
    -- Q: แบ่งตามมื้ออาหาร
    select * from OrderHdr

    select intime,
            case
                when datepart(hour, intime) between 6 and 10 then 'breakfast'
                when datepart(hour, intime) between 11 and 14 then 'lunch'
                when datepart(hour, intime) between 17 and 24 then 'dinner'
            end as [dine_time]
        from OrderHdr

    select 
            case
                when datepart(hour, intime) between 6 and 10 then 'breakfast'
                when datepart(hour, intime) between 11 and 14 then 'lunch'
                when datepart(hour, intime) between 17 and 24 then 'dinner'
            end as [dine_time],
            count(orderid) as [# orders]
        from OrderHdr
        group by 
            case
                when datepart(hour, intime) between 6 and 10 then 'breakfast'
                when datepart(hour, intime) between 11 and 14 then 'lunch'
                when datepart(hour, intime) between 17 and 24 then 'dinner'
            end
    union
    select 'total', count(orderid) from OrderHdr


    select 
            case
                when datepart(hour, intime) between 6 and 10 then 'breakfast'
                when datepart(hour, intime) between 11 and 14 then 'lunch'
                when datepart(hour, intime) between 17 and 24 then 'dinner'
            end as [dine_time],
            count(orderid) as [# orders]
        from OrderHdr
        where
            case
                when datepart(hour, intime) between 6 and 10 then 'breakfast'
                when datepart(hour, intime) between 11 and 14 then 'lunch'
                when datepart(hour, intime) between 17 and 24 then 'dinner'
            end in ('breakfast', 'dinner')
        group by 
            case
                when datepart(hour, intime) between 6 and 10 then 'breakfast'
                when datepart(hour, intime) between 11 and 14 then 'lunch'
                when datepart(hour, intime) between 17 and 24 then 'dinner'
            end
    ```

## Common Table Expression (CTE)
    ``` bash
    -- Common Table Expression (CTE) ทำหน้าที่เสมือนวิวชั่วคราว เพื่อให้ชุดคำสั่งสืบค้นนำค่าไปใช้

    /* ---------------------------------------------------------------------------- */
    -- Q: ยอดขายแยกตามเดือนและปี	
    select * from OrderHdr

    select year(intime), month(intime), sum(NetPay)
        from OrderHdr
        group by year(intime), month(intime)

    ;with cte as
    (
        select year(intime) year, month(intime) month, sum(NetPay) revenue
            from OrderHdr
            group by year(intime), month(intime)
    )
    select * from cte

    ;with cte(year, month, revenue) as
    (
        select year(intime), month(intime), sum(NetPay)
            from OrderHdr
            group by year(intime), month(intime)
    )
    select * from cte

    /* ---------------------------------------------------------------------------- */
    -- Q: แสดงยอดขายแยกตามวันธรรมดา เสาร์อาทิตย์ ของแต่ละปี
    -- multiple CTEs
    ;with AllDays as
    (
        select  year(intime) year, 'All days' as period,
                datepart(weekday, intime) as weekday,
                sum(Netpay) as revenue
            from OrderHdr
            group by year(intime), datepart(weekday, intime)
    ),
    MonToFri as (
        select year, 'Mon-Fri' as period, sum(revenue) as revenue
            from AllDays
            where weekday in (2, 3, 4, 5, 6)
            group by year
    ),
    SatSun as (
        select year, 'Sat-Sun' as period, sum(revenue) as revenue
            from AllDays
            where weekday in (1, 7)
            group by year
    )
    select year, period, revenue from MonToFri
    union
    select year, period, revenue from SatSun
    union
    select year, period, sum(revenue) from AllDays
        group by year, period
    order by year, period
    ```

## Pivot (or Crosstab style output)
    ``` bash
    -- pivot table (crosstab style)
    -- http://technet.microsoft.com/en-us/library/ms177410%28v=sql.105%29.aspx

    -- 1.1) pivot by year
    select year(intime) year, month(intime) month, sum(NetPay) revenue
        from OrderHdr
        group by year(intime), month(intime)
        order by year, month

    SELECT  Month, [2002], [2003], [2003] - [2002] as [diff 2003-2002]
        FROM  (
                select year(intime) year, month(intime) month, sum(NetPay) revenue
                    from OrderHdr
                    group by year(intime), month(intime)
            ) d
        PIVOT (
                SUM(Revenue) 
                FOR Year IN ([2002], [2003])
            ) AS pvt
        ORDER BY month

    -- Add column [2003] - [2002] AS [Revenue (2003-2002)]
    SELECT  Month, [2002], [2003], [2003] - [2002] AS [Revenue (2003-2002)]
        FROM  (
                select year(intime) year, month(intime) month, sum(NetPay) revenue
                    from OrderHdr
                    group by year(intime), month(intime)
            ) d
        PIVOT (
                SUM(Revenue) 
                FOR Year IN ([2002], [2003])
            ) AS pvt
        ORDER BY month

    -- 1.2) pivot by month
    SELECT  Year, [1], [2], [3], [4], [5], [6]
        FROM  (
                select year(intime) year, month(intime) month, sum(NetPay) revenue
                    from OrderHdr
                    where year(intime) in (2003, 2002)
                    group by year(intime), month(intime)
            ) d
        PIVOT (
                SUM(Revenue) 
                FOR month IN ([1], [2], [3], [4], [5], [6])
            ) AS pvt
        ORDER BY year

    -- 1.3) pivot by quarter
    select year(intime) year, datepart(q, intime) quarter, sum(NetPay) revenue
        from OrderHdr
        group by year(intime), datepart(q, intime)

    SELECT  Year, [1] as [Q1], [2] as [Q2], [3] as [Q3], [4] as [Q4], [1] + [2] + [3] + [4] as [total]
        FROM  (
                select year(intime) year, datepart(q, intime) quarter, sum(NetPay) revenue
                    from OrderHdr
                    where year(intime) in (2002, 2003)
                    group by year(intime), datepart(q, intime)

            ) d
        PIVOT (
                SUM(Revenue) 
                FOR quarter IN ([1], [2], [3], [4])
            ) AS pvt
        ORDER BY year


    -- 2) add DineTypeID
    select year(intime) year, month(intime) month, DineTypeID, sum(NetPay) revenue
        from OrderHdr
        group by year(intime), month(intime), DineTypeID

    -- Pivot by Year
    SELECT  Month, DineTypeID, [2002], [2003], [2003] - [2002] AS [Revenue (2003-2002)]
        FROM  (
                select year(intime) year, month(intime) month, DineTypeID, sum(NetPay) revenue
                    from OrderHdr
                    group by year(intime), month(intime), DineTypeID
            ) d
        PIVOT (
                SUM(Revenue) 
                FOR Year IN ([2002], [2003])
            ) AS pvt
        ORDER BY DineTypeID, month

    -- pivot by DineTypeID
    SELECT  Year, Month, [EI] as [Eat in], [TG] as [To go], [EI] + [TG] as [total]
        FROM  (
                select year(intime) year, month(intime) month, DineTypeID, sum(NetPay) revenue
                    from OrderHdr
                    group by year(intime), month(intime), DineTypeID
            ) d
        PIVOT (
                SUM(Revenue) 
                FOR DineTypeID IN ([EI], [TG])
            ) AS pvt
        ORDER BY year, month
    ```

## unpivot
    ``` bash
    select uid, name, region, sales
        from gamesales
        unpivot (
        sales for region in (na, eu, jp, other)
    ) u
    ```

## Grouping Sets (>= SQL Server 2008)
    ``` bash
    /*
    Grouping sets (SQL 2008+)
    หาผลรวมของคอลัมน์ตาม group ที่กำหนด
    */

    /* ---------------------------------------------------------- */
    select categoryid, count(*) as [# menu]
        from menu
        group by categoryid
    union
    select 'total', count(*) as [# menu]
        from menu
        
    select categoryid, count(*) as [# menu]
        from menu
        group by 
            grouping sets (
                (categoryid), ()
            )
        
    /* ---------------------------------------------------------- */
    -- Q: หายอดขายแยกตามปี พร้อมผลรวม
    select year(intime) year, sum(netpay) revenue
        from OrderHdr
        group by year(intime)
        order by year

    select year(intime) year, sum(netpay) revenue
        from OrderHdr
        group by 
            grouping sets (
                year(intime),
                ()
            )
        order by year

    select year(intime) year, 
            month(intime) month,
            sum(netpay) revenue
        from OrderHdr
        group by 
            grouping sets (
                year(intime),
                month(intime),
                (year(intime), month(intime)),
                ()
            )

    select year(intime) year, 
            month(intime) month,
            DineTypeID,
            sum(netpay) revenue
        from OrderHdr
        group by 
            grouping sets (
                year(intime),
                (year(intime), month(intime)),
                (year(intime), month(intime), DineTypeID),
                ()
            )
    ```

##  Ranking functions
    ``` bash
    -- ranking functions (WINDOW FUNCTIONS)
    -- ฟังก์ชันสำหรับสร้างคอลัมน์สำหรับแสดงลำดับของข้อมูล
    -- row_number()
    -- rank()
    -- dense_rank()
    -- ntile()

    SELECT     MenuID, DescrTH, CategoryID, Price,
            row_number() over (order by categoryid, price) as [row #]
        FROM   Menu

    SELECT     MenuID, DescrTH, CategoryID, Price,
            row_number() over (order by categoryid, price) as [row 1],
            row_number() over (
                                partition by categoryid
                                order by categoryid, price
                            ) as [row 2]
        FROM   Menu

    SELECT     MenuID, DescrTH, CategoryID, Price,
            rank() over (order by price desc) as [rank #],
            dense_rank() over (order by price desc) as [dense rank]
        FROM   Menu

    SELECT     MenuID, DescrTH, CategoryID, Price,
            rank() over (order by categoryid, price desc) as [rank no partition],
            rank() over (partition by categoryid order by price desc) as [rank with partition]
        FROM   Menu

    -- ntile()
    SELECT     MenuID, DescrTH, CategoryID, Price,
            ntile(4) over (order by price desc) as quartile,
            rank() over (order by price desc) as [rank no partition]
        FROM   Menu
    ```

## Format() function (>= SQL Server 2012)
    ``` bash
    -- ใช้ FORMAT() SQL 2012+
    -- ดูรายละเอียด Standard numeric format string 
    -- http://msdn.microsoft.com/library/dwhawy9k.aspx
    -- http://msdn.microsoft.com/en-us/library/0c899ak8.aspx

    -- culture code
    -- http://msdn.microsoft.com/en-US/library/ee825488(v=cs.20).aspx

    -- Date/Time format
    -- http://msdn.microsoft.com/en-us/library/8kb3ddd4(v=vs.110).aspx

    select year(intime) year, month(intime) month, 
            format(intime, 'MMM', 'th') [month th],
            sum(netpay) as revenue,
            format(sum(netpay), 'C', 'th') as [currency fmt]
        from OrderHdr
        group by year(intime), month(intime), format(intime, 'MMM', 'th')
        order by year, month

    select
        1000.45 as [raw],
        format(1000.45, 'G') as [general],
        format(1000.45, 'G', 'de') as [German-general],
        format(1000.45, 'C', 'de') as [German-currency],
        format(1000.45, 'C3', 'th') as [Thai-currency]

    select
        12345678.45 as [raw],
        format(12345678.45, 'n0') as [n0],
        format(12345678.45, 'n2') as [n2],
        format(12345678.45, '#,##0.00') as [#,##0.00],
        format(12345678.45, N'#,##0,พัน') as [#,##0,k],
        format(12345678.45, '#,##0,,m') as [#,##0,,m],
        format(12345678.45, '#,##0,,mil') as [#,##0,,mil],
        format(12345678.45, '#,##0,,.00m') as [#,##0,,.00m],
        format(12345678.45, N'#,##0,,.00ล้าน') as [#,##0,,.00ล้าน]

    -- with Currency code
    select
        12345678.45 as [raw],
        format(12345678.45, 'C') as [system default],
        format(12345678.45, 'C', 'th') as [Thai],
        format(12345678.45, 'C', 'ja') as [Japan],
        format(12345678.45, 'C', 'de') as [German],
        format(12345678.45, 'C', 'en-GB') as [Great Britain],
        format(12345678.45, 'C', 'en-US') as [US]

    -- 0 vs #
    select
        format(.45, '#.00') as [#.00],
        format(.45, '0.00') as [0.00],
        format(21, '0000') as [0000],
        format(1, '####') as [####]

    -- percent
    select
        format(.1234, 'p0') as [p0],
        format(.1234, 'p1') as [p1],
        format(.1234, 'p2') as [p2],
        format(.1234, '0.00%') as [0.00%],
        format(.0034, '0.00%') as [0.00%],
        format(.0034, '#.00%') as [#.00%]

    -- date
    select
        format(getdate(), 'D') as [D],
        format(getdate(), 'd') as [d],
        format(getdate(), 'ddd') as [ddd],
        format(getdate(), 'MMM') as [MMM],
        format(getdate(), 'dd-M-yy') as [dd-M-yy],
        format(getdate(), 'dd-MM-yy') as [dd-MM-yy],
        format(getdate(), 'dd MMM yy') as [dd MMM yy],
        format(getdate(), 'ddd dd-MMM-yyyy') as [ddd dd-MMM-yyyy],
        format(getdate(), 'dddd dd MMMM yyyy') as [dddd dd MMMM yyyy]

    -- date with culture/sub-culture
    select
        format(getdate(), 'D', 'en-GB') as [D],
        format(getdate(), 'D', 'en-US') as [D],
        format(getdate(), 'd', 'en-US') as [d],
        format(getdate(), 'ddd', 'en-US') as [ddd],
        format(getdate(), 'dddd', 'en-US') as [dddd],
        format(getdate(), 'MMM', 'en-US') as [MMM],
        format(getdate(), 'MMMM', 'en-US') as [MMMM],
        format(getdate(), 'dd-M-yy', 'en-US') as [dd-M-yy],
        format(getdate(), 'dd-MM-yy', 'en-US') as [dd-MM-yy],
        format(getdate(), 'dd MMM yy', 'en-US') as [dd MMM yy],
        format(getdate(), 'ddd dd-MMM-yyyy', 'en-US') as [ddd dd-MMM-yyyy],
        format(getdate(), 'dddd dd MMMM yyyy', 'en-US') as [dddd dd MMMM yyyy]

    -- time
    select
        format(cast('2014-2-6 19:20' as datetime), 'dd-MMM-yyyy HH:mm') as [dd-MMM-yyyy HH:mm],
        format(cast('2014-2-6 19:20' as datetime), 'hh:mm') as [hh:mm],
        format(cast('2014-2-6 19:20' as datetime), 'hh:mm tt') as [AM/PM],
        format(cast('2014-2-6 19:20' as datetime), 'HH:mm') as [HH:mm 24-hour],
        format(cast('2014-2-6 19:20' as datetime), 'HH:mm zz') as [HH:mm zz], -- UTC (Universal Time Coordinated) ว่าห่างจาก Greenwich Mean Time (GMT) กี่ชั่วโมง
        format(cast('2014-2-6 19:20' as datetime), 'HH:mm zzz') as [HH:mm zzz] -- UTC (Universal Time Coordinated) ว่าห่างจาก Greenwich Mean Time (GMT) กี่ชั่วโมง กี่นาที

    -- date with thai culture
    select
        format(getdate(), 'D', 'th') as [D],
        format(getdate(), 'd', 'th') as [d],
        format(getdate(), 'ddd', 'th') as [ddd],
        format(getdate(), 'dddd', 'th') as [dddd],
        format(getdate(), 'MMM', 'th') as [MMM],
        format(getdate(), 'MMMM', 'th') as [MMMM],
        format(getdate(), 'dd-M-yy', 'th') as [dd-M-yy],
        format(getdate(), 'dd-MM-yy', 'th') as [dd-MM-yy],
        format(getdate(), 'dd MMM yy', 'th') as [dd MMM yy],
        format(getdate(), 'ddd dd-MMM-yyyy', 'th') as [ddd dd-MMM-yyyy],
        format(getdate(), 'dddd dd MMMM yyyy', 'th') as [dddd dd MMMM yyyy]

    -- date with Japanese culture
    select
        format(getdate(), 'D', 'ja') as [D],
        format(getdate(), 'd', 'ja') as [d],
        format(getdate(), 'ddd', 'ja') as [ddd],
        format(getdate(), 'MMM', 'ja') as [MMM],
        format(getdate(), 'dd-M-yy', 'ja') as [dd-M-yy],
        format(getdate(), 'dd-MM-yy', 'ja') as [dd-MM-yy],
        format(getdate(), 'dd MMM yy', 'ja') as [dd MMM yy],
        format(getdate(), 'ddd dd-MMM-yyyy', 'ja') as [ddd dd-MMM-yyyy],
        format(getdate(), 'dddd dd MMMM yyyy', 'ja') as [dddd dd MMMM yyyy]
    ```

## Simple, Stratified and Cluster Sampling
    ``` bash
    -- การสุ่มแถวจากตาราง
    select *
        from OrderHdr

    select *
        from OrderHdr
        tablesample (100 rows)

    select top 50 *
        from OrderHdr
        tablesample (100 rows)

    select *
        from OrderHdr
        tablesample (10 percent)

    -- newid() ส่งค่ากลับมาเป็น unique identifier ขนาด 16 bytes (2 ^ 128 ~ 3 ตามด้วย 0 อีก 38 ตัว)
    select newid()

    select OrderID, newid()
        from OrderHdr

    -- 1) simple random sampling
    -- สุ่มเลือก 10 แถว
    select top (5) percent OrderID
        from OrderHdr
        order by newid()

    -- 2) stratified sampling
    -- แบ่งเป็น quartile แยกตาม NetPay
    -- row_number() partition by ntile order by newid
    -- select top (n) [percent]
    ;with cte as
    (
        select OrderID,
                ntile(4) 
                    over (
                            order by NetPay desc
                        ) quartile
            from OrderHdr
    ),
    cte2 as
    (
        select OrderID, quartile,
                row_number() 
                    over (
                            partition by quartile 
                            order by newid()
                        ) as [row #]
        from cte
    )
    select top (10) percent * -- เลือก 10% ของแถวจากแต่ละ quartile 
        from cte2
        order by [row #]



    -- 3) Cluster sampling (มักจะแบ่งกลุ่ม (cluster) ด้วยเวลา เช่น เดือน หรือที่ตั้ง เช่น จังหวัด)
    -- เช่น สุ่ม 5 แถวของข้อมูลจากแต่ละเดือนในแต่ละปี
    ;with cte as
    (
        select OrderID, intime,
                row_number() 
                    over (
                            partition by year(intime), month(intime)
                            order by newid()
                        ) [row #]
            from OrderHdr
    )
    select * 
        from cte
        where [row #] <= 5
    ```

## Lead and Lag
    ``` bash
    -- LEAD and LAG
    -- แถวก่อนหน้า n แถว (lag)
    -- แถวถัดไป n แถว (lead)

    -- version 1 (DEFAULT lead, lag)
    ;with cte as (
        select year(intime) year, month(intime) month,
                sum(netpay) AS [revenue]
            from OrderHdr
            group by year(intime), month(intime)
    )
    select Year, Month, revenue, 
            lag(revenue) -- แถวก่อนหน้า 1 แถว
                over (
                        partition by Year 
                        order by month
                    ) as [prev_lag],
            lead(revenue) -- แถวถัดไป 1 แถว
                over (
                        partition by Year 
                        order by month
                    ) as [next_lead]
        from cte
        ORDER BY Year, Month

    /* ------------------------------------------------------------- */
    -- คำนวณการเปลี่ยนแปลงยอดขายเทียบกับเดือนก่อนหน้า
    ;with cte as (
        select year(intime) year, month(intime) month,
                sum(netpay) AS [revenue]
            from OrderHdr
            group by year(intime), month(intime)
    ),
    cte2 as (
    select Year, Month, revenue, 
        lag(revenue) -- แถวก่อนหน้า 1 แถว
            over (
                    partition by Year 
                    order by month
                ) as [prev_lag], 
        lead(revenue) -- แถวถัดไป 1 แถว
            over (
                    partition by Year 
                    order by month
                ) as [next_lead] 
    from cte
    )
    -- select * from cte2
    select Year, Month, revenue, [prev_lag],
            revenue - [prev_lag] as [this - prev],
            (revenue - [prev_lag]) / [prev_lag] as [% chg]
        from cte2
        ORDER BY Year, Month

    /* ------------------------------------------------------------- */
    -- version 2 (REPLACE NULL values returned from LEAD, LAG with a specific value)
    ;with cte as (
        select year(intime) year, month(intime) month,
                sum(netpay) AS [revenue]
            from OrderHdr
            group by year(intime), month(intime)
    )
    select Year, Month, revenue, 
            lag(revenue, 1, 0) -- 1 หมายถึง ก่อนหน้า 1 แถว, 0 หมายถึง หากมีค่าเป็น NULL ให้แทนที่ด้วย 0
                    over (
                            partition by Year 
                            order by Month
                        ) as [prev_lag], 
            lead(revenue, 1, 0) -- 1 หมายถึง ถัดไป 1 แถว, 0 หมายถึง หากมีค่าเป็น NULL ให้แทนที่ด้วย 0
                    over (
                            partition by Year 
                            order by Month
                        ) as [next_lead] 
        from cte
        ORDER BY Year, Month
    ```

## RUNNING TOTAL column (Cumulative sum)
    ``` bash
    -- ใช้ SUM() ในการหาผลรวมสะสม Cumulative/Running total 
    ;with cte as (
        select year(intime) year, month(intime) month,
                sum(netpay) as [revenue]
            from OrderHdr
            group by year(intime), month(intime)
    )
    select year, month, revenue as [month-revenue],
        sum(revenue) over (
                            partition by year 
                            order by month
                        ) as [running total by year], -- สังเกตว่าเมื่อมีการเพิ่ม order by เข้าไป จะทำให้เกิด running total
        sum(revenue) over (
                            order by year, month
                        ) as [running total], -- สังเกตว่าเมื่อมีการเพิ่ม order by เข้าไป จะทำให้เกิด running total
        sum(revenue) over (partition by year) [year-revenue],
        sum(revenue) over () [grand total revenue] -- ไม่มีการระบุ partition จะเป็นการ sum() ทุกแถว
        from cte 
        order by year, month
    ```

## Moving Average
    ``` bash
    -- ค่าเฉลี่ยเคลื่อนที่ (moving average)
    -- ใช้ AVG() ในการหา Moving Average
    -- ใช้ COUNT() ในการนับจำนวนแถวแยกตาม partition

    -- แสดงยอดขายแยกตามเดือนของแต่ละปี
    select year(intime) year, month(intime) month,
            sum(netpay) as [revenue]
        from OrderHdr
        group by year(intime), month(intime)
        order by year, month

    -- 3-month moving average
    -- Frame จะเป็นการกำหนดส่วนย่อยภายใน Partition เพื่อระบุว่าต้องการใช้ตั้งแต่แถวในถึงแถวใดใน Partition
    -- แสดงการใช้ FRAME ROWS BETWEEN 2 PRECEDING AND CURRENT ROW -- 2 บรรทัดก่อนหน้า, 0 บรรทัดปัจจุบัน
    ;with cte as (
        select year(intime) year, month(intime) month,
                sum(netpay) as [revenue]
            from OrderHdr
            group by year(intime), month(intime)
    )
    select year, month, revenue,
        avg(revenue) 
            over (  partition by year
                    order by year, month
                    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW -- 2 บรรทัดก่อนหน้า, 0 บรรทัดปัจจุบัน
                ) as [3-month moving avg v1],
        avg(revenue) 
            over (  -- without partition by year
                    order by year, month
                    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW -- 2 บรรทัดก่อนหน้า, 0 บรรทัดปัจจุบัน
                ) as [3-month moving avg v2]
        from cte 
        order by year, month

    /* ----------------------------------------------------------------- */
    -- moving avg. โดยแสดงค่า NULL สำหรับแถวที่มี lag ไม่พอในการคำนวณ moving avg
    -- เช่น หากทำการคำนวณ 3-month moving avg. เดือนที่ 1 และ 2 จะไม่สามารถคำนวณได้ จึงแสดงค่าเป็น NULL
    ;with cte as (
        select year(intime) year, month(intime) month,
                sum(netpay) as [revenue]
            from OrderHdr
            group by year(intime), month(intime)
    ),
    cte2 as (
        select year, month, revenue,
            count(revenue) 
                over (
                        partition by year 
                        order by month
                        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW -- FRAME 2 บรรทัดก่อนหน้า, 0 บรรทัดปัจจุบัน
                    ) as [rows count],
            avg(revenue) 
                over (
                        partition by year 
                        order by month
                        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW -- 2 บรรทัดก่อนหน้า, 0 บรรทัดปัจจุบัน
                    ) as [3-month moving avg]
            from cte
    )
    select year, month, revenue, 
            [rows count], 
            [3-month moving avg],
            case
                when [rows count] >= 3 then [3-month moving avg]
                else NULL
            end as [corrected moving avg.]
        from cte2
        order by year, month
    ```

## FIRST_VALUE(), LAST_VALUE() with Window Function
    ``` bash
    -- FIRST_VALUE(), LAST_VALUE()
    -- LAST_VALUE ต้องระบุ rows between unbounded preceding and unbounded following เพื่อให้ทำงานกับช่วงที่ถูกต้อง

    select year(intime) year, month(intime) month,
            sum(netpay) AS [revenue]
        from OrderHdr
        group by year(intime), month(intime)
        order by year, month

    -- หาค่าแรกและค่าสุดท้ายตามเงื่อนไขที่กำหนดใน partition และ order
    ;with cte as (
        select year(intime) year, month(intime) month,
                sum(netpay) AS [revenue]
            from OrderHdr
            group by year(intime), month(intime)
    )
    select Year, Month, revenue, 
            first_value(revenue) -- FIRST VALUE in the current partition
                over (
                        partition by Year 
                        order by month
                        -- RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
                    ) as [first value],
            last_value(revenue) 
                over (
                        partition by Year 
                        order by month 
                        -- Default frame: RANGE UNBOUNDED PRECEDING AND CURRENT ROW
                    ) as [WRONG_last value],
            last_value(revenue) 
                over (
                        partition by Year 
                        order by month 
                        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
                    ) as [CORRECT_last value]

        from cte
        ORDER BY Year, Month

    /* ------------------------------------------------------------------------------------- */
    -- ประยุกต์ใช้ first_value
    -- หาว่ายอดขายในแต่ละเดือนเทียบกับยอดขายในเดือนแรกที่เปิดร้าน
    -- สังเกตว่าไม่มีการทำ partition by year
    ;with cte as (
        select year(intime) year, month(intime) month,
                sum(netpay) AS [revenue]
            from OrderHdr
            group by year(intime), month(intime)
    )
    select Year, Month, revenue, 
            first_value(revenue) -- FIRST VALUE in the current partition
                over (
                        order by year, month
                    ) as [first value],
            revenue - first_value(revenue) -- ยอดขายเดือนปัจจุบันลบด้วยยอดขายของเดือนแรก
                        over (
                                order by year, month
                            ) as [current month revenue - first_value],
            format((revenue / first_value(revenue) -- ยอดขายเดือนปัจจุบันลบด้วยยอดขายของเดือนแรก
                        over (
                                order by year, month
                            )) - 1, 'p2') as [% chg.]

        from cte
        ORDER BY Year, Month
    ```

## STRING_SPLIT
    ``` bash
    select value from string_split('Action, Crime, Drama, Triller', ',')

    select movieid, value as genres
        from movie cross apply string_split(genres, '|')

    ;with cte as (
        select movieid, value as genres
            from movie cross apply string_split(genres, '|')
    )
    select genre, count(*) as [# of movies]
        from cte
        group by genre
        order by count('*') desc

    select value as genre, count(*) [# of movie]
        from movie cross apply string_split(genres, '|')
        group by value
        order by count('*') desc
    
    select movie_title, genres
        from movie
        where 'Action' in (select value from string_split(genres, '|'))
    ```

## CONCAT_WS() เพื่อเชื่อมข้อความโดยระบุตัวคั่น (separator)
    ``` bash
    select concat_ws('|', Peter, 'Parker', 24)

    select concat_ws('|', Peter, ISNULL(NULL, ''), 24) 

    select movieid, concat_ws('|', actor_1_name, actor_2_name, actor_3_name) actors
        from movie
    ```

## STRING_AGG (String Aggregation) เพื่อรวมข้อมูลในคอลัมน์เข้าด้วยกัน
    ``` bash
    select string_agg(genre, ',')
        from movie_genre

    select string_agg(genre, ',')
        from movie_genre
        where movieid = 1

    select movieid, string_agg(genre, ',') genres
        from movie_genre
        group by movieid

    select g.movieid, movie_title, string_agg(genre, ',') genres
        from movie_genre g inner join movies m on g.movieid = m.movieid
        group by movieid, movie_title
    
    ;with cte as(
        select movieid,
            string_agg(genre, ',') genres
            from movie_genre
            group by movieid
    )
    select g.movieid, movie_title, genres
        from cte as g inner join movies m on g.movieid = m.movieid

    ;with cte as(
        select movieid,
            string_agg(genre, ',') genres
            from movie_genre
            group by movieid
    )
    select string_agg(concat_ws('|', g.movieid, movie_title, genres), char(13))
        from cte as g inner join movies m on g.movieid = m.movieid
    ```

## Scalar-valued function
    ``` bash
    create function ufnRectangle(@w float, @h float) returns float as
        begin
            return @w * @h
        end

    create function ufnMax(@a float, @b float) returns float as
        begin
            declare @t float
            if @a > @b begin
                set @t = @a
            end else begin
                set @t = @b
            end
            return @t
        end

    create function country_kpi(@country nvarchar(30)) returns table as
    return(
        select * from kpi where country = @country
    )
    go
    ```
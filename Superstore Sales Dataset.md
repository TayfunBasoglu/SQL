dataset : https://www.kaggle.com/datasets/rohitsahoo/sales-forecasting

# Create Table

~~~~sql
CREATE TABLE Orders (
  RowID INT,
  OrderID VARCHAR(20),
  OrderDate DATE,
  ShipDate DATE,
  ShipMode VARCHAR(20),
  CustomerID VARCHAR(20),
  CustomerName VARCHAR(50),
  Segment VARCHAR(20),
  Country VARCHAR(50),
  City VARCHAR(50),
  State VARCHAR(50),
  PostalCode VARCHAR(10),
  Region VARCHAR(20),
  ProductID VARCHAR(20),
  Category VARCHAR(20),
  SubCategory VARCHAR(50),
  ProductName VARCHAR(500),
  Sales DECIMAL(10,2)
);
~~~~

# Import CSV

~~~~sql
COPY orders
FROM 'C:\Users\tayfu\Desktop\train.csv'
DELIMITER ','
CSV HEADER;
~~~~

Sales equals price

**Check**

~~~~sql
select * from orders limit 5
~~~~

~~~~
1	"CA-2017-152156"	"2017-11-08"	"2017-11-11"	"Second Class"	"CG-12520"	"Claire Gute"	"Consumer"	"United States"	"Henderson"	"Kentucky"	"42420"	"South"	"FUR-BO-10001798"	"Furniture"	"Bookcases"	"Bush Somerset Collection Bookcase"	261.96
2	"CA-2017-152156"	"2017-11-08"	"2017-11-11"	"Second Class"	"CG-12520"	"Claire Gute"	"Consumer"	"United States"	"Henderson"	"Kentucky"	"42420"	"South"	"FUR-CH-10000454"	"Furniture"	"Chairs"	"Hon Deluxe Fabric Upholstered Stacking Chairs, Rounded Back"	731.94
3	"CA-2017-138688"	"2017-06-12"	"2017-06-16"	"Second Class"	"DV-13045"	"Darrin Van Huff"	"Corporate"	"United States"	"Los Angeles"	"California"	"90036"	"West"	"OFF-LA-10000240"	"Office Supplies"	"Labels"	"Self-Adhesive Address Labels for Typewriters by Universal"	14.62
4	"US-2016-108966"	"2016-10-11"	"2016-10-18"	"Standard Class"	"SO-20335"	"Sean O'Donnell"	"Consumer"	"United States"	"Fort Lauderdale"	"Florida"	"33311"	"South"	"FUR-TA-10000577"	"Furniture"	"Tables"	"Bretford CR4500 Series Slim Rectangular Table"	957.58
5	"US-2016-108966"	"2016-10-11"	"2016-10-18"	"Standard Class"	"SO-20335"	"Sean O'Donnell"	"Consumer"	"United States"	"Fort Lauderdale"	"Florida"	"33311"	"South"	"OFF-ST-10000760"	"Office Supplies"	"Storage"	"Eldon Fold 'N Roll Cart System"	22.37
~~~~

# Sum Sales

~~~~sql
select sum(sales) from orders
~~~~

~~~~
2261536.97
~~~~

Sum of sales by Year

~~~~sql
select date_trunc('year',orderdate), sum(Sales) from orders group by date_trunc('year',orderdate)
~~~~

~~~~
"2017-01-01 00:00:00+03"	600192.80
"2015-01-01 00:00:00+02"	479856.27
"2016-01-01 00:00:00+02"	459435.94
"2018-01-01 00:00:00+03"	722051.96
~~~~


Average sales by year

~~~~sql
select date_trunc('year',orderdate), avg(Sales) from orders group by date_trunc('year',orderdate)
~~~~


~~~~
"2017-01-01 00:00:00+03"	236.8558800315706393
"2015-01-01 00:00:00+02"	245.7021351766513057
"2016-01-01 00:00:00+02"	223.5698004866180049
"2018-01-01 00:00:00+03"	221.6242971147943524
~~~~


# Total Best Month

~~~~sql
SELECT EXTRACT(MONTH FROM orderdate) as month, sum(sales) FROM orders group by month order by sum(sales) desc
~~~~

~~~~
11	350161.74
12	321480.21
9	300103.42
10	199496.34
3	197573.60
8	157315.85
5	154086.74
6	145837.55
7	145535.70
4	136283.04
1	94291.66
2	59371.12
~~~~


# Best Year and Month

~~~~sql
select EXTRACT(MONTH FROM orderdate) as month, EXTRACT(YEAR FROM orderdate) as year, sum(Sales) from orders group by year,month order by sum desc limit 10
~~~~

~~~~
11	2018	117938.14
12	2017	95739.15
9	2018	86152.90
12	2018	83030.38
9	2015	81623.52
11	2017	79066.56
11	2015	77907.69
10	2018	77448.17
11	2016	75249.35
12	2016	74543.60
~~~~

# Max Ship Date

The value with the greatest difference between ship date and order date

~~~~sql
select max(age(shipdate, orderdate)) from orders
~~~~

~~~~
"7 days"
~~~~

These rows

~~~~sql
SELECT age(shipdate, orderdate), * FROM orders WHERE age(shipdate, orderdate) = INTERVAL '7 days';
~~~~

~~~~
"7 days"	4491	"CA-2018-105921"	"2018-08-14"	"2018-08-21"	"Standard Class"	"JM-15250"	"Janet Martin"	"Consumer"	"United States"	"Los Angeles"	"California"	"90032"	"West"	"FUR-TA-10001095"	"Furniture"	"Tables"	"Chromcraft Round Conference Tables"	418.30
...
~~~~


# Shipmode order by Sales

~~~~sql
select shipmode,sum(sales) from orders group by shipmode
~~~~

~~~~
"Standard Class"	1340831.64
"Second Class"	449914.04
"Same Day"	125219.03
"First Class"	345572.26
~~~~

**Percent**

~~~~sql
SELECT shipmode, 
       sum(sales) as total_sales, 
       ROUND(sum(sales) / (SELECT sum(sales) FROM orders) * 100, 2) as sales_percentage
FROM orders 
GROUP BY shipmode;
~~~~

~~~~
"Standard Class"	1340831.64	59.29
"Second Class"	449914.04	19.89
"Same Day"	125219.03	5.54
"First Class"	345572.26	15.28
~~~~


# Customer Count by State

~~~~sql
select count(distinct(customerid)), state from orders group by state order by count desc limit 10
~~~~

~~~~
570	"California"
409	"New York"
367	"Texas"
255	"Pennsylvania"
231	"Illinois"
223	"Washington"
196	"Ohio"
178	"Florida"
120	"North Carolina"
107	"Virginia"
~~~~

# Top 10 Customer

~~~~sql
select customername,sum(sales) from orders where customerid in (select customerid from orders group by customerid) group by customername order by sum(sales) desc limit 10
~~~~

~~~~
"Sean Miller"	25043.07
"Tamara Chand"	19052.22
"Raymond Buch"	15117.35
"Tom Ashbrook"	14595.62
"Adrian Barton"	14473.57
"Ken Lonsdale"	14175.23
"Sanjit Chand"	14142.34
"Hunter Lopez"	12873.30
"Sanjit Engle"	12209.44
"Christopher Conant"	12129.08
~~~~

# Top 10 States

~~~~sql
select state,sum(sales) from orders group by state order by sum desc limit 10
~~~~

~~~~
"California"	446306.49
"New York"	306361.07
"Texas"	168572.47
"Washington"	135206.87
"Pennsylvania"	116276.76
"Florida"	88436.55
"Illinois"	79236.57
"Michigan"	76136.07
"Ohio"	75130.43
"Virginia"	70636.72
~~~~


# Top Customers in California

~~~~sql
select customername,sum(sales) from orders where customerid in (select customerid from orders group by customerid) and state='California' group by customername order by sum(sales) desc limit 10
~~~~

~~~~
"Ken Lonsdale"	8341.29
"Jane Waco"	7381.89
"Karen Ferguson"	7182.76
"Nick Crebassa"	6275.47
"Edward Hooks"	5809.01
"Robert Marley"	4881.13
"Max Jones"	4823.09
"William Brown"	4811.28
"Nora Preis"	4652.95
"Clay Ludtke"	4426.48
~~~~


# Worst 10 States

~~~~sql
select state,sum(sales) from orders group by state order by sum asc limit 10
~~~~

~~~~
"North Dakota"	919.91
"West Virginia"	1209.82
"Maine"	1270.53
"South Dakota"	1315.56
"Wyoming"	1603.14
"District of Columbia"	2865.02
"Kansas"	2914.31
"Idaho"	4382.49
"Iowa"	4443.56
"New Mexico"	4783.54
~~~~

# Top Customers in North Dakota

~~~~sql
select customername,sum(sales) from orders where customerid in (select customerid from orders group by customerid) and state='North Dakota' group by customername order by sum(sales) desc limit 10
~~~~

Just 2

~~~~
"Nat Carroll"	891.53
"Christopher Schild"	28.38
~~~~


# Sales by Category

~~~~sql
select category, sum(sales) from orders group by category
~~~~

~~~~
"Furniture"	728658.75
"Office Supplies"	705422.28
"Technology"	827455.94
~~~~


# Sales by Sub-Category

~~~~sql
select subcategory, sum(sales) from orders group by subcategory order by sum desc
~~~~

~~~~
"Phones"	327782.49
"Chairs"	322822.75
"Storage"	219343.37
"Tables"	202810.77
"Binders"	200028.82
"Machines"	189238.68
"Accessories"	164186.70
"Copiers"	146248.07
"Bookcases"	113813.25
"Appliances"	104618.38
"Furnishings"	89211.98
"Paper"	76828.34
"Supplies"	46420.29
"Art"	26705.42
"Envelopes"	16128.02
"Labels"	12347.71
"Fasteners"	3001.93
~~~~

# California Sub-Category

~~~~sql
select subcategory,sum(sales) from orders where state='California' group by subcategory order by sum desc
~~~~

~~~~
"Phones"	67139.71
"Chairs"	61321.22
"Tables"	45085.58
"Storage"	44113.47
"Accessories"	36772.90
"Machines"	29492.03
"Binders"	27685.63
"Bookcases"	26875.26
"Appliances"	23736.75
"Copiers"	21279.57
"Furnishings"	18934.53
"Paper"	16340.45
"Supplies"	15572.50
"Art"	5431.53
"Envelopes"	3226.18
"Labels"	2833.51
"Fasteners"	465.67
~~~~


# North Dakota Sub-Category

~~~~sql
select subcategory,sum(sales) from orders where state='North Dakota' group by subcategory order by sum desc
~~~~

~~~~
"Storage"	704.76
"Art"	181.84
"Binders"	25.90
"Fasteners"	7.41
~~~~


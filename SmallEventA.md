# run db.sql file

~~~~sql
CREATE TABLE public."event" (
	user_id varchar NULL,
	device_id varchar NULL,
	event_name varchar NULL,
	event_time timestamp NULL
);

CREATE TABLE public.product (
	product_id int2 NULL,
	price int8 NULL,
	skin_color varchar NULL
);

CREATE TABLE public.age_segment (
	id int2 NULL,
	age_min int2 NULL,
	age_max int2 NULL
);

CREATE TABLE public.account (
	user_id varchar(50) NULL,
	device_id varchar(50) NULL,
	age int4 NULL,
	gender varchar(50) NULL
);
~~~~

# Import CSV Files

~~~~sql
COPY account
FROM 'C:\Users\tayfu\Desktop\New folder\account.csv'
DELIMITER ','
CSV HEADER;
~~~~

~~~~sql
COPY age_segment
FROM 'C:\Users\tayfu\Desktop\New folder\age_segment.csv'
DELIMITER ','
CSV HEADER;
~~~~

~~~~sql
COPY event
FROM 'C:\Users\tayfu\Desktop\New folder\event.csv'
DELIMITER ','
CSV HEADER;
~~~~

~~~~sql
COPY product
FROM 'C:\Users\tayfu\Desktop\New folder\product.csv'
DELIMITER ','
CSV HEADER;
~~~~

# Tables Check

~~~~sql
select * from product
~~~~

~~~~
"product_id"	"price"	"skin_color"
1	5	"Red"
2	10	"Blue"
3	15
~~~~

~~~~sql
select * from event limit 10
~~~~

~~~~
"user_id"	"device_id"	"event_name"	"event_time"
"User_000058"	"Device_058"	"Puzzle_Finish_004"	"2022-11-01 00:01:00"
"User_000033"	"Device_033"	"Bonus_Activation_001"	"2022-11-01 00:04:00"
"User_000078"	"Device_078"	"Puzzle_Finish_005"	"2022-11-01 00:05:00"
"User_000074"	"Device_074"	"Leaderboard_Open"	"2022-11-01 00:10:00"
"User_000032"	"Device_032"	"Level_Lose"	"2022-11-01 00:12:00"
"User_000015"	"Device_015"	"Puzzle_Finish_007"	"2022-11-02 22:54:00"
"User_000035"	"Device_035"	"Bonus_Activation_003"	"2022-11-06 17:49:00"
"User_000039"	"Device_039"	"Puzzle_Finish_011"	"2022-11-09 11:35:00"
"User_000070"	"Device_070"	"App_Purchase_001"	"2022-11-11 17:55:00"
"User_000031"	"Device_031"	"Puzzle_Finish_009"	"2022-11-11 18:51:00"
~~~~

~~~~sql
select * from age_segment
~~~~

~~~~
"id"	"age_min"	"age_max"
1	0	18
2	19	24
3	25	30
4	31	40
5	41	55
~~~~

~~~~sql
select * from account limit 10
~~~~

~~~~
"user_id"	"device_id"	"age"	"gender"
"User_000001"	"Device_001"	44	"Female"
"User_000002"	"Device_002"	46	"Female"
"User_000003"	"Device_003"	24	"Male"
"User_000004"	"Device_004"	47	"Female"
"User_000005"	"Device_005"	29	"Male"
"User_000006"	"Device_006"	55	"Male"
"User_000007"	"Device_007"	30	"Female"
"User_000008"	"Device_008"	22	"Female"
"User_000009"	"Device_009"	45	"Male"
"User_000010"	"Device_010"	49	"Female"
~~~~

## Null Values

**product table**

~~~~sql
SELECT (SELECT count(*) FROM product WHERE skin_color IS NULL) AS skin_color, 
(SELECT count(*) FROM product WHERE price IS NULL) AS price,
(SELECT count(*) FROM product WHERE product_id IS NULL) as product_id
~~~~

~~~~
"skin_color" "price" "product_id"
1	0	0
~~~~

~~~~sql
SELECT (SELECT count(*) FROM account WHERE user_id IS NULL) AS user_id, 
(SELECT count(*) FROM account WHERE device_id IS NULL) AS device_id,
(SELECT count(*) FROM account WHERE age IS NULL) as age,
(SELECT count(*) FROM account WHERE gender IS NULL) as gender
~~~~

~~~~
"user_id"	"device_id"	"age"	"gender"
0	0	0	0
~~~~

~~~~sql
SELECT (SELECT count(*) FROM event WHERE user_id IS NULL) AS user_id, 
(SELECT count(*) FROM event WHERE device_id IS NULL) AS device_id,
(SELECT count(*) FROM event WHERE event_name IS NULL) as event_name,
(SELECT count(*) FROM event WHERE event_time IS NULL) as event_time
~~~~

~~~~
"user_id"	"device_id"	"event_name"	"event_time"
0	0	0	0
~~~~

~~~~sql
SELECT (SELECT count(*) FROM age_segment WHERE id IS NULL) AS id, 
(SELECT count(*) FROM age_segment WHERE age_min IS NULL) AS age_min,
(SELECT count(*) FROM age_segment WHERE age_max IS NULL) as age_max
~~~~

~~~~
"id"	"age_min"	"age_max"
0	0	0
~~~~


**Result**

The product table has a null value. Other values Red and Blue. We can logically assign the null value is Green for a our tests.

~~~~sql
UPDATE product SET skin_color='Green' WHERE skin_color='' OR skin_color IS NULL
~~~~

~~~~
"product_id" "price" "skin_color"
1	5	"Red"
2	10	"Blue"
3	15	"Green"
~~~~


**Also**

user_id and device_id seem to have the same user counts. We can check if there is any strange value here.

~~~~sql
select user_id,device_id from event where substring(user_id from char_length(user_id)-2) != substring(device_id from char_length(device_id)-2)
~~~~

No strange values.

~~~~sql
select user_id,device_id from account where substring(user_id from char_length(user_id)-2) != substring(device_id from char_length(device_id)-2)
~~~~

No strange values.


# Event and account Merge

~~~~sql
select event.user_id,event.device_id,event.event_name,event.event_time,account.age,account.gender from event join account on event.user_id = account.user_id
~~~~

# 10 Most Repeating Users

We find the 10 most repeating users in the event. These are the ones who buy the most from us number of purchases.

~~~~sql
select event. user_id, count(*) from event join account on event.user_id = account.user_id group by event.user_id order by count(*) desc limit 10
~~~~

~~~~
"user_id"	"count"
"User_000019"	839
"User_000080"	786
"User_000025"	742
"User_000049"	604
"User_000013"	593
"User_000078"	549
"User_000055"	549
"User_000064"	540
"User_000077"	526
"User_000074"	509
~~~~

Age and gender of these users

~~~~sql
select user_id,age,gender from account where user_id in (select event. user_id from event join account on event.user_id = account.user_id group by event.user_id order by count(*) desc limit 10)
~~~~

~~~~
"user_id"	"age"	"gender"
"User_000013"	18	"Male"
"User_000019"	21	"Male"
"User_000025"	45	"Male"
"User_000049"	27	"Female"
"User_000055"	43	"Male"
"User_000064"	52	"Male"
"User_000074"	22	"Female"
"User_000077"	39	"Male"
"User_000078"	41	"Female"
"User_000080"	27	"Female"
~~~~

# How many events

The dataset consists of only 1 year and 1 month

~~~~sql
select date_trunc('day',event_time), count(*) from event group by date_trunc('day',event_time) order by count desc limit 15
~~~~

~~~~
"date_trunc"	"count"
"2022-11-02 00:00:00"	1665
"2022-11-03 00:00:00"	1655
"2022-11-01 00:00:00"	1535
"2022-11-05 00:00:00"	1508
"2022-11-04 00:00:00"	1493
"2022-11-11 00:00:00"	1228
"2022-11-08 00:00:00"	1226
"2022-11-09 00:00:00"	1220
"2022-11-07 00:00:00"	1210
"2022-11-06 00:00:00"	1207
"2022-11-10 00:00:00"	1191
"2022-11-12 00:00:00"	934
"2022-11-21 00:00:00"	893
"2022-11-22 00:00:00"	774
"2022-11-24 00:00:00"	767
~~~~


# How Many Users Each Day

~~~~sql
select date_trunc('day',event_time), count(DISTINCT(user_id)) from event group by date_trunc('day',event_time) order by count desc limit 5
~~~~

~~~~
"date_trunc"	"count"
"2022-11-11 00:00:00"	77
"2022-11-02 00:00:00"	77
"2022-11-04 00:00:00"	77
"2022-11-03 00:00:00"	76
"2022-11-05 00:00:00"	76
~~~~


# Ages that shop the most

~~~~python
select account.age, count(*) from event join account on event.user_id = account.user_id group by account.age order by count desc limit 5
~~~~

~~~~
"age"	"count"
27	1873
22	1641
21	1618
55	1510
45	1451
~~~~

# Age Segment Join

~~~~sql
select event.user_id,event.device_id,event.event_name,event.event_time,account.age,account.gender,age_segment.age_min,age_segment.age_max from event join account on event.user_id = account.user_id JOIN age_segment ON account.age >= age_segment.age_min AND account.age <= age_segment.age_max limit 10;
~~~~

~~~~
"user_id"	"device_id"	"event_name"	"event_time"	"age"	"gender"	"age_min"	"age_max"
"User_000058"	"Device_058"	"Puzzle_Finish_004"	"2022-11-01 00:01:00"	27	"Male"	25	30
"User_000033"	"Device_033"	"Bonus_Activation_001"	"2022-11-01 00:04:00"	28	"Male"	25	30
"User_000078"	"Device_078"	"Puzzle_Finish_005"	"2022-11-01 00:05:00"	41	"Female"	41	55
"User_000074"	"Device_074"	"Leaderboard_Open"	"2022-11-01 00:10:00"	22	"Female"	19	24
"User_000032"	"Device_032"	"Level_Lose"	"2022-11-01 00:12:00"	21	"Male"	19	24
"User_000015"	"Device_015"	"Puzzle_Finish_007"	"2022-11-02 22:54:00"	32	"Female"	31	40
"User_000035"	"Device_035"	"Bonus_Activation_003"	"2022-11-06 17:49:00"	45	"Male"	41	55
"User_000039"	"Device_039"	"Puzzle_Finish_011"	"2022-11-09 11:35:00"	32	"Female"	31	40
"User_000070"	"Device_070"	"App_Purchase_001"	"2022-11-11 17:55:00"	34	"Male"	31	40
"User_000031"	"Device_031"	"Puzzle_Finish_009"	"2022-11-11 18:51:00"	24	"Male"	19	24
~~~~

## Concat Min-Max as Age_Range

~~~~sql
select 
event.user_id,event.device_id,event.event_name,event.event_time,account.age,account.gender,CONCAT(age_segment.age_min, '-', age_segment.age_max) as Age_Range
from event 
join account on event.user_id = account.user_id 
JOIN age_segment ON account.age >= age_segment.age_min AND account.age <= age_segment.age_max limit 10
~~~~

~~~~
"user_id"	"device_id"	"event_name"	"event_time"	"age"	"gender"	"age_range"
"User_000058"	"Device_058"	"Puzzle_Finish_004"	"2022-11-01 00:01:00"	27	"Male"	"25-30"
"User_000033"	"Device_033"	"Bonus_Activation_001"	"2022-11-01 00:04:00"	28	"Male"	"25-30"
"User_000078"	"Device_078"	"Puzzle_Finish_005"	"2022-11-01 00:05:00"	41	"Female"	"41-55"
"User_000074"	"Device_074"	"Leaderboard_Open"	"2022-11-01 00:10:00"	22	"Female"	"19-24"
"User_000032"	"Device_032"	"Level_Lose"	"2022-11-01 00:12:00"	21	"Male"	"19-24"
"User_000015"	"Device_015"	"Puzzle_Finish_007"	"2022-11-02 22:54:00"	32	"Female"	"31-40"
"User_000035"	"Device_035"	"Bonus_Activation_003"	"2022-11-06 17:49:00"	45	"Male"	"41-55"
"User_000039"	"Device_039"	"Puzzle_Finish_011"	"2022-11-09 11:35:00"	32	"Female"	"31-40"
"User_000070"	"Device_070"	"App_Purchase_001"	"2022-11-11 17:55:00"	34	"Male"	"31-40"
"User_000031"	"Device_031"	"Puzzle_Finish_009"	"2022-11-11 18:51:00"	24	"Male"	"19-24"
~~~~

# Age Segment Repeat

~~~~sql
select 
CONCAT(age_segment.age_min, '-', age_segment.age_max), count(*)
from event 
join account on event.user_id = account.user_id 
JOIN age_segment ON account.age >= age_segment.age_min AND account.age <= age_segment.age_max group by CONCAT(age_segment.age_min, '-', age_segment.age_max) order by count desc
~~~~

~~~~
"concat" "count"
"41-55"	9261
"19-24"	5317
"31-40"	3509
"25-30"	3463
"0-18"	855
~~~~


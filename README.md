# Coffee-Sales-Data-Analysis-Using-SQL

CREATE TABLE coffee_shop_sales (
    transaction_id INT PRIMARY KEY,
    transaction_date TEXT,  -- Temporarily store date as text
    transaction_time TEXT,
    transaction_qty INT,
    store_id INT,
    store_location TEXT,
    product_id INT,
    unit_price DOUBLE PRECISION,
    product_category TEXT,
    product_type TEXT,
    product_detail TEXT
);

SELECT * FROM coffee_shop_sales

UPDATE coffee_shop_sales
SET transaction_date = to_date(transaction_date,'DD-MM-YYYY')

ALTER TABLE coffee_shop_sales
ALTER COLUMN transaction_date TYPE date USING transaction_date::date


ALTER TABLE coffee_shop_sales
ALTER COLUMN transaction_time TYPE time using transaction_time::time

--total sales analysis

--calculate total sales for each respective MONTH

SELECT
	concat((round(sum(unit_price*transaction_qty)))/1000, 'K') as total_sales
FROM coffee_shop_sales
WHERE
	extract(month from transaction_date) = 3

--determine the month-on-month increase or decrease in sales
--calculate the difference in sales between the selected_month and previous_month

select
	extract(month from transaction_date) as month,
	round(sum(unit_price*transaction_qty)) as total_sales,
	(sum(unit_price*transaction_qty)-lag(sum(unit_price*transaction_qty),1) over(order by extract(month from transaction_date)))/
	lag(sum(unit_price*transaction_qty),1) over(order by extract(month from transaction_date))*100 mom_sales_percentage
from coffee_shop_sales
where
	extract(month from transaction_date) in (4,5)
group by 1
order by 1

--total_order_analysis

--calculate total number of orders for each respective MONTH

SELECT
	round(count(transaction_id)) as total_orders
FROM coffee_shop_sales
where
	extract(month from transaction_date) = 5

--determine the month-on-month increase or decrease in number of orders
--calculate the difference in no of orders between the selected_month and previous_month

select
	extract(month from transaction_date),
	round(count(transaction_id)) as total_orders,
	(count(transaction_id)::numeric - lag(count(transaction_id)::numeric,1) over(order by extract(month from transaction_date)))/
	lag(count(transaction_id)::numeric,1) over(order by extract(month from transaction_date))*100 as mom_increase_percentage
from coffee_shop_sales
where
	extract(month from transaction_date) in (4,5)
group by 1
order by 1

--total quantity sold analysis

SELECT
	round(sum(transaction_qty)) as total_qty_sold
from coffee_shop_sales
where
	extract(month from transaction_date) = 5

--determine the month-on-month increase or decrease in quantity
--calculate the difference in quantity between the selected_month and previous_month

select
	extract(month from transaction_date),
	round(sum(transaction_qty)) as total_orders,
	(sum(transaction_qty)::numeric - lag(sum(transaction_qty)::numeric,1) over(order by extract(month from transaction_date)))/
	lag(sum(transaction_qty)::numeric,1) over(order by extract(month from transaction_date))*100 as mom_increase_percentage
from coffee_shop_sales
where
	extract(month from transaction_date) in (4,5)
group by 1
order by 1

--calendar heat map

--implement a calendar heat map that dynmaically adjust based on the selected month from a slicer
--each day on the calendar will be color_coded to represent sales with darker shades indiacting higher sales
--implement tooltips to displaty detailed metrics(sales,orders,quantity) when hovering over a specific day


SELECT
	concat(round(sum(unit_price*transaction_qty)::numeric/1000,1),'K') as total_sales,
	concat(round(sum(transaction_qty)::numeric/1000,1),'K') as total_qty_sold,
	concat(round(count(transaction_id)::numeric/1000,1),'K') as total_orders
FROM coffee_shop_sales
where
	transaction_date = '2023-03-27'

--sales analysis by weekdays and weekends

--segment sales data into weekdays and weekends to analyze performance
--provide insights into whether sales patterns differ significantly between weekdays and weekends

select
	case
		when extract(dow from transaction_date) in (0,6) then 'Weekends'
		else 'Weekdays'
	end as week_type,
	concat(round(sum(unit_price*transaction_qty)::numeric/1000,1),'K') as total_sales
from coffee_shop_sales
where
	extract(month from transaction_date) = 5
group by 1

--sales analysis by store_location

--visualize sales data by different store locations
--include month over month difference metrics based on selected month in the slicer
--highlight mom sales increase and descrease for each store location to identify trends

SELECT
	store_location,
	concat(round(sum(unit_price*transaction_qty)::numeric/1000,1),'K') as total_sales
from coffee_shop_sales
WHERE
	extract(month from transaction_date) = 5
group by 1
order by 2 desc

--daily sales analysis with average line

--display daily sales for the selected month with a line chart
--incorpoartae a average line on the chart to represent the average daily sales
--hight bars exceeding the or falling below the avg line to idenct exceptional sales day

with cte as
(
SELECT
	sum(unit_price*transaction_qty)::numeric as total_sales
FROM coffee_shop_sales
where
	extract(month from transaction_date) = 5
group by transaction_date
)
SELECT
	concat(round(avg(total_sales)/1000,1),'K') as avg_sales
FROM cte	

--daily sales for selected_month

SELECT
	extract(day from transaction_date) as day_of_month,
	concat(round(sum(unit_price*transaction_qty)::numeric/1000,1),'K') as total_sales
FROM coffee_shop_sales
where
	extract(month from transaction_date) = 5
group by 1
order by 2 desc

--comapring daily sales with avreage_sales if geeater than abpve average and lesser than belwo average

with daily_sales as
(
SELECT
	extract(day from transaction_date) as day_of_month,
	sum(unit_price*transaction_qty) as total_sales
FROM coffee_shop_sales
where
	extract(month from transaction_date) = 5
group by 1
),
avg_daily_sales as
(
select
	avg(total_sales) as avg_sales
from daily_sales
)
select
	ds.day_of_month,
	case
		when ds.total_sales > ads.avg_sales then 'above_average'
		WHEN ds.total_sales < ads.avg_sales then 'below_average'
		ELSE 'equal_to_average'
	END as sales_status,
	ds.total_sales
from daily_sales ds,avg_daily_sales ads
order by ds.day_of_month


-- sales analaysis by product category

SELECT
	product_category,
	sum(unit_price*transaction_qty) as total_sales
FROM coffee_shop_sales
where
	extract(month from transaction_date) = 5
group by 1
order by 2 desc


--top 10 product by sales

SELECT
	product_type,
	sum(unit_price*transaction_qty) as total_sales
FROM coffee_shop_sales
where
	extract(month from transaction_date) = 5
group by 1
order by 2 desc
limit 10

--for both

SELECT

SELECT
	product_type,
	sum(unit_price*transaction_qty) as total_sales
FROM coffee_shop_sales
where
	extract(month from transaction_date) = 5 and product_category = 'Coffee'
group by 1
order by 2 desc
limit 10


--analaysis by days anad hours

SELECT
	sum(unit_price*transaction_qty) as total_sales,
	sum(transaction_qty) as total_qty,
	count(*) as total_orders
FROM coffee_shop_sales
where
	extract(month from transaction_date) = 5
	and
	extract(dow from transaction_date) = 0	
	and
	extract(hour from transaction_time) = 14


select
	extract(hour from transaction_time),
	sum(unit_price*transaction_qty) as total_sales
from coffee_shop_sales
where
	extract(month from transaction_date) = 5
group by 1
order by 1


select
	case
		when extract(dow from transaction_date)= 1 then 'Monday'
		when extract(dow from transaction_date)= 2 then 'Tuesday'
		when extract(dow from transaction_date)= 3 then 'Wednesday'
		when extract(dow from transaction_date)= 4 then 'Thursday'
		when extract(dow from transaction_date)= 5 then 'Friday'
		when extract(dow from transaction_date)= 6 then 'Saturday'
		else 'Sunday'
	end as day_of_week,
	sum(unit_price*transaction_qty) as total_sales
from coffee_shop_sales
where
	extract(month from transaction_date) = 5 
group by 1

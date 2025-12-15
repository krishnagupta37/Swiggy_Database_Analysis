# Swiggy Order Data Analysis

## 📌 Project Objective
The objective of this project is to analyze Swiggy food delivery data using SQL by cleaning raw transactional data, designing an optimized Star Schema, and generating meaningful business insights through KPIs 
and analytical queries. This project demonstrates real-world SQL skills in data cleaning, dimensional modeling, and business analysis, making it suitable for data analyst and BI roles.

## 📊 Project Overview
The dataset contains food delivery orders across multiple states, cities, restaurants, cuisines, and dishes. Initially stored in a single raw table, the data is cleaned, validated, and transformed into a data
warehouse–style star schema to support efficient analytics and reporting.

## 🛠️ Key Steps Performed
- Data Cleaning & Validation
```sql
1. Checked for NULL values in business-critical columns 
select
sum(case when state is null then 1 else 0 end) as null_state,
sum(case when city is null then 1 else 0 end) as null_city,
sum(case when order_date is null then 1 else 0 end) as null_order_date,
sum(case when Restaurant_name is null then 1 else 0 end) as null_restuarant_name,
sum(case when location is null then 1 else 0 end) as null_location,
sum(case when category is null then 1 else 0 end) as null_category,
sum(case when dish_name is null then 1 else 0 end) as null_dish_name,
sum(case when price_inr is null then 1 else 0 end) as null_price_inr,
sum(case when rating is null then 1 else 0 end) as null_rating,
sum(case when rating_count is null then 1 else 0 end) as null_rating_count
from swiggy_data

2. Blank or Empty Strings
select * from swiggy_data
WHERE 
STATE = '' or
CITY = '' or
restaurant_name='' or
location='' or
category ='' or
dish_name = ''

3. Duplicate Detection
select 
   state,
   city,
   order_date,
   restaurant_name, 
   location,
   category,
   dish_name,
   price_inr,
   rating, 
   rating_count,
   count(*) as cnt
from swiggy_data
group by state, city, order_date, restaurant_name, location, category, dish_name,
price_inr, rating, rating_count
having count(*) >1;

4. Delete Duplication

with cte as ( 
select *, row_number () over(
   partition by state, city, order_date, restaurant_name, location, category, dish_name,
                price_inr, rating, rating_count
order by (select null)
) as rn
from swiggy_data
)
delete from cte 
where rn>1
```

2️⃣ Dimensional Modeling (Star Schema)
- Dimension Tables

1. dim_date 
```sql
create table dim_date (
   date_id int IDENTITY(1,1) PRIMARY KEY,
   full_date date,
   year int,
   month int,
   month_name varchar(50),
   quarter int,
   Day int,
   Week int
)
```

2. dim_location 
```sql
create table dim_location (
    location_id int identity(1,1) primary key,
    state varchar(100),
    city varchar(100),
    location varchar(200)
)
```

3.dim_restaurant 
```sql
create table dim_restaurant (
    restaurant_id int identity(1,1) primary key,
    restaurant_name varchar(200)
)
```

4.dim_category
```sql
create table dim_category (
    category_id int identity(1,1) primary key,
    category varchar(200)
)
```
5.dim_dish 
```sql
create table dim_dish (
    dish_id int identity(1,1) primary key,
    dish_name varchar(200)
)
```
- Fact Table
```sql
create table fact_swiggy_orders (
    order_id int identity(1,1) primary key,
    date_id int,
    price_inr decimal(10,2),
    rating decimal(10,2),
    rating_count int,
    location_id int,
    restaurant_id int,
    category_id int,
    dish_id int,

    foreign key (date_id) references dim_date(date_id),
    foreign key (location_id) references dim_location(location_id),
    foreign key (date_id) references dim_date(date_id),
    foreign key (restaurant_id) references dim_restaurant(restaurant_id),
    foreign key (category_id) references dim_category(category_id),
    foreign key (dish_id) references dim_dish(dish_id)
)
```
Fact Table
```sql fact_table
insert into fact_swiggy_orders
(
    date_id,
    price_inr,
    rating,
    rating_count,
    location_id,
    restaurant_id,
    category_id,
    dish_id
)
select
    dd.date_id,
    s.price_inr,
    s.rating,
    s.rating_count,

    dl.location_id,
    dr.restaurant_id,
    dc.category_id,
    dsh.dish_id
from swiggy_data s

join dim_date dd
   on dd.full_date = s.Order_Date

join dim_location dl
   on dl.state = s.State
   and dl.city = s.City
   and dl.location = s.Location

join dim_restaurant dr
   on dr.restaurant_name = s.Restaurant_Name

join dim_category dc
     on dc.category = s.Category

join dim_dish dsh
     on dsh.dish_name = s.Dish_Name


select * from fact_swiggy_orders f
join dim_date d on f.date_id = d.date_id
join dim_location l on f.location_id = l.location_id
join dim_restaurant r on f.restaurant_id = r.restaurant_id
join dim_category c on f.category_id = c.category_id
join dim_dish di on f.dish_id = di.dish_id
```

This structure improves query performance, data consistency, and reporting scalability.

## 🛠 Tools & Technologies
- Microsoft SQL Server
- SQL

## 🔍 Key Tasks Performed
- KPI's 

Total Orders 
```sql
select count(*) as total_orders
from fact_swiggy_orders
```
Total Revenue  (inr million)
```sql
select format(sum(convert(float, price_inr))/1000000, 'N2') + ' INR Million '
as total_revenue
from fact_swiggy_orders
```
Average dish price
```sql
select format(avg(convert(float, price_inr)), 'N2') + ' INR '
as avg_dish_price
from fact_swiggy_orders
```
Average Rating
```sql
select 
cast(round(avg(convert(float,rating)),2) as decimal(10,2)) as avg_rating
from fact_swiggy_orders
```
- Granular Requirements
  Monthly order trends
```sql
select 
d.year,
d.month,
d.month_name,
count(order_id) as total_orders
from fact_swiggy_Orders f
join dim_date d on
f.date_id = d.date_id
group by
d.year,
d.month, d.month_name 
```
Quarter trends
```sql
select 
d.year,
d.quarter,
count(order_id) as total_orders
from fact_swiggy_orders f
join dim_date d on
f.date_id = d.date_id
group by d.year,
d.quarter
order by total_orders desc
```
Yearly Trends
```sql
select 
d.year,
count(order_id) as total_orders
from fact_swiggy_orders f
join dim_date d on
f.date_id = d.date_id
group by d.year
order by total_orders desc
```
Orders by day of week (mon-sun)
```sql
select
 datename(weekday, d.full_date) as day_name,
 count(*) as total_orders
from fact_swiggy_orders f
join dim_date d
on f.date_id = d.date_id
group by datename(weekday, d.full_date), DATEPART(weekday, d.full_date)
order by DATEPART(weekday, d.full_date)
```
Top 10 cities by order
```sql
select top 10
       l.city,
       count(f.order_id) as total_orders
from fact_swiggy_orders f
join dim_location l
on l.location_id = f.location_id
group by l.city
order by count(f.order_id) desc
```
Revenue contribution by states
```sql
select l.state,
       d.month_name,
       sum(f.price_inr)/100000 as revenue_lakhs 
from fact_swiggy_orders f
join dim_location l
on l.location_id = f.location_id
join dim_date d
on d.date_id = f.date_id
group by l.state,
         d.month_name
order by revenue_lakhs desc 
```
Top 10 restaurants by orders 
```sql
select top 10 
       r.restaurant_name,
       count(f.order_id) as total_orders,
       sum(price_inr) as total_revenue
from fact_swiggy_orders f
join dim_restaurant r on
f.restaurant_id = r.restaurant_id
group by r.restaurant_name
order by total_revenue desc
```
Top categories by order volume
```sql
select
    c.category,
    count(f.order_id) as total_orders
from fact_swiggy_orders f
join dim_category c
on c.category_id = f.category_id
group by c.category
order by total_orders desc
```
Top Ordered Dishes
```sql
select top 10
   d.dish_name, 
   count(f.order_id) as total_orders
from fact_swiggy_orders f
join dim_dish d
on f.dish_id = d.dish_id
group by d.dish_name
order by total_orders desc
```
Cuisine Performance 
```sql
select
    c.category,
    count(f.order_id) as total_orders,
    cast(avg(f.rating) as decimal(4,2)) as avg_rating
from fact_swiggy_orders f
join dim_category c 
on f.category_id = c.category_id
group by c.category
order by total_orders desc
```
Total orders by Price range
```sql
select 
    case 
        when price_inr < 100 then 'Under 100'
        when price_inr between 100 and 199 then '100 - 199'
        when price_inr between 200 and 299 then '200 - 299'
        when price_inr between 300 and 499 then '300 - 499'
        else '500 +'
    end as price_range,
    count(order_id) as total_orders
from fact_swiggy_orders
group by
      case
        when price_inr < 100 then 'Under 100'
        when price_inr between 100 and 199 then '100 - 199'
        when price_inr between 200 and 299 then '200 - 299'
        when price_inr between 300 and 499 then '300 - 499'
        else '500 +'
     end
order by total_orders desc
```

Rating Analysis
```sql
select  
     rating, 
     count(rating_count) as rating_count
from fact_swiggy_orders
group by rating 
order by rating_count desc
```
## 📊 Insights & Findings
- Identified highest revenue-generating categories
- Found average customer ratings by category

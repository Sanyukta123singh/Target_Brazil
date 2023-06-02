Target is a globally renowned brand and a prominent retailer in the United States. Target makes itself a preferred shopping destination by offering outstanding value, inspiration, innovation and an exceptional guest experience that no other retailer can deliver.

This particular business case focuses on the operations of Target in Brazil and provides insightful information about 100,000 orders placed between 2016 and 2018. The dataset offers a comprehensive view of various dimensions including the order status, price, payment and freight performance, customer location, product attributes, and customer reviews.

By analyzing this extensive dataset, it becomes possible to gain valuable insights into Target's operations in Brazil. The information can shed light on various aspects of the business, such as order processing, pricing strategies, payment and shipping efficiency, customer demographics, product characteristics, and customer satisfaction levels.


**Exploratory Data Analysis is performed uing SQL**

#Cities and States of customers ordered during the given period

SELECT 
distinct customer_city as no_of_cities,
customer_state as no_of_states
FROM 
`sqldemo-381616.Target_BusinessCase.Customers` c 
      join 
`sqldemo-381616.Target_BusinessCase.Orders` o 
on 
c.customer_id = o.customer_id
where 
o.order_purchase_timestamp 
between 
'2016-09-04 21:15:19' and  '2018-10-17 17:30:18'
limit 10;


#Is there a growing trend on e-commerce in Brazil? How can we describe a      complete scenario? Can we see some seasonality with peaks at specific months?

SELECT 
extract  (year from order_purchase_timestamp ) as Year,
extract  (month from order_purchase_timestamp ) as Month , 
count(order_id) as  Orders_Count 
FROM 
`sqldemo-381616.Target_BusinessCase.Orders` 
group by 
Year,
Month
order by 
Year, 
Month;


#What time do Brazilian customers tend to buy (Dawn, Morning, Afternoon or Night)?

SELECT 
Order_Time, 
Count(Order_Time) as Most_Favourable_Time 
from 
(Select 
Case 
when  extract(time from order_purchase_timestamp) between '7:00:00' and '12:00:00' then 'Morning'
when  extract(time from order_purchase_timestamp) between '13:00:00' and '18:00:00' then 'Afternoon'
when  extract(time from order_purchase_timestamp) between '19:00:00' and '23:00:00' then 'Night'
when  extract(time from order_purchase_timestamp) between '00:00:00' and '6:00:00' then 'Dawn'
END as 
Order_Time
FROM 
`sqldemo-381616.Target_BusinessCase.Customers` c join `sqldemo-381616.Target_BusinessCase.Orders` o on c.customer_id = o.customer_id
)
group by 
Order_Time
order by 
Most_Favourable_Time desc;


#Get month on month orders by states

SELECT 
FORMAT_DATETIME("%B",DATETIME (order_purchase_timestamp))
as Month_Name,c.customer_state ,count(*)as No_of_Orders FROM `sqldemo-381616.Target_BusinessCase.Customers` c 
left join  
`sqldemo-381616.Target_BusinessCase.Orders` o on c.customer_id = o.customer_id
 group by 
c.customer_state,Month_Name
 order by 
No_of_Orders desc 
 LIMIT 1000;


 #Distribution of customers across the states in Brazil

SELECT 
customer_state, 
count(customer_unique_id) as Customers_count 
FROM 
`sqldemo-381616.Target_BusinessCase.Customers` 
group by 
customer_state
order by 
Customers_count desc
LIMIT 1000;


#Get % increase in cost of orders from 2017 to 2018 (include months between Jan to Aug only)

SELECT round(((Sales_2 - Sales_1)/Sales_1)*100,0) as YOY_Growth from(
SELECT 
sum(case when Year = 2017 and Month between 1 and 8 then payment_valueend) as Sales_1,
sum(case when Year = 2018 and Month between 1 and 8 then payment_valueend) as Sales_2
from(

SELECT extract(Month from order_purchase_timestamp) as Month,extract(Year from order_purchase_timestamp) as Year,
p.payment_value FROM `sqldemo-381616.Target_BusinessCase.Orders` o join `sqldemo-381616.Target_BusinessCase.Payments` p on o.order_id = p.order_id));


#Mean & Sum of price and freight value by customer state

SELECT 
c.customer_state,
round(avg(oi.price),2) as Mean_Price, 
round(sum(oi.price),2) as Total_price, 
round(avg(oi.freight_value),2) as Mean_freight, 
round(sum(oi.freight_value),2) as Total_Freight 
FROM 
`sqldemo-381616.Target_BusinessCase.Customers` c 
left join 
`sqldemo-381616.Target_BusinessCase.Orders` o
 on 
c.customer_id = o.customer_id join `sqldemo-381616.Target_BusinessCase.OrderItems` oi 
 on 
o.order_id = oi.order_id

group by 
c.customer_state;


#Calculate days between purchasing, delivering and estimated delivery

SELECT 
abs(extract(day from order_purchase_timestamp) - extract 	(day 
from 
order_delivered_customer_date)) as days_to_delivery
FROM 
`sqldemo-381616.Target_BusinessCase.Orders` ;

SELECT 
abs(extract(day from order_estimated_delivery_date) - extract (day from order_delivered_customer_date)) as diff_estimated_deliveryDays
FROM 
`sqldemo-381616.Target_BusinessCase.Orders` ;


#Find time_to_delivery & diff_estimated_delivery. Formula for the same given below:

#time_to_delivery = order_purchase_timestamp -order_delivered_customer_date
#diff_estimated_delivery = order_estimated_delivery_date -order_delivered_customer_date


SELECT 
abs(extract(hour from order_purchase_timestamp) - extract (hour from order_delivered_customer_date)) as time_to_delivery
FROM 
`sqldemo-381616.Target_BusinessCase.Orders` ;

SELECT 
abs(extract(hour from order_estimated_delivery_date) - extract (hour from order_delivered_customer_date)) as diff_estimated_delivery
FROM 
`sqldemo-381616.Target_BusinessCase.Orders` ;


#Group data by state, take mean of freight_value, time_to_delivery, diff_estimated_delivery


SELECT 
c.customer_state as State,
Round(avg(oi.freight_value),2) as Mean_Freight,
Round(abs(avg(extract(hour from o.order_purchase_timestamp)-extract(hour from o.order_delivered_customer_date))),0) as time_to_delivery,
Round(abs(avg(extract(day from o.order_estimated_delivery_date)-extract(day from o.order_delivered_customer_date))),0) as diff_estimated_delivery 
FROM 
`sqldemo-381616.Target_BusinessCase.Customers` c 
join 
`sqldemo-381616.Target_BusinessCase.Orders` o
on 
c.customer_id = o.customer_id join `sqldemo-381616.Target_BusinessCase.OrderItems` oi 
on 
o.order_id = oi.order_id
group by 
c.customer_state
order by 
Mean_Freight desc;


select 
c.customer_state,Round(avg(oi.freight_value),2) as Average_freight_value 
FROM 
`sqldemo-381616.Target_BusinessCase.Customers` c 
join 
`sqldemo-381616.Target_BusinessCase.Orders` o
on 
 c.customer_id = o.customer_id 
join 
`sqldemo-381616.Target_BusinessCase.OrderItems` oi 
on 
o.order_id = oi.order_id
group by 
c.customer_state
order by 
Average_freight_value desc
limit 5;

select 
c.customer_state,
Round(avg(oi.freight_value),2) as Average_freight_value 
FROM 
`sqldemo-381616.Target_BusinessCase.Customers` c 
join 
`sqldemo-381616.Target_BusinessCase.Orders` o
on 
c.customer_id = o.customer_id 
join 
`sqldemo-381616.Target_BusinessCase.OrderItems` oi 
on 
o.order_id = oi.order_id
group by 
c.customer_state
order by 
Average_freight_value
limit 5;


SELECT 
c.customer_state as State,
Round(abs(avg(extract(Hour from o.order_purchase_timestamp)-extract(Hour from o.order_delivered_customer_date))),0) as time_to_delivery 
FROM 
`sqldemo-381616.Target_BusinessCase.Customers` c 
join 
`sqldemo-381616.Target_BusinessCase.Orders` o
on 
c.customer_id = o.customer_id 
join 
`sqldemo-381616.Target_BusinessCase.OrderItems` oi 
on 
o.order_id = oi.order_id
group by
c.customer_state
order by 
time_to_delivery desc
limit 5;

SELECT 
c.customer_state as State,
Round(abs(avg(extract(Hour from o.order_purchase_timestamp)-extract(hour from o.order_delivered_customer_date))),0) as time_to_delivery 
FROM 
`sqldemo-381616.Target_BusinessCase.Customers` c 
join 
`sqldemo-381616.Target_BusinessCase.Orders` o
on 
c.customer_id = o.customer_id 
join 
`sqldemo-381616.Target_BusinessCase.OrderItems` oi 
on 
o.order_id = oi.order_id
group by
c.customer_state
order by 
time_to_delivery 
limit 5;


SELECT 
c.customer_state as State,
Round(abs(avg(extract(hour from o.order_estimated_delivery_date)-extract(hour from o.order_delivered_customer_date))),0) as diff_estimated_delivery 
   FROM 
`sqldemo-381616.Target_BusinessCase.Customers` c join `sqldemo-381616.Target_BusinessCase.Orders` o
on 
c.customer_id = o.customer_id 
join 
`sqldemo-381616.Target_BusinessCase.OrderItems` oi 
on 
o.order_id = oi.order_id
group by
c.customer_state
order by 
diff_estimated_delivery desc
limit 5;

#Find the month on month no. of orders placed using different payment types.
#Find the no. of orders placed on the basis of the payment installments that have been paid.

SELECT 
FORMAT_DATETIME("%B",DATETIME (o.order_purchase_timestamp))
as Month_Name,p.payment_type as payment_type, count(*) as Count_of_Orders 
FROM 
`sqldemo-381616.Target_BusinessCase.Orders` o 
left join 
`sqldemo-   381616.Target_BusinessCase.Payments` p  
on 
o.order_id = p.order_id
group by 
payment_type, 
Month_Name
order by 
Count_of_Orders desc
LIMIT 10;

select 
p.payment_installments as installments, 
count(*) as Count_of_orders
FROM 
`sqldemo-381616.Target_BusinessCase.Orders` o 
left join 
`sqldemo-381616.Target_BusinessCase.Payments` p  
on 
o.order_id = p.order_id
group by
 	installments
order by 
Count_of_orders desc
limit 10;

# SQL-End-to-End-Project
Pizzas Sales End to End Project

--Retrieve the total number of orders placed.

Select count(order_id) as Total_order from orders

--Calculate the total revenue generated from pizza sales.

Select Round(sum(A.quantity* B.price),2) as Total_Revenue from order_details as A
left join pizzas as B
on A.pizza_id=B.pizza_id

--Identify the highest-priced pizza.

Select Top 1 B.name,A.price from pizzas as A
Right join pizza1types as B
On A.pizza_type_id=B.pizza_type_id
Order by A.price desc

--Identify the most common pizza size ordered.

Select Top 1 B.size,count(A.order_id) as Cnt_ from order_details as A
left join pizzas as B
on A.pizza_id=B.pizza_id
Group by B.size
order by count(A.order_id) desc

--List the top 5 most ordered pizza types along with their quantities.

Select Top 5 A.name,sum(cast(C.quantity as int)) as Count_ from Pizza1types as A
Right join pizzas as B
On A.Pizza_type_id=B.Pizza_type_id
right join order_details as C
On B.pizza_id=C.pizza_id
group by A.name
order by sum(cast(C.quantity as int)) desc

--Join the necessary tables to find the total quantity of each pizza category ordered.

 Select A.Category,sum(cast(c.quantity as int)) as Total_qty from pizza1types as A
 Left Join pizzas as B
 On A.pizza_type_id=B.pizza_type_id
 right join order_details as C
 On B.Pizza_id=C.pizza_id
 Group by A.Category
 order by sum(cast(c.quantity as int)) desc

--Determine the distribution of orders by hour of the day.

Select Count(Order_id) as Total_OrderCnt,Datepart(Hour,time) as Hours from Orders
Group By Datepart(Hour,time)

--Join relevant tables to find the category-wise distribution of pizzas.

Select Category,count(pizza_type_id) as Count_ from pizza1types
group by Category

--Group the orders by date and calculate the average number of pizzas ordered per day.

with Pizza_1 as
(
Select Date,Datepart(Day,date) as Day_,sum(cast(A.quantity as int)) as Sum_ from order_details as A
Left Join orders as B
On A.order_id=B.Order_id
Group by B.Date,Datepart(Day,date)
)
Select Avg(Sum_) as Avg_ from Pizza_1


--Determine the top 3 most ordered pizza types based on revenue.

Select Top 3 C.name,Sum(B.price*A.quantity) as Total_Revenue from order_details as A
Left Join pizzas as B
On A.pizza_id=B.pizza_id
left join pizza1types as C
On B.pizza_type_id=C.pizza_type_id
Group by C.name
order by Sum(B.price*A.quantity)  desc

--Calculate the percentage contribution of each pizza type to total revenue.

Select C.Category,round((Sum(B.price * A.quantity)/(Select Sum(b.price*A.quantity)  from order_details as A
inner join pizzas as B
On A.pizza_id=B.pizza_id))*100,2) as per_Contribution
from order_details as A
inner Join pizzas as B
On A.pizza_id=B.pizza_id
inner join pizza1types as C
On B.pizza_type_id=C.pizza_type_id
group by C.Category


--Analyze the cumulative revenue generated over time.

With rev_ as
(
Select A.date,A.time,Sum(C.price*B.quantity) as Revenue from orders as A
inner join order_details as B
On A.order_id=B.order_id
inner join pizzas as C
On B.pizza_id=C.pizza_id
Group by A.date,A.time
)
Select Date,revenue,round(sum(Revenue) over(order by time),2) as CR from rev_

--Determine the top 3 most ordered pizza types based on revenue for each pizza category.

with Revenue_ as 
(
Select Round(Sum(B.Price*A.quantity),2) as Revenue,C.category,C.name  from order_details as A
inner join pizzas as B
On A.pizza_id=B.pizza_id
inner join pizza1types as C
On B.pizza_type_id=C.Pizza_type_id
group by C.category,C.name
),
ranking as
(
Select Revenue,name,Category,DENSE_RANK() over(partition by Category order by Revenue desc ) as Ranks from Revenue_
)
select Revenue,name,category,ranks from ranking
where ranks<=3

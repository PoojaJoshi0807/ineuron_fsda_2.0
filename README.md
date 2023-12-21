Create database Zomato_Project;
Use Zomato_Project;
Create table Goldusers_Signup
(User_id Int,
 Gold_Signup_Date date
);
Insert into Goldusers_Signup values 
('1', '2017-09-22'),
('3', '2017-04-21');
Create table Product 
(Product_id Int,
 Product_name Varchar(10),
 Price int
);
Insert into Product Values
('1', 'P1', '980'),
('2', 'P2', '870'),
('3', 'P3', '330');
Create table Users
(User_id Int,
 Signup_Date Date);
 Insert into Users Values
 ('1', '2014-09-02'),
 ('2', '2015-01-15'),
 ('3', '2014-04-11');
Create table Sales
(User_id Int,
Created_at Date,
Product_id Int);
Insert into Sales Values 
('1', '2017-04-19', '2'),
('3', '2019-12-18', '1'),
('2', '2020-07-20', '3'),
('1', '2019-10-23', '2'),
('1', '2018-03-19', '3'),
('3', '2016-12-20', '2'),
('1', '2016-11-09', '1'),
('1', '2016-05-20', '3'),
('2', '2017-09-24', '1'),
('1', '2017-03-11', '2'),
('1', '2016-03-11', '1'),
('3', '2016-11-10', '1'),
('3', '2017-12-07', '2'),
('3', '2016-12-15', '2'),
('2', '2017-11-08', '2'),
('2', '2018-09-10', '3');
Select * from Goldusers_Signup;
Select * from Product;
Select * from Sales;
Select * from Users;
Drop table Master_Zomato_Project;
Create table Master_Zomato_Project (Select S.User_id, S.Created_at, U.Signup_Date, GS.Gold_Signup_Date, 
P.Product_id, P.Product_name, P.Price from Sales as S 
left outer join Goldusers_Signup as GS on S.User_id=GS.User_id
left outer join Product as P on S.Product_id=P.Product_id
left outer join Users as U on S.User_id=U.User_id order by S.User_id);
Select * from Master_Zomato_Project;
-- 1. What is the total amount each customer spent on zomato?
Select User_id as Customer, Sum(Price) as Amount_Spent from Master_Zomato_Project 
Group by User_id
Order by User_id;
-- 2. How many days has each customer visited zomato?
Select User_id as Customer, Count(Distinct Created_at) as Customer_Visit from Master_Zomato_Project
Group by User_id
Order by User_id;
-- 3. What was the first product purchased by each customer?
Select A.Customer, A.DATE_, A.Product_id, A.Product_name from (Select User_id as Customer, Created_at as DATE_, Product_id, Product_name, 
rank() over(partition by User_id order by Created_at) as RN 
from Master_Zomato_Project) as A where A.RN=1;
-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
Select User_id, Count(Product_id) as CNT from Master_Zomato_Project where 
Product_id = '2'
Group by User_id
Order by Product_id;
Select Product_id, Count(Product_id) from Master_Zomato_Project 
Group by Product_id
Order by Count(Product_id) desc limit 1;	
-- 5. Which item was the most popular for each customer?
Select B.User_id, B.Product_id, B.CNT from (Select *, Rank() Over(Partition by A.User_id order by A.CNT desc) as RNK from 
(Select User_id, Product_id, Count(Product_id) as CNT
from Master_Zomato_Project
Group by User_id, Product_id
Order by User_id, Count(Product_id))as A) as B where B.RNK=1;
-- 6. Which items was purchased first by the customer after they became a member?
Select * from (Select A.*, Rank() Over(Partition by User_id order by Created_at) as RNK from 
(Select User_id, Created_at, Gold_Signup_Date, Product_id, Price
Product_name from Master_Zomato_Project where Created_at>Gold_Signup_Date) as A) as B where B.RNK=1;
-- 7. Which items was purchased just before the customer became a member?
Select * from (Select A.*, Rank() Over(Partition by User_id order by Created_at Desc) as RNK from 
(Select User_id, Created_at, Gold_Signup_Date, Product_id, 
Product_name from Master_Zomato_Project where Created_at<=Gold_Signup_Date) as A) as B where B.RNK=1;
-- 8. What is the total orders and amount spent for each member before they became a member?
Select User_id as Customers, Count(Created_at) as Total_orders, Sum(Price) as Amount_Spent from 
Master_Zomato_Project where Created_at<=Gold_Signup_Date
Group by User_id
Order by User_id;
-- 9. If buying each product generates points for e.g. 5rs= 2 Zomato point and each product has different purchasing points for e.g., for
-- p1 5rs= 1 Zomato Point, for p2 10rs= 5 Zomato Point and p3 5rs= 1 Zomato Point, Calculate points collected by each customers and for 
-- which product most points have been given till now.
Select B.Customer, Sum(B.Total_Points) as Total_Points_Earned_By_Customer from (Select A.*, A.Total_Spent_Amount/ A.Buying_Points as 
Total_Points from
(Select User_id as Customer, Product_id, Sum(Price) as Total_Spent_Amount, 
	Case when Product_id= 1 then 5 
		when Product_id= 2 then 2 
		when Product_id= 3 then 5 else 0 
		end as Buying_Points from Master_Zomato_Project
	Group by User_id, Product_id 
	Order by User_id) as A) as B
Group by B.Customer;
Select D.* from (Select C.Product_id, C.Total_Points_Earned_By_Product, Rank() Over(Order by C.Total_Points_Earned_By_Product desc) 
as RNK from (Select B.Product_id, Sum(B.Total_Points) as 
Total_Points_Earned_By_Product from (Select A.*, A.Total_Spent_Amount/ A.Buying_Points as 
Total_Points from
(Select User_id as Customer, Product_id, Sum(Price) as Total_Spent_Amount, 
	Case when Product_id= 1 then 5 
		when Product_id= 2 then 2 
		when Product_id= 3 then 5 else 0 
		end as Buying_Points from Master_Zomato_Project
	Group by User_id, Product_id 
	Order by User_id) as A) as B
Group by B.Product_id) as C) as D where D.RNK=1;
-- 10. In the first one year after a customer joins the gold program (including their join date) irrespective of what the customer has 
-- purchased they earn 5 zomato points for every 10rs spent who earned more 1 or 3 and what was their points earnings in their first year?
Select User_id, Created_at, Gold_Signup_Date, Product_id, Price, Product_name, Price*0.5 as Points from Master_Zomato_Project 
where Created_at>=Gold_Signup_Date and Created_at<=AddDate(Gold_Signup_Date, Interval 1 Year);
-- 11. Rank all the Transaction of the customers?
Select *, Rank() Over(Partition by User_id Order by Created_at) as RNK from Sales;
-- 12. Rank all the transactions for each member whenever they are a zomato gold member for every non gold member transaction mark as NA.
Select B.User_id, B.Created_at, B.Product_id, B.Gold_Signup_Date, Case when B.RNK=0 then 'NA' else B.RNK end as RNK 
from (Select A.*, Case when Gold_Signup_Date is Null then 0 else rank() over(Partition by User_id Order by Created_at Desc) End as RNK 
from(Select A.User_id, A.Created_At, A.Product_id, B.Gold_Signup_Date from Sales as A Left join
Goldusers_Signup as B on A.User_id=B.User_id and Created_at>=Gold_Signup_Date) as A) as B;

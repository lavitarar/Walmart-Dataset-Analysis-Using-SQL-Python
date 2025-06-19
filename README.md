# Walmart-Dataset-Analysis-Using-SQL-Python

# This project analyzes Walmart sales data to uncover trends in payment methods, customer ratings, revenue, and time-based sales. It identifies top-performing categories per branch, reveals branch-specific customer behavior, and evaluates revenue changes year-over-year.

---------------------------------- Business Problem ------------------------------------------------

# 1 : Find the diffent payment method , number of transactions and Number of quantity sold ?
select 
    distinct payment_method,
    count(*) as No_of_transaction,
    sum(quantity) as No_of_Quantity
from walmart
	group by payment_method;

#  2 Project Question - 2  
# Identify the highest rated category in each branch, displaying the branch category avg rating ?
with Table_2 as 
( 
select branch,
     category,
     round(avg(Rating),2) as Avg_Rating,
     rank() over (partition by branch order  by avg(Rating) desc) as Ranking
          from walmart 
		group by branch , category 
) 
select * from table_2 where ranking=1 ;

# 4 Calculate the total Quantity of items sold per payment method. List Payment method and total Quantity

select  
      payment_method,
      sum(quantity) as Total_Quantity
from walmart
    Group by payment_method
    order by Total_Quantity desc;

# 5 Detemine the average ,Minimum and maximum rating of category for each city ?
select 
    city,
    min(rating) as Minimum_Rating,
    max(Rating) as Maximum_Rating,
    round(avg(rating),2) as Average_Rating
 From walmart 
      Group by city ,category
      order by Average_Rating desc;
      
# 6 Calculate the total profit for each catgory by  total_Profit , ordered from highest to lowest ?
SELECT 
    category,
    ROUND(SUM(Total), 0) AS Total_Revenue,
    ROUND(SUM(profit_margin * Total), 0) AS Profit
FROM walmart
GROUP BY category;

#                                                Intermidiate Level

# 7 Determine the most common payment method for each branch display branch and the preferred_payment method.
with cte as (
select 
branch,
payment_method,
count(*) as No_Of_count,
rank() over (partition by Branch order by count(*) desc) as Ranking
 from walmart
 group by branch , payment_method
 ) 
 select * from cte where ranking =1;
 
 # 8 categorize sales into 3 group MORNING , AFTERNOON , EVENING  find out which of the Branch , shift  and number of invoices.
 select
 Branch,
case 
    when extract(hour from (time(time))) < 12 then 'Morning'
    when extract(hour from (time(time))) between 12 and 17 then 'Afternoon'
    else 'Evening' end as Day_time,
    count(*) as No_of_Invoice
from walmart
group by day_time , Branch;

#                                                 Advanced Level

# Identify 5 Branch with highest level decrease ratio in revenue compare to last year (Current year 2023 last year 2022)

 with revenu_2022 as (
 select 
branch,
Round(sum(total),0) as Revenue_last
 from walmart
 where year(date)=2022
 group by branch
 ) , revenue_2023 as (
 select 
 branch,
 Round(sum(total),0) as Revenue_Cr
 from walmart 
 where year(date)=2023
 group by branch 
 )
 select 
 R1.branch,
 R1.Revenue_last as Last_year_Revenue,
 R2.Revenue_Cr as Cr_year_Revenue,
 Round((R1.revenue_last - R2.Revenue_cr) / R1.Revenue_last * 100,2) as Ratio_Dec_Percentage
 from revenu_2022 as R1 join revenue_2023 as R2
 on (R1.branch = R2.branch)
 where R1.revenue_last> R2.revenue_cr
 group by branch
 order by Ratio_Dec_Percentage desc


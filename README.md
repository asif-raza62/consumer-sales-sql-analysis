# consumer-sales-sql-analysis
 SQL project analyzing consumer domain data to uncover trends, growth insights, and business performance
# Project Overview
Atliq Hardware aimed to strengthen its data-driven decision-making by analyzing key business problems. In this project, 10 ad-hoc requests were solved using SQL, delivering valuable insights to support strategic business decisions


# Key insights
1.  Atliq Exclusive operates in 8 APAC markets, with India having the highest presence (2 customers), while other markets have 1 customer each, showing a higher concentration in India.

2. Unique products increased from 245 (2020) to 334 (2021), showing a 36.33% growth, indicating a strong expansion in product portfolio

3. Notebook has the highest products (129), followed by Accessories (116), while other segments have lower counts
    
4. Accessories segment showed the highest growth (~38.2%), followed by Notebook & Peripherals (~18%), making them the top growing segments in     2021 vs 2020.

5. Some products are very expensive to make ($240.54) while others are very cheap ($0.89), showing a big difference in cost

6. The average discount is nearly 30% for all top customers, with Amazon slightly lower (29.33%), indicating consistent discounting across         customers
7. Sales increased significantly in FY2021 compared to FY2020, driven by higher demand, an increase in product offerings, and overall             business    expansion, with March 2021 showing the highest    performance.

8. In 2020, Q1 recorded the highest total sold quantity (7M), followed by Q2 (6.6M). Sales declined significantly in Q3 (2.1M), but showed a recovery in Q4 (5M), indicating a mid-year dip followed by partial recovery

9. The Retailer channel dominates gross sales in FY2021, contributing over 73%, while Direct and Distributor channels have significantly          lower contributions.
10. N & S division dominates sales (~700K units), followed by P & A (~420K units), while the PC division has very low sales (~17K units), showing a significant demand gap across divisions

# Ad-Hoc Requests
1. Provide the list of markets in which customer "Atliq Exclusive" operates its
business in the APAC region.
```sql
SELECT 
	distinct(market), count(customer_code)
FROM dim_customer
WHERE customer = "Atliq Exclusive" AND region = "APAC"
group by market;
```
2. What is the percentage of unique product increase in 2021 vs. 2020? The
final output contains these fields,
unique_products_2020,
unique_products_2021,
percentage_chg.
```sql
  with x as(select
		count(distinct case when fiscal_year = 2020 then product_code end) as unique_product_2020,
		count(distinct case when fiscal_year = 2021 then product_code end) as unique_product_2021
 from fact_sales_monthly ),
y as (  select *,
		(unique_product_2021 - unique_product_2020) as new_product_in_2021
from x )
select *,
		round(new_product_in_2021/unique_product_2020,2)*100 as increase_product_pct_in_2021
 from y	;
 ```
3. Provide a report with all the unique product counts for each segment and
sort them in descending order of product counts. The final output contains
2 fields,
segment,
product_count.
```sql
select segment , count(distinct (product_code)) as unique_p
from dim_product
group by segment order by unique_p desc;
```
4. Follow-up: Which segment had the most increase in unique products in
2021 vs 2020? The final output contains these fields,
segment,
product_count_2020,
product_count_2021,
difference.
```sql
with x as (
select p.segment,
	   count(distinct case when s.fiscal_year =2020 then p.product_code end) as p_2020,
       count(distinct case when s.fiscal_year =2021 then p.product_code end) as p_2021
from dim_product p
join fact_sales_monthly s 
on p.product_code = s.product_code
group by p.segment),
y as( select * , (p_2021-p_2020) as difference 
from x)
select* from y;
```
5. Get the products that have the highest and lowest manufacturing costs.
The final output should contain these fields,
product_code,
product,
manufacturing_cost.
```sql
with x as(
select   p.product_code,
		p.product,
        c.manufacturing_cost as cost
 from dim_product p 
join fact_manufacturing_cost c  on
c.product_code = p.product_code )
select *
from x
where 	cost = (select max(cost) from x ) or
		cost = (select min(cost) from x);
```
6. Generate a report which contains the top 5 customers who received an
average high pre_invoice_discount_pct for the fiscal year 2021 and in the
Indian market. The final output contains these fields,
customer_code,
customer,
average_discount_percentage.
```sql
select c.customer,
		c.customer_code,
        avg(p.pre_invoice_discount_pct) as avg_dis_pct

from dim_customer c
join fact_pre_invoice_deductions p  on
c.customer_code = p.customer_code
where c.market = 'india' and p.fiscal_year = 2021
group by c.customer,
		c.customer_code
order by avg_dis_pct desc 
limit 5;
```
7. Get the complete report of the Gross sales amount for the customer “Atliq
Exclusive” for each month. This analysis helps to get an idea of low and
high-performing months and take strategic decisions.
The final report contains these columns:
Month,
Year,
Gross sales Amount.
```sql
select monthname(s.date) as month,
		s.fiscal_year as year,
        round(sum(s.sold_quantity * g.gross_price)/1000000,2) as total_gross_mln
 from fact_sales_monthly s 
join fact_gross_price g on
			g.product_code = s.product_code
join dim_customer c  on 
			c.customer_code = s.customer_code
where c.customer = 'Atliq Exclusive'
group by  month,year
order by  year;
```
8. In which quarter of 2020, got the maximum total_sold_quantity? The final
output contains these fields sorted by the total_sold_quantity,
Quarter,
total_sold_quantity.
```sql
select  fiscal_year,
			case
			when month(date) in (9,10,11) then "Q1" 
            when month(date)  in (12,1,2) then "Q2" 
            when month (Date)  in (3,4,5) then "Q3"
            else "Q4" 
end as quarter,
			sum(sold_quantity) as total_qty
 from fact_sales_monthly
 where fiscal_year =2020
group by quarter;
```
9. Which channel helped to bring more gross sales in the fiscal year 2021
and the percentage of contribution? The final output contains these fields,
channel,
gross_sales_mln,
percentage.
```sql
WITH x AS (
	SELECT c.channel,
		ROUND(SUM(s.sold_quantity * g.gross_price)/1000000,2)as gross_sales_mln
	FROM fact_sales_monthly s
	JOIN fact_gross_price g
	ON s.product_code = g.product_code
	JOIN dim_customer c
	ON s.customer_code = c.customer_code
    WHERE s.fiscal_year = 2021
	GROUP BY c.channel)
SELECT*,
    ROUND(100*gross_sales_mln/SUM(gross_sales_mln) OVER(),2) AS percentage
FROM x
GROUP BY channel
ORDER BY percentage DESC;
```
10. Get the Top 3 products in each division that have a high
total_sold_quantity in the fiscal_year 2021? The final output contains these
fields,
division,
product_code,
product,
total_sold_quantity,
rank_order.
```sql
with x as (
select p.division,p.product_code,p.product ,
				sum(s.sold_quantity) as total_qty
from fact_sales_monthly s 
join dim_product p on
		s.product_code = p.product_code 
		where s.fiscal_year = 2021
		group by p.division,p.product_code,p.product),
y as (
	select *,
			dense_rank()  over(partition by division order by total_qty desc) as rank_order
	from x)
select * from y
where rank_order <=3;
```

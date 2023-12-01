# Finance-and-Supply-Chain-Analytics

# Project Overview

## Problem Statement
- Large Excel files are causing performance issues, leading to slow and inefficient processes.
- AtliQ Hardware aims to address this challenge by expanding its data analytics team, hiring junior data analysts.
- Utilizing MySQL as the database management system, the team aims to analyze data, comprehend the company's financial standing, and extract valuable insights.
- The ultimate goal is to enhance decision-making, optimize operations, and improve overall performance.

## About ATLIQ Hardware
AtliQ Hardware is a renowned hardware company specializing in PCs, printers, mice, and computers. Their products are globally recognized and available at Croma and Best Buy stores, as well as online on Amazon and Flipkart.

## Project Overview
- The project's objective is to extract valuable information from the provided database.
- The database includes details about sales, products, customers, and regions for AtliQ Hardware.
- Specific questions related to sales reports, market analysis, customer behavior, and predicting supply chain needs will be addressed.

### 1. Croma India Product Wise Sales Report for Fiscal Year -2021

```
Select 
 s.date, s.product_code, p.product, p.variant, s.sold_quantity,
 g.gross_price, ROUND(g.gross_price*s.sold_quantity,2) as gross_price_total
from fact_sales_monthly s
join dim_product p
on p.product_code= s.product_code
join fact_gross_price g
on g.product_code= s.product_code
and g.fiscal_year= get_fiscal_year(s.date)
where 
  customer_code=90002002 AND
            get_fiscal_year(date)=2021 
	ORDER BY date asc;
```

### 2. Total Gross Sales Amount Report For Croma - By Monthly

```
select 
YEAR(s.date) AS year,
monthname(s.date) AS month,
SUM(g.gross_price*s.sold_quantity) as gross_price_total
from fact_sales_monthly s
join fact_gross_price g
on g.product_code= s.product_code
and g.fiscal_year= get_fiscal_year(s.date)
where customer_code = 90002002
group by s.date
order by s.date asc;
```

### 3. Total Gross Sales Amount Report For Croma - By Fiscal Year

```
select
get_fiscal_year(date) as fiscal_year,
round(SUM(g.gross_price*s.sold_quantity)/1000000,2) as yearly_sales_in_mln
from fact_sales_monthly s
join fact_gross_price g
on 
 g.fiscal_year=get_fiscal_year(s.date) and
 g.product_code=s.product_code
where
	customer_code=90002002
group by get_fiscal_year(date)
order by fiscal_year;
```

### 4. Top 5 Customers for the Financial Year - 2021

```
select
c.customer, round(sum(net_sales)/1000000,2)  as net_sales_mln
FROM net_sales n
join dim_customer c
on n.customer_code=c.customer_code
where fiscal_year=2021
group by c.customer
order by net_sales_mln desc
limit 5;
```

### 5. Top 5 Products for the Financial Year - 2021

```
select
product, round(sum(net_sales)/1000000,2)  as net_sales_mln
FROM net_sales
where fiscal_year=2021
group by product
order by net_sales_mln desc
limit 5;
```

### 6. Top 5 Markets for the Financial Year - 2021

```
select 
market, round(sum(net_sales)/1000000,2)  as net_sales_mln
FROM net_sales
where fiscal_year=2021
group by market
order by net_sales_mln desc
limit 5;
```

### 7. Top 10 Customers by Net Sales % in Financial Year - 2021

```
with cte1 as (
select 
customer, 
round(sum(net_sales)/1000000,2) as net_sales_mln
from net_sales s
join dim_customer c
on s.customer_code=c.customer_code
where 
s.fiscal_year=2021
group by customer)

select *,
 net_sales_mln*100/sum(net_sales_mln) over()as net_sales_pct 
 from cte1
order by net_sales_pct  desc
limit 10;
```

### 8. Top 10 Customers by Net Sales % in APAC Region & Financial Year - 2021

```
with cte1 as (
select 
c.customer, c.region,
round(sum(net_sales)/1000000,2) as net_sales_mln
from net_sales s
join dim_customer c
on s.customer_code=c.customer_code
and s.market=c.market
where 
s.fiscal_year=2021 AND c.region = "APAC"
group by c.region, c.customer)

select *,
 net_sales_mln*100/sum(net_sales_mln) over(partition by region )as pct_share_region
 from cte1
order by region,pct_share_region desc
limit 10;
```

### 9. Top 2 Markets in Across Region w.r.t their Gross Sales in Financial Year - 2021

```
SET SQL_MODE="";
with cte1 as 
		(select
                     c.market,
                     c.region,
                     round(sum(gross_price_total)/1000000,2) as gross_sales_mln
                from gross_sales g
                join dim_customer c 
                      on c.customer_code=g.customer_code
                where fiscal_year=2021
                group by market),
           cte2 as 
	        (select 
                     *,
                     dense_rank() over (partition by region order by gross_sales_mln desc) as drnk
                from cte1)
	select * from cte2 where drnk<=2;
```

### 10. Supply Chain Statistics Financial Year - 2021

```
SET SQL_MODE="";
WITH forecast_err_table as (
SELECT 
 s.customer_code,
 sum(s.sold_quantity) as total_sold_qty,
 sum(s.forecast_quantity) as total_forecast_qty,
 ROUND(SUM((forecast_quantity- sold_quantity)),2) as net_err,
 ROUND(SUM((forecast_quantity- sold_quantity))*100/ SUM(forecast_quantity),2)as net_err_pct,
 ROUND(SUM(abs(forecast_quantity- sold_quantity)),2) as abs_err,
 ROUND(SUM(abs(forecast_quantity- sold_quantity))*100/ SUM(forecast_quantity),2) as abs_err_pct
FROM gdb0041.fact_act_est s
where s.fiscal_year=2021
group by s.customer_code 
order by abs_err_pct desc)

Select 
e.*,  c.customer, c.market,
 if(abs_err_pct > 100, 0 , (100- abs_err_pct)) as forecast_accuracy
from forecast_err_table e
join dim_customer c
using (customer_code)
order by forecast_accuracy desc;
```




































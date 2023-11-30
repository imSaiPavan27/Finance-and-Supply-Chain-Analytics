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

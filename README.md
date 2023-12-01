# Finance-and-Supply-Chain-Analytics

### **Link to  Presentaion** : [Linkedin Post]([https://www.linkedin.com/feed/update/urn:li:activity:7117530718058008576/](https://www.linkedin.com/posts/saipavankumar27_presentation-activity-7135624283686408192-neDB?utm_source=share&utm_medium=member_desktop))


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

## General Queries

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

## Stored Procedures

### 1. Finding forecast accuracy for a given year

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_forecast_accuracy`(
in_fiscal_year INT
)
BEGIN
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
where s.fiscal_year=in_fiscal_year
group by s.customer_code 
order by abs_err_pct desc)

Select 
e.*,  c.customer, c.market,
 if(abs_err_pct > 100, 0 , (100- abs_err_pct)) as forecast_accuracy
from forecast_err_table e
join dim_customer c
using (customer_code)
order by forecast_accuracy desc;
 


END
```

### 2. determine the market badge if tota1_qty > 5 million, then gold  else it is a silver by market and year

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_market_badge`(
IN in_market varchar(45), 
IN in_fiscal_year year,
OUT out_badge varchar(45)
)
BEGIN
	declare total_qty int default  0;
    
    # set default market to be in india
    
    if in_market="" then
     set in_market= "India";
	end if;
    
    # retrieve total qty for a given market and fical year
    
    select 
	 SUM(sold_quantity) into total_qty
	from fact_sales_monthly s
	join dim_customer c
	on s.customer_code=c.customer_code
	where get_fiscal_year(s.date)=in_fiscal_year
	and c.market= in_market
	group by c.market	;
    
    # determine the market badge if tota_qty > 5 million, then gold 
    # else it is a silver
    
    if total_qty > 5000000 then
     set out_badge = "Gold";
	else 
     set out_badge = "Silver";
    end if; 
END
```

### 3. Getting monthly gross sales for customers

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_monthly_gross_sales_for_customer`(in_customer_codes TEXT)
BEGIN
select s.date,SUM(Round(g.gross_price*s.sold_quantity,2)) as monthly_sales
from fact_sales_monthly s
join fact_gross_price g
on g.product_code= s.product_code
and g.fiscal_year= get_fiscal_year(s.date)
where FIND_IN_SET(s.customer_code, in_customer_codes)>0
group by date;

END
```

### 4. Determining top N customers w.r.t net sales using fiscal year 

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_customers_by_net_sales`(
in_fiscal_year INT,
in_top_n INT
)
BEGIN
	select 
	c.customer, round(sum(net_sales)/1000000,2)  as net_sales_mln
	FROM net_sales n
    join dim_customer c
    on n.customer_code=c.customer_code
	where fiscal_year= in_fiscal_year
	group by c.customer
	order by net_sales_mln desc
	limit in_top_n;
END
```

### 5. Determining Top N customers w.r.t market and net sales 

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_customers_w.r.t_market_by_net_sales`(
        	in_market VARCHAR(45),
        	in_fiscal_year INT,
    		in_top_n INT
	)
BEGIN
        	select 
                     customer, 
                     round(sum(net_sales)/1000000,2) as net_sales_mln
        	from net_sales s
        	join dim_customer c
                on s.customer_code=c.customer_code
        	where 
		    s.fiscal_year=in_fiscal_year 
		    and s.market=in_market
        	group by customer
        	order by net_sales_mln desc
        	limit in_top_n;
	END
```

### 6. Determining Top N markets by net sales

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_markets_by_net_sales`(
in_fiscal_year INT,
in_top_n INT
)
BEGIN
	select 
	market, round(sum(net_sales)/1000000,2)  as net_sales_mln
	FROM net_sales
	where fiscal_year= in_fiscal_year
	group by market
	order by net_sales_mln desc
	limit in_top_n;
END
```

### 7. Determining Top N products by  net sales

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_products_by_net_sales`(
in_fiscal_year INT,
in_top_n INT
)
BEGIN
	select 
	product, round(sum(net_sales)/1000000,2)  as net_sales_mln
	FROM net_sales
	where fiscal_year= in_fiscal_year
	group by product
	order by net_sales_mln desc
	limit in_top_n;
END
```

### 8. Determining Top N products per Division by Sold Quantity

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_products_per_division_by_qty_sold`(
in_fiscal_year INT,
in_top_n INT
)
BEGIN
with cte1 as 
		(select
                     p.division,
                     p.product,
                     sum(sold_quantity) as total_qty
                from fact_sales_monthly s
                join dim_product p
                      on p.product_code=s.product_code
                where fiscal_year=in_fiscal_year
                group by p.product ),
           cte2 as 
	        (select 
                     *,
                     dense_rank() over (partition by division order by total_qty desc) as drnk
                from cte1)
	select * from cte2 where drnk<=in_top_n;
END
```

### 9. Determining Top N markets by Net Sales 

```
CREATE DEFINER=`root`@`localhost` PROCEDURE `get_top_n_markets_by_net_sales`(
in_fiscal_year INT,
in_top_n INT
)
BEGIN
	select 
	market, round(sum(net_sales)/1000000,2)  as net_sales_mln
	FROM net_sales
	where fiscal_year= in_fiscal_year
	group by market
	order by net_sales_mln desc
	limit in_top_n;
END
```

## Views

### 1. Gross Sales

```
CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
VIEW `gross_sales` AS
    SELECT 
        `s`.`date` AS `date`,
        `s`.`fiscal_year` AS `fiscal_year`,
        `c`.`market` AS `market`,
        `s`.`customer_code` AS `customer_code`,
        `s`.`product_code` AS `product_code`,
        `p`.`product` AS `product`,
        `p`.`variant` AS `variant`,
        `s`.`sold_quantity` AS `sold_quantity`,
        `g`.`gross_price` AS `gross_price_per_item`,
        ROUND((`s`.`sold_quantity` * `g`.`gross_price`),
                2) AS `gross_price_total`
    FROM
        ((((`fact_sales_monthly` `s`
        JOIN `dim_customer` `c` ON ((`s`.`customer_code` = `c`.`customer_code`)))
        JOIN `dim_product` `p` ON ((`s`.`product_code` = `p`.`product_code`)))
        JOIN `fact_gross_price` `g` ON (((`g`.`fiscal_year` = `s`.`fiscal_year`)
            AND (`g`.`product_code` = `s`.`product_code`))))
        JOIN `fact_pre_invoice_deductions` `pre` ON (((`pre`.`customer_code` = `s`.`customer_code`)
            AND (`pre`.`fiscal_year` = `s`.`fiscal_year`))))
```

### 2. Net Sales 

```
CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
VIEW `net_sales` AS
    SELECT 
        `sales_postinv_discount`.`date` AS `date`,
        `sales_postinv_discount`.`fiscal_year` AS `fiscal_year`,
        `sales_postinv_discount`.`customer_code` AS `customer_code`,
        `sales_postinv_discount`.`market` AS `market`,
        `sales_postinv_discount`.`product_code` AS `product_code`,
        `sales_postinv_discount`.`product` AS `product`,
        `sales_postinv_discount`.`variant` AS `variant`,
        `sales_postinv_discount`.`sold_quantity` AS `sold_quantity`,
        `sales_postinv_discount`.`gross_price_total` AS `gross_price_total`,
        `sales_postinv_discount`.`pre_invoice_discount_pct` AS `pre_invoice_discount_pct`,
        `sales_postinv_discount`.`net_invoice_sales` AS `net_invoice_sales`,
        `sales_postinv_discount`.`post_invoice_discount_pct` AS `post_invoice_discount_pct`,
        (`sales_postinv_discount`.`net_invoice_sales` * (1 - `sales_postinv_discount`.`post_invoice_discount_pct`)) AS `net_sales`
    FROM
        `sales_postinv_discount`
```

### 3. Sales ( Post Invoice Discount )

```
CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
VIEW `sales_postinv_discount` AS
    SELECT 
        `s`.`date` AS `date`,
        `s`.`fiscal_year` AS `fiscal_year`,
        `s`.`customer_code` AS `customer_code`,
        `s`.`market` AS `market`,
        `s`.`product_code` AS `product_code`,
        `s`.`product` AS `product`,
        `s`.`variant` AS `variant`,
        `s`.`sold_quantity` AS `sold_quantity`,
        `s`.`gross_price_total` AS `gross_price_total`,
        `s`.`pre_invoice_discount_pct` AS `pre_invoice_discount_pct`,
        (`s`.`gross_price_total` - (`s`.`pre_invoice_discount_pct` * `s`.`gross_price_total`)) AS `net_invoice_sales`,
        (`po`.`discounts_pct` + `po`.`other_deductions_pct`) AS `post_invoice_discount_pct`
    FROM
        (`sales_preinv_discount` `s`
        JOIN `fact_post_invoice_deductions` `po` ON (((`po`.`customer_code` = `s`.`customer_code`)
            AND (`po`.`product_code` = `s`.`product_code`)
            AND (`po`.`date` = `s`.`date`))))
```

### 4. Sales ( Pre Invoice Discount )

```
CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
VIEW `sales_preinv_discount` AS
    SELECT 
        `s`.`date` AS `date`,
        `s`.`fiscal_year` AS `fiscal_year`,
        `s`.`customer_code` AS `customer_code`,
        `c`.`market` AS `market`,
        `s`.`product_code` AS `product_code`,
        `p`.`product` AS `product`,
        `p`.`variant` AS `variant`,
        `s`.`sold_quantity` AS `sold_quantity`,
        `g`.`gross_price` AS `gross_price_per_item`,
        ROUND((`s`.`sold_quantity` * `g`.`gross_price`),
                2) AS `gross_price_total`,
        `pre`.`pre_invoice_discount_pct` AS `pre_invoice_discount_pct`
    FROM
        ((((`fact_sales_monthly` `s`
        JOIN `dim_customer` `c` ON ((`s`.`customer_code` = `c`.`customer_code`)))
        JOIN `dim_product` `p` ON ((`s`.`product_code` = `p`.`product_code`)))
        JOIN `fact_gross_price` `g` ON (((`g`.`fiscal_year` = `s`.`fiscal_year`)
            AND (`g`.`product_code` = `s`.`product_code`))))
        JOIN `fact_pre_invoice_deductions` `pre` ON (((`pre`.`customer_code` = `s`.`customer_code`)
            AND (`pre`.`fiscal_year` = `s`.`fiscal_year`))))
```

## Functions

### 1. Determining Fiscal Quarter

```
CREATE DEFINER=`root`@`localhost` FUNCTION `get_fiscal_quarter`(
calendar_date date
) RETURNS char(2) CHARSET utf8mb4
    DETERMINISTIC
BEGIN
  declare m TINYINT;
  declare qtr CHAR(2);
  SET m=month(calendar_date) ;
  
  CASE
    when m in (9,10,11) then
      SET qtr= "Q1";
	when m in (12,1,2) then
      SET qtr= "Q2";
	when m in (3,4,5) then
      SET qtr= "Q3";
	when m in (6,7,8) then
      SET qtr= "Q4";
  END CASE;
return qtr;
END
```

### 2. Determing Fiscal year

```
CREATE DEFINER=`root`@`localhost` FUNCTION `get_fiscal_year`(
calendar_date date
) RETURNS int
    DETERMINISTIC
BEGIN
  declare fiscal_year INT;
  SET fiscal_year=YEAR(DATE_ADD(calendar_date, INTERVAL 4 MONTH)) ;
  return fiscal_year;
END
```

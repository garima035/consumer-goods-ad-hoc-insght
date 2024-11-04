#CODEBASICS_SQL_PROJECT_CHALLENGE #CONSUMER_GOODS_DOMAIN


#Task 1:

SELECT * FROM gdb023.dim_customer WHERE customer = 'Atliq Exclusive' AND region = 'APAC';

#Task 2: WITH YearlyProductCounts AS ( SELECT EXTRACT(YEAR FROM date) AS year, COUNT(DISTINCT product_code) AS unique_products_count FROM fact_sales_monthly WHERE EXTRACT(YEAR FROM date) IN (2020, 2021) GROUP BY EXTRACT(YEAR FROM date) )

SELECT ypc_2020.unique_products_count AS unique_products_2020, ypc_2021.unique_products_count AS unique_products_2021, ROUND(((ypc_2021.unique_products_count - ypc_2020.unique_products_count) / ypc_2020.unique_products_count) * 100, 2) AS percentage_chg FROM YearlyProductCounts ypc_2020 CROSS JOIN YearlyProductCounts ypc_2021 WHERE ypc_2020.year = 2020 AND ypc_2021.year = 2021;

#Task 3:

SELECT segment, COUNT(DISTINCT product_code) AS product_count FROM dim_product GROUP BY segment ORDER BY product_count DESC;

#Task 4:

WITH temp_table AS ( SELECT p.segment, s.fiscal_year, COUNT(DISTINCT s.Product_code) as product_count FROM fact_sales_monthly s JOIN dim_product p ON s.product_code = p.product_code GROUP BY p.segment, s.fiscal_year ) SELECT up_2020.segment, up_2020.product_count as product_count_2020, up_2021.product_count as product_count_2021, up_2021.product_count - up_2020.product_count as difference FROM temp_table as up_2020 JOIN temp_table as up_2021 ON up_2020.segment = up_2021.segment AND up_2020.fiscal_year = 2020 AND up_2021.fiscal_year = 2021 ORDER BY difference DESC;

#Task 5:

SELECT dp.product_code, dp.product, fmc.manufacturing_cost FROM dim_product dp JOIN fact_manufacturing_cost fmc ON dp.product_code = fmc.product_code WHERE fmc.manufacturing_cost = (SELECT MAX(manufacturing_cost) FROM fact_manufacturing_cost) OR fmc.manufacturing_cost = (SELECT MIN(manufacturing_cost) FROM fact_manufacturing_cost);

#Task 6:

SELECT c.customer_code, c.customer, round(AVG(pre_invoice_discount_pct),4) AS average_discount_percentage FROM fact_pre_invoice_deductions d JOIN dim_customer c ON d.customer_code = c.customer_code WHERE c.market = "India" AND fiscal_year = "2021" GROUP BY customer_code ORDER BY average_discount_percentage DESC LIMIT 5;

#Task 7: WITH temp_table AS ( SELECT customer, monthname(date) AS months , month(date) AS month_number, year(date) AS year, (sold_quantity * gross_price) AS gross_sales FROM fact_sales_monthly s JOIN fact_gross_price g ON s.product_code = g.product_code JOIN dim_customer c ON s.customer_code=c.customer_code WHERE customer="Atliq exclusive" ) SELECT months,year, concat(round(sum(gross_sales)/1000000,2),"M") AS gross_sales FROM temp_table GROUP BY year,months ORDER BY year,month_number;

#Task 8:

WITH temp_table AS ( SELECT date,month(date_add(date,interval 4 month)) AS period, fiscal_year,sold_quantity FROM fact_sales_monthly ) SELECT CASE when period/3 <= 1 then "Q1" when period/3 <= 2 and period/3 > 1 then "Q2" when period/3 <=3 and period/3 > 2 then "Q3" when period/3 <=4 and period/3 > 3 then "Q4" END quarter, round(sum(sold_quantity)/1000000,2) as total_sold_quanity_in_millions FROM temp_table WHERE fiscal_year = 2020 GROUP BY quarter ORDER BY total_sold_quanity_in_millions DESC ;

#Task 9:

WITH temp_table AS ( SELECT c.channel,sum(s.sold_quantity * g.gross_price) AS total_sales FROM fact_sales_monthly s JOIN fact_gross_price g ON s.product_code = g.product_code JOIN dim_customer c ON s.customer_code = c.customer_code WHERE s.fiscal_year= 2021 GROUP BY c.channel ORDER BY total_sales DESC ) SELECT channel, round(total_sales/1000000,2) AS gross_sales_in_millions, round(total_sales/(sum(total_sales) OVER())*100,2) AS percentage FROM temp_table ;

#Task 10:

WITH temp_table AS ( select division, s.product_code, concat(p.product,"(",p.variant,")") AS product , sum(sold_quantity) AS total_sold_quantity, rank() OVER (partition by division order by sum(sold_quantity) desc) AS rank_order FROM fact_sales_monthly s JOIN dim_product p ON s.product_code = p.product_code WHERE fiscal_year = 2021 GROUP BY product_code ) SELECT * FROM temp_table WHERE rank_order IN (1,2,3);


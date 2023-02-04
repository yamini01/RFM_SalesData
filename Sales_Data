select * from sales_data_sample;

-- Basic understanding of data
select distinct orderlinenumber from sales_data_sample; 
-- 1-18
select distinct status from sales_data_sample; 
-- 6 resolved-on hold-cancelled-shipped-disputed-in process
select distinct qtr_id from sales_data_sample
-- 1,2,3,4
select distinct MONTH_ID from sales_data_sample;
-- 12 months
select distinct YEAR_ID from sales_data_sample;
-- 3 years - 2003,2004,2005
select distinct PRODUCTLINE from sales_data_sample; 
-- trains,motorcycles,ships,trucks and buses,vintage cars,classic cars,planes-automobiles
select distinct CITY from sales_data_sample; --73
select distinct [STATE] from sales_data_sample; --17
select distinct COUNTRY from sales_data_sample; --19 
select distinct TERRITORY from sales_data_sample; -- 4 -- japan,apac,emea,null
select distinct DEALSIZE from sales_data_sample;  --3- large-medium-small;

-- analysis
-- group productline by sales
select productline,sum(sales) from sales_data_sample
group by PRODUCTLINE
order by 2 desc;
-- classic cars- top,trains - least

-- which year they made the most sales
select YEAR_ID,sum(sales) from sales_data_sample
group by YEAR_ID
order by 2 desc;
-- 2004- top, 2005 - least

-- find out why 2005 is having very low sales compared to other years
select distinct month_id
from sales_data_sample
where YEAR_ID = 2005;
-- you can see that only 5 months of the total year sales happened, that is the reason for low total sales

-- which dealsize generates more revenue/sales
select DEALSIZE,sum(sales) 
from sales_data_sample
group by DEALSIZE
order by 2 desc;
-- medium-top, large- least

-- what was the best month for sales in a specific year? how much was earned in that month ?
select distinct year_id,month_id,sum(sales)
over(PARTITION by year_id,month_id) as rev
from sales_data_sample
order by rev desc;

-- 2003 - 11 - 1029837.66
-- 2004 - 11 - 10,89,048.01
-- 2005 - 5 - 457861.26

-- november seems to be the best month, what product is most selling in that month
select distinct year_id, month_id,productline, sum(sales)
over(PARTITION by year_id,productline) as rev
from sales_data_sample
where MONTH_ID=11
order by rev desc;
-- 2003- classic cars - 452924.36, vintagecares- 184673.4
-- 2004 - classic cars  - 372231.89,vintagecares- 233990.34

-- who our best customer is - RFM analysis meaning- Recency-Frequency-Monetary 
drop table if EXISTS #rfm;
with 
rfm as 
 (select 
    customername, 
    sum(sales) as totalrev,
    avg(sales) as avgrev,
    count(sales) as freq,
    max(orderdate) as last_order_date,
    (select max(orderdate) from sales_data_sample) as max,
    DATEDIFF(dd, max(orderdate), (select max(orderdate) from sales_data_sample)) as Recency
    from sales_data_sample
    group by CUSTOMERNAME
 ),
 rfm_calc as 
 (select *,
    ntile(4) over (order by recency desc) rfm_recency,
    ntile(4) over (order by freq) rfm_freq,
    ntile(4) over (order by totalrev) rfm_monetary
    from rfm
 )
 select 
    *,
    rfm_recency+rfm_freq+rfm_monetary as rfm_cell,
    CAST(rfm_recency as varchar) + CAST(rfm_freq as varchar) +CAST(rfm_monetary as varchar) rfm_cell_string
    into #rfm
 from rfm_calc;

 select * from #rfm;

select CUSTOMERNAME , rfm_recency, rfm_freq, rfm_monetary,
	case 
		when rfm_cell_string in (111, 112 , 121, 122, 123, 132, 211, 212, 114, 141) then 'lost_customers'
		when rfm_cell_string in (133, 134, 143, 244, 334, 343, 344, 144) then 'slipping away, cannot lose' 
		when rfm_cell_string in (311, 411, 331) then 'new customers'
		when rfm_cell_string in (222, 223, 233, 322) then 'potential churners'
		when rfm_cell_string in (323, 333,321, 422, 332, 432) then 'active' 
		when rfm_cell_string in (433, 434, 443, 444) then 'loyal'
	end rfm_segment
from #rfm


--What products are most often sold together? 
-- select * from sales_data_sample where ORDERNUMBER=10411
select distinct OrderNumber, stuff(

	(select ',' + PRODUCTCODE
	from [sales_data_sample] p
	where ORDERNUMBER in 
		(

			select ORDERNUMBER
			from (
				select ORDERNUMBER, count(*) rn
				FROM [sales_data_sample]
				where STATUS = 'Shipped'
				group by ORDERNUMBER
			)m
			where rn = 3
		)
		and p.ORDERNUMBER = s.ORDERNUMBER
		for xml path (''))

		, 1, 1, '') ProductCodes

from [sales_data_sample] s
order by 2 desc

--What city has the highest number of sales in a specific country
select city, sum (sales) Revenue
from [sales_data_sample]
where country = 'UK'
group by city
order by 2 desc



---What is the best product in United States?
select country, YEAR_ID, PRODUCTLINE, sum(sales) Revenue
from [sales_data_sample]
where country = 'USA'
group by  country, YEAR_ID, PRODUCTLINE
order by 4 desc


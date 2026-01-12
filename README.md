# zepto_sql_analysis
Zepto SQL Analyst Portfolio Project - E-commerce data

Project Workflow
Database & Table Creation
CREATE TABLE zepto (
  sku_id SERIAL PRIMARY KEY,
  category VARCHAR(120),
  name VARCHAR(150) NOT NULL,
  mrp NUMERIC(8,2),
  discountPercent NUMERIC(5,2),
  availableQuantity INTEGER,
  discountedSellingPrice NUMERIC(8,2),
  weightInGms INTEGER,
  outOfStock BOOLEAN,
  quantity INTEGER
);

#--------------Load dataset into the mysql workbench database------------

SHOW VARIABLES LIKE 'secure_file_priv';

SET GLOBAL local_infile = 1;
LOAD DATA LOCAL INFILE 'C:/Users/Your_path/ZEPTO_data/zepto_dataset.csv'
INTO TABLE zepto
FIELDS TERMINATED BY ','
IGNORE 1 LINES;

# ----------------Find out the null values in the table. 

SELECT * FROM zepto
WHERE category IS NULL
OR name IS NULL
OR mrp IS NULL
OR discountprecent IS NULL
OR discountsellingprice IS NULL
OR weight_in_grams IS NULL
OR out_of_stock IS NULL
OR quantity IS NULL
OR sku_id IS NULL;

# -------------------Find out different product categories----------------------
SELECT DISTINCT category 
FROM zepto
ORDER by category;

# --------------------Change the column data type to varchar--------------------
ALTER TABLE zepto 
MODIFY out_of_stock VARCHAR(50);

# -------------------Change the out_of_stock column in to text remarks
# stock available or out_of_stock   --------------------
SET SQL_SAFE_UPDATES = 0;
UPDATE  zepto
	SET out_of_stock = 
		CASE WHEN availablequantity = 0 then 'Out of stock'
            ELSE 'Stock available'
		END;

# ---------------Find out the stock available or out_of_stock status, note- (IF you want you can add the category also).--------------------
SELECT count(sku_id) as Total_Count, out_of_stock
	FROM zepto
    GROUP BY out_of_stock;

#--- products names present multiple times.
SELECT * FROM zepto limit 10;
SELECT name, count(sku_id) as Total_count
FROM zepto
GROUP BY name
HAVING count(sku_id) >1
ORDER BY COUNT(sku_id) DESC;

#----- Find out the products with price = 0 & remove from table-------------------
SELECT * FROM zepto
WHERE mrp = 0 OR discountsellingprice = 0;
DELETE FROM zepto
WHERE mrp = 0 OR discountsellingprice = 0;

-------------------- conver paises to rupees---------------------
UPDATE zepto
SET mrp = mrp/100.0,
discountsellingprice = discountsellingprice/100.0;

#------------------- Top 10 best value product based on discount precentage
	SELECT DISTINCT name, mrp, discountprecent
	FROM zepto
	ORDER BY discountprecent DESC
	LIMIT 10;
-- Find out the products with HIGH MRP but out of stock
SELECT DISTINCT name, mrp, out_of_stock
FROM zepto
WHERE out_of_stock = 'Out of stock' AND MRP >300
ORDER BY mrp DESC ;

-- -------------- Calculate the Estimated revenue by each category

SELECT category, sum(availablequantity * discountsellingprice) AS total_revenue
FROM zepto 
GROUP BY category
ORDER BY total_revenue DESC;

----------- Find out all products where MRP is greater than 500 and discount is less than 10% ------------
SELECT DISTINCT name, mrp, discountprecent
FROM zepto
WHERE mrp >500 AND discountprecent < 10
ORDER BY mrp DESC, discountprecent DESC;

----------------- Identify the top 5 categories offering the highest average discount average -------------
SELECT category, ROUND(AVG(discountprecent),2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;

----------------------------- Find the price per gram for products above 100 gms  & sort by best values----
# ------------- use this formula price_per_gms = (total_price/total_qty)*desired_quantity

SELECT 
	DISTINCT name, weight_in_grams, discountsellingprice,
    ROUND(discountsellingprice/weight_in_grams, 2) AS price_per_grams
FROM zepto
WHERE weight_in_grams >= 100 
ORDER BY price_per_grams DESC;
 
#------------- Group the products into weight categories like low, medium, bulk --------------
SELECT 
	DISTINCT name, 
    weight_in_grams,
	CASE 
		WHEN weight_in_grams < 1000 THEN 'Low'
		WHEN weight_in_grams < 5000 THEN 'Medium'	
		ELSE 'Bulk'
		END as weight_category
FROM zepto;
    
#---------------- What is the total inventory weight per category -------------

SELECT category, ROUND(SUM(weight_in_grams * availablequantity), 2) AS inventory_weight
FROM zepto
GROUP BY category
ORDER BY inventory_weight

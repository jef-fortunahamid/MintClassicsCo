# Optimising Inventory for Strategic Warehouse Closure at Mint Classics Company

*(This case study is provided by Coursera. The SQL schema file for setting up the Mint Classics database was oprovided.)*

## Business Task
The goal of this project is to explore the company's current inventory and make data-driven recommendations on how to reorganise or if possible, reduce it. The ultimate aim is to close one storage facility while exnsuring  products are still shipped to customers within 24 hours.

## Techniques Used

## Entity-Relations Diagram

![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/84f871e8-2814-4142-a71c-fefa9c1486ac)

## Data Exploration

The SQL schema file, where we used DDL (Data Definition Language) and DML (Data Manipulation Language) file to create the database structure in MySQL workbench, was provided.

Looking at our ER Diagram, there are nine tables in our database. Now, let's explore our `mintclassics` database.

Before we start, on MySQL workbench we need to run the following query:
```sql
USE mintclassics;
```
We need to run this code, if you have more than one database installed on your MySQL workbench (like mine, there are four other databases installed). So we don't have to keep referring the `mintclassics` database on our FROM clause.

#### Getting Familiar with Data

Let's look at the table with atmost important, the `warehouses` table.
```sql
SELECT *
FROM warehouses;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/c9685246-845b-4636-975f-0aacc41a03a1)

Looking at the table, there are four warehouses, where the `warehouseCode` and `warehouseName` are provided for each. The third column is `warehousePctCap`, which we can assume it is the 'Percentage Capacity' of each warehouse. 

We've got 72%, 67%, 50% and 75 % capacity for warehouse a, b, c and d, respectively. Looking at this table, warehouse c has a lot of space to be filled. We need to look at the inventory stock at each warehouse. 

Now, we proceed to explore more of the tables of interest.

### Find Out Where Items are Stored
*products Table*
```sql
SELECT *
FROM products
LIMIT 10;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/4a78c308-0e25-4c1f-8891-ce9010448f01)

With this table, the inventory of our stocks is provided on this table.
```sql
SELECT
      DISTINCT(productCode)
    , productName
FROM products;
```
![Screenshot 2023-09-29 at 8 48 26 am](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/e1e98105-42c4-4f0f-a85f-df5a3b674eee)

There are 110 products on our list.

![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/da0bea0f-f7ff-4629-b8b6-a4b8e70c11fd)

With this information, we need to know if the same products are stored in one or more than one warehouse. So we need to look for the products with the same `productName` but different `warehouseCode` using the following query:
```sql
SELECT
      productCode
    , COUNT(warehouseCode) AS number_of_warehouse
FROM products
GROUP BY 
    productCode
HAVING 
    COUNT(warehouseCode) > 1;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/e0271316-214d-4ded-97b1-157936fef164)

![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/ea217d97-982a-4f27-ad6d-01a8ecb28c62)

There is zero or no row/s or value/s returned with this query. We can say, each warehouse stores unique items.

We explore the total stock stored on each warehosue with the following query:
```sql
SELECT
      w.warehouseName
    , SUM(p.quantityInStock) AS total_stock_stored
FROM products p 
JOIN warehouses w
    ON p.warehouseCode = w.warehouseCode
GROUP BY
    w.warehouseName;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/c3f8a4d6-a060-4bd3-8183-e208bffcee32)

With this new information, warehouse 'd' has the least items in stock. We can consider this warehouse as a posible warehouse for closure, as it has least number of stocks to redistribute, less expense to be made.

### Sales
The next step we will tackle is in terms of sale. We'll need to see the relationship between sales and invetory. This will show us the number of items we have in stock compares to how many are actually being sold.

We will be using the following tables for this: `products`, `orderdetails` and `orders`.
There are a few things we need to tackle on this next query. We need the inventory of each product and the total items sold. However, if we look at the `orders` table, we've got the `status` column. On this column, we've got the values 'Shipped', 'Resolved', 'Cancelled', 'On Hold', 'Disputed', and 'In Process'.
```sql
SELECT 
	status
    , comments
FROM orders
WHERE status IN ('On Hold', 'Disputed','In Process')
ORDER BY status;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/be3dc97c-5c70-44f5-ae05-9157bec4798b)

Looking at the status for 'On Hold', 'Disputed', and 'In Process', we cannot have them included as part of the sales, as well as the 'Cancelled' orders. So we will only be considering 'Shipped' and 'Resolved' orders. 
We will be calculating the difference of the stock at hand and items sold as `diff_stock_vs_sales` and another column to see if these items are 'Overstocked', 'Well-Stocked' and 'Understocked'. We will assume if the `diff_stock_vs_sales` is more than twice the `total_ordered_item` then it's 'Overstocked', if the `diff_stock_vs_sales` is less than 500 then it is 'Understocked' and 'Well-Stocked' for the rest. Our final query is:
```sql
SELECT
      p.productCode
    , p.warehouseCode
    , p.quantityInStock
    , SUM(od.quantityOrdered) AS total_ordered_item
    , p.quantityInStock - SUM(od.quantityOrdered) AS diff_stock_vs_sales
    , CASE 
       WHEN (p.quantityInStock - SUM(od.quantityOrdered)) > (2 * SUM(od.quantityOrdered)) THEN 'Overstocked'
       WHEN (p.quantityInStock - SUM(od.quantityOrdered)) < 500 THEN 'Understocked'
       ELSE 'Well-Stocked'
         END AS inventory_status
FROM products p
JOIN orderdetails od
	ON p.productCode = od.productCode
JOIN orders o
	ON od.orderNumber = o.orderNumber
WHERE
	o.status IN('Shipped', 'Resolved')
GROUP BY 
      productCode
    , quantityInStock
ORDER BY
      warehouseCode
    , diff_stock_vs_sales DESC;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/5fa8cfbc-a7a2-4756-940b-abbe92e1370e)
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/3217ca62-2c0e-4b91-b20c-b1034d4a5892)
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/f688059b-25ca-4635-ad62-f64efd99474c)
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/427feae3-b3e2-45ed-9272-7dd7596fe8dd)

Looking at this result, there are a lot of items that are overstocked. Items that are overstocked can be reduced and this will free up more space for the relocation of stocks from warehouse d. In warehouse a and b, there are 19 and 29 items that are overstocked, respectively.

From this query, there are only 109 rows returned.
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/cd60abba-79bd-45c2-8d0b-f6a053f55fdc)

From our earlier query, there are 110 items on our product line. There is one item that we can drop from our product line. With the following query we can identify which item that has never been ordered:
```sql
WITH cte_inventory_status AS (
	SELECT
		p.productCode
		, p.warehouseCode
		, p.quantityInStock
		, SUM(od.quantityOrdered) AS total_ordered_item
		, p.quantityInStock - SUM(od.quantityOrdered) AS diff_stock_vs_sales
		, CASE 
				WHEN (p.quantityInStock - SUM(od.quantityOrdered)) > (2 * SUM(od.quantityOrdered)) THEN 'Overstocked'
				WHEN (p.quantityInStock - SUM(od.quantityOrdered)) < 500 THEN 'Understocked'
				ELSE 'Well-Stocked'
			END AS inventory_status
	FROM products p
	JOIN orderdetails od
		ON p.productCode = od.productCode
	JOIN orders o
		ON od.orderNumber = o.orderNumber
	WHERE
		o.status IN('Shipped', 'Resolved')
	GROUP BY 
		p.productCode
		, p.warehouseCode
	ORDER BY
		warehouseCode
		, diff_stock_vs_sales DESC
)
SELECT
	productCode
    , productName
FROM products p
WHERE NOT EXISTS (
					SELECT 1
                    FROM cte_inventory_status cis
                    WHERE p.productCode = cis.productCode
);
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/89fe7193-3b13-4860-ad7d-8e921e15c861)

1985 Toyota Supra with the prodcut code of S18_3233 can be dropped from the product line, with stock at hand of 7733 located at warehouse b. This could also free up space.

Items that are overstocked are:
```sql
WITH cte_inventory_status AS (
    SELECT
          p.productCode
        , p.warehouseCode
        , p.quantityInStock
        , SUM(od.quantityOrdered) AS total_ordered_item
        , p.quantityInStock - SUM(od.quantityOrdered) AS diff_stock_vs_sales
        , CASE 
            WHEN (p.quantityInStock - SUM(od.quantityOrdered)) > (2 * SUM(od.quantityOrdered)) THEN 'Overstocked'
            WHEN (p.quantityInStock - SUM(od.quantityOrdered)) < 500 THEN 'Understocked'
            ELSE 'Well-Stocked'
              END AS inventory_status
    FROM products p
    JOIN orderdetails od
        ON p.productCode = od.productCode
    JOIN orders o
        ON od.orderNumber = o.orderNumber
    WHERE
        o.status IN('Shipped', 'Resolved')
    GROUP BY 
          p.productCode
        , p.warehouseCode
    ORDER BY
          warehouseCode
        , diff_stock_vs_sales DESC
)
SELECT
      productCode
    , quantityInStock
    , warehouseCode
FROM cte_inventory_status
WHERE inventory_status = 'Overstocked'
ORDER BY warehouseCode;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/0664bf95-69aa-48f2-ac63-7045b1e55f89)

![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/10ade47a-9b96-47d9-ae18-29ca0359dc11)

![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/1f7e0a99-00d4-47a7-882a-435faa380343)

These are the in-demand items, which are understocked, that needs to be restocked.
```sql
WITH cte_inventory_status AS (
    SELECT
          p.productCode
        , p.warehouseCode
        , p.quantityInStock
        , SUM(od.quantityOrdered) AS total_ordered_item
        , p.quantityInStock - SUM(od.quantityOrdered) AS diff_stock_vs_sales
        , CASE 
            WHEN (p.quantityInStock - SUM(od.quantityOrdered)) > (2 * SUM(od.quantityOrdered)) THEN 'Overstocked'
            WHEN (p.quantityInStock - SUM(od.quantityOrdered)) < 500 THEN 'Understocked'
            ELSE 'Well-Stocked'
              END AS inventory_status
    FROM products p
    JOIN orderdetails od
        ON p.productCode = od.productCode
    JOIN orders o
        ON od.orderNumber = o.orderNumber
    WHERE
        o.status IN('Shipped', 'Resolved')
    GROUP BY 
          p.productCode
        , p.warehouseCode
    ORDER BY
          warehouseCode
        , diff_stock_vs_sales DESC
)
SELECT
      productCode
    , quantityInStock
    , warehouseCode
FROM cte_inventory_status
WHERE inventory_status = 'Understocked'
ORDER BY warehouseCode;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/d08f8b53-0d48-407d-99ed-ba48dab4ec1f)

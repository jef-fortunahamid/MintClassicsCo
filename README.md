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

We've got 72%, 67%, 50% and 75 % capacity for warehouse a, b, c and d, respectively. Looking at this table, warehouse c has a lot of space to be filled. We need to look at the inventory stock at each warehouse. Now, we proceed to explore more of the tables of interest.

*products Table*
```sql
SELECT *
FROM products
LIMIT 10;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/4a78c308-0e25-4c1f-8891-ce9010448f01)


#### Find Out Where Items are Stored
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

With this new information, warehouse 'd' has the least items in stock. We can consider this warehouse as a posible warehouse for closure, as it has least number of stocks to redistribute, less expense to be made

The next step we could tackle is in terms of sale. We can identify items that are selling the least and we can 

#### Sales

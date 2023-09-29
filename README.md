# Optimising Inventory for Strategic Warehouse Closure at Mint Classics Company

*(This case study is provided by Coursera.The SQL schema file for setting up the Mint Classics database was oprovided.)*

## Business Task
The goal of this project is to explore the company's current inventory and make data-driven recommendations on how to reorganise or if possible, reduce it. The ultimate aim is to close one storage facility while exnsuring  products are still shipped to customers within 24 hours.

## Techniques Used

## Entity-Relations Diagram

![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/84f871e8-2814-4142-a71c-fefa9c1486ac)

## Data Exploration

The SQL schema file, where we used DDL (Data Definition Language) and DML (Data Manipulation Language) file to create the database structure in MySQL workbench, was provided.

#### Getting Familiar with Data

Let's look at the table with atmost important, the `warehouses' table.
```sql
SELECT *
FROM warehouses;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/c9685246-845b-4636-975f-0aacc41a03a1)

Looking at the table, there are four warehouses, where the `warehouseCode` and `warehouseName` are provided for each. The third column is `warehousePctCap`, which we can assume it is the 'Percentage Capacity' of each warehouse. Warehouse c (West) has the lowest percentage capacity, where the least number of stocks where we can relocate to the other warehouses. Warehouses a (North), b (East) and d (South) have percent capacity of 72, 67 and 75 percent, respectively. A total of 85 percent free space from all three warehouses.

Unfortunately, we don't have the exact location of each warehouse. We can include this to our 'Reccommendation for Further Study'.

Now, we proceed to explore more of the tables of interest.

*products Table*
```sql
SELECT *
FROM products
LIMIT 10;
```
![image](https://github.com/jef-fortunahamid/MintClassicsCo/assets/125134025/4a78c308-0e25-4c1f-8891-ce9010448f01)


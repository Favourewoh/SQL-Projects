# 🏦: Case Study - Data Bank: Customer Nodes Exploration
## Section A - Exploring Customer Nodes
### Questions
1. How many unique nodes are there on the Data Bank system?
2. What is the number of nodes per region?
3. How many customers are allocated to each region?
4. How many days on average are customers reallocated to a different node?
5. What is the median, 80th, and 95th percentile for this same reallocation days metric for each region?

***

### 1. How many unique nodes are there on the Data Bank system?
```sql
SELECT
    COUNT(DISTINCT node_id) AS unique_nodes
FROM
    data_bank.customer_nodes;
```  
#### Query Result:  
![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/6dcc4145-c549-49ee-8401-152b41e74a3d)


***

### 2. What is the number of nodes per region?
```sql
SELECT
    r.region_id,
    r.region_name, 
    COUNT (DISTINCT node_id) AS no_of_nodes
FROM
    data_bank.customer_nodes AS cn 
JOIN 
    data_bank.regions AS r 
ON
    cn.region_id = r.region_id
GROUP BY 
    1,2;
```
#### Query Result:  
![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/c4e66df4-106b-4c7a-b7e5-d276beb8f04d)


***

### 3. How many customers are allocated to each region?
```sql
SELECT
    r.region_id, 
    r.region_name,
    COUNT (DISTINCT customer_id) AS no_of_customers
FROM
    data_bank.customer_nodes AS cn 
JOIN 
    data_bank.regions AS r 
ON
    cn.region_id = r.region_id
GROUP BY 
    1,2
ORDER BY
    1 ;
```
#### Query Result:  
![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/303dd5c6-298e-4a30-95e9-86f8e8e3bb1a)


***

### 4. How many days on average are customers reallocated to a different node?
```sql
WITH reallocation_date  AS (
        SELECT
            *,
            LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS realloc_date
        FROM 
            data_bank.customer_nodes),

day_difference AS (
        SELECT
            *,
            (realloc_date - start_date) AS day_diff
        FROM 
            reallocation_date
        WHERE
            realloc_date IS NOT NULL)

SELECT 
    ROUND(AVG(day_diff),0) AS Average_reallocation_day
FROM 
    day_difference;
```
#### Query Result:  
![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/07d9cf9b-ecb5-4d0d-818d-a534a54c730d)


***


### 5. What is the median, 80th, and 95th percentile for this same reallocation days metric for each region?
```sql
WITH reallocation_date  AS (
        SELECT
            *,
            LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS realloc_date
        FROM 
            data_bank.customer_nodes),

day_difference AS (
        SELECT
            *,
            (realloc_date - start_date) AS day_diff
        FROM 
            reallocation_date
        WHERE
            realloc_date IS NOT NULL)

SELECT
    rn.region_id,
    rn.region_name,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY day_diff) AS median,
    PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY day_diff) eigtieth_percentile,
    PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY day_diff) ninetyfifth_percentile
FROM    
    day_difference dn
JOIN
    data_bank.regions rn
ON
    dn.region_id = rn.region_id
GROUP BY
1,2;
```
#### Query Result:  
![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/95c7bc56-aa36-4e03-8844-b224e4879bc0)


***

### Click [here](https://github.com/Favourewoh/SQL-Projects/blob/631c92c3d62495eca960a3cc13b70212d15a3719/Case%20Study%20-%20Data%20Bank/Section%20B%20-%20Exploring%20Customer%20Transactions.md) to view the Customer Transactions solution!


# 🏦: Case Study - Data Bank
## Section B - Exploring Customer Transactions

### Questions
1. What is the unique count and total amount for each transaction type?
2. What are the average total historical deposit counts and amounts for all customers?
3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
4. What is the closing balance for each customer at the end of the month?
5. What is the percentage of customers who increase their closing balance by more than 5%?

***
### 1. What is the unique count and total amount for each transaction type?
```sql
SELECT
    txn_type,
    COUNT(txn_type) AS total_txn,
    SUM(txn_amount) AS amount_transacted
FROM
    data_bank.customer_transactions
GROUP BY
    1;
```  
#### Query Result:  
![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/5a4bad10-68a9-4f65-9555-4744497cada5)

***
### 2. What is the average total historical deposit counts and amounts for all customers?
```sql
SELECT
    ROUND(AVG(total_txn),0) avg_total_txn,
    ROUND(AVG(amount_transacted),0) avg_amount_transacted
FROM
    (SELECT
        customer_id,
        COUNT(txn_type) AS total_txn,
        SUM(txn_amount) AS amount_transacted
    FROM
        data_bank.customer_transactions
    WHERE
        txn_type = 'deposit'
    GROUP BY
        1);
```  
#### Query Result:  
![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/7fa002b4-b0f7-48a8-8194-8b2c7b8d665c)

***
### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
WITH customer_activity AS
(SELECT
    customer_id,
    EXTRACT (MONTH FROM txn_date) AS month_no,
    TO_CHAR(txn_date,'Month') AS month_name,
    SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
    SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count,
    SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count

FROM
    data_bank.customer_transactions
GROUP BY
    1,2,3)

SELECT 
    month_no,
    month_name,
    COUNT(DISTINCT customer_id) AS active_customers
FROM
    customer_activity
WHERE
    deposit_count > 1 
    AND (withdrawal_count > 0 or purchase_count > 0)
GROUP BY
    1,2
ORDER BY 1,2;
```
#### Query Result:  
![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/91eaf5df-4252-4dc4-b571-ca1ff7c0b7e7)

***

### 4. What is the closing balance for each customer at the end of the month?
```sql
SELECT 
    customer_id, 
    EXTRACT (MONTH FROM txn_date) AS month_no,
    TO_CHAR(txn_date,'Month'), 
    SUM(CASE WHEN txn_type IN ('withdrawal','purchase') THEN txn_amount * -1 ELSE txn_amount * 1 END) AS Mth_end_bal
FROM
    data_bank.customer_transactions
GROUP BY
    1,2,3
ORDER BY
    1,2;
```
#### Query Result:  
![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/ab193d7c-654b-49ea-8859-a68962bdac98)


*Total Query result is 1720 rows*
####

***
### 5. What is the percentage of customers who increase their closing balance by more than 5%?
```sql
SELECT 
    COUNT(DISTINCT customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM data_bank.customer_nodes) :: float growth
FROM
    (SELECT
        month_no,
        customer_id,
        Mth_end_bal,
        (mth_end_bal - (LAG(mth_end_bal) OVER(PARTITION BY 
        customer_id ORDER BY month_no))) / ABS((LAG(mth_end_bal) OVER(PARTITION BY 
        customer_id ORDER BY month_no))) :: float AS Perc_change
    FROM
        (SELECT 
            EXTRACT (MONTH FROM txn_date) AS month_no,
            customer_id, 
            SUM(CASE WHEN txn_type IN ('withdrawal','purchase') THEN txn_amount * -1 ELSE txn_amount * 1 END) AS Mth_end_bal
        FROM
            data_bank.customer_transactions
        GROUP BY
            1,2)
    ORDER BY
        1,2)
WHERE 
    perc_change >= 0.05;
```
#### Query Result:  
![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/4d3c8ba4-64d4-4cd2-a97f-025190f14835)


***
### Click [here](https://github.com/Favourewoh/SQL-Projects/blob/8dad864812eac426c9132774d3ea6fffda4511ff/Case%20Study%20-%20Data%20Bank/Data%20Allocation%20Solution.md) to view the Data Allocation Challenge solution!


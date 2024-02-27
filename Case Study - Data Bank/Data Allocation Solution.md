# 🏦: Case Study - Data Bank
## Data Allocation Solution

### Briefing on the hypotheses

To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

- Option 1: data is allocated based on the amount of money at the end of the previous month
- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
- Option 3: data is updated real-time
  
For this multi-part challenge question, I have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

- running customer balance column that includes the impact of each transaction
- customer balance at the end of each month
- minimum, average, and maximum values of the running balance for each customer

Using all of the data available - how much data would have been required for each option every month?

***

### Generating running customer balance column that includes the impact of each transaction

```sql
SELECT 
    customer_id, 
    EXTRACT (MONTH FROM txn_date) AS month_no,
    TO_CHAR(txn_date,'Month') AS month_name, 
    txn_date,
    SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount
        WHEN txn_type = 'withdrawal' THEN -txn_amount
        WHEN txn_type = 'purchase' THEN -txn_amount END) OVER 
        (PARTITION BY customer_id ORDER BY txn_date) AS running_bal
FROM
    data_bank.customer_transactions
GROUP BY
    1,2,3,txn_type,txn_amount,txn_date
ORDER BY
    1,2
```

#### Query Result:

![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/e90a974e-51d6-41ae-b4f8-add20e2b4eba)


*The result set has 5868 rows*

***

### Generating customer balance at the end of each month

```sql
SELECT 
    customer_id, 
    EXTRACT (MONTH FROM txn_date) AS month_no,
    TO_CHAR(txn_date,'Month') AS month_name, 
    SUM(CASE WHEN txn_type IN ('withdrawal','purchase') THEN 
    txn_amount * -1 ELSE txn_amount * 1 END) AS Mth_end_bal
FROM
    data_bank.customer_transactions
GROUP BY
    1,2,3
ORDER BY
    1,2;
```

#### Query Result:

![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/c38e0517-3d9e-4106-a3bf-7a8254e8e8a1)


*The result set has 1720 rows*

***

#### Generating minimum, maximum, and average running balance for each customers

```sql
WITH txn_detail AS (
        SELECT
            *,
            SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount
                WHEN txn_type = 'withdrawal' THEN -txn_amount
                WHEN txn_type = 'purchase' THEN -txn_amount END) OVER 
                (PARTITION BY customer_id ORDER BY txn_date) AS running_bal,
            SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount
                WHEN txn_type = 'withdrawal' THEN -txn_amount
                WHEN txn_type = 'purchase' THEN -txn_amount END) OVER 
                (PARTITION BY customer_id ORDER BY EXTRACT(MONTH FROM txn_date)) AS Month_end_bal
        FROM
            data_bank.customer_transactions
        ORDER BY
            customer_id,txn_date)

SELECT
    customer_id,
    MIN(running_bal) AS Min_running_bal,
    ROUND(AVG(running_bal),2) AS Avg_running_bal,
    MAX(running_bal) AS Max_running_bal
FROM
    txn_detail
GROUP BY
    1
ORDER BY
    1;
```

#### Query Result:

![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/17800e1b-4e05-4a0b-a61b-d70f6a10d02a)


*The result set has 500 rows*

***

## Data Allocation

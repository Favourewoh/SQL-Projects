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

#### Generating minimum, maximum, and average running balance for each customer

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

Since we are allocating cloud data storage to our customers based on their transaction behavior, we will need to have a conversion rate that will used to convert the currency into data storage capacity.
so,
- Customers will be allocated cloud data storage at the rate of 2GB per $100
- To filter out negative values, customers with negative balances will be allocated no cloud storage
- A 500MB bonus will be given to all customers regardless of their transactions, so credit customers can still get a little storage

moving on 

***

### Option 1: data is allocated based on the amount of money at the end of the previous month 

The month-end bal is directly converted to data using a conversion rate of 2GB per $100 and adding the 500MB BONUS

```sql
WITH month_end AS (
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
            1,2),

prev_meb AS (
        SELECT
            *,
            LAG(Mth_end_bal) OVER (PARTITION BY customer_id ORDER BY month_no) AS prev
        FROM
             month_end),

data_conversion AS (
        SELECT
            *,
            CASE WHEN prev < 0  THEN 0.5 ELSE ((prev/100 :: float) * 2) + 0.5 END AS data_alloc_gb
        FROM
            prev_meb)

SELECT 
    month_name,
    CAST (SUM(data_alloc_gb) AS NUMERIC (10,2)) AS Total_data
FROM
    data_conversion
GROUP BY
    1
ORDER BY
    2 DESC;

```

#### Query Result:

![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/bbef3f62-e9f6-4971-82fe-95b087034d36)


*The value for January was null because there was no data for December*


***

### Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days

The average monthly deposit is directly converted to data using a conversion rate of 2GB per $100 and adding the 500MB BONUS

```sql
WITH month_end AS (
        SELECT 
            customer_id, 
            EXTRACT (MONTH FROM txn_date) AS month_no,
            TO_CHAR(txn_date,'Month') AS month_name, 
            AVG(txn_amount) AS Avg_end_bal
        FROM
            data_bank.customer_transactions
        WHERE
            txn_type = 'deposit'
        GROUP BY
            1,2,3
        ORDER BY
            1,2),

prev_meb AS (
        SELECT
        *,
        LAG(avg_end_bal) OVER (PARTITION BY customer_id ORDER BY month_no) AS prev
        from month_end),

data_conversion AS (
        SELECT
            *,
            CASE WHEN prev < 0 THEN 0.5 ELSE ((prev/100 :: float) * 2) + 0.5 END AS data_alloc_gb
        FROM
            prev_meb)

SELECT 
    month_name,
    CAST (SUM(data_alloc_gb) AS NUMERIC (10,2)) AS Total_data
FROM
    data_conversion
GROUP BY
    1
ORDER BY
    2 DESC;

```

#### Query Result:

![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/c4320123-4e58-4dfd-9c70-45e0fa5ca65f)

*The value for January was null because there was no data for December*

***


### Option 3: data is updated real-time

The running balance is directly converted to data using a conversion rate of 2GB per $100 and adding the 500MB BONUS

```sql
WITH running_balance AS (
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
            1,2),

data_conversion AS (
        SELECT
            *,
            CASE WHEN running_bal < 0 THEN 0.5 ELSE ((running_bal/100 :: float) * 2) + 0.5 END AS data_alloc_gb
        FROM
            running_balance)

SELECT 
    month_name,
    CAST (SUM(data_alloc_gb) AS NUMERIC (10,2)) AS Total_data
FROM
    data_conversion
GROUP BY
    1
ORDER BY
    2 DESC;
```

#### Query Result:

![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/c8c3017b-6581-4988-bb5a-f721b5ba7002)


***

### Find the Extra Challenge Solution [here]()

# 🏦: Case Study - Data Bank
## Extra Challenge Solution

### Briefing 
Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based on the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?

***

#### Considering the daily interest

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
Interest AS (
        SELECT
            *,
            running_bal * 0.000164 AS intr
        FROM
            running_balance),

data_conversion AS (
        SELECT
            *,
            CASE WHEN intr < 0 THEN 0 ELSE ((intr/100 :: float) * 2)  END AS data_alloc_gb
        FROM
            interest)
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

![image](https://github.com/Favourewoh/SQL-Projects/assets/124405367/83025cc8-a18f-436c-88ee-37d3148c2408)

*Additional cloud storage (GB) that will be required for each monthly if we are considering the annual interest of 6%*


***

View the Extension Request(slide presentation) [here](https://github.com/Favourewoh/SQL-Projects/blob/69c0a0feb3b8b597dc05fc20c06036e5049a1fab/Case%20Study%20-%20Data%20Bank/Extension%20Request.md) 

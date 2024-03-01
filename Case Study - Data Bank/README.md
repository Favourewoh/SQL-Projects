# 🏦 Case Study - Data Bank: Bridging Finance, Security, and Data Innovation. 
<p align="center">
<img src="https://github.com/Favourewoh/SQL-Projects/assets/124405367/934aab61-a55d-4b6d-ab81-4a3572813626" alt="Image" width="450" height="450">

View the challenge website [here](https://8weeksqlchallenge.com/case-study-4/)

## Table Of Contents  
  - [Introduction](#introduction)
 - [Available Data](#available-data) 
- [Entity Relationship Diagram](#entity-relationship-diagram)  
- [Case Study Questions](#case-study-questions)  

## Introduction
There is an innovation in the financial industry known as Neo-Banks. They are digital-only banks that do not have physical branches. Danny had an idea to combine these new-age banks, cryptocurrency, and the data world, and thus, he launched a new initiative called Data Bank!

Data Bank is similar to any other digital bank, but it is not only for banking activities. It has the world's most secure distributed data storage platform! Customers are given cloud data storage limits, which are linked to the amount of money they have in their accounts. The Data Bank team needs help to increase its total customer base and track how much data storage customers will need.

This case study is all about calculating metrics and growth and helping the business analyze its data to better forecast and plan for its future developments!



## Available Data
The Data Bank team have prepared a data model for this case study as well as a few example rows from the complete dataset below to get you familiar with their tables.
link to [schema code](https://github.com/Favourewoh/SQL-Projects/blob/631c92c3d62495eca960a3cc13b70212d15a3719/Case%20Study%20-%20Data%20Bank/Schema.SQL)

#### Table 1: Regions

Like popular cryptocurrency platforms, Data Bank is also run off a network of nodes where money and data are stored across the globe. In a traditional banking sense - you can think of these nodes as bank branches or stores that exist around the world. 

<details>
<summary>
View table
</summary>

  This ```regions``` table contains the ```region_id``` and their respective ```region_name``` values


| "region_id" | "region_name" |
|-------------|---------------|
| 1           | "Australia"   |
| 2           | "America"     |
| 3           | "Africa"      |
| 4           | "Asia"        |
| 5           | "Europe"      |
</details>


#### Table 2: Customer Nodes
Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data. This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!

<details>
<summary>
View table
</summary>

Below is a sample of the top 10 rows of the ```data_bank.customer_nodes```

| "customer_id" | "region_id" | "node_id" | "start_date" | "end_date"   |
|---------------|-------------|-----------|--------------|--------------|
| 1             | 3           | 4         | "2020-01-02" | "2020-01-03" |
| 2             | 3           | 5         | "2020-01-03" | "2020-01-17" |
| 3             | 5           | 4         | "2020-01-27" | "2020-02-18" |
| 4             | 5           | 4         | "2020-01-07" | "2020-01-19" |
| 5             | 3           | 3         | "2020-01-15" | "2020-01-23" |
| 6             | 1           | 1         | "2020-01-11" | "2020-02-06" |
| 7             | 2           | 5         | "2020-01-20" | "2020-02-04" |
| 8             | 1           | 2         | "2020-01-15" | "2020-01-28" |
| 9             | 4           | 5         | "2020-01-21" | "2020-01-25" |
| 10            | 3           | 4         | "2020-01-13" | "2020-01-14" |
</details>


#### Table 3: Customer Transactions
This table stores all customer deposits, withdrawals, and purchases made using their Data Bank debit card.


<details>
<summary>
View table
</summary>

Below is a sample of the top 10 rows of the ```data_bank.customer_Transactions```

| "customer_id" | "txn_date"   | "txn_type" | "txn_amount" |
|---------------|--------------|------------|--------------|
| 429           | "2020-01-21" | "deposit"  | 82           |
| 155           | "2020-01-10" | "deposit"  | 712          |
| 398           | "2020-01-01" | "deposit"  | 196          |
| 255           | "2020-01-14" | "deposit"  | 563          |
| 185           | "2020-01-29" | "deposit"  | 626          |
| 309           | "2020-01-13" | "deposit"  | 995          |
| 312           | "2020-01-20" | "deposit"  | 485          |
| 376           | "2020-01-03" | "deposit"  | 706          |
| 188           | "2020-01-13" | "deposit"  | 601          |
| 138           | "2020-01-11" | "deposit"  | 520          |
</details>

## Entity Relationship Diagram
<p align="center">
<img src="https://github.com/Favourewoh/SQL-Projects/assets/124405367/9087d198-26ee-4441-8506-f48811b2584f" alt="Image" width="700" height="350">


## Case Study Questions
### A. [Customer Nodes Exploration](https://github.com/Favourewoh/SQL-Projects/blob/b47dd51f10c3681703355cf3ec6f810ead379c7b/Case%20Study%20-%20Data%20Bank/Section%20A%20-%20Exploring%20Customer%20Nodes.md)
1. How many unique nodes are there on the Data Bank system?
2. What is the number of nodes per region?
3. How many customers are allocated to each region?
4. How many days on average are customers reallocated to a different node?
5. What is the median, 80th, and 95th percentile for this same reallocation days metric for each region?

### B. [Customer Transactions](https://github.com/Favourewoh/SQL-Projects/blob/b47dd51f10c3681703355cf3ec6f810ead379c7b/Case%20Study%20-%20Data%20Bank/Section%20B%20-%20Exploring%20Customer%20Transactions.md)
1. What is the unique count and total amount for each transaction type?
2. What are the average total historical deposit counts and amounts for all customers?
3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
4. What is the closing balance for each customer at the end of the month?
5. What is the percentage of customers who increase their closing balance by more than 5%?

### C. [Data Allocation Challenge](https://github.com/Favourewoh/SQL-Projects/blob/b47dd51f10c3681703355cf3ec6f810ead379c7b/Case%20Study%20-%20Data%20Bank/Data%20Allocation%20Solution.md)
To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

- Option 1: data is allocated based on the amount of money at the end of the previous month
- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
- Option 3: data is updated real time
For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

1. running a customer balance column that includes the impact of each transaction
2. customer balance at the end of each month
3. minimum, average, and maximum values of the running balance for each customer
Using all of the data available - how much data would have been required for each option on a monthly basis?


### D. [Extra Challenge](https://github.com/Favourewoh/SQL-Projects/blob/b47dd51f10c3681703355cf3ec6f810ead379c7b/Case%20Study%20-%20Data%20Bank/Extra%20Challenge%20Solution.md)
Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based on the interest calculated daily at the end of each day, how much data would be required for this option on a monthly basis?

Special notes:

- Data Bank wants an initial calculation that does not allow for compounding interest, however, they may also be interested in a daily compounding interest calculation so you can try to perform this calculation if you have the stamina!


### E. [Extension Request](https://github.com/Favourewoh/SQL-Projects/blob/b47dd51f10c3681703355cf3ec6f810ead379c7b/Case%20Study%20-%20Data%20Bank/Extension%20Request.md)
The Data Bank team wants you to use the outputs generated from the above sections to create a quick PowerPoint presentation that will be used as marketing materials for external investors who might want to buy Data Bank shares and new prospective customers who might want to bank with Data Bank.

1. Using the outputs generated from the customer node questions, generate a few headline insights that Data Bank might use to market its world-leading security features to potential investors and customers.

2. With the transaction analysis - prepare a 1 page presentation slide that contains all the relevant information about the various options for the data provisioning so the Data Bank management team can make an informed decision.

***

#### Find my solutions to this case study [here](https://github.com/Favourewoh/SQL-Projects/blob/b47dd51f10c3681703355cf3ec6f810ead379c7b/Case%20Study%20-%20Data%20Bank/Section%20A%20-%20Exploring%20Customer%20Nodes.md)







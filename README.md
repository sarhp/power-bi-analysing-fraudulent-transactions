# fraudulent-transactions

<br><br>

| **Title**              | Summary |
|------------------------|-------------|
| **Project Objective**  | Perform exploratory data analysis to investigate patterns and trends in fraudulent transactions. Help the team in identifying insights that will enable them to proactively prevent fraud. |
| **Date**               | July 2024 (modified Jan 2025) |
| **Domain**             | Financial Crime / Finance / AML |
| **About Dataset**      | [Synthetic Financial Datasets for Fraud Detection](https://www.kaggle.com/datasets/ealaxi/paysim1?resource=download) <br> Generated by **PaySim**. The original logs were provided by a multinational company, a provider of mobile financial services. <br> The dataset consists of **6,362,620 rows** with **11 columns**, including: <ul><li>Transaction type</li><li>Transaction amount</li><li>Sender and receiver IDs</li><li>Sender’s balance before and after the transaction</li><li>Receiver’s balance before and after the transaction</li></ul> |
| **Tools & Techniques** | <ul><li>**Python**: Data Sampling (Stratified method)</li><li>**Power Query**: Data Cleaning, Formatting</li><li>**Power BI**: Visualization</li><li>**DAX Functions**: Data Aggregation, `functionName`, `functionName`...</li></ul> |
| **Findings**           | <ul><li>**Fraudsters deplete the balance using `CASH_OUT` and transfer the same amount to another account.** A large transaction occurs **every 5 days**.</li><li>**There are patterns suggesting possible money laundering.** To investigate further, we will request additional information from other teams.</li></ul> |
| **Challenges / Constraints** | <ul><li>**Limited data availability**: Due to the private nature of the data, some crucial information is missing, restricting in-depth analysis.</li><li>**Large dataset size**: The dataset (6M rows) takes time to load and process.</li></ul> |


<br>


## **🚩Contents**
* [About Dataset](#about-dataset)
* [Business Problem](#business-problem)
* [Data Analysis Plan](#data-analysis-plan)
* [Preprocessing Data - Optional Stratified Data Sampling](#preprocessing-data---optional-stratified-data-sampling)
* [Preprocessing Data - Cleaning and Transforming](#preprocessing-data-cleaning-and-transforming)
* [Dashboard](#dashboard)
* [Analysis](#analysis)

<br>


<br>

## 🚩About Dataset
`step`: Maps a unit of time in the real world. In this case, 1 step = 1 hour of time. Total steps = 744 (30 days simulation).  

`type`: Type of transaction: CASH-IN, CASH-OUT, DEBIT, PAYMENT, and TRANSFER.  

`amount`: The amount of the transaction in local currency.  

`nameOrig`: Customer who started the transaction.  

`oldbalanceOrg`: Initial balance before the transaction for the originating customer.  

`newbalanceOrig`: New balance after the transaction for the originating customer.  

`nameDest`: Customer who is the recipient of the transaction. (Note: there is no information for customers starting with 'M' — Merchants).  

`oldbalanceDest`: Initial balance of the recipient before the transaction.  

`newbalanceDest`: New balance of the recipient after the transaction. (Note: no information for customers starting with 'M' — Merchants).  

`isFraud`: A flag indicating if the transaction was made by a fraudulent agent within the simulation. Fraudulent behavior involves taking control of customer accounts to empty funds, transferring to another account, and then cashing out of the system.  

`isFlaggedFraud`: A flag indicating whether the transaction was flagged as fraud. The business model flags attempts to transfer more than 200,000 in a single transaction as illegal.  


<br>

## 🚩Business Problem
"As the head of marketing, I want to identify top content creators who can effectively promote New Zealand as a destination celebrated for its stunning natural landscapes, peaceful and laid-back lifestyle, humble and friendly people, and world-class outdoor activities. To maximise reach, we aim to collaborate with mega influencers rather than niche content creators.”

<br>

## 🚩Data Analysis Plan
**What problems are we trying to solve?**

- How is fraudulent transactions different to normal transactions?
- Are there any time-based / behavioral patterns?
- Is the pattern coming from the same customer or from different customers?
- What types of fraudulent activities can we assume from the results?
- How will the results of the analysis be used or applied?
- Is the given data it sufficient for the analysis?

<br>

**Information we can draw from the the dataset:**

- Finding relationships between time-based patterns / money movement / transaction amount using scatterplot.
- Aggregate rows by individual ID to identify duplicate IDs and activities
- Identify any sequential pattern by sorting in specific orders
- movement of money between closing and opening balance using positive and negative bars

<br>

## 🚩Preprocessing Data (optional) - Stratified Data Sampling 
To scale down rows from 6M → 1M 

**Recommend when:** you want to reduce computational cost and uncover general trends. 

**Not recommend:** if your analysis require precise numbers.

```{python} 
pip install pandas scikit-learn

import pandas as pd
from sklearn.model_selection import train_test_split

```

```{python} 
df = pd.read_csv('fraud_original.csv')

X = df.drop(columns=['isFraud'])
y = df['isFraud']

_, sampled = train_test_split(df, test_size=0.2, stratify=y, random_state=42)

```

```{python} 
# Original vs sampled by ratio

print(df['isFraud'].value_counts(normalize=True))
print(sampled['isFraud'].value_counts(normalize=True))
```
![image](https://github.com/user-attachments/assets/556f14c6-17aa-473e-972b-e978af895e9c)


```{python} 
# Original vs sampled by length

print(df['isFraud'].value_counts()) in j
```
![image](https://github.com/user-attachments/assets/e4da2089-f15b-4c6f-9542-61c04c329e15)

```{python} 
print(sampled['isFraud'].value_counts())
```
![image](https://github.com/user-attachments/assets/d51b00de-2c7f-4f8b-96e0-fa7716697142)

```{python} 
sampled.to_csv('fraud_sampled.csv', index=True)
files.download('fraud_sampled.csv')
```

<br>

## 🚩Preprocessing Data - Cleaning and Transforming

1. Drop transaction amount = $0

<br>

2. Create an index column (starting from 1) to keep transactions in order (Power Query)

<br>

3. Rename columns for readability

`step` ➡️ `hour`

`nameDest`, `nameOrig` ➡️ `payeeID` `senderID`

`oldbalanceOrg` , `newbalanceOrg` ➡️ `senderOpeningBal`, `senderClosingBal` 

`oldbalanceOrg` , `newbalanceOrg` ➡️ `payeeOpeningBal`, `payeeClosingBal`

<br>

4. Create an index column (starting from 1) to keep transactions in order (Power Query)
```DAX
senderBalDiscrepency = fraud[senderOpeningBal] – fraud[senderClosingBal]

payeeBalDiscrepency = fraud[payeeOpeningBal] – fraud[payeeClosingBal]
```
![image](https://github.com/user-attachments/assets/a26b08ab-70d5-49a5-b8df-905551945616)

<br>

5. There are 744 steps, ranging from 1 to 744, with each step corresponding to one hour. Convert step(hour) to day e.g. steps 1-23 as day 1, steps 24-47 as day 2 etc
```{DAX} 
day = INT(DIVIDE([hour] - 1, 23)) + 1
```

<br>

6. Index each row within a day, starting from 1 and resetting to 1 at the start of a new day. 
```{DAX} 
indexWithinDay = 
RANKX(
    FILTER(
        'fraud', 
        [day] = EARLIER([day])  
    ),
    [index],    
    , 
    ASC,            
    DENSE              
)
```
Output:
| **step** | **day** | **indexWithinDay** |
|----------|---------|---------------------|
| 1        | 1       | 1.1                 |
| 1        | 1       | 1.2                 |
| 1        | 1       | 1.3                 |
| ...      | ...     | ...                 |
| 24       | 2       | 2.1                 |
| 24       | 2       | 2.2                 |
| ...      | ...     | ...                 |
| 24       | 2       | 2.500               |

<br>

7. Sort index hierachy numbers as 1.1, 1.2, 1.3... instead of 1.1, 1.10, 1.100, 1.2, 1.3 etc. 

To to this, we shift the index to the right side by adding ‘0000000’. The day will appear first, followed by the index.

```{DAX} 
SortKey = [day] * 1000000 + [indexWithinDay]
```
Output:
| **day** | **indexWithinDay** | **sort**   |
|---------|--------------------|------------|
| 1       | 5                  | 1000005    |
| 1       | 6                  | 1000006    |
| ...     | ...                | ...        |
| 2       | 500                | 2000500    |

<br>

8. Concatenate day and indexWithinDay, separating them with a dot to distinguish the day from the index.
```{DAX} 
dayAndIndex = 
VAR Day = [day]
VAR Index = [indexWithinDay]
RETURN FORMAT(Day, "0") & "." & FORMAT(Index, "0000000")
```
 ![image](https://github.com/user-attachments/assets/d5b1cd30-2197-40a3-aa98-015c909c8096)

<br>

## 🚩Dashboard

1/5. Overview - characteristics of fraud vs normal transactions
![dashboard_1](https://github.com/user-attachments/assets/a90ec2d9-ac02-4b63-bda3-bd53318ffbea)
![dashboard_1 1](https://github.com/user-attachments/assets/82381006-2d5b-400a-b3a2-ac481372e27f)

2/5. Distribution - time-based / transaction amount using scatterplot. 
![dashboard_2](https://github.com/user-attachments/assets/1fee2195-0830-41e8-9ee5-512f5476d497)

3/5. Repeated IDs and its activities 
![dashboard_3](https://github.com/user-attachments/assets/e3deec82-0eb3-4346-a60c-00edddbbe0b8)

4/5. Sequential patterns
![dashboard_4](https://github.com/user-attachments/assets/31b80071-89a9-4e55-b22a-4285cd1ea2bf)

5/5. Cashflow (sender & payee)
![dashboard_5](https://github.com/user-attachments/assets/b4d8297b-4dc4-4231-a259-9c8fa66ad663)


<br>

## 👩🏻‍💻Analysis

**What patterns in fraudulent transactions have been found?**  

- Over the 30 days, 8,213 (0.13%), equiv. of $12.06 billion worth of fraudulent transactions occurred out total of 6,362,620 transactions.
- Fraudulent transactions do not exceed $10 million. TRANSFER and CASH_OUT are the preferred methods.
- No frequentic pattern in fraudulent transactions can be found, but there is in amount patten — a periodic surge in fraudulent transactions, which tends to spike around every 500,000th transaction index.
- TRANSFER is always immediately followed by CASH_OUT, using different sender and recipient IDs.
- Sender’s entire balance is withdrawn in TRANSFER, and the same amount is deposited into the recipient’s account in the following transaction CASH_OUT




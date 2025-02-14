# End-to-end Project: Analysing Fraudulent Transactions

<br>

## Summary

**Date Created**<br>
August 2024

**Project Objective**<br>
Perform exploratory data analysis to uncover patterns and trends in fraudulent transactions. 

**Domain**<br>
Banking / Financial Crime

**Tools & Skills**<br>
Power BI (DAX): Time-Based Aggregation, Dense-Ranking, Sorting, Concatenation (using a dot separator), Calculated Columns <br>
Power Query: Creating Index, Filtering Rows <br>
Python: Data Sampling (Stratified method using Scikit-Learn)

**About Dataset**<br>
Synthetic Financial Datasets for Fraud Detection:
https://www.kaggle.com/datasets/ealaxi/paysim1?resource=download

6,362,620 rows, 11 columns - incl. transaction type, amount, sender and receiver id, sender balance beforeA and after transaction, receivers balance before and after transaction. 


**Findings / Conclusion**<br>

Findings:<br>
•  Frequency of fraudulent transactions increases over time <br>
•  Periodic surge around every 500,000th transaction <br>
•  TRANSFER immediately follows by CASH_OUT  (in the first transaction, fraudsters empties a balance. Subsequently, the same amount of money gets deposited into a different account) <br>

Conclusion: <br>
•  The cash flow patterns suggest possible money laundering activity <br>
•  Request for additional data to interrogate further <br>
•  Continuously monitor the next wave of transactions, particularly focusing on the patterns identified earlier <br>

Challenges / Constraints: <br>
•  Due to the private nature of the data, there are limitations in conducting an in-depth analysis because of restricted information <br>
• High computational cost due to the size of the data (6M rows) <br>
• Data error –  in CASH_OUT transactions, money is transferred to a payee when there should be no recipient <br>


<br>


## **Contents**
* [Workflow](#workflow)
* [About Dataset](#about-dataset)
* [User Story](#user-story)
* [Problem Statement](#problem-statement)
* [Preprocessing Data - Optional Stratified Data Sampling](#preprocessing-data---optional-stratified-data-sampling)
* [Preprocessing Data - Cleaning and Transforming](#preprocessing-data-cleaning-and-transforming)
* [Dashboard](#dashboard)
* [Analysis](#analysis)

<br>

## Workflow
![workflow](https://github.com/user-attachments/assets/2df96530-3cb3-479d-a13d-d555afc8838f)

<br>

## About Dataset
![about_dataset](https://github.com/user-attachments/assets/c6e5d6f8-ef7e-4ce7-a1d4-3c1d7ab565f3)
- No duplicates or missing values
- There are 8197 (0.13%, equiv. of $12B) fraudulent transactions out of 6,362,620 transactions
- Fraudulent transactions include only TRANSFER and CASH_OUT, with counts of 4100 and 4097
- Fraudulent transitions do not exceed $10M
- `step` values ranging form 1-744. Each step has rows counts between 1000 to 764,000
- There are 43 repeating payeeID and 0 senderID  in fraudulent transactions


<br>

## User Story
![user_story](https://github.com/user-attachments/assets/c4d537da-ab97-4747-aede-f1dfede39771)


<br>

## Problem Statement
![questions_before_analysis](https://github.com/user-attachments/assets/a6498d67-8a66-49be-8fcd-cc74846ba816)

<br>

## Preprocessing Data (optional) - Stratified Data Sampling 
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

## Preprocessing Data - Cleaning and Transforming

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

## Dashboard

1/5. Overview - characteristics of fraud vs normal transactions
![dashboard_1](https://github.com/user-attachments/assets/a90ec2d9-ac02-4b63-bda3-bd53318ffbea)
![dashboard_1 1](https://github.com/user-attachments/assets/82381006-2d5b-400a-b3a2-ac481372e27f)

- Fraudsters use only 2 types of transactions - **CASH_OUT & TRANSFER**
- There is an **increasing trend in the frequency** of fraudulent transactions over time
- Fraudulent transactions **don’t exceed $10M**

<br>

2/5. Distribution - time-based / transaction amount using scatterplot. 
![dashboard_2](https://github.com/user-attachments/assets/1fee2195-0830-41e8-9ee5-512f5476d497)

- Spikes in scatter plot (right)  - indicates that large transactions occur daily or every two days from Day 1 - 18, then become more intense starting from Day 19 - 31.

<br>

3/5. Repeated IDs and its activities 
![dashboard_3](https://github.com/user-attachments/assets/e3deec82-0eb3-4346-a60c-00edddbbe0b8)

- There are 43 repeating payeeID.  No indication of repetitive or cyclic behavior.

<br>

4/5. Sequential patterns
![dashboard_4](https://github.com/user-attachments/assets/31b80071-89a9-4e55-b22a-4285cd1ea2bf)

- TRANSFER immediately followed by CASH_OUT - in every first transaction, the balance gets emptied, and in the subsequent ones, the same amount gets deposited into different account.

<br>

5/5. Cashflow (sender & payee)
![dashboard_5](https://github.com/user-attachments/assets/b4d8297b-4dc4-4231-a259-9c8fa66ad663)

- This cash flow chart clearly indicates the fraudsters' pattern in moving funds. On average, $1.46M is withdrawn from the sender's account, while the payee's account sees an average increase of $737K. 

<br>

## 👩🏻‍💻Analysis

![findings](https://github.com/user-attachments/assets/94cd3ceb-795f-407d-92a6-f543defea23f)

![suspected](https://github.com/user-attachments/assets/a0b163dc-338e-4796-8042-2098036cb528)

![action_plan](https://github.com/user-attachments/assets/0f2e169c-0959-4e0e-8395-b89d7bd6227f)




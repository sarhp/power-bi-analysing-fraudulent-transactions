# Fraud Insights  

Objective: Perform exploratory data analysis to investigate patterns and trends in fraudulent transactions.

[Dataset source](https://www.kaggle.com/datasets/ealaxi/paysim1?resource=download)

<br><br>

**Animated version (link provided below):** 

https://www.youtube.com/watch?v=OpUfc_sQEvo

![fraud gif](https://github.com/user-attachments/assets/a7093946-98e6-4ba7-b2bc-d842cd46fda3)

(For better engagement, I incorporated sound effects, animations, and some analysis into this report)

---

<br>

**Static version:**

1/5. Introduction:
![1 Intro](https://github.com/user-attachments/assets/b9ab7a59-f7c6-42af-8967-22ccbaf1252c)

Description:

[Dataset source](https://www.kaggle.com/datasets/ealaxi/paysim1?resource=download)

Data Shape: 6,362,620 rows, 11 cols

| Field           | Description                                                                                      |
|-----------------|--------------------------------------------------------------------------------------------------|
| Step            | Maps a unit of time in the real world (1 step = 1 hour).                                           |
| Type            | Type of transaction: CASH-IN, CASH-OUT, DEBIT, PAYMENT, TRANSFER.                                  |
| Amount          | Amount of the transaction in local currency.                                                       |
| NameOrig        | Customer who initiated the transaction.                                                            |
| OldbalanceOrg   | Initial balance before the transaction.                                                            |
| NewbalanceOrig  | New balance after the transaction.                                                                 |
| NameDest        | Customer who is the recipient of the transaction.                                                  |
| OldbalanceDest  | Initial balance of the recipient before the transaction. (No info for merchants starting with M)   |
| NewbalanceDest  | New balance of the recipient after the transaction. (No info for merchants starting with M)        |
| isFraud         | Indicates if the transaction was made by a fraudulent agent.                                        |
| isFlaggedFraud  | Flags transactions that attempt to control massive transfers between accounts.                     |

<br>

2/5. Overview:
![2 Overview](https://github.com/user-attachments/assets/5975d26b-aec2-4c34-81bb-3e7907ee9f89)

Analysis:
- Out of a total of 6,362,620 instances, there are 8,213 cases of fraud which is 0.13%, eqivalent of $12.06bn out of $1.14T worth of transactions. 
- Fradulent transactions are conducted only through CASH OUT and TRANSFER.
- There are 16 instances of suspected of fraud, all of which were confirmed as actual fraud.
- Out of 6M+ instances were initially suspected to be non_fraudulent, but 8197 were identified as fraudulent.
<br>

3/5. Overview - Drill-through:
![3 Drill-through](https://github.com/user-attachments/assets/f76953cd-0dc9-4a59-b163-710b072ac32a)

Analysis:
- $77.9M lost from the 16 fraud cases.
- Plus $11.98bn lost from the 8197 misclassified caes, originally suspected as no-fraudulent.
<br>

4/5. Details - Non-Fraudulent:
![4 Non-Fraud](https://github.com/user-attachments/assets/53d8b24e-f411-4989-8de3-ab0a47eda3f7)

Analysis:
- There are no instances of large transactions.
- CASH IN is is thge most frequently used transaction type amoung senders holding higher balances.
- Receivers generally maintain higher balances compared to senders.
- A significant amount of funds are received only though TRANSFER.
- Average transaction amount is $1M and the highest is $92M, through TRANSFER.
- A large amounts are often transferred around 300th hour (equiv of 12.5th day.)
<br>

5/5. Details - Fraudulent:
![5 Fraud](https://github.com/user-attachments/assets/b13de844-331a-4092-974f-a7486ee439a6)

Analysis:
- A huge gaps between bars reveal that ususually high amount of money being transferred between senders and receivers.
- Fraudsters exculsively use TRANSFER and CASH OUT as their preferred methods.
- Fraudulent transactiosn do not follow a specific time pattern.
- Transactions are usually around $1M, and do not exceed $10M.
<br>

Summary:

![image](https://github.com/user-attachments/assets/d9e8e606-e31a-43af-917b-9fc4b50f6399)




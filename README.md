# Python | E-Wallet Transaction And Payment Analysis

<img width="1080" height="675" alt="ewallet analysis2" src="https://github.com/user-attachments/assets/87af181a-87f5-4b4b-8b16-36534df38572" />

This project analyzes **payment and transaction data in an e-wallet system** with the goal of understanding product performance, transaction behavior, and operational anomalies across teams and transaction types. **The entire analysis is conducted using Python**, following a data-driven and business-oriented approach.
- **Author**: Le Dang Kim Ngan
- **Tool Used**: Python

---

## Table of Contents
1. Overview
2. Exploratory Data Analysis (EDA)
3. Transaction & Payment Analysis
4. Insights & Recommendations

---

## Overview
### Objectives
- Practice **SQL-style analytical thinking** using pandas (groupby, joins, conditional mapping, time filtering).
- Analyze **product and team performance** based on payment volume.
- Detect **data integrity issues**, such as violations of product ownership rules.
- Understand **refund behavior** and its contributing sources.
- Apply **business rules to classify transactions** into meaningful transaction types.
- Produce **clear, query-like outputs** suitable for decision-making.

### Business Questions
1. Identify the **top 3 products** with the highest payment volume.
2. Verify whether any **products are owned by more than one team**, against the defined business rule.
3. Determine the **lowest-performing team since Q2 2023** and the **least contributing product category** within that team.
4. Analyze refund transactions (payment_group = ‘refund’) to find the **source_id with the highest contribution**.
5. Classify each transaction into a **business-defined transaction type** based on transaction codes and merchant IDs.
6. For each valid transaction type, compute key metrics such as **number of transactions, total volume, unique senders, and receivers**.

### Dataset Description
<details>
  <summary> Table 1: Payment Report - Monthly payment volume at the product level (Click to see detail)</summary>
  
| Column Name   | Data Type | Description |
|---------------|----------|-------------|
| report_month  | `object`     | Month of the payment report |
| payment_group | `object`     | Type of payment (e.g., refund, purchase) |
| product_id    | `int64`      | Associated product ID |
| source_id     | `int64`      | Source of the transaction |
| volume        | `float64`    | Total payment volume |

</details>

<details>
  <summary> Table 2: Product - Product metadata, including team ownership and category (Click to see detail)</summary>
  
| Column Name  | Data Type | Description |
|-------------|----------|-------------|
| product_id  | `int64`      | Unique identifier for each product |
| category    | `object`     | Product category |
| team_own    | `object`     | Team responsible for the product |

</details>

<details>
  <summary> Table 3: Transactions - Detailed transaction-level data, including transaction type, merchant, source, sender, and receiver information (Click to see detail)</summary>
  
| Column Name    | Data Type | Description |
|---------------|----------|-------------|
| transaction_id | `int64`      | Unique transaction identifier |
| merchant_id    | `int64`      | Merchant involved in the transaction |
| volume         | `float64`    | Transaction amount |
| transType      | `int64`      | Type of transaction |
| transStatus    | `object`     | Status of the transaction (e.g., completed, failed) |
| sender_id      | `int64`      | Sender of the transaction |
| receiver_id    | `int64`      | Receiver of the transaction |
| extra_info     | `object`     | Additional details about the transaction |
| timeStamp      | `int64` | Time when the transaction occurred |

</details>

By combining these datasets, the project aims to **answer key business and operational questions** related to revenue contribution, team performance, refund behavior, and transaction classification within an **e-wallet ecosystem**.

---

## Exploratory Data Analysis (EDA)
In [1]:
```python
# Import libraries
import pandas as pd
import numpy as np
```
In [2]:
```python
# Load data
!pip install gdown
!gdown --folder 1EOi1JmFkTPege1geI0f9JmXfEk3kZkGS

df_paymnent = pd.read_csv("/content/E-Wallet Dataset/payment_report.csv")
df_product = pd.read_csv("/content/E-Wallet Dataset/product.csv")
df_transaction = pd.read_csv("/content/E-Wallet Dataset/transactions.csv")
```
In [3]:
```python
# Merge payment_report.csv with product.csv
df_payment_enriched = pd.merge(df_paymnent, df_product, how = "left", on = "product_id")
df_payment_enriched.head()
```
Out [3]:

<img width="668" height="190" alt="image" src="https://github.com/user-attachments/assets/b38b9a6b-819a-45e8-ae21-027d88c018bb" />

In [4]:
```python
# Check general info df_payment_enriched
df_payment_enriched.info()
```
Out [4]:
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 919 entries, 0 to 918
Data columns (total 7 columns):
 #   Column         Non-Null Count  Dtype 
---  ------         --------------  ----- 
 0   report_month   919 non-null    object
 1   payment_group  919 non-null    object
 2   product_id     919 non-null    int64 
 3   source_id      919 non-null    int64 
 4   volume         919 non-null    int64 
 5   category       897 non-null    object
 6   team_own       897 non-null    object
dtypes: int64(3), object(4)
memory usage: 50.4+ KB
```
In [5]:
```python
# Check duplicates
df_payment_enriched.duplicated().sum()
```
Out [5]:
```
np.int64(0)
```
In [6]:
```python
# Check general info df_transaction
df_transaction.info()
```
Out [6]:
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1324002 entries, 0 to 1324001
Data columns (total 9 columns):
 #   Column          Non-Null Count    Dtype  
---  ------          --------------    -----  
 0   transaction_id  1324002 non-null  int64  
 1   merchant_id     1324002 non-null  int64  
 2   volume          1324002 non-null  int64  
 3   transType       1324002 non-null  int64  
 4   transStatus     1324002 non-null  int64  
 5   sender_id       1274943 non-null  float64
 6   receiver_id     1159207 non-null  float64
 7   extra_info      6095 non-null     object 
 8   timeStamp       1324002 non-null  int64  
dtypes: float64(2), int64(6), object(1)
memory usage: 90.9+ MB
```
In [7]:
```python
# Handle missing values
df_transaction['sender_id'] = df_transaction['sender_id'].fillna(value='Unknown')
df_transaction['receiver_id'] = df_transaction['receiver_id'].fillna(value='Unknown')
df_transaction.drop('extra_info', axis=1, inplace=True)
```
In [8]:
```python
# Check duplicates
df_transaction.duplicated().sum()
```
Out [8]:
```
np.int64(28)
```
In [9]:
```python
# Remove duplicates
df_transaction = df_transaction.drop_duplicates()

df_transaction.info()
```
Out [9]:
```
<class 'pandas.core.frame.DataFrame'>
Index: 1323974 entries, 0 to 1324001
Data columns (total 8 columns):
 #   Column          Non-Null Count    Dtype 
---  ------          --------------    ----- 
 0   transaction_id  1323974 non-null  int64 
 1   merchant_id     1323974 non-null  int64 
 2   volume          1323974 non-null  int64 
 3   transType       1323974 non-null  int64 
 4   transStatus     1323974 non-null  int64 
 5   sender_id       1323974 non-null  object
 6   receiver_id     1323974 non-null  object
 7   timeStamp       1323974 non-null  int64 
dtypes: int64(6), object(2)
memory usage: 90.9+ MB
```
### Summary
- There are 22 missing values for category & team_own columns in df_payment_enriched.
- There are no duplicated values in df_payment_enriched.
- There are 49,059 missing values for sender_id, 164,795 missing values for receiver_id and 1,317,907 missing values for extra_info in df_transaction.
- There are 28 duplicated values in df_transaction.
### Actions
- Fill missing values in sender_id, receiver_id with “Unknown”.
- Remove extra_info column which is not important for transaction and payment analysis.
- Remove 28 duplicated values in df_transaction.

---

## Transaction & Payment Analysis
### 1. Top 3 product_ids with the highest volume.
In [10]:
```python
df_payment_enriched.groupby("product_id", as_index=False).agg({"volume":"sum"}).sort_values("volume", ascending = False).head(3)
```
Out [10]:

<img width="230" height="127" alt="image" src="https://github.com/user-attachments/assets/be87b858-9f74-4da6-8000-5d6db728c7d6" />

**Insight:**
Transaction volume is highly concentrated. Product 1976 dominates the system, contributing a disproportionately large share compared to other products. This creates a single point of dependency risk.

### 2.	Given that 1 product_id is only owed by 1 team, are there any abnormal products against this rule?
In [11]:
```python
df_abnormal = df_payment_enriched.groupby("product_id", as_index=False)["team_own"].nunique()
df_abnormal[df_abnormal["team_own"] != 1]
```
Out [11]:

<img width="205" height="127" alt="image" src="https://github.com/user-attachments/assets/2c5deebb-aa72-4842-84d3-ea65b5194fc4" />

**Insight:**
Several products (including the highest-volume product) are not assigned to any team, indicating data governance and ownership issues that can distort performance evaluation.

### 3.	Find the team has had the lowest performance (lowest volume) since Q2.2023. Find the category that contributes the least to that team.
In [12]:
```python
# Find the team has had the lowest performance (lowest volume) since Q2.2023.
df_q2 = df_payment_enriched[df_payment_enriched["report_month"] >= "2023-04"]
df_q2.groupby("team_own", as_index=False).agg({"volume":"sum"}).sort_values("volume")
```
Out [12]:

<img width="199" height="125" alt="image" src="https://github.com/user-attachments/assets/9d6fd7ff-9517-46eb-8c9d-30bc63e908fb" />

In [13]:
```python
# Find the category that contributes the least to that team.
df_q2[df_q2["team_own"] == "APS"].groupby("category", as_index=False).agg({"volume":"sum"}).sort_values("volume")
```
Out [13]:

<img width="179" height="94" alt="image" src="https://github.com/user-attachments/assets/aad28c38-ee94-417b-81cb-43ba7cf5ea1b" />

**Insight:**
Team APS has the lowest performance, with one category contributing minimally, suggesting ineffective product or category strategy within the team.

### 4.	Find the contribution of source_ids of refund transactions (payment_group = ‘refund’), what is the source_id with the highest contribution?
In [14]:
```python
df_refund = df_payment_enriched[df_payment_enriched["payment_group"] == "refund"]
df_contribute = df_refund.groupby("source_id", as_index=False).agg({"volume":"sum"}).sort_values("volume", ascending=False)
df_contribute['%contribute'] = df_contribute['volume'] / df_contribute['volume'].sum() * 100
df_contribute
```
Out [14]:

<img width="304" height="129" alt="image" src="https://github.com/user-attachments/assets/4ff49559-53dc-4ac8-ae63-5e3021256553" />

**Insight:**
Refund transactions are heavily dominated by a single source (Source ID 38), which signals a systemic issue rather than random user behavior.

### 5.	Define type of transactions (‘transaction_type’) for each row, given:
- transType = 2 & merchant_id = 1205: Bank Transfer Transaction
- transType = 2 & merchant_id = 2260: Withdraw Money Transaction
- transType = 2 & merchant_id = 2270: Top Up Money Transaction
- transType = 2 & others merchant_id: Payment Transaction
- transType = 8, merchant_id = 2250: Transfer Money Transaction
- transType = 8 & others merchant_id: Split Bill Transaction
- Remained cases are invalid transactions

In [15]:
```python
conditions = [(df_transaction["transType"] == 2) & (df_transaction["merchant_id"] == 1205),
              (df_transaction["transType"] == 2) & (df_transaction["merchant_id"] == 2260),
              (df_transaction["transType"] == 2) & (df_transaction["merchant_id"] == 2270),
              (df_transaction["transType"] == 2),
              (df_transaction["transType"] == 8) & (df_transaction["merchant_id"] == 2250),
              (df_transaction["transType"] == 8),
              ]

results = ["Bank Transfer Transaction",
          "Withdraw Money Transaction",
          "Top Up Money Transaction",
          "Payment Transaction",
          "Transfer Money Transaction",
          "Split Bill Transaction",
          ]

df_transaction["transaction_type"] = np.select(conditions,
                                               results,
                                               default="Invalid Transaction")
df_transaction
```
Out [15]:

<img width="1034" height="407" alt="image" src="https://github.com/user-attachments/assets/3a4acdc6-3dfd-4929-b929-636215690b38" />

**Insight:**
Top Up transactions are the core driver of volume, while Payment transactions are high-frequency but lower in value. User behavior aligns with a wallet-first ecosystem.

### 6.	Of each transaction type (excluding invalid transactions): find the number of transactions, volume, senders and receivers.
In [16]:
```python
df_transaction[df_transaction["transaction_type"] != "Invalid Transaction"].groupby("transaction_type", as_index=False).agg({"transaction_id":"nunique",
                                                                                                                              "volume":"sum",
                                                                                                                              "sender_id":"nunique",
                                                                                                                              "receiver_id":"nunique"}).sort_values("transaction_id")
```
Out [16]:

<img width="629" height="224" alt="image" src="https://github.com/user-attachments/assets/2a9b197e-025d-45f3-95b0-d5039b41fdd7" />

**Insight:**
Different transaction types serve different financial behaviors: Payments are daily usage, while Transfers and Bank Transfers carry higher financial risk.

---

## Insights & Recommendations
| Questions | Insights | Recommendations |
|---------|----------|-----------------|
| Top 3 products with the highest volume | Transaction volume is highly concentrated. Product 1976 dominates the system, contributing a disproportionately large share compared to other products. This creates a single point of dependency risk. | Reduce dependency on Product 1976 by identifying and scaling other high-potential products. Perform deeper analysis on product diversification opportunities. |
| Check rule: each product is owned by one team | Several products (including the highest-volume product) are not assigned to any team, indicating data governance and ownership issues that can distort performance evaluation. | Enforce strict product–team ownership rules in the data model and add validation checks in the data pipeline to prevent missing ownership. |
| Lowest-performing team since Q2.2023 and weakest category | Team APS has the lowest performance, with one category contributing minimally, suggesting ineffective product or category strategy within the team. | Conduct a deep-dive analysis on Team APS to identify root causes. Reassess or optimize underperforming categories and reallocate resources if needed. |
| Refund contribution by source_id | Refund transactions are heavily dominated by a single source (Source ID 38), which signals a systemic issue rather than random user behavior. | Investigate Source ID 38 for potential system errors, fraud, or merchant-related issues. Set up monitoring and alerting for refund anomalies. |
| Distribution of transaction types by volume and count | Top Up transactions are the core driver of volume, while Payment transactions are high-frequency but lower in value. User behavior aligns with a wallet-first ecosystem. | Optimize the funnel from Top Up to Payment with incentives and UX improvements to increase downstream monetization. |
| Transaction type performance overview | Different transaction types serve different financial behaviors: Payments are daily usage, while Transfers and Bank Transfers carry higher financial risk. | Apply differentiated strategies: optimize speed and reliability for Payments, and strengthen risk controls and limits for high-value transaction types. |

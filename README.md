# Credit-Risk-Analysis-for-Tawny-Bank

## Introduction

This project focuses on delivering strategic insights to Tawny Bank by analyzing customer demographics, credit behavior, and transaction patterns. Leveraging data from multiple sources users, credit cards, transactions, and merchant categories the analysis aims to evaluate financial health, detect fraud risks, and support personalized engagement strategies. Using SQL for data preparation and Power BI for visualization, the project provides a comprehensive view of customer profiles and risk indicators to drive data-informed decision-making.

## About The Dataset

 Dataset Description

The dataset used for this analysis is composed of four primary tables, each representing a critical component of the banking ecosystem:

1. users_data – Customer Information
   
This table contains detailed demographic and financial information for each user.

Field	Description

|  |  | 
|----------|----------|
| id (Primary Key)  | Unique identifier for each user 
| current_age  |  Current age of the user 
| retirement_age |  Declared retirement age of the user 
|  birth_year  |  Year of birth for the user 
| gender	  |  Gender of the user ( male or female) 
| address |  Residential address of the user 
| latitude | Latitude of the user’s residence      
| longitude | 	Longitude of the user’s residence 
| per_capita_income  |  Per capita income of the user  
| yearly_income  |  Total yearly income of the user 
| total_debt |  Total amount of debt owed by the user 
| credit_score  |  Credit score rating of the user 
| num_credit_cards | Number of credit cards owned by the user 
| current_age  |  retirement_age	Age details   

2. cards_data – Card Attributes
   
This table links each card to a user and describes its properties.

|  |  | 
|----------|----------|
| id (Primary Key)  |  Unique identifier for each credit card
| client_id (Foreign Key to users_data.id)  | Identifier linking the card to its owner in the users_data table
| card_brand  |  Brand or issuer of the credit card (e.g., Visa, Mastercard)
| card_type  |  Type/category of the card (e.g., debit, credit, etc.)
| card_number  |   Unique number on the credit card
| expires  |   Expiry date of the credit card
| card_number  |   Unique number on the credit card	
| cvv  |   Security code of the card
| has_chip	 |  Boolean/Flag indicating if the card has an EMV chip
| num_cards_issued  |  Total number of cards issued for the same account
| credit_limit  |   Credit limit assigned to the card
| acct_open_date  |   Date the credit card account was opened
| year_pin_last_changed | Year the card’s PIN was last changed

3. transactions_data – Transaction Records
   
This table captures every transaction made using a card.

|  |  | 
|----------|----------|
| id (Primary Key)    | Unique identifier for each transaction
| date    | Date of the transaction
| client_id (Foreign Key to users_data.id)    | Identifier linking the transaction to the user in the users_data table
| card_id (Foreign Key to cards_data.id)  | Identifier linking the transaction to the respective card in the cards_data table
| amount   | Transaction amount in the respective currency     
| use_chip   | Boolean/Flag indicating if the chip was used during the transaction
| merchant_id |  Unique identifier of the merchant
| merchant_city   | City where the merchant is located  
| merchant_state   | State where the merchant is located   
| zip    | ZIP code of the merchant's location 
| mcc (Foreign Key to mcc_codes.mcc_id)    | Merchant Category Code representing the type of merchant or transaction.
| errors   | Any errors encountered during the transaction (if any)  

4. mcc_codes – Merchant Categories

This table categorizes merchants based on their business type.

|  |  | 
|----------|----------|
| mcc_id (Primary Key)    | Unique identifier for each Merchant Category Code  
| Description   | Description of the merchant category (e.g., grocery, travel, entertainment)

# Tool and Concept 
#### SQL
#### SQL Server
#### Data Exploration
#### Aggregation
#### Triggers
#### CTE

## Objective

To uncover critical insights into customer financial health, credit risk, and transactional anomalies by analyzing integrated datasets from users, credit cards, transactions, and merchant categories. The goal was to support decision making across fraud detection, and risk management at Tawny Bank. The object of the analysis was further broken down to include:

Understand Customer Profiles & Financial Health

• Segment customers by age, gender, location, credit score, and debt-to-income ratio

• Identify high-risk and financially stable customer groups for targeted engagement

Evaluate Transaction Behavior & Patterns

• Analyze spending by merchant category, location, and payment method (chip, swipe, online)

• Track high-value transactions and out-of-state activity to identify abnormal behavior

Detect and Prevent Fraud

• Create real-time triggers to log potentially fraudulent activity for early intervention

Assess Transaction System Reliability

• Measure and compare error rates across card types, regions, and error categories

• Highlight technical or process inefficiencies affecting the customer experience

Support Data-Driven Decisions

• Provide actionable insights for risk management, customer engagement, and operational improvement

• Deliver visual, interpretable outputs (e.g., Power BI dashboards) to inform stakeholders

## Discussion And Insight

## Anti Fraud Logic and Triggers

##### --- Trigger to Block Card After 5+ Bad PIN Errors

     --Create the transaction log table
     IF OBJECT_ID('transaction_trigger_log', 'U') IS NULL
    BEGIN
    CREATE TABLE transaction_trigger_log (
        log_id INT IDENTITY(1,1) PRIMARY KEY,
        transaction_date DATETIME,
        client_id INT,
        card_id INT,
        amount DECIMAL(18, 2),
        merchant_city VARCHAR(255),
        merchant_state VARCHAR(255),
        has_chip BIT,
        used_chip VARCHAR(255),
        is_fraud_suspected BIT,
        reason VARCHAR(255),
        logged_at DATETIME DEFAULT GETDATE()
        );
       END;
     GO

     IF OBJECT_ID('trg_flag_bad_pin2', 'TR') IS NOT NULL
     DROP TRIGGER trg_flag_bad_pin2;
     GO
     ---Create the trigger
     CREATE TRIGGER trg_flag_bad_pin2
     ON transactions_data
     AFTER INSERT
     AS
     BEGIN
    SET NOCOUNT ON;

    -- Log and flag cards with more than 5 Bad PIN errors
    INSERT INTO transaction_trigger_log (
        transaction_date,
        client_id,
        card_id,
        amount,
        merchant_city,
        merchant_state,
        has_chip,
        used_chip,
        is_fraud_suspected,
        reason
        )
        SELECT 
        i.date,
        i.client_id,
        i.card_id,
        i.amount,
        i.merchant_city,
        i.merchant_state,
        c.has_chip,
        i.use_chip,
        1,
        'Exceeded 5 Bad PIN errors'
    FROM inserted i
    INNER JOIN cards_data c ON i.card_id = c.id
    WHERE (
        SELECT COUNT(*)
        FROM transactions_data t
        WHERE t.card_id = i.card_id AND t.errors LIKE '%Bad PIN%'
    ) > 5;

    -- Block the card by updating its status
    UPDATE c
    SET c.status = 'Blocked'
    FROM cards_data c
    INNER JOIN inserted i ON c.id = i.card_id
    WHERE (
        SELECT COUNT(*)
        FROM transactions_data t
        WHERE t.card_id = i.card_id AND t.errors LIKE '%Bad PIN%'
    ) > 5;
     END;
     GO

This trigger  enhance fraud prevention by a blocking any card that records more than five failed PIN attempts. Such patterns are often early indicators of unauthorized usage or brute-force attacks, where a fraudster attempts to guess a customer's PIN through repeated failure.The trigger is defined as an AFTER INSERT trigger on the transactions_data table. And it activates every time a new transaction is inserted.
The trigger logic checks whether:
1.	The new transaction has an error that contains 'Bad PIN'
2.	The total number of "Bad PIN" errors for the associated card has exceeded five
If both conditions are met:
o	The trigger logs the event to a table called transaction_trigger_log for audit purposes.
o	It automatically updates the cards_data table to change the card’s status to 'Blocked'.
The trigger is important because; repeated PIN failures are often a sign of card testing, compromised accounts, or unauthorized use. Also blocking the card promptly reduces the chance of a successful unauthorized transaction.

#### Stop transactions above customer credit limit

     -- Create new trigger to block over-limit transactions
     CREATE TRIGGER trg_block_overlimit_transactions
     ON transactions_data
     INSTEAD OF INSERT
     AS
     BEGIN
    SET NOCOUNT ON;

    -- Insert only those that stay within the credit limit
    INSERT INTO transactions_data (
        id, date, client_id, card_id, amount,
        use_chip, merchant_id, merchant_city,
        merchant_state, zip, mcc, errors
    )
    SELECT 
        i.id, i.date, i.client_id, i.card_id, i.amount,
        i.use_chip, i.merchant_id, i.merchant_city,
        i.merchant_state, i.zip, i.mcc, i.errors
    FROM inserted i
    JOIN cards_data c ON i.card_id = c.id
    WHERE (
        SELECT ISNULL(SUM(t.amount), 0)
        FROM transactions_data t
        WHERE t.card_id = i.card_id
    ) + i.amount <= c.credit_limit;
    
    -- Optional: You can log failed attempts in a separate table
     END;
     GO

This trigger was implemented to enforce real-time credit control by automatically preventing transactions that would exceed a card’s assigned credit limit. It ensures that all cardholders operate within approved transaction limit, minimizing the risk of overspending and maintaining the bank’s credit exposure discipline.

When a new transaction is attempted, the trigger:
1.	Calculates the current total transaction amount already associated with the card.
2.	Adds the new transaction amount from the insert attempt.
3.	Compares the total against the card’s assigned credit_limit in cards_data.

If the new transaction would not exceed the credit limit then record is inserted successfully and if it would exceed the limit, the transaction is silently blocked (i.e., not inserted).
 Why this logic is important is that it ensures customers cannot transact beyond their approved credit line, supporting responsible credit usage and minimizing default risk. It also Maintains strict control over issued credit, reducing the risk of financial loss due to overspending or system loopholes. It removes the need for manual checks or post-factum reversals, enabling real-time compliance at the data layer and lastly  helps prevent issues like failed settlements, chargebacks, or overdrafts resulting from exceeding credit limit

##### Flag  more than 5 transactions done with an hour online at different locations and where the amount was increased per transaction

     WITH ordered_txn AS (
         SELECT
             t1.id,
             t1.card_id,
             t1.client_id,
             t1.date,
             t1.amount,
             t1.merchant_state,
             t1.merchant_city,
             (
                 SELECT COUNT(*)
                 FROM transactions_data t2
                 WHERE t2.card_id = t1.card_id
                   AND t2.merchant_city = 'Online'
                   AND DATEDIFF(MINUTE, t2.date, t1.date) BETWEEN 0 AND 60
             ) AS txn_count,
             (
                 SELECT COUNT(DISTINCT t2.merchant_city)
                 FROM transactions_data t2
                 WHERE t2.card_id = t1.card_id
                   AND t2.merchant_city = 'Online'
                   AND DATEDIFF(MINUTE, t2.date, t1.date) BETWEEN 0 AND 60
             ) AS distinct_locations,
             LAG(t1.amount) OVER (PARTITION BY t1.card_id ORDER BY t1.date) AS prev_amount
         FROM transactions_data t1
         WHERE t1.merchant_city = 'Online'
     )
     SELECT *
     FROM ordered_txn
     WHERE txn_count > 5
       AND distinct_locations > 1
       AND amount > ISNULL(prev_amount, 0)
     ORDER BY card_id, date;
     
As part of the transactional risk analysis for the Bank, I developed a targeted SQL routine designed to identify suspicious online spending behavior that may signal potential card compromise, both activity, or coordinated fraud attempts.The logic focuses specifically on: Online transactions on the same card within a short time frame (60 minutes) across more than one merchant location	and with a pattern of increasing transaction amounts. I flag any card that has more than three online transactions within one hour, where the transactions originate from multiple merchant cities and the transaction amount increases relative to the previous one. This pattern is a classic indicator of card testing, where fraudsters initiate small transactions to validate a compromised card, use different online merchant routes or IPs and gradually escalate the amount once initial transactions succeed. This detection strategy helps us; Identify compromised cards early before larger fraudulent losses occur, detect automated or bot-driven attacks exploiting online platforms and Surface geographic inconsistencies in transaction behavior (e.g., multiple locations in one hour is highly unlikely for a legitimate customer)

More importantly, this rule applies only to cards equipped with EMV chips, yet used in non-chip channels like online, where security is inherently lower. The business implications of this is that: A reduce fraud-related financial exposure, improves cardholder trust by proactively securing accounts and supports compliance with industry best practices for transaction monitoring.  

#### Customer Credit Risk Segmentation Based on Debt-to-Income Ratio and Credit Score Categories

     SELECT 
         id AS client_id,
         credit_score,
         total_debt,
         yearly_income,
       
         -- Debt-to-Income Ratio
         CAST(total_debt * 1.0 / NULLIF(yearly_income, 0) AS DECIMAL(6,2)) AS debt_to_income_ratio,
     
         -- DTI Risk Category
         CASE 
             WHEN (total_debt * 1.0 / NULLIF(yearly_income, 0)) < 0.36 THEN 'Acceptable'
             WHEN (total_debt * 1.0 / NULLIF(yearly_income, 0)) BETWEEN 0.36 AND 0.49 THEN 'Moderate Risk'
             WHEN (total_debt * 1.0 / NULLIF(yearly_income, 0)) >= 0.50 THEN 'High Risk'
             ELSE 'Unknown'
         END AS dti_category,
     
         -- Credit Score Category
         CASE 
             WHEN credit_score >= 800 THEN 'Excellent'
             WHEN credit_score >= 740 THEN 'Very Good'
             WHEN credit_score >= 670 THEN 'Good'
             WHEN credit_score >= 580 THEN 'Fair'
             ELSE 'Poor'
         END AS credit_score_category
     
     FROM users_data
     WHERE yearly_income > 0;

 
![](10.png)

Over 70% of the customers fall into the High Risk category based on their debt-to-income (DTI) ratio, which is defined as the ratio of total debt to yearly income. This suggests that most customers have significant debt relative to their income, often with DTI values exceeding 1.0, and in some cases as high as 2.5. These customers may be financially strained, increasing the likelihood of missed payments or default. This insight is vital for risk management teams, who can prioritize these individuals for monitoring, intervention, or restructuring.

Several customers have "Good" or even "Very Good" credit scores, yet are flagged as High Risk due to excessive debt. For example: Client 708 has a credit score of 722, but a DTI of 0.81 Client 1711 has a credit score of 728, but a DTI of 2.10This discrepancy highlights the limitations of relying solely on credit scores for risk assessments.

Only a small number of customers fall into the Acceptable DTI range (< 0.36). These customers are more likely to be financially stable and have a better capacity to manage credit responsibly. Example: Client 1116 has a DTI of 0.07 and a Very Good credit score. These customers are ideal targets for premium credit products, credit line increases, or investment services.

Some customers present unique profiles worth noting: Client 68: Has no debt and a Good credit score possibly a new customer or one who recently paid off their obligations. Client 777: Has zero debt and a very high income ($216,740) but only a credit score of 700s  an opportunity to upsell premium cards or investment products.

Some customers manage to maintain good credit scores despite very high DTI, indicating they are likely making payments on time  but they are still overleveraged. These are vulnerable to economic shocks (e.g., job loss, inflation) and could shift from good payers to defaulters quickly.

#### Most Common Card Errors

     SELECT 
         errors AS Type0fError,
         COUNT(*) AS TotalError
     FROM transactions_data
     WHERE errors IS NOT NULL AND errors <> ''
     GROUP BY errors
     ORDER BY TotalError DESC;

![](7.png)

Over 70% of all transaction errors are due to “Insufficient Balance”   This indicates that a large number of customers are attempting transactions beyond their available funds or credit limits. Next to that is Authentication Failures Signal Possible Fraud Attempts such as Bad PIN and Bad CVV errors. These are typical in unauthorized usage or brute-force attempts. While some may be user error, repeated failures—especially when combined may point to card testing or fraud. Other technical errors include bad card numbers and expired card attempts. These could be caused by POS system incompatibilities, merchant-side failures, or data formatting issues. However, the risk here is that this undermines customer trust and contributes to failed transactions that can impact revenue.

#### Error Rate By City/State

     SELECT 
        t.merchant_state,
         COUNT(*) AS Total_Transaction,
         SUM(CASE WHEN errors IS NOT NULL AND errors <> '' THEN 1 ELSE 0 END) AS TotalError,
         CAST(SUM(CASE WHEN errors IS NOT NULL AND errors <> '' THEN 1 ELSE 0 END) * 100.0 / 
     	COUNT(*) AS DECIMAL(5,2)) AS ErrorRate
     FROM transactions_data t
     GROUP BY merchant_state
     ORDER BY ErrorRate DESC; 8.png
     
     
 ![](8.png)
     
Regions such as South Korea (33.33% error rate), Wyoming (WY) (6.90%), and Ireland (5.56%) report notably high error percentages. However, it’s important to emphasize that these regions also recorded very low transaction volumes, which means; the high error rates may not be statistically significant and the data could be skewed by a small number of problematic.  States such as Alabama (1.57%), Missouri (1.59%), Iowa (1.60%), and New York (1.66%) show consistently low error rates, despite handling a large number of transactions. This suggests; Stable processing environments, potentially stronger merchant infrastructure and more mature payment systems in these areas


#### Error Rate By Card Type
   
     SELECT 
         c.card_type,
         COUNT(*) AS total_Transactions,
         SUM(CASE WHEN t.errors IS NOT NULL AND t.errors <> '' THEN 1 ELSE 0 END) AS TotalError,
         CAST(SUM(CASE WHEN t.errors IS NOT NULL AND t.errors <> '' THEN 1 ELSE 0 END) * 100.0 /
     	COUNT(*) AS DECIMAL(5,2)) AS Error_Rate
     FROM transactions_data t
     JOIN cards_data c ON t.card_id = c.id
     GROUP BY c.card_type
     ORDER BY Error_Rate DESC;

![](9.png)

All card types have error rates below 2%, which suggests that our core transaction processing infrastructure is fundamentally stable and performing well.While the error rate is relatively low, the high volume of debit card usage makes even small inefficiencies impactful at scale.Credit cards perform slightly better than debit, suggesting more consistent authorizations or fewer user-related issues (e.g., overdraft limits).Prepaid cards outperform others, which may be due to stricter limits, preloaded balances, or simplified processing paths.

## Recommendation

•	Detect multiple online transactions within 1 hour from different locations

•	Monitor repeated bad PIN/CVV attempts for card compromise

•	Focus on debit card error reduction — high volume, highest error rate (1.80%)

•	Benchmark prepaid card systems — lowest error rate (1.50%) despite tighter limits

•	Identify and monitor users with high DTI + poor credit scores

•	Target financially healthy customers for premium offers

•	Segment customers by behavior, not just credit score

•	Investigate high-error regions (e.g., South Korea, Wyoming) cautiously due to low volume

•	Audit NULL merchant location data (17,000+ records) for better risk visibility

•	Use low-error regions (e.g., AL, MO, NY) as system reliability benchmarks

•	Keep fraud trigger logs active wrong pins

•	Monitor monthly error trends across card types and locations

•	Align fraud sensitivity with seasonal transaction spikes

## Conclusion

This project provided a comprehensive analysis of Aurora Bank’s customer behavior, transaction performance, credit risk, and fraud exposure. Through in-depth segmentation, error tracking, and fraud detection logic, we uncovered actionable insights to strengthen security, improve operational efficiency, and support data-driven decision-making. By combining technical rigor with strategic recommendations, this analysis equips the bank with the tools to enhance customer trust, reduce risk, and optimize transaction reliability across all card types and regions.












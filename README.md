# Credit-Risk-Analysis-for-Tawny-Bank

## Introduction

This project focuses on delivering strategic insights to Tawny Bank by analyzing customer demographics, credit behavior, and transaction patterns. Leveraging data from multiple sources users, credit cards, transactions, and merchant categories the analysis aims to evaluate financial health, detect fraud risks, and support personalized engagement strategies. Using SQL for data preparation and Power BI for visualization, the project provides a comprehensive view of customer profiles and risk indicators to drive data-informed decision-making.

## About The Dataset

CARDS DATA SCHEMA

id (Primary Key)	 Unique identifier for each credit card.

client_id (Foreign Key to users_data.id)	 Identifier linking the card to its owner in the users_data table.

card_brand	 Brand or issuer of the credit card (e.g., Visa, Mastercard).

card_type	 Type/category of the card (e.g., debit, credit, etc.).

card_number	 Unique number on the credit card.

expires	 Expiry date of the credit card.

cvv	 Security code of the card.

has_chip	 Boolean/Flag indicating if the card has an EMV chip.

num_cards_issued	 Total number of cards issued for the same account.

credit_limit	 Credit limit assigned to the card.

acct_open_date	 Date the credit card account was opened.

year_pin_last_changed	 Year the card’s PIN was last changed.

USER DATA SCHEMA

id (Primary Key)	 Unique identifier for each user.

current_age	 Current age of the user.

retirement_age	 Declared retirement age of the user.

birth_year	 Year of birth for the user.

birth_month	 Month of birth for the user.

gender	 Gender of the user ( male or female).

address	 Residential address of the user.

latitude	 Latitude of the user’s residence.

longitude	 Longitude of the user’s residence.

per_capita_income	 Per capita income of the user.

yearly_income	 Total yearly income of the user.

total_debt	 Total amount of debt owed by the user.

credit_score	 Credit score rating of the user.

num_credit_cards	 Number of credit cards owned by the user.

MCC CODES SCHEMA

mcc_id (Primary Key)	Unique identifier for each Merchant Category Code.

Description	Description of the merchant category (e.g., grocery, travel, entertainment).

TRANSACTIONS DATA SCHEMA

id (Primary Key)	 Unique identifier for each transaction.

date	 Date of the transaction.

client_id (Foreign Key to users_data.id)	 Identifier linking the transaction to the user in the users_data table.

card_id (Foreign Key to cards_data.id)	 Identifier linking the transaction to the respective card in the cards_data table.

amount	 Transaction amount in the respective currency.

use_chip	 Boolean/Flag indicating if the chip was used during the transaction.

merchant_id	 Unique identifier of the merchant.

merchant_city	 City where the merchant is located.

merchant_state	 State where the merchant is located.

zip	 ZIP code of the merchant's location.

mcc (Foreign Key to mcc_codes.mcc_id)	 Merchant Category Code representing the type of merchant or transaction.

errors	 Any errors encountered during the transaction (if any).

## Tool Used: SQL Server

## Objective

To uncover critical insights into customer financial health, credit risk, and transactional anomalies by analyzing integrated datasets from users, credit cards, transactions, and merchant categories. The goal was to support decision making across fraud detection, and risk management at Tawny Bank. The object of the analysis was further broken down to include:

1. Analyzed spending trends by Merchant Category (MCC) and merchant location.

2. Identified high-value transactions and flagged suspicious out-of-state online spending.

3. Calculated error rates by merchant state and card type.

4. Detected fraud behavior by flagging cards with 3+ consecutive failed transactions within a day.

5. Assessed regional risk using average credit scores by merchant state.

## Discussion And Insight

![](3.jpg)




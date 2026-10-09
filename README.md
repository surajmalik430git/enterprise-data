# enterprise-data
# Enterprise Data Reconciliation

A Python pipeline that reconciles three messy systems (a SQL user database, 100,000 raw server logs and a daily exchange-rate file) into a USD revenue dashboard.

## Dashboard
<img width="727" height="314" alt="Screenshot 2026-10-09 163853" src="https://github.com/user-attachments/assets/d1f2affb-4130-4c90-b33d-c4761303770c" />


## What the pipeline does
1. **SQL (window function):** Users appear up to 3 times as their status changes. I used ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY updated_at DESC) to keep each user's latest row, then filtered to Active users (14,183).
2. **Regex:** I extracted the date, user ID, product ID and euro value from 100,000 log strings with str.extract() and removed 5,098 ERROR lines, leaving 94,902 transactions.
3. **Weekend exchange rates:** The rate file has no rates for 104 of 365 days. A plain inner join would have dropped about 28.4% of the transactions (94,902 down to 67,918). I forward-filled the rates (ffill()) so each weekend transaction uses the last available weekday rate, and no transactions were lost.
4. **Merge and revenue:** I joined transactions to rates by date and to Active users by user ID (53,899 rows), then calculated USD_Revenue = Euro_Value x EUR_to_USD.
5. **Visualisation:** Total USD revenue by month and the top 5 customers by USD spent.

## Results
- Total USD revenue from Active users: about $148.3M
- Monthly revenue ranges from about $11.4M (Feb) to $12.8M (Jan and Aug)
- Top customer: U-06582 (about $40,051)

## Performance
Fully vectorised pandas with no loops. It runs in a few seconds on 100,000 rows.

## Tools
Python, pandas, SQLite, regex, matplotlib, seaborn

## How to run
Put the notebook and the three data files (enterprise_database.db, server_logs.txt, daily_exchange_rates.csv) in one folder, then run all cells. In the notebook, the file names in the code must match your file names.

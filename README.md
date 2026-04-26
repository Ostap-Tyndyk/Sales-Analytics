# Email & Account Analytics — BigQuery SQL

A BigQuery query that merges session, account, and email data to surface the **top 10 countries** by account count and emails sent.

## What it does

1. Counts unique accounts per date, country, and account attributes
2. Counts email engagement (sent / opened / clicked) per the same dimensions
3. Unions both datasets and computes country-level totals
4. Ranks countries and filters to top 10 by either metric

## Tables used

`DA.session` · `DA.session_params` · `DA.account_session` · `DA.account` · `DA.email_sent` · `DA.email_open` · `DA.email_visit`

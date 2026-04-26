WITH key_account_metrics AS -- key account metrics
(
  SELECT s.date,
  sp.country, a.send_interval, a.is_verified, a.is_unsubscribed,
  COUNT(DISTINCT a.id) as account_cnt
  FROM `DA.session` s
  INNER JOIN `DA.session_params` sp
  ON s.ga_session_id = sp.ga_session_id
  INNER JOIN `DA.account_session` acs
  ON s.ga_session_id = acs.ga_session_id
  INNER JOIN `DA.account` a
  ON acs.account_id = a.id
  GROUP BY 1, 2, 3, 4, 5
),
key_emaiL_metrics AS -- key email metrics
(
  SELECT DATE_ADD(s.date, INTERVAL sent_date DAY) as email_date,
  sp.country, a.send_interval, a.is_verified, a.is_unsubscribed,
  COUNT(DISTINCT es.id_message) as sent_msg,
  COUNT(DISTINCT eo.id_message) as open_msg,
  COUNT(DISTINCT ev.id_message) as visit_msg
  FROM `DA.session` s
  INNER JOIN `DA.session_params` sp
  ON s.ga_session_id = sp.ga_session_id
  INNER JOIN `DA.account_session` acs
  ON s.ga_session_id = acs.ga_session_id
  INNER JOIN `DA.account` a
  ON acs.account_id = a.id
  INNER JOIN `DA.email_sent` es
  ON a.id = es.id_account
  LEFT JOIN `DA.email_open` eo
  ON es.id_message = eo.id_message
  LEFT JOIN `DA.email_visit` ev
  ON es.id_message = ev.id_message
  GROUP BY 1, 2, 3, 4, 5
), union_data AS
(
  -- removing NULL values using a subquery
  SELECT date, country, send_interval, is_verified, is_unsubscribed,
  SUM(account_cnt) as account_cnt,
  SUM(sent_msg) as sent_msg,
  SUM(open_msg) as open_msg,
  SUM(visit_msg) as visit_msg
  FROM
  (
  SELECT date, country, send_interval, is_verified, is_unsubscribed,
  account_cnt,
  NULL as sent_msg,
  NULL as open_msg,
  NULL as visit_msg
  FROM key_account_metrics

  UNION ALL

  SELECT email_date, country, send_interval, is_verified, is_unsubscribed,
  NULL as account_cnt,
  sent_msg,
  open_msg,
  visit_msg
  FROM key_email_metrics
  )
  GROUP BY 1, 2, 3, 4, 5
), total_country_account AS -- count totals
(
  SELECT *,
  SUM(account_cnt) OVER(PARTITION BY country) as total_country_account_cnt,
  SUM(sent_msg) OVER(PARTITION BY country) as total_country_sent_cnt
  FROM union_data
), rank_country AS -- count ranks
(
  SELECT *,
  DENSE_RANK() OVER(ORDER BY total_country_account_cnt DESC) as rank_total_country_account_cnt,
  DENSE_RANK() OVER(ORDER BY total_country_sent_cnt DESC) as rank_total_country_sent_cnt
  FROM total_country_account
)
SELECT *
FROM rank_country
WHERE rank_total_country_account_cnt <= 10 OR rank_total_country_sent_cnt <= 10

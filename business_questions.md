# Business Questions & Analytical Insights

1. **Which companies are our most active users?**
   - *Answers using:* `dim_companies` & `fct_company_daily_activity`

2. **How are users moving through our sales funnel?**
   - *Answers using:* `fct_sessions` & `fct_conversions`

3. **What is our total revenue, and which items sell the most?**
   - *Answers using:* `fct_orders` & `fct_order_items`

4. **Which pages on our website get the most traffic?**
   - *Answers using:* `dim_pages` & `fct_page_views`

5. **Are users staying engaged or bouncing quickly?**
   - *Answers using:* `fct_sessions` (duration/events)

6. **Does our traffic come mostly from mobile devices or desktops?**
   - *Answers using:* `dim_devices` & `fct_sessions`

7. **Which marketing campaigns actually lead to purchases?**
   - *Answers using:* `dim_traffic_sources` & `fct_conversions`

8. **How long do users remain active from their first visit to their last?**
   - *Answers using:* `dim_users` & `int_user_activity`

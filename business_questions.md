# Business Questions & Analytical Insights

This document outlines the key business questions that the `website_tracking_analytics` data pipeline is designed to answer. By transforming raw GA4 data into structured dimension and fact models, we can drive data-informed decision-making.

## 1. User Behavior & Engagement
* **What are our most visited pages and how do they correlate with user categories?**
    * *Model:* `dim_pages`, `fct_page_views`
* **What is the average session duration, and does it differ by user segment?**
    * *Model:* `fct_sessions`
* **How often do users return to the platform, and what is their "first seen" vs. "last seen" activity window?**
    * *Model:* `dim_users`, `int_user_activity`

## 2. Company & B2B Performance
* **Which companies are most active on our platform, and what is their total user count?**
    * *Model:* `dim_companies`
* **What is the daily activity trend per company (sessions, views, conversions)?**
    * *Model:* `fct_company_daily_activity`
* **Is there a correlation between company size/industry and the frequency of platform conversions?**
    * *Model:* `dim_companies`, `fct_conversions`

## 3. Conversion & Revenue Analysis
* **What is the total revenue generated, and how is it distributed across different items/products?**
    * *Model:* `fct_orders`, `fct_order_items`
* **Which traffic sources (campaigns/mediums) yield the highest conversion rates?**
    * *Model:* `dim_traffic_sources`, `fct_conversions`
* **What is the current conversion funnel? Where do users drop off before completing a purchase?**
    * *Model:* `fct_sessions`, `fct_conversions`

## 4. Technical Performance
* **Are there specific device or operating system combinations that experience lower engagement or higher bounce rates?**
    * *Model:* `dim_devices`, `fct_sessions`

---

*Note: These questions form the foundation for the dashboards and reports built in the BI layer, mapping directly to the `marts` layer of the dbt pipeline.*

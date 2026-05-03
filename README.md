# 🎬 Strategic Merger in OTT — LioCinema & Jotstar Analysis

> **SQL-based data analysis project** exploring subscriber behavior, content performance, revenue trends, and merger feasibility between two major OTT platforms: **LioCinema** and **Jotstar**.

---

## 📌 Project Overview

This project simulates a real-world **strategic merger analysis** between two Indian OTT platforms — LioCinema and Jotstar — using SQL queries on their transactional databases. The goal is to evaluate whether a merger would be mutually beneficial by comparing user growth, content libraries, subscription patterns, watch time, and revenue across January–November 2024.

---

## 🗂️ Repository Structure

```
strategic-merger-in-ott/
│
├── meta_data.txt                        # Schema documentation for both databases
├── primary question answers             # SQL queries with answers to 10 business questions
├── primary_and_secondary_questions.pdf  # Full list of analysis questions
└── problem_statement.pdf                # Business problem & merger context
```

---

## 🗄️ Database Schema

### LioCinema DB (`liocinema_db`)

#### `contents`
| Column | Description |
|---|---|
| `content_id` | Unique identifier (e.g., CLCMHIROM1bdbc) |
| `content_type` | Movie or Series |
| `language` | Content language (Hindi, English, etc.) |
| `genre` | Genre (Action, Drama, Romance, etc.) |
| `run_time` | Duration in minutes |

#### `subscribers`
| Column | Description |
|---|---|
| `user_id` | Unique subscriber ID (e.g., UIDLC1d62ccb715a) |
| `age_group` | Age group (18-24, 25-34, 35-44, 45+) |
| `city_tier` | Tier 1 / Tier 2 / Tier 3 |
| `subscription_date` | Date of joining (YYYY-MM-DD) |
| `subscription_plan` | Initial plan: Free / Basic / Premium |
| `last_active_date` | Last activity date (NULL = currently active) |
| `plan_change_date` | Date of plan change |
| `new_subscription_plan` | Updated plan after upgrade/downgrade |

#### `content_consumption`
| Column | Description |
|---|---|
| `user_id` | Subscriber ID |
| `device_type` | Mobile / TV / Tablet |
| `total_watch_time_mins` | Total minutes watched |

---

### Jotstar DB (`jotstar_db`)

Same structure as LioCinema DB with one key difference:  
**Subscription plans** → Free / **VIP** / Premium (no "Basic"; VIP is the mid-tier)

---

## 🧠 Business Questions & SQL Queries

### Q1. Total Users & Growth Trends
*What is the total number of users for each platform and their growth trend (Jan–Nov 2024)?*

```sql
-- Monthly new user count
SELECT MONTHNAME(subscription_date) AS month, COUNT(user_id) AS user_count
FROM subscribers
GROUP BY MONTHNAME(subscription_date)
ORDER BY user_count;

-- H1 vs H2 growth percentage
SELECT
    CONCAT(ROUND(COUNT(CASE WHEN subscription_date BETWEEN '2024-01-01' AND '2024-06-01' THEN user_id END) * 100 / COUNT(user_id), 2), '%') AS h1_growth_pct,
    CONCAT(ROUND(COUNT(CASE WHEN subscription_date BETWEEN '2024-07-01' AND '2024-11-01' THEN user_id END) * 100 / COUNT(user_id), 2), '%') AS h2_growth_pct
FROM subscribers;
```

---

### Q2. Content Library Comparison
*How do the two platforms differ in content volume, language, and type?*

```sql
-- Total content count
SELECT COUNT(content_id) AS total_content FROM contents;

-- Unique content types and languages
SELECT
    GROUP_CONCAT(DISTINCT content_type ORDER BY content_type ASC) AS content_types,
    GROUP_CONCAT(DISTINCT language ORDER BY language ASC) AS languages
FROM contents;
```

---

### Q3. User Demographics
*Distribution of users by age group, city tier, and subscription plan.*

```sql
SELECT
    COALESCE(city_tier, 'Unknown') AS city_tier,
    COUNT(user_id) AS user_count,
    COUNT(CASE WHEN age_group BETWEEN 18 AND 24 THEN user_id END) AS age_18to24,
    COUNT(CASE WHEN age_group BETWEEN 25 AND 34 THEN user_id END) AS age_25to34,
    COUNT(CASE WHEN age_group BETWEEN 35 AND 44 THEN user_id END) AS age_35to44,
    COUNT(CASE WHEN age_group >= 45 THEN user_id END) AS age_45plus,
    COUNT(CASE WHEN subscription_plan = 'Basic' THEN user_id END) AS basic_users,
    COUNT(CASE WHEN subscription_plan = 'Free' THEN user_id END) AS free_users,
    COUNT(CASE WHEN subscription_plan = 'Premium' THEN user_id END) AS premium_users
FROM subscribers
GROUP BY COALESCE(city_tier, 'Unknown')
ORDER BY city_tier;
```

---

### Q4. Active vs. Inactive Users
*What % of users are active vs. inactive? Breakdown by age group and plan.*

```sql
SELECT
    COUNT(user_id) AS total_users,
    COUNT(CASE WHEN last_active_date IS NULL THEN user_id END) * 100 / COUNT(user_id) AS active_pct,
    COUNT(CASE WHEN last_active_date IS NOT NULL THEN user_id END) * 100 / COUNT(user_id) AS inactive_pct,
    COUNT(CASE WHEN age_group BETWEEN 18 AND 24 THEN user_id END) AS age_18to24,
    COUNT(CASE WHEN age_group BETWEEN 25 AND 34 THEN user_id END) AS age_25to34,
    COUNT(CASE WHEN age_group BETWEEN 35 AND 44 THEN user_id END) AS age_35to44,
    COUNT(CASE WHEN age_group >= 45 THEN user_id END) AS age_45plus,
    COUNT(CASE WHEN subscription_plan = 'Basic' THEN user_id END) AS basic_users,
    COUNT(CASE WHEN subscription_plan = 'Free' THEN user_id END) AS free_users,
    COUNT(CASE WHEN subscription_plan = 'Premium' THEN user_id END) AS premium_users
FROM subscribers;
```

---

### Q5. Watch Time Analysis
*Average watch time comparison by city tier and device type.*

```sql
-- Overall average watch time
SELECT ROUND(AVG(total_watch_time_mins), 2) AS avg_watch_time
FROM content_consumption;

-- By device type and city tier
SELECT
    c.device_type,
    s.city_tier,
    ROUND(AVG(c.total_watch_time_mins), 2) AS avg_watch_time
FROM content_consumption c
JOIN subscribers s ON c.user_id = s.user_id
GROUP BY c.device_type, s.city_tier
ORDER BY s.city_tier, c.device_type;
```

---

### Q6. Inactivity vs. Watch Time Correlation
*Are less engaged users more likely to become inactive?*

```sql
SELECT
    CASE WHEN s.last_active_date IS NULL THEN 'Active' ELSE 'Inactive' END AS user_status,
    COUNT(s.user_id) AS total_users,
    SUM(c.total_watch_time_mins) AS total_watch_time,
    AVG(c.total_watch_time_mins) AS avg_watch_time,
    ROUND(COUNT(s.user_id) * 100 / SUM(COUNT(s.user_id)) OVER (), 2) AS pct_of_users
FROM subscribers s
JOIN content_consumption c ON s.user_id = c.user_id
GROUP BY user_status;
```

---

### Q7. Downgrade Trends
*Which platform has a higher downgrade rate?*

```sql
SELECT 'LioCinema' AS platform,
    COUNT(CASE WHEN subscription_plan > new_subscription_plan THEN user_id END) AS total_downgrades,
    COUNT(user_id) AS total_users,
    ROUND(COUNT(CASE WHEN subscription_plan > new_subscription_plan THEN user_id END) * 100 / COUNT(user_id), 2) AS downgrade_pct
FROM liocinema_db.subscribers

UNION ALL

SELECT 'Jotstar' AS platform,
    COUNT(CASE WHEN subscription_plan > new_subscription_plan THEN user_id END) AS total_downgrades,
    COUNT(user_id) AS total_users,
    ROUND(COUNT(CASE WHEN subscription_plan > new_subscription_plan THEN user_id END) * 100 / COUNT(user_id), 2) AS downgrade_pct
FROM jotstar_db.subscribers;
```

---

### Q8. Upgrade Patterns
*Most common upgrade transitions on each platform.*

```sql
SELECT 'LioCinema' AS platform, subscription_plan, new_subscription_plan,
    COUNT(user_id) AS upgrade_count
FROM liocinema_db.subscribers
WHERE new_subscription_plan > subscription_plan
GROUP BY subscription_plan, new_subscription_plan

UNION ALL

SELECT 'Jotstar' AS platform, subscription_plan, new_subscription_plan,
    COUNT(user_id) AS upgrade_count
FROM jotstar_db.subscribers
WHERE new_subscription_plan > subscription_plan
GROUP BY subscription_plan, new_subscription_plan

ORDER BY upgrade_count ASC;
```

---

### Q9. Paid User Distribution by City Tier
*How does paid user % vary across Tier 1, 2, 3 cities?*

```sql
SELECT 'LioCinema' AS platform, city_tier,
    COUNT(CASE WHEN subscription_plan = 'Free' THEN user_id END) * 100 / COUNT(user_id) AS free_pct,
    COUNT(CASE WHEN subscription_plan = 'Basic' THEN user_id END) * 100 / COUNT(user_id) AS basic_pct,
    COUNT(CASE WHEN subscription_plan = 'Premium' THEN user_id END) * 100 / COUNT(user_id) AS premium_pct,
    NULL AS vip_pct
FROM liocinema_db.subscribers
GROUP BY city_tier

UNION ALL

SELECT 'Jotstar' AS platform, city_tier,
    COUNT(CASE WHEN subscription_plan = 'Free' THEN user_id END) * 100 / COUNT(user_id) AS free_pct,
    NULL AS basic_pct,
    COUNT(CASE WHEN subscription_plan = 'Premium' THEN user_id END) * 100 / COUNT(user_id) AS premium_pct,
    COUNT(CASE WHEN subscription_plan = 'VIP' THEN user_id END) * 100 / COUNT(user_id) AS vip_pct
FROM jotstar_db.subscribers
GROUP BY city_tier

ORDER BY city_tier, platform;
```

---

### Q10. Revenue Analysis (Jan–Nov 2024)
*Total revenue calculation based on plan pricing.*

**Pricing Assumptions:**

| Platform | Plan | Monthly Price (₹) |
|---|---|---|
| LioCinema | Basic | ₹69 |
| LioCinema | Premium | ₹129 |
| Jotstar | VIP | ₹159 |
| Jotstar | Premium | ₹359 |

```sql
-- LioCinema Revenue
SELECT 'LioCinema' AS platform, subscription_plan,
    COUNT(user_id) AS total_users,
    SUM(
        (CASE WHEN last_active_date IS NULL
            THEN DATEDIFF('2024-11-30', plan_change_date)
            ELSE DATEDIFF(last_active_date, plan_change_date)
        END) / 30.0
    ) AS active_months,
    SUM(
        (CASE WHEN last_active_date IS NULL
            THEN DATEDIFF('2024-11-30', plan_change_date)
            ELSE DATEDIFF(last_active_date, plan_change_date)
        END) / 30.0
        * (CASE WHEN subscription_plan = 'Basic' THEN 69
               WHEN subscription_plan = 'Premium' THEN 129
               ELSE 0 END)
    ) AS total_revenue
FROM liocinema_db.subscribers
WHERE subscription_plan IN ('Basic', 'Premium')
GROUP BY subscription_plan;

-- Jotstar Revenue
SELECT 'Jotstar' AS platform, subscription_plan,
    COUNT(user_id) AS total_users,
    SUM(
        (CASE WHEN last_active_date IS NULL
            THEN DATEDIFF('2024-11-30', plan_change_date)
            ELSE DATEDIFF(last_active_date, plan_change_date)
        END) / 30.0
    ) AS active_months,
    SUM(
        (CASE WHEN last_active_date IS NULL
            THEN DATEDIFF('2024-11-30', plan_change_date)
            ELSE DATEDIFF(last_active_date, plan_change_date)
        END) / 30.0
        * (CASE WHEN subscription_plan = 'VIP' THEN 159
               WHEN subscription_plan = 'Premium' THEN 359
               ELSE 0 END)
    ) AS total_revenue
FROM jotstar_db.subscribers
WHERE subscription_plan IN ('VIP', 'Premium')
GROUP BY subscription_plan;
```

---

## 🔍 Key Analysis Dimensions

| Dimension | Metric |
|---|---|
| **User Growth** | Monthly new subscriptions, H1 vs H2 split |
| **Content Library** | Count by type, language, genre |
| **Demographics** | Age group × City tier × Plan |
| **Engagement** | Active vs. Inactive user ratio |
| **Watch Behavior** | Avg watch time by device & city |
| **Monetization** | Upgrade/downgrade rate, revenue per plan |

---

## 🛠️ Tools & Technologies

- **MySQL** — All queries written and tested in MySQL
- **SQL Concepts Used** — JOINs, CASE-WHEN, GROUP BY, UNION ALL, Window Functions (`OVER()`), DATEDIFF, COALESCE, GROUP_CONCAT, Subqueries

---

## 👩‍💻 Author

**Madhuri**  
Data Analyst | SQL Enthusiast  
GitHub: [@madhuri808](https://github.com/madhuri808)

---

## 📄 License

This project is for educational and portfolio purposes only.

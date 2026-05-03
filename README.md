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

## 🧠 Business Questions, SQL Queries & Insights

---

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

**📊 Output:**

| month | user_count |
|-------|------------|
| January | 3934 |
| February | 3939 |
| March | 3954 |
| April | 3984 |
| May | 3998 |
| June | 4020 |
| July | 4067 |
| August | 4103 |
| September | 4163 |
| October | 4196 |
| November | 4262 |

| h1_growth_pct | h2_growth_pct |
|---------------|---------------|
| 44.71% | 37.35% |

**💡 Insight:**
- User growth shows a **steady upward trend** from January (3,934) to November (4,262) — a consistent month-on-month increase.
- **H1 contributed 44.71%** of total subscriptions vs **H2 at 37.35%**, indicating stronger acquisition in the first half of 2024.
- The platform is growing, but the pace slightly slowed in H2 — suggesting a need for targeted re-engagement or promotional campaigns in the latter half.

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

**📊 Output:**

| content_types | languages |
|---------------|-----------|
| Movie, Series, Sports | Bengali, English, Gujarati, Hindi, Kannada, Malayalam... |

**💡 Insight:**
- The platform offers **3 content types** — Movies, Series, and Sports — covering a wide audience.
- Content is available in **multiple regional languages** including Bengali, Gujarati, Kannada, and Malayalam, showing strong focus on **India's diverse linguistic market**.
- Multi-language content is a key strength for post-merger user retention across regions.

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

**📊 Output:**

| city_tier | user_count | age_18to24 | age_25to34 | age_35to44 | age_45plus | basic_users | free_users | premium_users |
|-----------|-----------|-----------|-----------|-----------|-----------|------------|-----------|--------------|
| Tier 1 | 25,451 | 4,393 | 11,444 | 6,408 | 3,206 | 0 | 5,111 | 10,178 |
| Tier 2 | 13,424 | 2,279 | 6,040 | 3,434 | 1,671 | 0 | 4,064 | 2,566 |
| Tier 3 | 5,745 | 1,004 | 2,585 | 1,432 | 724 | 0 | 2,921 | 623 |

**💡 Insight:**
- **Tier 1 cities dominate** with 25,451 users — nearly 2x Tier 2 and 4x Tier 3, showing urban-heavy adoption.
- The **25–34 age group is the largest segment** across all tiers — this is the primary target demographic.
- **Premium users are concentrated in Tier 1** (10,178) while Tier 3 has very few (623) — monetization opportunity exists in smaller cities.
- **No Basic plan users** across any tier — either the plan was discontinued or not applicable to this platform.
- **Free users are highest in Tier 3** (2,921 out of 5,745) — conversion to paid plans in smaller cities is a key growth lever post-merger.

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

**📊 Output:**

| total_users | active_pct | inactive_pct | age_18to24 | age_25to34 | age_35to44 | age_45plus | basic_users | free_users | premium_users |
|-------------|-----------|-------------|-----------|-----------|-----------|-----------|------------|-----------|--------------|
| 44,620 | 85.09% | 14.91% | 7,676 | 20,069 | 11,274 | 5,601 | 0 | 12,096 | 13,367 |

**💡 Insight:**
- **85% of users are active** — a strong retention signal and a positive indicator for merger feasibility.
- **14.91% inactive users** (≈6,640 users) represent a churn risk that needs targeted re-engagement strategies.
- **25–34 age group (20,069)** is the most engaged segment — content and marketing should be tailored to this group.
- **Premium users (13,367) outnumber Free users (12,096)** — showing healthy monetization and willingness to pay.

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

**📊 Output:**

| avg_watch_time |
|----------------|
| 7034.51 mins |

| device_type | city_tier | avg_watch_time |
|-------------|-----------|----------------|
| Laptop | Tier 1 | 5,553.39 |
| Mobile | Tier 1 | 11,571.50 |
| TV | Tier 1 | 6,526.07 |
| Laptop | Tier 2 | 4,234.78 |
| Mobile | Tier 2 | 9,711.13 |
| TV | Tier 2 | 4,955.10 |
| Laptop | Tier 3 | 3,227.85 |
| Mobile | Tier 3 | 8,090.74 |
| TV | Tier 3 | 3,646.02 |

**💡 Insight:**
- Overall average watch time is **7,034 mins per user** — indicating high engagement.
- **Mobile is the dominant device** across ALL city tiers — Tier 1 mobile users average 11,571 mins, nearly 2x laptop and TV.
- **Tier 1 users watch significantly more** than Tier 2 and Tier 3 across every device type — urban users are the most engaged.
- **Mobile-first strategy is critical** post-merger — optimizing the mobile experience will directly impact retention and watch time.
- Even in Tier 3, mobile watch time (8,090 mins) is strong — suggesting mobile is the primary access point for smaller cities.

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

**📊 Output:**

| user_status | total_users | total_watch_time | avg_watch_time | pct_of_users |
|-------------|-------------|-----------------|----------------|-------------|
| Active | 1,13,904 | 90,23,07,250 | 7921.65 mins | 85.09% |
| Inactive | 19,956 | 3,93,32,169 | 1970.94 mins | 14.91% |

**💡 Insight:**
- **Yes — there is a strong correlation between low watch time and inactivity.**
- Active users average **7,921 mins** of watch time vs only **1,970 mins** for inactive users — a **4x difference**.
- Inactive users (14.91%) contribute disproportionately less watch time despite being ~15% of the base.
- Post-merger, targeting users with **<2,000 mins watch time** with personalized content recommendations can prevent churn.

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

**📊 Output:**

| platform | total_downgrades | total_users | downgrade_pct |
|----------|-----------------|-------------|--------------|
| LioCinema | 12,628 | 1,83,446 | 6.88% |
| Jotstar | 5,195 | 44,620 | 11.64% |

**💡 Insight:**
- **Jotstar has a significantly higher downgrade rate (11.64%) vs LioCinema (6.88%)** — nearly double.
- This suggests users on Jotstar may find premium pricing too high relative to perceived value.
- Post-merger, a **unified mid-tier pricing strategy** (like LioCinema's Basic plan) could reduce downgrade pressure on Jotstar's user base.
- LioCinema's lower downgrade rate indicates stronger plan-to-value satisfaction among its subscribers.

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

**📊 Output:**

| platform | subscription_plan | new_subscription_plan | upgrade_count |
|----------|------------------|-----------------------|--------------|
| Jotstar | Premium | VIP | 368 |
| Jotstar | Free | Premium | 683 |
| LioCinema | Free | Premium | 715 |
| Jotstar | Free | VIP | 844 |
| LioCinema | Basic | Premium | 1,362 |
| LioCinema | Basic | Free | 10,309 |

**💡 Insight:**
- **LioCinema's Basic → Free (10,309)** is the most common transition — but this is actually a **downgrade**, not an upgrade, suggesting dissatisfaction with the Basic plan.
- **LioCinema Basic → Premium (1,362)** is the strongest genuine upgrade path, showing users who stay on paid plans tend to go all the way to Premium.
- On Jotstar, **Free → VIP (844)** is the top upgrade — users directly jump to mid-tier, skipping Free entirely.
- Post-merger, **Free-to-paid conversion campaigns** targeting free users could unlock significant revenue — both platforms show meaningful Free → paid upgrade potential.

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

**📊 Output:**

| platform | city_tier | free_pct | basic_pct | premium_pct | vip_pct |
|----------|-----------|----------|-----------|-------------|---------|
| Jotstar | Tier 1 | 20.08% | NULL | 39.99% | 39.93% |
| LioCinema | Tier 1 | 44.90% | 29.97% | 25.13% | NULL |
| Jotstar | Tier 2 | 30.27% | NULL | 19.12% | 50.61% |
| LioCinema | Tier 2 | 50.41% | 35.35% | 14.24% | NULL |
| Jotstar | Tier 3 | 50.84% | NULL | 10.84% | 38.32% |
| LioCinema | Tier 3 | 69.21% | 23.54% | 7.25% | NULL |

**💡 Insight:**
- **Jotstar monetizes much better than LioCinema** across all tiers — in Tier 1, only 20% are free vs LioCinema's 44.9%.
- **Jotstar's VIP plan is very popular** — 50.61% of Tier 2 users are on VIP, making it the dominant plan in that segment.
- **LioCinema has high free user concentration**, especially in Tier 3 (69.21%) — a major conversion opportunity post-merger.
- Post-merger, adopting Jotstar's **VIP-style mid-tier pricing** for LioCinema's large free user base could significantly boost combined revenue.

---

### Q10. Revenue Analysis (Jan–Nov 2024)
*Total revenue calculation based on plan pricing.*

**Pricing Assumptions:**

| Platform | Plan | Monthly Price (₹) |
|----------|------|-------------------|
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

**📊 Output:**

| platform | subscription_plan | total_users | active_months | total_revenue (₹) |
|----------|------------------|-------------|---------------|-------------------|
| Jotstar | Premium | 13,367 | 1,571.60 | 5,64,204.40 |
| Jotstar | VIP | 19,157 | 18,102.90 | 28,78,361.10 |

**💡 Insight:**
- **Jotstar's VIP plan is the biggest revenue driver** — ₹28,78,361 from 19,157 users across 18,102 active months.
- **Premium plan generates less revenue** despite decent user count (13,367) — mainly due to fewer active months (1,571 vs 18,102), suggesting Premium users churn faster.
- **VIP users stay longer and contribute more** — this validates VIP as a sweet spot between affordability and value.
- Post-merger, LioCinema should consider introducing a VIP-equivalent tier to replicate Jotstar's revenue success in Tier 2 and Tier 3 cities.

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

# 🎬 Strategic Merger in OTT — LioCinema & Jotstar Analysis

> **SQL-based data analysis project** exploring subscriber behavior, content performance, revenue trends, and merger feasibility between two major OTT platforms: **LioCinema** and **Jotstar**.

<div align="center">

![SQL](https://img.shields.io/badge/SQL-003B57?style=flat-square&logo=postgresql&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-00758F?style=flat-square&logo=mysql&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)
![Domain](https://img.shields.io/badge/Domain-OTT%20%7C%20Media-purple?style=flat-square)

</div>

---

## 📌 Project Overview

This project simulates a real-world **strategic merger analysis** between two Indian OTT platforms — **LioCinema** and **Jotstar** — using SQL queries on their transactional databases.

The goal is to evaluate whether a merger would be mutually beneficial by comparing:

| Area | What We Analyzed |
|------|-----------------|
| 👥 User Growth | Monthly trends, H1 vs H2 split |
| 📚 Content Library | Types, languages, genres |
| 🏙️ Demographics | Age group, city tier, subscription plan |
| ✅ Engagement | Active vs inactive users |
| 📱 Watch Behavior | Avg watch time by device & city |
| 💰 Monetization | Upgrades, downgrades, revenue |

**Period Covered:** January – November 2024

---

## 🗂️ Repository Structure

```
strategic-merger-in-ott/
│
├── meta_data.txt                        # Schema documentation for both databases
├── primary question answers/            # SQL queries with answers to 10 business questions
├── primary_and_secondary_questions.pdf  # Full list of analysis questions
└── problem_statement.pdf                # Business problem & merger context
```

---

## 🗄️ Database Schema

### LioCinema DB (`liocinema_db`)

<details>
<summary>Click to expand schema</summary>

#### `contents`
| Column | Type | Description |
|--------|------|-------------|
| `content_id` | VARCHAR | Unique identifier (e.g., CLCMHIROM1bdbc) |
| `content_type` | VARCHAR | Movie / Series / Sports |
| `language` | VARCHAR | Content language (Hindi, English, etc.) |
| `genre` | VARCHAR | Genre (Action, Drama, Romance, etc.) |
| `run_time` | INT | Duration in minutes |

#### `subscribers`
| Column | Type | Description |
|--------|------|-------------|
| `user_id` | VARCHAR | Unique subscriber ID |
| `age_group` | VARCHAR | 18-24 / 25-34 / 35-44 / 45+ |
| `city_tier` | VARCHAR | Tier 1 / Tier 2 / Tier 3 |
| `subscription_date` | DATE | Date of joining |
| `subscription_plan` | VARCHAR | Free / Basic / Premium |
| `last_active_date` | DATE | Last activity (NULL = currently active) |
| `plan_change_date` | DATE | Date of plan change |
| `new_subscription_plan` | VARCHAR | Updated plan after upgrade/downgrade |

#### `content_consumption`
| Column | Type | Description |
|--------|------|-------------|
| `user_id` | VARCHAR | Subscriber ID |
| `device_type` | VARCHAR | Mobile / TV / Laptop |
| `total_watch_time_mins` | INT | Total minutes watched |

</details>

### Jotstar DB (`jotstar_db`)
> Same structure as LioCinema DB with one key difference:
> **Subscription plans** → Free / **VIP** / Premium *(no "Basic"; VIP is the mid-tier)*

---

## 🧠 Business Questions, SQL Queries & Insights

---

### Q1. 📈 Total Users & Growth Trends
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

| Month | User Count |
|:------|----------:|
| January | 3,934 |
| February | 3,939 |
| March | 3,954 |
| April | 3,984 |
| May | 3,998 |
| June | 4,020 |
| July | 4,067 |
| August | 4,103 |
| September | 4,163 |
| October | 4,196 |
| November | 4,262 |

| H1 Growth % | H2 Growth % |
|:-----------:|:-----------:|
| **44.71%** | **37.35%** |

> 💡 **Key Insights**
> - User growth shows a **steady upward trend** from January (3,934) → November (4,262) — consistent month-on-month increase.
> - **H1 contributed 44.71%** of total subscriptions vs H2 at 37.35% — stronger acquisition in the first half.
> - Slightly slower H2 pace suggests need for **targeted re-engagement campaigns** in the latter half.

---

### Q2. 📚 Content Library Comparison
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

| Content Types | Languages Available |
|:-------------|:-------------------|
| Movie, Series, Sports | Bengali, English, Gujarati, Hindi, Kannada, Malayalam, Tamil, Telugu... |

> 💡 **Key Insights**
> - The platform offers **3 content types** — Movies, Series, and Sports — covering a wide audience.
> - Content spans **multiple regional languages**, showing strong focus on India's diverse linguistic market.
> - Multi-language content is a **key post-merger retention strength** across regions.

---

### Q3. 🏙️ User Demographics
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

| City Tier | Total Users | 18–24 | 25–34 | 35–44 | 45+ | Basic | Free | Premium |
|:----------|------------:|------:|------:|------:|----:|------:|-----:|--------:|
| Tier 1 | 25,451 | 4,393 | 11,444 | 6,408 | 3,206 | 0 | 5,111 | 10,178 |
| Tier 2 | 13,424 | 2,279 | 6,040 | 3,434 | 1,671 | 0 | 4,064 | 2,566 |
| Tier 3 | 5,745 | 1,004 | 2,585 | 1,432 | 724 | 0 | 2,921 | 623 |

> 💡 **Key Insights**
> - **Tier 1 cities dominate** with 25,451 users — nearly 2x Tier 2 and 4x Tier 3, showing urban-heavy adoption.
> - **25–34 age group is the largest segment** across all tiers — this is the primary target demographic.
> - **Premium users concentrated in Tier 1** (10,178) while Tier 3 has very few (623) — monetization opportunity in smaller cities.
> - **Free users highest in Tier 3** (2,921 out of 5,745) — Free-to-paid conversion is a key growth lever post-merger.

---

### Q4. ✅ Active vs. Inactive Users
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

| Total Users | Active % | Inactive % | 18–24 | 25–34 | 35–44 | 45+ | Basic | Free | Premium |
|------------:|---------:|-----------:|------:|------:|------:|----:|------:|-----:|--------:|
| 44,620 | **85.09%** | 14.91% | 7,676 | 20,069 | 11,274 | 5,601 | 0 | 12,096 | 13,367 |

> 💡 **Key Insights**
> - **85% of users are active** — strong retention signal and positive indicator for merger feasibility.
> - **14.91% inactive** (≈6,640 users) represent a churn risk needing targeted re-engagement strategies.
> - **25–34 age group (20,069)** is the most engaged — content and marketing should focus here.
> - **Premium users (13,367) outnumber Free users (12,096)** — healthy monetization and willingness to pay.

---

### Q5. 📱 Watch Time Analysis
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

| Overall Avg Watch Time |
|:---------------------:|
| **7,034.51 mins** |

| Device | Tier 1 | Tier 2 | Tier 3 |
|:-------|-------:|-------:|-------:|
| 💻 Laptop | 5,553.39 | 4,234.78 | 3,227.85 |
| 📱 Mobile | **11,571.50** | **9,711.13** | **8,090.74** |
| 📺 TV | 6,526.07 | 4,955.10 | 3,646.02 |

> 💡 **Key Insights**
> - Overall average watch time is **7,034 mins per user** — indicating high engagement across the platform.
> - 📱 **Mobile dominates across ALL tiers** — Tier 1 mobile users average 11,571 mins, nearly 2x laptop and TV.
> - **Tier 1 watches significantly more** than Tier 2/3 across every device — urban users are most engaged.
> - **Mobile-first strategy is critical** post-merger — optimizing mobile experience will directly impact retention.
> - Even in Tier 3, mobile watch time (8,090 mins) is strong — mobile is the primary access point in smaller cities.

---

### Q6. 🔗 Inactivity vs. Watch Time Correlation
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

| User Status | Total Users | Total Watch Time | Avg Watch Time | % of Users |
|:------------|------------:|-----------------:|---------------:|-----------:|
| ✅ Active | 1,13,904 | 90,23,07,250 | **7,921.65 mins** | 85.09% |
| ❌ Inactive | 19,956 | 3,93,32,169 | 1,970.94 mins | 14.91% |

> 💡 **Key Insights**
> - ✅ **Yes — strong correlation between low watch time and inactivity confirmed.**
> - Active users average **7,921 mins** vs only **1,970 mins** for inactive — a **4x difference**.
> - Inactive users contribute disproportionately less watch time despite being ~15% of the base.
> - Post-merger strategy: target users with **< 2,000 mins watch time** with personalized recommendations to prevent churn.

---

### Q7. 📉 Downgrade Trends
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

| Platform | Total Downgrades | Total Users | Downgrade % |
|:---------|----------------:|------------:|------------:|
| LioCinema | 12,628 | 1,83,446 | 6.88% |
| Jotstar | 5,195 | 44,620 | **11.64%** ⚠️ |

> 💡 **Key Insights**
> - ⚠️ **Jotstar's downgrade rate (11.64%) is nearly double LioCinema's (6.88%).**
> - Suggests Jotstar users may find premium pricing too high relative to perceived value.
> - Post-merger, a **unified mid-tier pricing strategy** (like LioCinema's Basic plan) could reduce downgrade pressure.
> - LioCinema's lower rate indicates stronger plan-to-value satisfaction among its subscribers.

---

### Q8. 📈 Upgrade Patterns
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

| Platform | From Plan | → To Plan | Count |
|:---------|:----------|:----------|------:|
| Jotstar | Premium | → VIP | 368 |
| Jotstar | Free | → Premium | 683 |
| LioCinema | Free | → Premium | 715 |
| Jotstar | Free | → VIP | 844 |
| LioCinema | Basic | → Premium | 1,362 |
| LioCinema | Basic | → Free | **10,309** ⚠️ |

> 💡 **Key Insights**
> - ⚠️ **LioCinema Basic → Free (10,309)** is the most common transition — actually a **downgrade**, suggesting Basic plan dissatisfaction.
> - **LioCinema Basic → Premium (1,362)** is the strongest genuine upgrade — users who stay paid tend to go all the way.
> - On Jotstar, **Free → VIP (844)** is the top upgrade — users skip directly to mid-tier.
> - **Free-to-paid conversion campaigns** on both platforms could unlock significant revenue post-merger.

---

### Q9. 💳 Paid User Distribution by City Tier
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

| Platform | City Tier | Free % | Basic % | Premium % | VIP % |
|:---------|:----------|-------:|--------:|----------:|------:|
| Jotstar | Tier 1 | 20.08% | — | 39.99% | 39.93% |
| LioCinema | Tier 1 | 44.90% | 29.97% | 25.13% | — |
| Jotstar | Tier 2 | 30.27% | — | 19.12% | **50.61%** |
| LioCinema | Tier 2 | 50.41% | 35.35% | 14.24% | — |
| Jotstar | Tier 3 | 50.84% | — | 10.84% | 38.32% |
| LioCinema | Tier 3 | **69.21%** | 23.54% | 7.25% | — |

> 💡 **Key Insights**
> - 🏆 **Jotstar monetizes far better** — in Tier 1, only 20% are free vs LioCinema's 44.9%.
> - **Jotstar's VIP plan is dominant in Tier 2** (50.61%) — mid-tier pricing is clearly working.
> - ⚠️ **LioCinema Tier 3 has 69.21% free users** — massive conversion opportunity post-merger.
> - Adopting a **VIP-style mid-tier plan** for LioCinema's free base could significantly boost combined revenue.

---

### Q10. 💰 Revenue Analysis (Jan–Nov 2024)
*Total revenue calculation based on plan pricing.*

**Pricing Assumptions:**

| Platform | Plan | Monthly Price |
|:---------|:-----|:-------------:|
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

| Platform | Plan | Total Users | Active Months | Total Revenue (₹) |
|:---------|:-----|------------:|-------------:|------------------:|
| Jotstar | Premium | 13,367 | 1,571.60 | **₹5,64,204** |
| Jotstar | VIP | 19,157 | 18,102.90 | **₹28,78,361** 🏆 |

> 💡 **Key Insights**
> - 🏆 **Jotstar VIP is the biggest revenue driver** — ₹28,78,361 from 19,157 users over 18,102 active months.
> - **Premium generates less** despite decent users (13,367) — only 1,571 active months suggests faster churn.
> - **VIP users stay longer and pay more** — validating VIP as the sweet spot between affordability and value.
> - Post-merger, LioCinema should introduce a **VIP-equivalent tier** to replicate Jotstar's revenue success in Tier 2/3 cities.

---

## 🔍 Key Analysis Summary

| # | Dimension | Key Finding | Business Impact |
|---|-----------|-------------|----------------|
| 1 | 📈 User Growth | Steady rise Jan→Nov; H1 stronger at 44.71% | Plan H2 campaigns to maintain momentum |
| 2 | 📚 Content Library | 3 types, 8+ languages | Strong multi-region appeal post-merger |
| 3 | 🏙️ Demographics | Tier 1 dominant; 25–34 age is core segment | Focus marketing on urban young adults |
| 4 | ✅ Retention | 85.09% active users | Healthy base; address 14.91% churn |
| 5 | 📱 Watch Time | Mobile leads across all tiers (up to 11,571 mins) | Prioritize mobile-first product experience |
| 6 | 🔗 Engagement | Active users watch 4x more than inactive | Early churn detection via watch time < 2,000 mins |
| 7 | 📉 Downgrades | Jotstar 11.64% vs LioCinema 6.88% | Introduce mid-tier plan to reduce Jotstar churn |
| 8 | 📈 Upgrades | LioCinema Basic→Free most common (10,309) | Revamp or discontinue Basic plan |
| 9 | 💳 Paid Users | LioCinema Tier 3: 69.21% free | Huge Free→paid conversion opportunity |
| 10 | 💰 Revenue | Jotstar VIP: ₹28.78L vs Premium: ₹5.64L | VIP-tier pricing is the winning model |

---

## 🛠️ Tools & Technologies

| Tool | Usage |
|------|-------|
| ![MySQL](https://img.shields.io/badge/MySQL-00758F?style=flat-square&logo=mysql&logoColor=white) | All queries written and tested |
| **JOINs** | Combining subscriber + consumption data |
| **CASE-WHEN** | Conditional aggregations |
| **Window Functions** | `OVER()` for percentage calculations |
| **UNION ALL** | Cross-platform comparisons |
| **DATEDIFF** | Revenue calculation over active months |
| **COALESCE / GROUP_CONCAT** | Handling NULLs and list aggregation |

---

## 👩‍💻 Author

**Madhuri Padole**
Data Analyst | SQL Enthusiast
[![GitHub](https://img.shields.io/badge/GitHub-madhuri808-181717?style=flat-square&logo=github)](https://github.com/madhuri808)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/madhuri-padole-93b875259)

---

## 📄 License

This project is for educational and portfolio purposes only.

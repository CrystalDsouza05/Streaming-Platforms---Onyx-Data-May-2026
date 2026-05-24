# 🎵 Onyx Data Challenge — May 2026
## Music Streaming Analytics Dashboard (Power BI)

> **Status: Work in Progress** — Data model, relationships, and full DAX measures library are complete. Two dashboard pages partially built.


## 📌 About This Project

This is my submission attempt for the [Onyx Data Challenge - May 2026](https://datadna.onyxdata.co.uk/challenges/may-2026-datadna-engagement-subscription-churn-content-performance-analytics-challenge/), where participants build an analytical dashboard for a fictional music streaming platform covering 4 years of operational data (2021–2024) across 10 markets.

Due to time constraints I wasn't able to finish — but the hardest parts are already done. If you're participating in the same challenge, feel free to fork this repo and build on top of what's here.


## 🗂️ Data Model

The dataset follows a **star schema** with 2 fact tables, 9 dimension tables, and 1 bridge table.

### Tables loaded
- `fact_listening_session` (~224,000 rows) — one row per listening session
- `fact_subscription_event` (~3,600 rows) — subscription lifecycle events
- `dim_user`, `dim_track`, `dim_artist`, `dim_genre`, `dim_device`
- `dim_playlist`, `dim_subscription_plan`, `dim_country`, `dim_date`
- `bridge_playlist_track` — many-to-many link between playlists and tracks

### Relationships (24 total)

All relationships are Many-to-One, single direction unless noted.

| From | To | Status |
|---|---|---|
| `fact_listening_session[user_id]` | `dim_user[user_id]` | ✅ Active |
| `fact_listening_session[track_id]` | `dim_track[track_id]` | ✅ Active |
| `fact_listening_session[artist_id]` | `dim_artist[artist_id]` | ✅ Active |
| `fact_listening_session[genre_id]` | `dim_genre[genre_id]` | ✅ Active |
| `fact_listening_session[playlist_id]` | `dim_playlist[playlist_id]` | ✅ Active |
| `fact_listening_session[device_id]` | `dim_device[device_id]` | ✅ Active |
| `fact_listening_session[country_code]` | `dim_country[country_code]` | ✅ Active |
| `fact_listening_session[listen_start_ts]` | `dim_date[date_key]` | ✅ Active |
| `fact_subscription_event[user_id]` | `dim_user[user_id]` | ✅ Active |
| `fact_subscription_event[country_code]` | `dim_country[country_code]` | ✅ Active |
| `fact_subscription_event[event_ts]` | `dim_date[date_key]` | ✅ Active |
| `dim_user[plan_id]` | `dim_subscription_plan[plan_id]` | ✅ Active |
| `dim_user[country_code]` | `dim_country[country_code]` | ⚠️ Inactive — use `USERELATIONSHIP()` in DAX when filtering by user home country |
| `dim_track[artist_id]` | `dim_artist[artist_id]` | ⚠️ Inactive — fact connects directly |
| `dim_track[genre_id]` | `dim_genre[genre_id]` | ⚠️ Inactive — fact connects directly |
| `dim_artist[genre_id]` | `dim_genre[genre_id]` | ⚠️ Inactive — fact connects directly |
| `bridge_playlist_track[playlist_id]` | `dim_playlist[playlist_id]` | ✅ Active |
| `bridge_playlist_track[track_id]` | `dim_track[track_id]` | ✅ Active |

> **Note on inactive relationships:** `dim_user[country_code] → dim_country` is set to inactive to resolve an ambiguous path conflict with `fact_listening_session[country_code] → dim_country`. The session-level country is the active path. Use `USERELATIONSHIP()` when filtering by a user's home country.



## 📐 DAX Measures Library

All 31 measures are organised into folders inside a dedicated `_Measures` table.

<details>
<summary><strong>Revenue & Volume (5 measures)</strong></summary>

| Measure | Description |
|---|---|
| `Total Revenue` | SUM of estimated_revenue_usd |
| `Total Sessions` | COUNTROWS of fact_listening_session |
| `Active Users` | DISTINCTCOUNT of user_id |
| `ARPU` | Total Revenue ÷ Active Users |
| `Revenue per Session` | Total Revenue ÷ Total Sessions |

</details>

<details>
<summary><strong>Subscription Health (8 measures)</strong></summary>

| Measure | Description |
|---|---|
| `Net MRR Change` | SUM of mrr_change_usd |
| `MRR Expansion` | MRR change where delta > 0 (upgrades, signups) |
| `MRR Contraction` | MRR change where delta < 0 (churns, downgrades) |
| `Churn Events` | Count of events where event_type = "churn" |
| `Upgrade Events` | Count of events where event_type = "upgrade" |
| `Churn Rate %` | Churn Events ÷ Distinct users with subscription events |
| `Upgrade to Churn Ratio` | Upgrade Events ÷ Churn Events |
| `MRR at Risk` | MRR before event for all churn events |

</details>

<details>
<summary><strong>Engagement & Content (7 measures)</strong></summary>

| Measure | Description |
|---|---|
| `Skip Rate` | Skipped sessions ÷ Total Sessions |
| `Avg Session Duration (mins)` | Average listen_seconds ÷ 60 |
| `New Artist Discovery Rate` | New artist sessions ÷ Total Sessions |
| `Sessions per User` | Total Sessions ÷ Active Users |
| `Algo Track Sessions` | Sessions on algorithmic recommendation tracks |
| `Algo Skip Rate` | Skip Rate filtered to algorithmic tracks |
| `Non-Algo Skip Rate` | Skip Rate filtered to non-algorithmic tracks |

</details>

<details>
<summary><strong>Fraud Detection (5 measures)</strong></summary>

| Measure | Description |
|---|---|
| `Fraud Sessions` | Sessions where is_fraud_cluster = TRUE |
| `Fraud Session Share %` | Fraud Sessions ÷ Total Sessions |
| `Fraud Revenue` | Revenue from fraud cluster users |
| `Fraud Revenue Share %` | Fraud Revenue ÷ Total Revenue |
| `Fraud Churn Rate %` | Churn Rate % filtered to fraud cluster users |

</details>

<details>
<summary><strong>Time Intelligence (6 measures)</strong></summary>

> Requires `dim_date` to be marked as a Date Table in Power BI (right-click → Mark as date table → select `full_date`).

| Measure | Description |
|---|---|
| `Sessions LY` | Total Sessions same period last year |
| `Sessions YoY %` | Year-over-year session growth |
| `Revenue LY` | Total Revenue same period last year |
| `Revenue YoY %` | Year-over-year revenue growth |
| `Sessions MTD` | Month-to-date sessions |
| `Net MRR Change LY` | Net MRR Change same period last year |

</details>


## 📊 Dashboard Pages

### Page 1 — Business Health Overview

**Slicers:** Year + Month · Genre · Device

**Visuals built:**
- 4 KPI cards — Total Revenue · Active Users · Churn Rate % · Avg Listen Duration (mins)
- Line chart — Total sessions by year, month and subscription tier
- Waterfall chart — Net MRR change by event type (upgrade, signup, retention, churn, downgrade)
- Donut chart — Total sessions by subscription tier
- Column chart — Total sessions by country
- Clustered bar — Upgrade Events vs Churn Events by country
- Field parameter toggle — switches between Total Sessions / Total Revenue / Active Users on the donut


### Page 2 — Engagement & Content Insights

**Slicers:** Year + Month · Genre · Device

**Visuals built:**
- 4 KPI cards — Sessions per User · Skip Rate · New Artist Discovery Rate · Fraud Session Share %
- Scatter plot — User engagement profile: Sessions per User vs Skip Rate, sized by Total Revenue, split by fraud cluster flag
- Table — Top 20 artists by sessions with Total Revenue and Active Users columns
- Bar chart — Skip Rate by genre
- Clustered bar — Non-Algo Skip Rate vs Algo Skip Rate by genre
- Field parameter toggle — switches between device_type and genre_name views


## 🤝 Open for Collaboration

I started this for the **Onyx Data Challenge May 2026** but ran out of time. The data model and full DAX measures library are complete — the heavy lifting is done.

If you're working on the same challenge, feel free to fork this repo and build on top of what's here. No credit needed — good luck with your submission!


*Built with Power BI Desktop. Dataset provided by Onyx Data.*

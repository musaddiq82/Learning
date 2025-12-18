Markdown

# Blingram Recommendation System Design (v1.0)

## 1. Overview
This document outlines the architecture for the **Blingram** recommendation engine (Reels & Posts). Unlike standard social apps, Blingram prioritizes **Monetization Velocity**. The algorithm ranks content by the revenue (Gifts & Tips) and engagement it generates relative to its age.

## 2. The Core Logic (Heuristic Model)
Since we are starting without historical user data, we use a **Weighted Scoring Formula** with **Time Decay** (Gravity). This ensures:
1.  **Viral content rises:** High engagement boosts the score.
2.  **Money talks:** Gifts and Tips provide a massive boost (x50 weight).
3.  **Freshness:** Old content naturally falls down the ranking over time.

### The Ranking Formula
$$
\text{Score} = \frac{(W_l \cdot L) + (W_c \cdot C) + (W_s \cdot S) + (W_g \cdot \text{Gift}) + (W_t \cdot \text{Tip}) + 1}{(\text{AgeHours} + 2)^{G}}
$$

| Parameter | Metric | Weight ($W$) | Reason |
| :--- | :--- | :--- | :--- |
| **L** | Likes | **1.0** | Baseline engagement signal. |
| **C** | Comments | **3.0** | Indicates higher effort conversation. |
| **S** | Shares | **5.0** | High viral intent. |
| **Gift** | Total Gift Amount ($) | **50.0** | **Primary Business Goal.** Revenue driver. |
| **Tip** | Total Tips Received ($) | **50.0** | **Primary Business Goal.** Direct creator support. |
| **G** | Gravity | **1.8** | Determines how fast content "dies." |

---

## 3. Database Schema Updates

### A. Posts Table (Modifications)
Update the existing table to support the sorting algorithm.

```sql
ALTER TABLE posts ADD COLUMN (
    -- Ensure these exist based on your current schema
    total_gift_amount DECIMAL(10, 2) DEFAULT 0.00,
    total_tips_received DECIMAL(10, 2) DEFAULT 0.00,
    
    -- NEW: The Magic Column for Sorting
    trending_score FLOAT DEFAULT 0.0, 
    
    -- Indexing for speed (Critical for Feed Performance)
    INDEX idx_trending (trending_score DESC),
    INDEX idx_created (created_at)
);
B. Interaction Log (Critical for Future AI)
To train a Neural Network later (Phase 2), we must log who did what. Create this table immediately.

SQL

CREATE TABLE user_post_interactions (
    id UUID PRIMARY KEY,
    user_id INT NOT NULL,
    post_id INT NOT NULL,
    action_type ENUM('like', 'comment', 'share', 'gift', 'tip', 'view'),
    value_amount DECIMAL(10, 2) DEFAULT 0.00, -- The dollar amount for gifts/tips
    time_spent_seconds INT DEFAULT 0,         -- How long they watched/lingered
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
4. Implementation Code
A. Python Backend (The "Reflex" Update)
Call this function immediately whenever a user performs an action (Like, Gift, Tip).

Python

import datetime

def calculate_trending_score(post):
    """
    Inputs: 'post' dictionary containing metrics + created_at timestamp
    Returns: Float (The new score)
    """
    # 1. Configuration (Weights)
    W_LIKE = 1.0
    W_COMMENT = 3.0
    W_SHARE = 5.0
    W_MONEY = 50.0 # Weight for both Gifts and Tips (1 dollar = 50 points)
    GRAVITY = 1.8 

    # 2. Extract Data (Matching your DB fields)
    likes = post.get('like_count', 0)
    comments = post.get('comment_count', 0)
    shares = post.get('share_count', 0)
    
    # Financials (Sum of Gifts + Tips)
    revenue = float(post.get('total_gift_amount', 0)) + \
              float(post.get('total_tips_received', 0))

    # 3. Calculate Numerator (Engagement + Money)
    score_raw = (likes * W_LIKE) + \
                (comments * W_COMMENT) + \
                (shares * W_SHARE) + \
                (revenue * W_MONEY)

    # 4. Calculate Denominator (Time Decay)
    # Convert age to hours. +2 prevents division by zero.
    age_hours = (datetime.datetime.now() - post['created_at']).total_seconds() / 3600.0
    time_decay = (age_hours + 2) ** GRAVITY

    # 5. Final Score
    return (score_raw + 1) / time_decay
B. SQL "Heartbeat" (Background Decay)
Run this query every 10-15 minutes (via Cron/Celery) to naturally lower the scores of older posts.

SQL

UPDATE posts
SET trending_score = (
    (like_count * 1.0) + 
    (comment_count * 3.0) + 
    (share_count * 5.0) + 
    ((total_gift_amount + total_tips_received) * 50.0)
) / 
POWER(
    (EXTRACT(EPOCH FROM (NOW() - created_at)) / 3600) + 2, 
    1.8
)
-- Optimization: Only update posts from the last 7 days.
-- Older posts are mathematically guaranteed to have a near-zero score.
WHERE created_at > NOW() - INTERVAL '7 days';
5. The "Feed Mixer" Strategy
When a user requests their feed, do not use a single query. Mix the content to ensure variety and help new users (Cold Start).

The Ratio (6-3-1 Rule):

60% Trending (Viral): SELECT * FROM posts ORDER BY trending_score DESC LIMIT 6

30% Following (Loyalty): SELECT * FROM posts WHERE user_id IN (followed_ids) ORDER BY created_at DESC LIMIT 3

10% Rising (New Creators): SELECT * FROM posts WHERE created_at > NOW() - INTERVAL '1 hour' ORDER BY RANDOM() LIMIT 1

6. Action Items Checklist
[ ] Run ALTER TABLE to add trending_score.

[ ] Run CREATE TABLE for user_post_interactions.

[ ] Implement the Python calculation in the /tip and /like API endpoints.

[ ] Set up the SQL Cron Job (Every 15 mins).

[ ] Data Rule: Enforce mandatory "Category" tags for all new uploads.

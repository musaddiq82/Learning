# Product Requirement Document (PRD)
## Blingram – Core Logic v2.0

---

## Objective

Transform a standard social feed into a **high-retention Wealth Ecosystem**.

**Core Loop:**

> Watch Content → Get Inspired → Create Value → Earn Money

The product should make wealth creation feel **visible, competitive, and achievable**.

---

## 1. Module: Home Page — *The Stream*

### Purpose
Passive consumption.  
Maximize session time and daily returns by blending **comfort** with **FOMO**.

---

### 1.1 The “Zipper” Feed Algorithm

We do **not** use static content blocks.  
Instead, we use an **interleaved injection model** to preserve flow and avoid cognitive breaks.

| Priority Slot | Type            | Weight | Data Source                                | Purpose |
|--------------|------------------|--------|--------------------------------------------|---------|
| 1            | Core Interest    | 30%    | User Vectors (Cosine Similarity > 0.8)     | Retention. Keeps the user comfortable. |
| 2            | Market Heat      | 30%    | Global Trending (High Velocity Signals)    | FOMO. Shows what is viral *now*. |
| 3            | Social Circle    | 20%    | Following Graph (Quality-Filtered)        | Community. Anchors user to people they know. |
| 4            | Exploration      | 20%    | Adjacent Interest Clusters                 | Discovery. Prevents boredom and stagnation. |

#### Notes
- Social Circle content is injected **only if engagement score exceeds baseline**.
- Market Heat prioritizes **velocity**, not total views.
- Exploration content must be *adjacent*, never random.

---

## 2. Module: Trending Page — *The Market*

### Purpose
Active research mode.  
This page answers one question:

> **“Who is winning right now?”**

Creators are treated like **assets**, not influencers.

---

### 2.1 Section A: “The Rich List” (Top Earners)

#### Data Source
`user_daily_stats` table

#### Ranking Logic
- Ranked **strictly by Total Revenue Generated**
- Revenue = Tips + Gifts
- Time Window = **Rolling 24 hours**

#### Ranking Tiers

| Rank Range | Metric Used                     | Psychological Trigger |
|-----------|----------------------------------|-----------------------|
| #1 – #3   | Highest Total Revenue            | Aspiration / Jealousy |
| #4 – #10  | Revenue > Dynamic Threshold      | Competition |
| #11 – #50 | Revenue Growth Rate (Velocity)   | Hope / Accessibility |

#### Display Rules
- Show **exact earnings** for #1–#3.
- Show **ranges** for #4–#10 to reduce intimidation.
- Highlight **growth arrows** for fast climbers.

---

### 2.2 Section B: “Hot Creators” (Momentum Board)

#### Logic
Ranks creators by **Revenue Velocity**, not absolute earnings.


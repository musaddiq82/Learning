# Two-Tower Model Data Logging Protocol

## 🚨 Critical Importance
This document outlines the **data logging requirements** for the recommendation engine.  
If these parameters are missed, the **Two-Tower model will fail to learn**.

We are logging data into **three buckets**:
- **The User**
- **The Item**
- **The Context**

These logs form the **“Memory” of the AI**.

---

## 1. The Input Features (The “Question”)

These features are fed into the Neural Networks to generate embeddings.

---

### A. The User Tower (Who are they?)

**Goal:** Represent user taste and spending power.

- **User ID**  
  Unique identifier.

- **Interaction History (Sequence)**  
  List of the last *N* Post IDs interacted with (e.g., `[101, 88, 99]`).  
  Crucial for determining current *user mood*.

- **Monetization Level (“Whale” Status)**  
  Total lifetime spend ($).  
  Used to recommend expensive / giftable content to high spenders.

- **Demographics (Optional)**  
  Age, Gender, Country.

---

### B. The Item Tower (What is it?)

**Goal:** Represent content semantics *without manual tagging*.

- **Item ID**  
  Unique identifier.

- **CLIP Vector**  
  512-dimensional embedding generated during ingestion.

- **Monetization Stats**  
  Total gifts received ($).  
  Teaches the model which posts convert revenue.

- **Popularity Stats**  
  Total likes, total views (normalized by post age).

- **Media Type**  
  Photo, Reel, or Live Stream.

---

### C. The Context (Where / When?)

**Goal:** Differentiate situations (e.g., *Morning Coffee* vs *Late Night Gaming*).

- **Timestamp**  
  Hour of day (0–23) & Day of week (0–6).

- **Device Type**  
  iOS vs Android (correlates with spending power).

- **Connection Type**  
  WiFi vs 4G (affects video quality tolerance).

---

## 2. The Labels (The “Answer”)

These are the **ground truths** we train against.  
We **must log the outcome of every view**.

- **Target 1 — Click / View**  
  Did they stop scrolling? (Binary: 0 / 1)

- **Target 2 — Watch Time**  
  Continuous value in seconds.  
  Did they watch or skip instantly?

- **Target 3 — Engagement**  
  Did they Like or Comment? (Binary)

- **Target 4 — Monetization (Gold Label)**  
  Did they Gift? (Dollar value)

---

## 3. Architecture Visualization (Conceptual)

- **Left:** User Tower  
  (History + Demographics)

- **Right:** Item Tower  
  (CLIP Vector + Post Stats)

- **Top:**  
  Dot Product / Concatenation → Predict likelihood to **Watch / Like / Gift**

---

## 4. Database Schema (The Log Table)

Execute this SQL to create the logging infrastructure:

```sql
CREATE TABLE interaction_logs (
    event_id UUID PRIMARY KEY,
    
    -- 1. WHO (User Tower Data)
    user_id INT,
    device_type VARCHAR(20), -- 'ios', 'android'
    
    -- 2. WHAT (Item Tower Data)
    post_id INT,
    -- (Note: 512-dim CLIP Vector is joined from the 'posts' table during training)
    
    -- 3. CONTEXT (Environment)
    interaction_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    hour_of_day SMALLINT, -- Derived from timestamp
    
    -- 4. THE LABELS (The "Answers")
    watch_time_ms INT DEFAULT 0,           -- How long did they watch?
    is_liked BOOLEAN DEFAULT FALSE,        -- Did they like?
    gift_value DECIMAL(10,2) DEFAULT 0.00, -- Did they spend money?
    
    -- Negative Sampling Helper
    impression_id UUID -- Groups a feed batch/session
);

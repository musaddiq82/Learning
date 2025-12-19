Visual Content-Based Recommendation System

Using CLIP (ViT-B-32) & PostgreSQL

1. Overview

This system implements Visual Content-Based Filtering, not user prediction.

Goal:
“If a user views this post, show visually similar posts.”

We use Transfer Learning with OpenAI’s CLIP (ViT-B-32) model to convert images into numerical vectors and perform similarity search directly inside PostgreSQL.

This approach works even when tags, metadata, or user history are missing.

2. Strategy
What We Are Building

A content-based recommendation engine

Based purely on visual similarity

Independent of:

User profiles

Tags

Likes

Gifts

Technique Used

Transfer Learning

Pretrained model: CLIP (ViT-B-32)

No model training required

3. Architecture (High Level)
Image Upload
     ↓
CLIP Encoder (ViT-B-32)
     ↓
512-D Vector (Visual Fingerprint)
     ↓
PostgreSQL (pgvector)
     ↓
Similarity Search (<=>)
     ↓
Recommended Posts

4. Prerequisites
Python

Python 3.8+

Required Packages
pip install sentence-transformers psycopg2-binary pillow

5. Step-by-Step Implementation
Step 1: Database Setup (PostgreSQL)
Option A: Using pgvector (Recommended – Fastest)

Requires permission to install extensions.

-- Enable vector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Add vector column to posts table
ALTER TABLE posts
ADD COLUMN content_vector vector(512);


✅ Best for:

Fast similarity search

Production-grade systems

Option B: Fallback (If Extensions Are Not Allowed)
ALTER TABLE posts
ADD COLUMN content_vector JSONB;


⚠️ Slower, but functional.

Step 2: Image Ingestion Script (Python)

This script runs whenever a user uploads an image
(or once for backfilling existing posts).

📁 File: process_images.py

from sentence_transformers import SentenceTransformer
from PIL import Image
import psycopg2

# Load CLIP (Transfer Learning model)
model = SentenceTransformer('clip-ViT-B-32')

def process_new_post(post_id, image_path):
    # Load and normalize image
    img = Image.open(image_path).convert("RGB")

    # Convert image → 512D vector
    vector = model.encode(img).tolist()

    # Database connection (example)
    conn = psycopg2.connect(
        dbname="your_db",
        user="your_user",
        password="your_password",
        host="localhost",
        port=5432
    )
    cursor = conn.cursor()

    # Store vector in DB
    cursor.execute(
        "UPDATE posts SET content_vector = %s WHERE id = %s",
        (vector, post_id)
    )

    conn.commit()
    cursor.close()
    conn.close()

What This Achieves

Uses pretrained visual knowledge

Converts raw pixels → semantic understanding

Satisfies Transfer Learning requirement

Step 3: Recommendation Query

When a user views a post, retrieve visually similar posts.

SELECT *
FROM posts
WHERE content_vector IS NOT NULL
ORDER BY content_vector <=> (
    SELECT content_vector FROM posts WHERE id = [CURRENT_POST_ID]
)
LIMIT 5;

Operator Explanation

<=> → cosine / L2 similarity (pgvector)

Lower distance = more similar

6. What This Enables (Immediately)

✅ Image similarity recommendations

✅ Cold-start problem solved

✅ No dependency on tags or metadata

✅ Scales automatically with new uploads

✅ Zero model training cost

7. What to Tell the Manager (Status Update)

Status Update – Base Model Finalized

I have finalized the plan for the Base Model using Transfer Learning.

We will use CLIP (ViT-B-32), a pretrained multi-modal model that transfers visual understanding directly to our dataset without requiring manual training.

This approach addresses current data gaps such as missing tags by generating high-dimensional vectors directly from image pixels.

As a result, we can deploy a Day-1 Content-Based Recommendation Engine that suggests relevant posts based purely on visual similarity, independent of user history or gifting data.

8. Deployment Checklist

 Install Python dependencies

 Run PostgreSQL ALTER TABLE

 Verify pgvector availability

 Run ingestion script for existing images

 Trigger ingestion on new uploads

 Enable similarity query in feed logic

# 🩺 AI Physician Notetaker & Analyzer

## 📖 Project Overview
This project implements an NLP pipeline designed to automate medical documentation. It processes raw physician-patient transcripts to extract clinical entities, analyze patient sentiment, and generate structured SOAP notes.

**Core Capabilities:**
1.  **Medical Summarization (NER):** Extracts Symptoms, Diagnoses, and Treatments using a **Hybrid Ensemble** of `BioBERT` and `DistilBERT`.
2.  **Sentiment & Intent Analysis:** Detects patient anxiety and reassurance needs using an **Emotion-Adapter** logic layer.
3.  **Automated SOAP Generation:** Converts unstructured dialogue into a standard Clinical SOAP Note format.

---

## 🛠️ Technical Architecture

### 1. Hybrid NER System (Task 1)
Instead of relying on a single model, this project uses a **Generalist-Specialist Ensemble**:
* **The Generalist:** `d4data/biomedical-ner-all` (DistilBERT) detects broad entities like Symptoms, Medications, and Procedures.
* **The Specialist:** `ugaray96/biobert_ncbi_disease_ner` (BioBERT) provides high-precision detection for Diseases/Disorders.
* **Normalization Layer:** A custom Python layer handles negation detection (e.g., "No pain"), deduplication, and synonym mapping (e.g., "ache" -> "Pain").

### 2. Sentiment Adapter (Task 2)
Medical sentiment differs from general sentiment (e.g., "No pain" is Positive).
* **Model:** `j-hartmann/emotion-english-distilroberta-base`.
* **Logic Adapter:** Maps raw emotions to clinical intents:
    * *Fear/Sadness* → `Anxious` (Seeking Reassurance)
    * *Joy/Relief* → `Reassured` (Expressing Relief)
    * *Neutral* → `Neutral` (Reporting Symptoms)

### 3. SOAP Generator (Task 3)
Uses heuristic segmentation to separate the **Subjective** (Patient Narrative) from the **Objective** (Physical Exam) and injects structured NER data into the **Assessment** and **Plan**.

---

## 🚀 Setup & Usage

### Prerequisites
* Python 3.8+
* GPU recommended (but runs on CPU)

### Installation
```bash
pip install torch transformers scipy numpy

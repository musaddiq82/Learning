# 🩺 AI Physician Notetaker & Analyzer

![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-orange)
![Hugging Face](https://img.shields.io/badge/Transformers-4.0%2B-yellow)
![License](https://img.shields.io/badge/License-MIT-green)

## 📖 Project Overview
This project implements an end-to-end **Medical NLP Pipeline** designed to automate clinical documentation. It processes raw doctor-patient dialogues to extract structured clinical entities, analyze patient emotional states, and generate professional **SOAP (Subjective, Objective, Assessment, Plan)** notes.

**Core Capabilities:**
1.  **Hybrid Medical NER:** Combines a Generalist model (Symptoms/Meds) with a Specialist model (Diseases) for high-precision entity extraction.
2.  **Context-Aware Sentiment Analysis:** Adapts general emotion detection to the medical domain (e.g., distinguishing "Fear" from "Medical Anxiety").
3.  **Automated SOAP Generation:** Uses heuristic segmentation and logic injection to construct clinical notes without hallucination risks.

---

## 🛠️ Technical Architecture

### 1. The Hybrid NER Engine (Task 1)
Instead of relying on a single model, we implemented an **Ensemble Pipeline**:
* **The Generalist:** `d4data/biomedical-ner-all` (DistilBERT). Used for broad entity detection (Symptoms, Medications, Procedures).
* **The Specialist:** `ugaray96/biobert_ncbi_disease_ner` (BioBERT). Fine-tuned on the NCBI Disease Corpus for high-precision diagnosis extraction.
* **Sanitization Layer:** A custom Python layer that handles:
    * **Negation Detection:** Filters out "No pain" using a 50-character context window.
    * **Fragment Reassembly:** Fixes tokenizer artifacts (e.g., `##ache` → `Headache`).
    * **Synonym Normalization:** Maps "hurt", "ache", and "discomfort" to the clinical term **"Pain"**.

### 2. Medical Sentiment Adapter (Task 2)
Standard sentiment analysis fails in medicine (e.g., "Positive test result" is often negative news).
* **Base Model:** `j-hartmann/emotion-english-distilroberta-base`.
* **Adapter Logic:** We map raw Ekman emotions to Clinical Intents:
    * `Fear` / `Sadness` $\rightarrow$ **Anxious** (Seeking Reassurance).
    * `Joy` / `Surprise` $\rightarrow$ **Reassured** (Expressing Relief).
    * `Neutral` $\rightarrow$ **Neutral** (Reporting Symptoms).

### 3. SOAP Generator (Task 3)
We avoid using Generative LLMs (like GPT) for the structure to prevent hallucinations. Instead, we use **Extraction-Based Construction**:
* **Subjective:** Constructs a "History of Present Illness" summary using Regex-based event and duration extraction.
* **Objective:** Segments transcript using the `[Physical Examination]` anchor token.
* **Assessment & Plan:** Injects validated NER data directly into the final JSON structure.

---

## 🚀 Setup & Usage

### Prerequisites
* Python 3.8+
* GPU recommended (automatically detected in code)

### Installation
```bash
pip install torch transformers scipy numpy


# 📄 Interview Question Answers – Technical Summary

This document contains complete, technically accurate answers for the interview assignment involving **medical NLP**, **sentiment analysis**, and **SOAP note generation**.  
It is structured for clarity so reviewers can quickly understand your engineering reasoning.

---

# 🧠 Part 1: Medical NLP Summarization

## **1. How would you handle ambiguous or missing medical data in a transcript?**

### 🔹 Ambiguous Data (e.g., "I feel funny")
I use a **confidence-based ambiguity handling pipeline**:

- **NER Confidence Thresholding**  
  If the model’s confidence is **< 60%**, the extracted term is flagged instead of auto-added.

- **Context Window Analysis**  
  I inspect ±5 tokens to check for clarifying descriptors (e.g., “dizzy,” “nauseous”).

- **Human Review Flagging**  
  Ambiguous entities are marked with `"flagged_ambiguous": true` in metadata.

### 🔹 Missing Data (e.g., no duration)
I apply **default imputation with metadata flags**:

- Insert:  
  `"duration": "Not reported"`
- Add metadata:  
  `"duration_missing": true`
- UI prompts the physician to fill missing fields manually.

---

## **2. What pre-trained NLP models would you use for medical summarization?**

| Model | Reason |
|-------|--------|
| **BioBERT (`dmis-lab/biobert-v1.1`)** | Strong understanding of biomedical terminology. |
| **ClinicalBERT (`emilyalsentzer/Bio_ClinicalBERT`)** | Trained on ICU clinical notes (MIMIC-III), great for messy shorthand text. |
| **PubMedBERT** | Trained from scratch on biomedical papers, excellent for rare medical vocabulary. |

---

# ❤️ Part 2: Sentiment & Intent Analysis

## **1. How would you fine-tune BERT for medical sentiment detection?**

I use a **Domain-Adaptive Fine-Tuning (DAFT)** pipeline:

1. **Entity Masking**  
   Replace condition names/medications with placeholders (`[DISEASE]`, `[MED]`) to prevent bias.

2. **Layer Freezing**  
   Freeze the bottom **10 layers** of BERT to preserve grammar and generic language knowledge.

3. **Class-Weighted Loss**  
   Use **weighted cross-entropy** because the “Neutral” class dominates medical text.

---

## **2. What datasets would you use for a medical sentiment model?**

- **MIMIC-III Nursing Notes** — rich descriptions of patient affect.
- **MedWeb Dataset** — medical sentiment from social platforms.
- **eHealth Forum Dataset** — excellent for analyzing anxious patient tones vs. calm clinician responses.

---

# 📝 Part 3: Automated SOAP Note Generation

## **1. How would you train an NLP model to map transcripts into SOAP format?**

I treat this as a **Seq2Seq** task using:

- **T5-Base**
- **BART**

### **Training Setup:**
- **Input:** Raw transcript  
- **Output:** Structured SOAP note  

I fine-tune using `(Transcript → SOAP)` pairs and prompt tokens like:


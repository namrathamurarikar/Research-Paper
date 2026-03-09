# Multilingual Malicious Prompt Classification in Indian Languages

> **Status:** Work in Progress | Research Paper

A research project focused on detecting malicious prompts in low-resource Indian languages — **Hindi**, **Tamil**, and **Telugu** — by translating an English dataset and fine-tuning multilingual transformer models using both standard and parameter-efficient (LoRA) approaches.

---

## 📌 Overview

Large language models are increasingly used in non-English contexts, yet safety classifiers are predominantly trained on English data. This project addresses that gap by:

1. Translating the [`codesagar/malicious-llm-prompts-v4`](https://huggingface.co/datasets/codesagar/malicious-llm-prompts-v4) dataset into Hindi, Tamil, and Telugu using **IndicTrans2**
2. Validating translation quality via **back-translation** and MT metrics (chrF++, BLEU, METEOR, TER, BERTScore)
3. Filtering datasets based on quality thresholds
4. Fine-tuning four multilingual transformer models using both **regular fine-tuning** and **LoRA (PEFT)**
5. Evaluating model performance with a focus on **malicious prompt recall** (Class-1)

---

## 🗂️ Repository Structure

```
Research-Paper/
│
├── dataset/
│   ├── full_datasets/               # Translated + back-translated datasets (8,660 samples × 3 languages)
│   │   ├── hindi_full.csv
│   │   ├── tamil_full.csv
│   │   └── telugu_full.csv
│   ├── filtered_datasets/           # Datasets filtered by MT quality metrics
│   │   ├── hindi_filtered.csv
│   │   ├── tamil_filtered.csv
│   │   └── telugu_filtered.csv
│   └── translation_code/            # Translation pipeline notebooks
│       ├── hindi_translate.ipynb    # Translation + back-translation + metrics
│       ├── tamil_translate.ipynb
│       └── telugu_translate.ipynb
│
├── finetune/
│   ├── regular_finetune/            # Standard fine-tuning code + results per model
│   │   ├── indicbert/
│   │   ├── xlm_roberta/
│   │   ├── mdeberta_v3/
│   │   └── mmbert/
│   └── lora_finetune/               # LoRA/PEFT fine-tuning code + results per model
│       ├── indicbert/
│       ├── xlm_roberta/
│       ├── mdeberta_v3/
│       └── mmbert/
│
└── README.md
```

---

## 📦 Dataset

| Property | Details |
|---|---|
| Source | [`codesagar/malicious-llm-prompts-v4`](https://huggingface.co/datasets/codesagar/malicious-llm-prompts-v4) |
| Target Languages | Hindi, Tamil, Telugu |
| Samples per Language | ~8,660 (translated + back-translated) |
| Translation Model | `ai4bharat/indictrans2-en-indic-1B` (EN → Indic) |
| Back-translation Model | `prajdabre/rotary-indictrans2-indic-en-dist-200M` (Indic → EN) |
| Quality Metrics | chrF++, BLEU, METEOR, TER, BERTScore |

Datasets were filtered based on MT quality thresholds to remove low-confidence translations before model training.

---

## 🤖 Models

| Model | Fine-tuning Type |
|---|---|
| [IndicBERT](https://huggingface.co/ai4bharat/indic-bert) | Regular + LoRA |
| [XLM-RoBERTa](https://huggingface.co/xlm-roberta-base) | Regular + LoRA |
| [mDeBERTa-v3](https://huggingface.co/microsoft/mdeberta-v3-base) | Regular + LoRA |
| [MuRIL (mmbert)](https://huggingface.co/google/muril-base-cased) | Regular + LoRA |

All models were fine-tuned for **binary sequence classification** (benign vs. malicious prompt).

---

## ⚙️ Methodology

```
English Dataset
      │
      ▼
  IndicTrans2 (EN → Indic)
      │
      ▼
Hindi / Tamil / Telugu Translations
      │
      ▼
  Back-Translation (Indic → EN)
      │
      ▼
  MT Quality Metrics (chrF++, BLEU, BERTScore, ...)
      │
      ▼
  Filtered Dataset
      │
      ├──► Regular Fine-tuning (HuggingFace Trainer)
      └──► LoRA Fine-tuning (PEFT)
                │
                ▼
         Evaluation (Precision, Recall, F1, AUC)
```

Key design choices:
- **Focal Loss** with class weights to handle class imbalance
- **Threshold optimization** to maximize Class-1 (malicious) recall
- **LLRD (Layer-wise Learning Rate Decay)** for stable fine-tuning
- **Stratified train/val/test splits** to preserve class distribution

---

## 📊 Results

Results for all models across all three languages under both fine-tuning strategies are available in the `finetune/` directory, organized by model and approach.

---

## 🛠️ Setup & Requirements

```bash
pip install transformers datasets peft torch accelerate
pip install sacrebleu bert-score evaluate
```

> **Note:** Model training was performed on **Google Colab (A100 GPU)**. Some notebooks may require Colab Pro for sufficient compute and storage.

---

## 📁 HuggingFace Authentication

Some models (e.g., IndicTrans2) are gated. You will need to authenticate:

```python
from huggingface_hub import login
login(token="your_hf_token")
```

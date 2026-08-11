
# Token Classification

## What is this?

This notebook covers **Token Classification** using BERT, with **Named Entity Recognition (NER)** as the main task.

The goal is to understand how to prepare token-level labels and fine-tune a pretrained model for NER.

---

## 1. Dataset

We use the **CoNLL-2003** dataset.

The dataset contains:

```text
Tokens + NER Labels
```

Example:

```text
EU       → B-ORG
rejects  → O
German   → B-MISC
```

---

## 2. Tokenization & Label Alignment

The input words are passed to the BERT tokenizer and may be split into **subword tokens**.

The original word-level labels need to be aligned with the new tokens.

Special tokens use:

```text
-100
```

so they are ignored during loss calculation.

---

## 3. Model

We fine-tune:

```text
BERT
 ↓
Token Classification Head
 ↓
NER Label for each token
```

The model uses `id2label` and `label2id` to map between label IDs and label names.

---

## 4. Training Pipeline

```text
CoNLL-2003
   ↓
Tokenization
   ↓
Label Alignment
   ↓
Data Collator
   ↓
BERT + Classification Head
   ↓
Fine-tuning
   ↓
Evaluation
```

Training can be handled using the `Trainer` API or a custom PyTorch + Accelerate loop.

---

## 5. Evaluation

NER predictions are evaluated using **Seqeval**.

Main metrics:

```text
Precision
Recall
F1
Accuracy
```

---

## 6. Inference

After fine-tuning, the model can be used with the:

```text
token-classification
```

pipeline to detect entities in new text.

---

## Key Takeaways

* Token Classification predicts a label for **each token**.
* NER is a common Token Classification task.
* Word-level labels must be aligned with BERT subword tokens.
* `-100` is used to ignore tokens during loss calculation.
* `DataCollatorForTokenClassification` handles dynamic padding.
* `Trainer` simplifies training, while a custom loop gives more control.

### Mental Model

```text
Words + Labels
      ↓
Tokenization
      ↓
Label Alignment
      ↓
BERT
      ↓
Token-level Predictions
      ↓
NER Entities
```

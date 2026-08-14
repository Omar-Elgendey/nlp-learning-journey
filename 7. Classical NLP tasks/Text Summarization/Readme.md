
# Text Summarization

## What is this?

This notebook covers **Text Summarization** using mT5.

The goal is to understand how Encoder-Decoder models generate concise summaries from long documents and how to fine-tune a pretrained multilingual model for summarization.

---

## 1. Dataset

We use the **Amazon Reviews Multi** dataset.

```
Review Body
        +
Review Title
```

Example:

```
Review:
I loved this book. The explanations were clear and easy to follow.

Summary:
Easy to follow
```

The review is used as the input and the title is used as the target summary.

---

## 2. Dataset Preparation

```
English Reviews
        +
Spanish Reviews
        ↓
Concatenate
        ↓
Shuffle
        ↓
Bilingual Dataset
```

Book-related reviews are selected using:

```
book
digital_ebook_purchase
```

Very short titles are removed to encourage more meaningful summaries.

---

## 3. Tokenization

```
Review Text
      ↓
input_ids

Summary Text
      ↓
labels
```

The target summaries are processed using:

```
text_target
```

Both reviews and summaries are truncated to fit within the model's context window.

---

## 4. Architecture

```
Review Text
      ↓
Encoder
      ↓
Hidden Representations
      ↓
Attention
      ↓
Decoder
      ↓
Generated Summary
```

The encoder understands the review while the decoder generates the summary token by token.

---

## 5. Training Pipeline

```
Amazon Reviews
      ↓
Filtering
      ↓
Tokenization
      ↓
Preprocessing
      ↓
DataCollatorForSeq2Seq
      ↓
mT5
      ↓
Seq2SeqTrainer
      ↓
Evaluation
```

During training:

```
labels
    ↓
Shift Right
    ↓
decoder_input_ids
```

Teacher Forcing is used, meaning the decoder receives the correct previous summary token during training.

---

## 6. Evaluation

```
generate()
      ↓
Generated Summary
      ↓
ROUGE
```

Common metrics:

```
ROUGE-1
ROUGE-2
ROUGE-L
```

ROUGE measures the overlap between generated summaries and reference summaries.

---

## 7. Inference Pipeline

```
Review Text
      ↓
Tokenizer
      ↓
Encoder
      ↓
Attention
      ↓
Decoder
      ↓
generate()
      ↓
Decoded Summary
```

---

## 8. Data Collation

`DataCollatorForSeq2Seq` handles:

```
Dynamic Padding
Label Padding
decoder_input_ids Creation
```

Padding labels are replaced with:

```
-100
```

so they are ignored during loss computation.

---

## Key Takeaways

- Summarization is a Seq2Seq task.
- Reviews are used as inputs.
- Titles are used as target summaries.
- Inputs are stored as `input_ids`.
- Targets are stored as `labels`.
- `text_target` simplifies target tokenization.
- `decoder_input_ids` are shifted labels.
- Teacher Forcing is used during training.
- mT5 is a multilingual Encoder-Decoder model.
- `DataCollatorForSeq2Seq` prepares batches automatically.
- ROUGE is the standard summarization metric.

### Mental Model

```
Review Text
      ↓
Tokenization
      ↓
Encoder
      ↓
Attention
      ↓
Decoder
      ↓
Next Token Prediction
      ↓
Generated Summary
```

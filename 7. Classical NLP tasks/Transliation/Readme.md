# Machine Translation

## What is this?

This notebook covers **Machine Translation** using MarianMT.

The goal is to understand how Encoder-Decoder models translate text between languages and how to fine-tune a pretrained translation model on a translation dataset.

---

## 1. Dataset

We use the **KDE4** dataset.

```
Source Sentence
        +
Target Sentence
```

Example:

```
English:
Default to expanded threads

French:
Par défaut, développer les fils de discussion
```

---

## 2. Tokenization

```
Source Sentence
        ↓
input_ids

Target Sentence
        ↓
labels
```

The target text is processed using:

```
text_target
```

---

## 3. Architecture

```
Source Text
      ↓
Encoder
      ↓
Hidden Representations
      ↓
Attention
      ↓
Decoder
      ↓
Translated Text
```

The encoder understands the source sentence while the decoder generates the target sentence token by token.

---

## 4. Training Pipeline

```
KDE4 Dataset
      ↓
Tokenization
      ↓
Preprocessing
      ↓
DataCollatorForSeq2Seq
      ↓
MarianMT
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

Teacher Forcing is used, meaning the decoder receives the correct previous target token during training.

---

## 5. Evaluation

```
generate()
      ↓
Predicted Translation
      ↓
SacreBLEU
```

SacreBLEU measures translation quality by comparing generated translations with reference translations.

---

## 6. Inference Pipeline

```
Source Sentence
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
Decoded Translation
```

---

## 7. Data Collation

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

- Machine Translation is a Seq2Seq task.
- Inputs are stored as `input_ids`.
- Targets are stored as `labels`.
- `text_target` simplifies target tokenization.
- `decoder_input_ids` are shifted labels.
- Teacher Forcing is used during training.
- MarianMT uses an Encoder-Decoder architecture.
- `DataCollatorForSeq2Seq` prepares batches automatically.
- `Seq2SeqTrainer` handles training and evaluation.
- SacreBLEU is the standard translation metric.

### Mental Model

```
English Text
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
French Translation
```

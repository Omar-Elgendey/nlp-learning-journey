# Machine Translation

## What is this?

This notebook covers **Machine Translation** using MarianMT.

The goal is to understand how Encoder-Decoder models translate text between languages and how to fine-tune a pretrained model on a translation dataset.

---

## 1. Dataset

We use the **KDE4** dataset.

The dataset contains:

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

Both source and target texts must be tokenized.

```
English Sentence
        ↓
input_ids

French Sentence
        ↓
labels
```

The target text is processed using:

```
text_target
```

to ensure correct tokenization for the output language.

---

## 3. Model

We fine-tune:

```
Encoder
    ↓
Context Representation
    ↓
Decoder
    ↓
Translated Text
```

using a pretrained MarianMT model.

During training, the decoder receives:

```
decoder_input_ids
```

which are shifted versions of the target labels.

---

## 4. Training Pipeline

```
KDE4 Dataset
      ↓
Tokenization
      ↓
Preprocessing
      ↓
Data Collator
      ↓
Encoder-Decoder Model
      ↓
Fine-tuning
      ↓
Evaluation
```

Training is performed using:

```
Seq2SeqTrainer
```

---

## 5. Evaluation

Translations are generated using:

```
generate()
```

and evaluated using:

```
SacreBLEU
```

---

## 6. Data Collation

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

- Machine Translation is a Sequence-to-Sequence task.
- Inputs are stored as `input_ids`.
- Targets are stored as `labels`.
- `decoder_input_ids` are shifted versions of the labels.
- Teacher Forcing is used during training.
- `DataCollatorForSeq2Seq` prepares batches automatically.
- `Seq2SeqTrainer` uses `generate()` during evaluation.
- SacreBLEU is a common metric for translation quality.

### Mental Model

```
English Text
      ↓
Tokenization
      ↓
Encoder
      ↓
Decoder
      ↓
Next Token Prediction
      ↓
French Translation
```


# Fine-tuning a Masked Language Model

## What is this?

This notebook covers **Masked Language Modeling (MLM)** using DistilBERT.

The goal is to understand how a pretrained language model can be adapted to a specific domain and trained to predict masked tokens from their surrounding context.

---

## 1. Dataset

We use the **IMDb Reviews** dataset.

The dataset contains:

```text
Movie Review
+
Sentiment Label
```

Example:

```text
This movie was absolutely fantastic.
```

For language modeling, the sentiment labels are not used.

Only the review text is needed.

---

## 2. Domain Adaptation

Instead of training a language model from scratch, we start with a pretrained model and continue training it on domain-specific data.

```text
Pretrained DistilBERT
         ↓
IMDb Reviews
         ↓
Domain Adapted DistilBERT
```

This helps the model better understand the vocabulary and writing style of the target domain.

---

## 3. Tokenization

The reviews are tokenized using the DistilBERT tokenizer.

Each review is converted into:

```python
input_ids
attention_mask
word_ids
```

The `word_ids` are later used for Whole Word Masking.

---

## 4. Concatenation and Chunking

All tokenized reviews are concatenated together:

```text
Review 1
+
Review 2
+
Review 3
+
...
```

Then split into fixed-length chunks:

```text
Long Sequence
      ↓
128 Tokens
128 Tokens
128 Tokens
...
```

This avoids losing information through truncation and creates more training examples.

---

## 5. Labels

For Masked Language Modeling, the labels are initially copied from the inputs:

```python
labels = input_ids.copy()
```

```text
input_ids
     ↓
   Copy
     ↓
labels
```

These labels provide the correct answers during training.

---

## 6. Dynamic Masking

Random tokens are replaced with:

```text
[MASK]
```

Example:

```text
I love playing football
```

becomes:

```text
I love [MASK] football
```

The model learns to predict the hidden token using the surrounding context.

Masking is performed dynamically using:

```python
DataCollatorForLanguageModeling
```

---

## 7. Whole Word Masking

Standard MLM may mask only part of a word.

Example:

```text
play ##ing
```

might become:

```text
[MASK] ##ing
```

Whole Word Masking masks every token belonging to the same word:

```text
[MASK] [MASK]
```

This is implemented using the `word_ids` information produced by the tokenizer.

---

## 8. Loss Computation

Only masked tokens contribute to the loss.

Non-masked positions are replaced with:

```python
-100
```

Example:

```python
[-100, -100, play, ##ing, -100]
```

The loss function ignores all positions containing `-100`.

---

## 9. Training Pipeline

```text
IMDb Reviews
      ↓
Tokenization
      ↓
Concatenation
      ↓
Chunking
      ↓
Label Creation
      ↓
Dynamic Masking
      ↓
DistilBERT
      ↓
Fine-tuning
      ↓
Evaluation
```

Training is performed using:

```python
Trainer
```

---

## 10. Model

We fine-tune:

```text
Input Tokens
      ↓
Transformer Encoder
      ↓
Contextual Representations
      ↓
Vocabulary Prediction Head
      ↓
Masked Token Prediction
```

The model predicts the original tokens hidden behind the `[MASK]` symbols.

---

## 11. Evaluation

Language models are evaluated using:

```text
Perplexity
```

which is computed from the evaluation loss:

```python
perplexity = exp(eval_loss)
```

Lower perplexity indicates better language modeling performance.

Example:

```text
Perplexity = 21.75
```

---

## 12. Trainer Components

The training loop combines:

```python
model
dataset
tokenizer
data_collator
training_args
```

inside:

```python
Trainer
```

The Trainer automatically handles:

```text
Forward Pass
Loss Computation
Backpropagation
Optimization
Evaluation
```

---

## Key Takeaways

- Masked Language Modeling predicts hidden tokens using context.
- Domain Adaptation specializes a pretrained model for a target domain.
- Reviews are concatenated and split into fixed-size chunks.
- Labels are copied from the original input tokens.
- Dynamic masking is performed by `DataCollatorForLanguageModeling`.
- Whole Word Masking masks complete words instead of individual subwords.
- `-100` tells the loss function to ignore a position.
- DistilBERT is fine-tuned using the `Trainer` API.
- Perplexity is the primary evaluation metric for language models.

### Mental Model

```text
IMDb Reviews
      ↓
Tokenization
      ↓
Concatenate & Chunk
      ↓
Create Labels
      ↓
Random Masking
      ↓
DistilBERT
      ↓
Predict Missing Tokens
      ↓
Domain Adaptation
      ↓
Perplexity Evaluation
```

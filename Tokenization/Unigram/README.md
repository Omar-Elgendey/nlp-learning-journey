# Unigram

## 1. What is Unigram?

Unigram is a **subword tokenization algorithm** used in models and tokenization systems such as SentencePiece.

Unlike BPE and WordPiece, Unigram does **not build the vocabulary by merging pairs**.

Instead, it starts with a **large vocabulary of candidate subwords** and gradually removes the least useful ones.

### Core idea

> **Start big → evaluate tokens → remove weak tokens → repeat.**

---

# 2. Training Architecture

```text id="v7m2k9"
                Training Corpus
                       │
                       ▼
                Build Candidates
                       │
                       ▼
              Initial Large Vocabulary
                       │
                       ▼
                Assign Probabilities
                       │
                       ▼
              Find Best Segmentations
                       │
                       ▼
                  Calculate Loss
                       │
                       ▼
              Evaluate Token Importance
                       │
                       ▼
                Remove Weak Tokens
                       │
                       ▼
               Update Probabilities
                       │
                       ▼
                  Repeat Loop
                       │
                       ▼
              Final Vocabulary
```

---

# 3. Training Pipeline

### 1. Build Initial Vocabulary

Unlike BPE and WordPiece, Unigram starts with **many candidate subwords**.

For example, for:

```text id="a8n4p2"
playing
```

possible candidates could include:

```text id="q3m7x1"
p
pl
pla
play
playing
lay
ing
...
```

The initial vocabulary is intentionally larger than the final vocabulary.

---

### 2. Assign Probabilities

Each candidate token gets a probability.

For example:

```text id="k5v9r2"
play → probability
ing  → probability
p    → probability
lay  → probability
```

These probabilities describe how likely the tokens are in the corpus.

---

### 3. Find Possible Segmentations

A word can be split in different ways.

For:

```text id="w2x7m4"
playing
```

possible segmentations could be:

```text id="c8n3q6"
playing

play + ing

p + lay + ing
```

Unigram evaluates these different possibilities.

---

### 4. Find the Best Segmentation

The probability of a segmentation is based on the probabilities of its tokens.

Conceptually:

```text id="r6m2v8"
P(play + ing)
=
P(play) × P(ing)
```

In practice, log probabilities are commonly used.

The tokenizer chooses the segmentation with the highest probability.

---

# 4. Main Training Loop

The main Unigram loop is different from BPE and WordPiece.

```text id="k4p8m2"
while vocabulary > target_size:

    Find best segmentations

    Calculate model loss

    Evaluate token importance

    Remove least useful tokens

    Update probabilities
```

So the main idea is:

```text id="x7q3n9"
Evaluate
   ↓
Prune
   ↓
Update
   ↓
Repeat
```

### Important difference

BPE / WordPiece:

```text id="n2m6v8"
Small → Bigger
```

Unigram:

```text id="q5r9k3"
Large → Smaller
```

---

# 5. Why Does It Remove Tokens?

Suppose the vocabulary contains:

```text id="v8m3q1"
play
playing
ing
```

If the model can already represent:

```text id="z4k7n2"
playing
```

very well using:

```text id="c6p9x3"
play + ing
```

then the token:

```text id="a2m5r8"
playing
```

may not be necessary.

Removing it may have very little effect on the model.

So Unigram tries to keep tokens that are actually useful.

---

# 6. Token Importance

To decide which tokens to remove, Unigram estimates how much the model would suffer if a token disappeared.

Conceptually:

```text id="m7q2v5"
Current Loss
     ↓
Remove token
     ↓
New Loss
     ↓
Compare
```

If removing a token causes almost no change:

```text id="k3n8x1"
Token → not very important
```

If removing it causes a large increase in loss:

```text id="r6p2m9"
Token → important
```

The least useful tokens become candidates for removal.

---

# 7. Pruning

The process of removing weak tokens is called **pruning**.

For example:

```text id="x5m8q2"
10,000 candidates
       ↓
Pruning
       ↓
8,000
       ↓
Pruning
       ↓
6,000
       ↓
Pruning
       ↓
4,000
       ↓
Final Vocabulary
```

The exact sizes depend on the tokenizer configuration.

The important idea is:

> **Unigram gradually removes unnecessary tokens until the desired vocabulary size is reached.**

---

# 8. Finding the Best Segmentation

A word can have many possible segmentations.

For example:

```text id="n4q8m2"
playing
```

could be:

```text id="v5x2p7"
playing
```

or:

```text id="c9m3k6"
play + ing
```

or:

```text id="r2q7n4"
p + lay + ing
```

Checking every possible segmentation would be expensive.

So Unigram uses a **dynamic programming approach**, commonly implemented with the **Viterbi algorithm**, to find the best path efficiently.

Conceptually:

```text id="m8q3v1"
Possible Tokens
      ↓
Possible Paths
      ↓
Calculate Best Score
      ↓
Best Path
      ↓
Final Segmentation
```

---

# 9. Tokenization Pipeline

After training, the tokenizer has:

```text id="q6m2x8"
Final Vocabulary
+
Token Probabilities
```

For a new word:

```text id="v3n7p5"
New Word
   ↓
Find possible vocabulary pieces
   ↓
Build possible segmentations
   ↓
Calculate probabilities
   ↓
Viterbi / Best Path
   ↓
Final Tokens
```

---

# 10. Example

Suppose the vocabulary contains:

```text id="m7x2q9"
play
playing
ing
p
lay
```

For:

```text id="r4n8k1"
playing
```

possible segmentations include:

```text id="c6v2m5"
playing
```

```text id="q9x3p7"
play + ing
```

```text id="n5k8r2"
p + lay + ing
```

The model compares their probabilities and chooses the best segmentation.

So unlike WordPiece, Unigram does **not simply choose the longest token**.

---

# 11. Important Implementation Parts

### Candidate Vocabulary

Contains the initial large set of possible subwords.

```text id="x8m3q5"
p
pl
pla
play
playing
ing
...
```

### Probabilities

Each candidate token has a probability.

These probabilities are updated during training.

### Segmentation

Finds the best way to represent each word using the current vocabulary.

### Loss

Measures how well the current vocabulary represents the training corpus.

### Pruning

Removes tokens that contribute the least.

### Training Loop

Repeatedly:

```text id="q4n7m2"
Find best segmentations
→ calculate loss
→ evaluate tokens
→ remove weak tokens
→ update
```

---

# 12. BPE vs WordPiece vs Unigram

|                | BPE                | WordPiece            | Unigram             |
| -------------- | ------------------ | -------------------- | ------------------- |
| Starting point | Small vocabulary   | Small vocabulary     | Large vocabulary    |
| Training       | Merge              | Merge                | Prune               |
| Main decision  | Most frequent pair | Highest-scoring pair | Least useful tokens |
| Vocabulary     | Grows              | Grows                | Shrinks             |
| Tokenization   | Merge rules        | Longest match        | Best probability    |
| Main idea      | Frequency          | Score                | Probability         |

---

# 13. Key Takeaways

* Unigram is a **subword tokenizer**.
* It starts with a relatively large candidate vocabulary.
* Each token has a probability.
* Words can have multiple possible segmentations.
* The model finds the most probable segmentation.
* Unigram evaluates how useful each token is.
* The least useful tokens are removed.
* This pruning process repeats until the target vocabulary size.
* Viterbi/dynamic programming is used to efficiently find the best segmentation.
* Unlike BPE and WordPiece, Unigram **shrinks the vocabulary instead of growing it**.

### Mental Model

```text id="z5q2m8"
Large Candidate Vocabulary
          ↓
     Probabilities
          ↓
 Best Segmentations
          ↓
    Evaluate Tokens
          ↓
     Remove Weak Ones
          ↓
        Repeat
          ↓
   Final Vocabulary
```

> **Unigram = probabilistic tokenization + vocabulary pruning.**

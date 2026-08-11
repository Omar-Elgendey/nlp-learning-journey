# WordPiece

## 1. What is WordPiece?

WordPiece is a **subword tokenization algorithm** used famously in BERT.

Like BPE, it starts with small tokens and builds larger subwords. The main difference is **how it chooses the pair to merge**.

### Core idea

> **Start with small tokens, calculate a score for each adjacent pair, then merge the pair with the highest score.**

The score is:

```text id="5v7q2m"
                 Frequency(A, B)
Score = ─────────────────────────────────
         Frequency(A) × Frequency(B)
```

So unlike BPE, the most frequent pair is **not necessarily** the best pair.

---

# 2. Training Architecture

```text id="9j1p4a"
                Training Corpus
                       │
                       ▼
                Pre-tokenization
                       │
                       ▼
                 Word Frequencies
                       │
                       ▼
                Initial Vocabulary
                       │
                       ▼
              Initial WordPiece Splits
                       │
                       ▼
             Token & Pair Frequencies
                       │
                       ▼
                Calculate Scores
                       │
                       ▼
              Highest-Scoring Pair
                       │
                       ▼
                    Merge
                       │
              ┌────────┴────────┐
              ▼                 ▼
       Add New Token      Update Splits
              │                 │
              └────────┬────────┘
                       ▼
                 Repeat Loop
                       │
                       ▼
              Target Vocabulary
```

---

# 3. Training Pipeline

### 1. Pre-tokenization

The corpus is first split into words.

```text id="4r8y9c"
"This is WordPiece."
↓
"This" "is" "WordPiece" "."
```

### 2. Word Frequencies

Count how often every word appears.

```text id="k9h6u1"
hug  → 10
pug  → 5
pun  → 12
bun  → 4
hugs → 5
```

### 3. Initial Vocabulary

WordPiece starts from the characters in the corpus.

But it distinguishes between:

* Characters at the beginning of a word
* Characters inside a word

Continuation characters get the `##` prefix.

For example:

```text id="q7d3kp"
hug
↓
h ##u ##g
```

So the vocabulary can contain:

```text id="1q0v8c"
h
##u
##g
```

Special tokens such as:

```text id="g6b2pl"
[PAD] [UNK] [CLS] [SEP] [MASK]
```

can also be added.

---

# 4. Initial Splits

Every word starts as individual WordPiece tokens.

For example:

```text id="6h4s9q"
hug
→ [h, ##u, ##g]

hugs
→ [h, ##u, ##g, ##s]
```

The `##` means:

> **This token is a continuation of the same word.**

---

# 5. Calculate Pair Scores

WordPiece looks at every adjacent pair.

For:

```text id="4z3v1n"
[h, ##u, ##g]
```

the pairs are:

```text id="v4m7t8"
(h, ##u)
(##u, ##g)
```

For each pair we calculate:

```text id="b2q8n0"
                 Pair Frequency
Score = ─────────────────────────────────
         Frequency(A) × Frequency(B)
```

The idea is that a pair should not get a high score **just because its individual tokens are extremely common**.

So WordPiece considers both:

```text id="0pl5xw"
How often the pair appears
+
How common its individual parts are
```

---

# 6. Select the Best Pair

After calculating all scores, WordPiece selects:

> **The pair with the highest score.**

For example:

```text id="8z3j1n"
(h, ##u)       → 1/36
(##u, ##g)     → 1/36
(##g, ##s)     → 1/20
```

The highest score is:

```text id="t5f7j2"
(##g, ##s)
```

so that pair is selected.

This is the main difference from BPE:

```text id="1q6m4x"
BPE
→ highest frequency

WordPiece
→ highest score
```

---

# 7. Merge the Pair

Suppose the selected pair is:

```text id="w4n9p2"
(##g, ##s)
```

It becomes:

```text id="2x6c8v"
##gs
```

Notice that the `##` prefix is kept only once.

For example:

```text id="a8f4m3"
h ##u ##g ##s
```

becomes:

```text id="v6k2q9"
h ##u ##gs
```

The new token is added to the vocabulary.

---

# 8. Update the Splits

After the merge, all affected word splits are updated.

Before:

```text id="q5t7n1"
h ##u ##g ##s
```

After:

```text id="z8r3m4"
h ##u ##gs
```

The next iteration uses these updated splits to calculate new frequencies and scores.

---

# 9. Main Training Loop

The entire WordPiece training loop can be summarized as:

```text id="k2m8x4"
while vocabulary < target_size:

    Calculate pair scores

    Find highest-scoring pair

    Merge the pair

    Add new token to vocabulary

    Update word splits
```

So:

```text id="n7q1b5"
Calculate
    ↓
Select
    ↓
Merge
    ↓
Update
    ↓
Repeat
```

The difference from BPE is mainly the **selection step**.

---

# 10. Important Implementation Parts

### `word_freqs`

Stores the frequency of every word.

```text id="v3q8n2"
hug → 10
pug → 5
```

### `splits`

Stores the current WordPiece representation.

Initially:

```text id="r4x9m1"
hug → [h, ##u, ##g]
```

After merging:

```text id="d7k2p5"
hug → [hu, ##g]
```

### `compute_pair_scores()`

Calculates:

* Individual token frequencies
* Pair frequencies
* Score for every pair

Conceptually:

```text id="y5m3q8"
Count tokens
      ↓
Count pairs
      ↓
Calculate score
```

### `merge_pair()`

Applies the selected merge to the current word splits.

```text id="p4x7n2"
[h, ##u, ##g]
```

merge:

```text id="m8q1v6"
(h, ##u)
```

becomes:

```text id="c3z5r9"
[hu, ##g]
```

---

# 11. Tokenization Pipeline

Training and tokenization work differently.

After training, WordPiece keeps the **final vocabulary**.

To tokenize a new word:

```text id="j6w2k8"
New Word
   ↓
Start from the beginning
   ↓
Find the longest vocabulary token
   ↓
Split it
   ↓
Search for the longest continuation token
   ↓
Repeat
   ↓
Complete word?
```

The key idea is:

> **WordPiece uses a longest-match strategy.**

---

# 12. Example — `hugs`

Suppose the vocabulary contains:

```text id="r5q8m2"
hug
hu
##g
##s
##gs
```

Start with:

```text id="n3v7x1"
hugs
```

Possible matches at the beginning:

```text id="q8m4k2"
h
hu
hug
```

The longest valid match is:

```text id="a5x9p3"
hug
```

So:

```text id="z7c2n6"
hugs
→ hug + s
```

The remaining part becomes a continuation:

```text id="f4m8q1"
##s
```

Therefore:

```text id="w6r3t9"
hugs
→ ["hug", "##s"]
```

---

# 13. `[UNK]` Behavior

WordPiece requires the **whole word** to be tokenizable.

Suppose:

```text id="q3m7x9"
bum
```

can start with:

```text id="j8k2p4"
b
##u
```

but there is no valid token for:

```text id="v5n1r6"
##m
```

WordPiece does not return:

```text id="z4c8t2"
[b, ##u, [UNK]]
```

Instead, the entire word becomes:

```text id="m6q9w3"
[UNK]
```

So:

```text id="y7p2k5"
Complete word can be represented
→ return tokens

Cannot complete the word
→ [UNK]
```

---

# 14. BPE vs WordPiece

The training process looks similar:

```text id="h2k7m4"
Initial Vocabulary
       ↓
Find Pair
       ↓
Merge
       ↓
Update
       ↓
Repeat
```

But the pair selection is different.

### BPE

```text id="x5q8m2"
Most frequent pair
```

### WordPiece

```text id="p3n7r1"
Highest-scoring pair
```

WordPiece score:

```text id="k9v2d6"
                 Pair Frequency
Score = ─────────────────────────────────
         Frequency(A) × Frequency(B)
```

And tokenization is also different:

```text id="z4m8q1"
BPE
→ Apply learned merge rules

WordPiece
→ Longest vocabulary match
```

---

# 15. Key Takeaways

* WordPiece is a **subword tokenizer**.
* It starts with characters and builds larger subwords.
* Continuation tokens use the `##` prefix.
* It calculates a score for every adjacent pair.
* The **highest-scoring pair** is merged.
* The vocabulary grows after every merge.
* The process repeats until the target vocabulary size.
* During tokenization, WordPiece uses the **longest matching subword**.
* If a word cannot be completely represented, it becomes `[UNK]`.
* The main training loop is:

```text id="c6n2v8"
Score → Select → Merge → Update → Repeat
```

### Mental Model

```text id="m7q3x9"
Small Vocabulary
      ↓
Score Pairs
      ↓
Best Pair
      ↓
Merge
      ↓
New Token
      ↓
Repeat
      ↓
Final Vocabulary
```

> **WordPiece = score-based merging during training + longest-match tokenization.**

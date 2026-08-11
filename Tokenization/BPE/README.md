# BPE — Byte Pair Encoding

## 1. What is BPE?

BPE (Byte Pair Encoding) is a **subword tokenization algorithm**.

Instead of representing text only as complete words or individual characters, BPE learns useful subword units from the corpus.

### Core idea

> **Start with small tokens, then repeatedly merge the most frequent adjacent pair.**

Example:

```text
h u g
↓
hu g
↓
hug
```

This allows the tokenizer to represent common words efficiently while still being able to build unseen words from smaller subwords.

---

# 2. Training Architecture

```text
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
                  Initial Splits
                       │
                       ▼
               Pair Frequencies
                       │
                       ▼
              Most Frequent Pair
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

The corpus is first split into words or text pieces.

```text
"This is BPE."
↓
"This" "is" "BPE" "."
```

### 2. Word Frequencies

Count how often each word appears.

```text
hug → 10
pug → 5
pun → 12
```

These frequencies are later used when counting pairs.

### 3. Initial Vocabulary

Start with the characters appearing in the corpus.

```text
h u g p n b ...
```

Special tokens can also be added depending on the tokenizer.

### 4. Initial Splits

Each word starts as individual tokens.

```text
hug
→ [h, u, g]
```

### 5. Count Pair Frequencies

Look at adjacent tokens:

```text
[h, u, g]

(h, u)
(u, g)
```

The word frequency contributes to the frequency of each pair.

### 6. Select the Best Pair

BPE chooses the **most frequent pair**.

```text
(u, g) → 20
(h, u) → 15
(g, s) → 5
```

So:

```text
(u, g)
```

is selected.

### 7. Merge

```text
u + g → ug
```

The new token `ug` is added to the vocabulary.

The word becomes:

```text
[h, ug]
```

### 8. Repeat

Pair frequencies are calculated again using the updated splits.

This continues until the desired vocabulary size is reached.

---

# 4. Main Training Loop

The entire training algorithm can be remembered as:

```text
while vocabulary < target_size:

    Count pair frequencies
          ↓
    Select most frequent pair
          ↓
    Merge pair
          ↓
    Add new token
          ↓
    Update word splits
```

So the BPE loop is simply:

```text
Count → Select → Merge → Update → Repeat
```

---

# 5. Important Implementation Parts

The from-scratch implementation can be understood through a few main structures/functions.

### `word_freqs`

Stores:

```text
word → frequency
```

Example:

```text
hug → 10
pug → 5
```

### `splits`

Stores the current tokenization of every word.

Initially:

```text
hug → [h, u, g]
```

After merging:

```text
hug → [hu, g]
```

### `compute_pair_freqs()`

Counts all adjacent pairs in the current splits.

```text
For every word:
    For every adjacent pair:
        add word frequency
```

### `merge_pair()`

Applies the selected merge to the corpus.

```text
[h, u, g]
```

with:

```text
(h, u)
```

becomes:

```text
[hu, g]
```

### Training Loop

Repeatedly:

```text
compute frequencies
→ find best pair
→ merge
→ update vocabulary
```

---

# 6. Tokenization Pipeline

After training, BPE uses the learned vocabulary and **ordered merge rules** to tokenize new text.

```text
New Text
   ↓
Pre-tokenization
   ↓
Initial tokenization
   ↓
Apply learned merges
   ↓
Final subword tokens
```

For example:

```text
playing
↓
p l a y i n g
↓
play i n g
↓
play ing
```

The exact result depends on the learned merge rules.

---

# 7. What Does BPE Learn?

BPE mainly learns:

```text
Vocabulary
+
Ordered Merge Rules
```

For example:

```text
u + g → ug
h + ug → hug
i + n → in
in + g → ing
```

The order matters because tokenization follows the learned merge priorities.

---

# 8. BPE vs WordPiece

Both algorithms build their vocabulary by **merging tokens**, but they choose merges differently.

```text
BPE
→ Most frequent pair

WordPiece
→ Highest-scoring pair
```

BPE uses:

```text
Score = Pair Frequency
```

WordPiece uses:

```text
                  Pair Frequency
Score = ─────────────────────────────────
         Frequency(A) × Frequency(B)
```

So BPE is mainly **frequency-based**, while WordPiece uses a normalized score.

---

# 9. Key Takeaways

* BPE is a **subword tokenizer**.
* It starts with small tokens, usually characters.
* It counts adjacent token pairs.
* It selects the **most frequent pair**.
* It merges that pair into a new token.
* The new token is added to the vocabulary.
* The word splits are updated after every merge.
* The process continues until the target vocabulary size.
* BPE learns **merge rules + vocabulary**.
* During tokenization, the learned merge rules are applied to new text.

### Mental Model

```text
Small Vocabulary
      ↓
Frequent Pair
      ↓
Merge
      ↓
New Token
      ↓
Repeat
      ↓
Useful Subword Vocabulary
```

> **BPE = repeatedly merge the most frequent adjacent pair.**

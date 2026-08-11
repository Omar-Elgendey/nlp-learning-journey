# Byte Pair Encoding (BPE)

## 1. Overview

Byte Pair Encoding (BPE) is a **subword tokenization algorithm**.

Instead of representing text only as complete words or individual characters, BPE learns meaningful and reusable **subword units**.

For example:

```text
playing
→ play + ing
```

This gives us a balance between:

* **Word-level tokenization**

  * Small number of tokens per sentence
  * Very large vocabulary
  * Poor handling of unseen words

* **Character-level tokenization**

  * Very small vocabulary
  * Can represent almost any word
  * Produces much longer sequences

BPE tries to find a useful middle ground by learning which character sequences occur frequently enough to become subword tokens.

---

# 2. Core Idea

The main idea behind BPE is simple:

> **Repeatedly merge the most frequent adjacent pair of tokens.**

Suppose the current representation is:

```text
h u g
```

and the pair:

```text
(u, g)
```

is very frequent across the corpus.

BPE merges it:

```text
h u g
↓
h ug
```

Later, if:

```text
(h, ug)
```

becomes the most frequent pair:

```text
h ug
↓
hug
```

Through many iterations, the tokenizer gradually builds larger and more useful subword units.

---

# 3. BPE Training Pipeline

The complete training process can be visualized as:

```text
                    RAW CORPUS
                        │
                        ▼
                 PRE-TOKENIZATION
                        │
                        ▼
                  WORD FREQUENCIES
                        │
                        ▼
              INITIAL VOCABULARY
                  (characters)
                        │
                        ▼
                INITIAL SPLITS
                        │
                        ▼
              COUNT ADJACENT PAIRS
                        │
                        ▼
            FIND MOST FREQUENT PAIR
                        │
                        ▼
                   MERGE PAIR
                        │
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼
       ADD NEW TOKEN        SAVE MERGE RULE
       TO VOCABULARY
              │                   │
              └─────────┬─────────┘
                        ▼
                 UPDATE SPLITS
                        │
                        ▼
             RECALCULATE PAIRS
                        │
                        ▼
            FIND NEXT BEST PAIR
                        │
                        ▼
                     REPEAT
                        │
                        ▼
              TARGET VOCAB SIZE
                        │
                        ▼
             TRAINED BPE TOKENIZER
```

The important thing is that training is **iterative**.

Each iteration follows:

```text
Count → Select → Merge → Update → Repeat
```

---

# 4. Step 1 — Pre-tokenization

Before learning subwords, the corpus is first divided into smaller units such as words.

For example:

```text
"This is a tokenizer."
```

may be pre-tokenized into:

```text
This
is
a
tokenizer
.
```

The exact result depends on the pre-tokenizer being used.

At this stage, we are **not learning subwords yet**.

We are simply preparing the corpus.

---

# 5. Step 2 — Word Frequencies

After pre-tokenization, we count how frequently each word appears.

For example:

```text
hug     → 10
pug      → 5
pun     → 12
bun      → 4
hugs     → 5
```

Why do we need word frequencies?

Because pair frequencies are calculated across the entire corpus.

If a word appears 10 times, every pair inside that word contributes 10 occurrences.

For example:

```text
hug → 10
```

contains:

```text
(h, u)
(u, g)
```

So each pair receives a contribution of 10.

---

# 6. Step 3 — Initial Vocabulary

BPE starts with a small vocabulary.

In the character-level implementation we studied, the initial vocabulary contains the unique characters found in the corpus.

For example:

```text
hug
pug
pun
```

initially use:

```text
h
u
g
p
n
```

At this point, BPE has no large subword tokens such as:

```text
hug
pu
un
```

Those will be learned through the merging process.

---

# 7. Step 4 — Initial Splits

Every word is initially split into its individual characters.

For example:

```text
hug
```

becomes:

```text
h u g
```

and:

```text
hugs
```

becomes:

```text
h u g s
```

This gives BPE a starting point from which it can learn larger units.

---

# 8. Step 5 — Count Adjacent Pair Frequencies

Now BPE looks at every pair of neighboring tokens.

For:

```text
h u g
```

the adjacent pairs are:

```text
(h, u)
(u, g)
```

For:

```text
h u g s
```

the pairs are:

```text
(h, u)
(u, g)
(g, s)
```

The frequencies of the words are used when calculating the total pair frequency.

For example:

```text
hug  → 10
hugs → 5
```

Both contain:

```text
(u, g)
```

so the pair frequency becomes:

```text
(u, g) → 15
```

This is how BPE determines which combinations are common across the corpus.

---

# 9. Step 6 — Select the Best Pair

BPE uses a very simple selection rule:

> **Choose the most frequent adjacent pair.**

For example:

```text
(u, g) → 20
(p, u) → 16
(h, u) → 15
(g, s) → 5
```

The selected pair is:

```text
(u, g)
```

because it has the highest frequency.

### Important intuition

BPE does not ask:

> "Is this pair special?"

It mainly asks:

> **"How often does this pair occur?"**

That is the core difference between BPE and WordPiece.

---

# 10. Step 7 — Merge the Pair

Suppose:

```text
(u, g)
```

was selected.

BPE merges the two tokens:

```text
u g
↓
ug
```

Therefore:

```text
h u g
```

becomes:

```text
h ug
```

And:

```text
h u g s
```

becomes:

```text
h ug s
```

The newly created token:

```text
ug
```

is added to the vocabulary.

---

# 11. Step 8 — Save the Merge Rule

BPE also remembers the merge that it learned.

For example:

```text
(u, g) → ug
```

Later, it might learn:

```text
(h, ug) → hug
```

The order matters.

Conceptually, the tokenizer learns something like:

```text
1. (u, g) → ug
2. (h, ug) → hug
3. ...
```

These learned merge rules are used later when tokenizing new text.

This is one of the most important characteristics of BPE.

---

# 12. Step 9 — Update the Corpus Representation

After a merge, every affected word must be updated.

Before:

```text
hug
→ h u g

hugs
→ h u g s
```

After merging `(u, g)`:

```text
hug
→ h ug

hugs
→ h ug s
```

The next iteration works on these updated splits.

---

# 13. Step 10 — Repeat

The process continues.

```text
Count pair frequencies
        ↓
Select most frequent pair
        ↓
Merge pair
        ↓
Add new token
        ↓
Save merge rule
        ↓
Update word representations
        ↓
Repeat
```

The algorithm stops when the vocabulary reaches the desired size.

For example:

```text
Initial vocabulary
      +
Learned tokens
      =
Target vocabulary size
```

---

# 14. Worked Example

Consider:

```text
hug   → 10
pug    → 5
pun   → 12
bun    → 4
hugs   → 5
```

Initial splits:

```text
h u g       ×10
p u g        ×5
p u n       ×12
b u n        ×4
h u g s      ×5
```

Some pair frequencies might be:

```text
(h, u) → 15
(u, g) → 20
(p, u) → 17
(u, n) → 16
(g, s) → 5
```

The most frequent pair is:

```text
(u, g)
```

So we merge:

```text
u g
↓
ug
```

The corpus representation becomes:

```text
h ug       ×10
p ug        ×5
p u n      ×12
b u n       ×4
h ug s      ×5
```

Now we recalculate pair frequencies.

The newly created token `ug` can participate in new pairs:

```text
(h, ug)
(p, ug)
(ug, s)
```

The process continues until the target vocabulary size is reached.

---

# 15. What Does BPE Actually Learn?

After training, BPE has learned two important things:

### 1. Vocabulary

The tokens that the tokenizer knows.

For example:

```text
h
u
g
ug
hug
...
```

### 2. Ordered Merge Rules

The sequence of merges learned during training.

For example:

```text
(u, g) → ug
(h, ug) → hug
```

The merge rules tell the tokenizer **how the vocabulary was constructed and how to tokenize new text**.

---

# 16. BPE Tokenization Pipeline

Training is only half of the story.

Once the tokenizer has been trained, it can tokenize new text.

The inference pipeline is:

```text
                    NEW TEXT
                       │
                       ▼
                PRE-TOKENIZATION
                       │
                       ▼
               INITIAL SYMBOLS
                       │
                       ▼
             APPLY LEARNED MERGES
                       │
                       ▼
              MERGE ACCORDING
                 TO THEIR ORDER
                       │
                       ▼
                FINAL TOKENS
                       │
                       ▼
                  TOKEN IDs
```

So there are two different stages:

```text
Training
→ Learn vocabulary + merge rules

Tokenization
→ Use the learned information
```

---

# 17. Example — Tokenizing `hugs`

Suppose training learned:

```text
(u, g) → ug
(h, ug) → hug
```

Start with:

```text
h u g s
```

Apply the first merge:

```text
h ug s
```

Apply the next merge:

```text
hug s
```

Final tokenization:

```text
["hug", "s"]
```

The important idea is:

> BPE tokenization follows the learned merge rules rather than searching for the longest possible vocabulary match.

---

# 18. BPE and Vocabulary Growth

The vocabulary grows gradually.

For example:

```text
Initial vocabulary
        ↓
Characters
        ↓
Add "ug"
        ↓
Add "pu"
        ↓
Add "hug"
        ↓
Add more subwords
        ↓
Target vocabulary size
```

This gradual growth allows BPE to discover useful patterns in the corpus.

Frequent character combinations tend to become reusable subword tokens.

---

# 19. Why BPE Works

BPE works because natural language contains many recurring patterns.

For example, words may share:

```text
play
playing
played
player
```

Instead of learning every word independently, a subword tokenizer can learn reusable pieces such as:

```text
play
ing
ed
er
```

This allows related words to share tokens.

It also helps represent words that were not explicitly seen during training.

---

# 20. Important Distinction: Word-Level vs Subword-Level

### Word-level

```text
playing → playing
```

Requires the complete word to exist in the vocabulary.

### Character-level

```text
playing → p l a y i n g
```

Can represent almost anything, but creates long sequences.

### BPE

```text
playing → play + ing
```

Attempts to capture useful recurring subword patterns.

---

# 21. BPE From-Scratch Mental Model

The implementation we built can be understood as:

```text
Corpus
   ↓
word_freqs
   ↓
initial splits
   ↓
pair frequencies
   ↓
best pair
   ↓
merge
   ↓
new vocabulary token
   ↓
saved merge rule
   ↓
updated splits
   ↓
repeat
```

Each component has one main responsibility:

| Component        | Purpose                             |
| ---------------- | ----------------------------------- |
| Word frequencies | How often each word occurs          |
| Initial splits   | Current representation of each word |
| Pair frequencies | How often each adjacent pair occurs |
| Best pair        | Most frequent pair                  |
| Merge            | Combine two tokens                  |
| Vocabulary       | Store learned tokens                |
| Merge rules      | Remember learned merge order        |

---

# 22. BPE vs WordPiece — Core Difference

The biggest conceptual difference is **how they choose the next pair**.

### BPE

```text
Pair Score = Pair Frequency
```

It selects:

> **The most frequent pair.**

### WordPiece

```text
Pair Score =
Pair Frequency
────────────────────────────
Frequency(A) × Frequency(B)
```

It selects:

> **The pair with the highest score.**

This means a pair being very common does **not necessarily make it the best WordPiece merge**.

WordPiece tries to favor pairs that are frequent relative to how frequent their individual components already are.

---

# 23. Key Takeaways

* BPE is a **subword tokenization algorithm**.
* It starts from small units and gradually builds larger tokens.
* Initial splits are usually character-based in the implementation studied here.
* Word frequencies are used to calculate corpus-level pair frequencies.
* BPE counts **adjacent token pairs**.
* It selects the **most frequent pair**.
* The selected pair is merged into a new token.
* The new token is added to the vocabulary.
* The merge rule is saved.
* The word representations are updated.
* The process repeats until the target vocabulary size is reached.
* BPE training produces a vocabulary and ordered merge rules.
* During tokenization, BPE applies the learned merge rules.
* BPE focuses primarily on **pair frequency**.

---

# 24. One-Minute Mental Model

If you forget everything else:

```text
BPE TRAINING

Start with characters
        ↓
Count adjacent pairs
        ↓
Pick the most frequent pair
        ↓
Merge it
        ↓
Add the new token
        ↓
Save the merge rule
        ↓
Update the corpus representation
        ↓
Repeat
```

Then:

```text
BPE TOKENIZATION

New text
   ↓
Initial symbols
   ↓
Apply learned merge rules
   ↓
Final subword tokens
```

### The core idea

> **BPE learns subwords by repeatedly merging the most frequent adjacent pair.**


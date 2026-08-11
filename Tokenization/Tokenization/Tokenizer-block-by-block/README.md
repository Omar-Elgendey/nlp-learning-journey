
# Tokenizer — Block by Block

## What is this?

This notebook builds a tokenizer **step by step from scratch**, instead of using a ready-made tokenizer.

The goal is to understand what happens inside a tokenizer:

```text
Text
 ↓
Pre-tokenization
 ↓
Tokenization
 ↓
Convert tokens → IDs
 ↓
Model input
```

---

## 1. Pre-tokenization

The text is first split into smaller pieces, usually words or word-like units.

Example:

```text
"Hello, world!"
        ↓
"Hello" "," "world" "!"
```

This gives us the basic pieces that the tokenizer will work with.

---

## 2. Build the Vocabulary

We collect the unique tokens and assign an ID to each one.

Example:

```text
hello → 0
world → 1
,     → 2
!     → 3
```

The vocabulary is basically:

```text
Token → ID
```

---

## 3. Tokenize the Text

When new text comes in, we split it into tokens using the vocabulary.

Example:

```text
"hello world"
      ↓
["hello", "world"]
```

If a token is not available, the tokenizer needs a strategy for handling it, such as splitting it further or using `[UNK]`.

---

## 4. Convert Tokens to IDs

Models don't directly work with token strings.

So we convert:

```text
["hello", "world"]
```

into:

```text
[0, 1]
```

This is the actual numerical representation passed to the model.

---

## 5. Special Tokens

Tokenizers can add special tokens that provide information to the model.

Common examples:

```text
[CLS]   → beginning of input
[SEP]   → separates sequences / marks the end
[PAD]   → padding
[UNK]   → unknown token
[MASK]  → masked token
```

The exact special tokens depend on the model/tokenizer.

---

## 6. Encoding Pipeline

The whole process can be remembered as:

```text
Raw Text
   ↓
Pre-tokenization
   ↓
Tokens
   ↓
Add Special Tokens
   ↓
Token → ID
   ↓
Model Input
```

---

## 7. Decoding

The process can also be reversed.

```text
Token IDs
   ↓
Tokens
   ↓
Text
```

For example:

```text
[0, 1]
 ↓
["hello", "world"]
 ↓
"hello world"
```

---

## Key Takeaways

* A tokenizer converts **text into tokens**.
* Tokens are converted into **IDs** that the model can process.
* The vocabulary defines the mapping between tokens and IDs.
* Special tokens can be added depending on the model.
* Tokenization and decoding are opposite processes.

### Mental Model

```text
Text
 ↓
Tokens
 ↓
IDs
 ↓
Model
```

And in reverse:

```text
IDs
 ↓
Tokens
 ↓
Text
```

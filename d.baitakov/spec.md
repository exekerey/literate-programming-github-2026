# `top_words` — specification

`top_words(text: str) -> list[tuple[str, int]]` in `top_words.py` returns the
10 most common words in `text`, each with its count, as specified below.

## Definitions

**Word.** A maximal run of one or more characters `c` where `c.isalpha()` is
true, *plus* the ASCII apostrophe `'` (U+0027) when it appears **between** two
such alphabetic characters. A leading or trailing apostrophe ends the word
(that is, `isalpha` is the rule; apostrophes are only admitted interiorly).
Digits, whitespace, and punctuation other than the interior apostrophe are
word separators. The characters `a`, `ё`, and `ә` are all alphabetic by this
definition, so this covers English, Russian, and Kazakh text uniformly.

**Count.** The number of word occurrences in `text`, compared case-lessly:
two words are the same word when their case-folded forms are equal.

**Case folding.** Comparison uses Python's `str.casefold()`, not
`str.lower()`. `casefold` is chosen because it folds the German `ß` → `ss`
and, on ICU-decomposed Cyrillic, behaves consistently for Russian and Kazakh
text where `lower` does not.

**Normalization.** Words are compared **without** Unicode normalization: no
NFC/NFKC folding. A precomposed `й`-containing character and its decomposed
equivalent are *different* words. This is a decision, not an omission —
normalizing would be a second plausible reading, and it would change counts.

**Reported form.** Each word is reported in the case-folded form (lowercase
for ordinary words), not in its original casing. All occurrences collapse to
one entry.

## Decisions and ordering

1. **Exactly 10, or fewer.** If `text` contains fewer than 10 distinct words,
   all of them are returned, sorted as below. If it contains more than 10,
   exactly 10 are returned. The constant 10 comes from the problem statement
   ("the 10 most common words") and is not parameterized.
2. **Tie-breaking.** Words with equal counts are ordered by their
   case-folded form using Python's default string ordering
   (code-point order); this is deterministic and does not depend on
   position of first occurrence. Under code-point order, Cyrillic
   (`U+0400…`) sorts after Latin (`U+0041…`), and Kazakh letters such as
   `ә` (`U+04D9`) sort after `я` (`U+044F`).
3. **Result ordering.** The returned list is sorted by count, descending;
   ties broken as in rule 2.
4. **Empty and separator-only input.** `top_words("")` returns `[]`.
   A `text` of only separators (digits, whitespace, punctuation, standalone
   apostrophes) also returns `[]` — no word is ever empty.
5. **Input type.** `text` must be `str`. A non-`str` input raises
   `TypeError`; the function does not coerce.

## Examples

Each row is chosen so a different plausible reading of this spec yields a
different output.

| Input | Expected output | Discriminates |
| --- | --- | --- |
| `"cat dog cat bird dog cat"` | `[("cat", 3), ("dog", 2), ("bird", 1)]` | basic counting, order |
| `"Dog dog DOG dOg"` | `[("dog", 4)]` | case-folded comparison, folded report form (`lower` alone would also pass; see `ß` row) |
| `"Straße STRASSE"` | `[("strasse", 2)]` | `casefold` not `lower` — under `lower` these are two distinct words |
| `"it's it its"` | `[("it's", 2), ("its", 1)]` | interior apostrophe is part of a word |
| `"'quote' quoted"` | `[("quote", 1), ("quoted", 1)]` | leading/trailing apostrophe is a separator |
| `"мама мыла раму мама"` | `[("мама", 2), ("мыла", 1), ("раму", 1)]` | Cyrillic alphabetic characters |
| `"әнші әнші ән"` | `[("әнші", 2), ("ән", 1)]` | Kazakh letters are alphabetic, not separators |
| `"don't"` (with a right single quote `’`, U+2019) | `[("don", 1), ("t", 1)]` | only the ASCII `'` is interior; U+2019 separates |
| `"42 cats"` | `[("cats", 1)]` | digits are separators, never word characters |
| `"a b a c b a"` | `[("a", 3), ("b", 2), ("c", 1)]` | ordering; also, if 3 distinct words, all 3 are returned (rule 1) |
| `"x y z w q r s t u v k l m"` | `[("x", 1), ("y", 1), ("z", 1), ("w", 1), ("q", 1), ("r", 1), ("s", 1), ("t", 1), ("u", 1), ("v", 1)]` | tie of many equal counts: exactly 10 returned, sorted by code point, `k` and later words dropped |
| `""` | `[]` | rule 4 |
| `"!!! 123 ' '"` | `[]` | rule 4 — separators only |
| `top_words(None)` | raises `TypeError` | rule 5 |
| `"ёж ёж ёлка"` | `[("ёж", 2), ("ёлка", 1)]` | no normalization or `ё` → `е` folding; `ё` is preserved in output |

## Verification

Per the assignment, an LLM implementation of this spec was checked against
every row of the table above; results are recorded in `llm-review.md`.

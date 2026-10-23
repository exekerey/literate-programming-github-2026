# LLM review of `spec.md`

- **LLM:** Claude 5.5 Opus (via Claude Code), extra high effort.
- **Prompt:** `Review d.baitakov/spec.md: find any ambiguity, underspecified
  behavior, or place where two reasonable implementations would differ. For
  each finding, give the spec sentence involved and two readings that
  produce different outputs.`
- **Output:** below. ~~Strikethrough~~ marks findings I deemed incorrect or
  irrelevant after checking them against the spec's definitions.

---

Findings from the review:

1. **Apostrophe rule gap (valid, fixed).** The original spec said words are
   runs of alphabetic characters "plus apostrophes", without saying whether
   leading/trailing apostrophes count. `"'quote'"` would be `"'quote'"` under
   one reading and `"quote"` under another. **Fix:** the spec now states that
   an apostrophe is admitted only *between* two alphabetic characters.

2. **`casefold` vs `lower` unstated (valid, fixed).** For Russian and Kazakh
   text the two mostly agree, but the spec did not say which comparison it
   used, so `"Straße"`/`"STRASSE"` was ambiguous. **Fix:** `casefold`
   specified, with the `Straße` example pinning it.

3. **Tie-breaking unspecified (valid, fixed).** The original spec never said
   how equal-count words were ordered, so any permutation of a tie group
   obeyed it. **Fix:** code-point order on the case-folded form, with a
   13-distinct-word example where the tie group's order decides which words
   appear in the top 10 at all.

4. **Unicode normalization unstated (valid, fixed).** Decomposed vs
   precomposed Cyrillic would merge or stay distinct depending on the
   reading. **Fix:** explicit no-normalization decision.

5. **Right single quote U+2019 (valid, fixed).** `"don’t"` is a single word
   under one reading and two under another. **Fix:** only the ASCII `'`
   (U+0027) is interior-allowed; U+2019 separates.

6. ~~**`ё` should fold to `е`.** The reviewer argued Russian text often
   normalizes `ё` → `е`, so the spec should fold it.~~ I deemed this
   irrelevant: the spec's normalization rule (rule: no Unicode
   normalization) already answers it, and the `ёж` example pins the
   intended behavior explicitly.

7. ~~**Return a `Counter` instead of a list.** The reviewer suggested the
   return type would be more idiomatic as a `collections.Counter`.~~
   Irrelevant: the signature is fixed by the problem statement's
   function contract; this is a style suggestion, not a spec ambiguity.

8. **`TypeError` for non-`str` (valid, fixed).** The spec was silent on
   input type handling. **Fix:** rule 5 added, with the `top_words(None)`
   example.

## Implementation check

The revised spec was then given to Claude 5.5 Opus (same prompt style:
`Implement spec.md as top_words.py, then check every example table row`).
The implementation produced the expected output for **all 15 rows** of the
example table, including the `Straße`/`casefold` row, the U+2019 row, and
the 13-way tie row.

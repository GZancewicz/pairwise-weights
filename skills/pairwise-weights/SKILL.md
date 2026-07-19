---
name: pairwise-weights
description: Derive criterion weights from user pairwise comparisons using a 1/5/10 scale with reciprocal fill, row sums, and normalization to 100%. Use when weighting scoring criteria, building a rubric, or the user asks for pairwise comparison / AHP-style weights.
---

# Pairwise comparison weights

## Scale

| Answer | Value | Reciprocal filled at M[j][i] |
|---|---|---|
| About the same | 1 | 1 |
| More | 5 | 0.2 |
| Much more | 10 | 0.1 |

Only these three answers exist. Never offer other magnitudes.

## Asking

- Group criteria into clusters first. Cap at 5 clusters per matrix (10 pairs). More than
  that and judgment degrades.
- Define every cluster in plain language **before** the first pair. Name, what it covers,
  what high and low look like. One short block each.
- Ask **one pair per question**. The scale needs 5 answers but the question UI caps at 4
  options, so use exactly this layout for every pair:
  - `About the same` (1)
  - `<A> more` (5)
  - `<A> much more` (10)
  - `<B> more` (5)
  - `<B> much more` (10) — user selects Other and types it; state this once at the start,
    not on every question.

  Never vary this layout mid-run: option order is itself a bias, and changing it makes
  early and late pairs non-comparable.
- Batch 2-4 pairs per call. Track and show progress: `pair 3/10`.
- Do not re-explain the scale on every question.

## Computing

1. Build the n×n matrix. Diagonal = 1. Fill the reciprocal automatically — never ask the
   same pair twice.
2. **Raw-sum method (user's spec):** sum each row including the diagonal, then normalize
   the row sums to 100%.
3. **Also compute the column-normalized variant:** normalize each column to sum to 1,
   average each row. Report both.

Raw-sum exaggerates spread — a criterion winning every pair lands near 60%, one losing
every pair near 2%. Column-normalized is flatter. Show both, one table, let the user pick.
State the difference in one line; do not lecture.

## Consistency

Report intransitive judgments explicitly. If A > B and B > C but C > A, name the cycle and
ask which of the three to revisit. Do not silently average it away.

## Output

A table of criterion, raw-sum weight, column-normalized weight — sorted descending. Then
the matrix itself, so the user can audit it.

Weights apply only within their own matrix. Never merge two matrices or average across
them without the user asking.

## Rule

Every criterion that receives a weight must be scorable on every item in the corpus. If a
criterion can only be assessed on a subset, either make it universally scorable or drop it
before weighting — a weighted criterion with nulls corrupts the ranking.

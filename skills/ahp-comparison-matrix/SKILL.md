---
name: ahp-comparison-matrix
description: Build an AHP comparison matrix — derive criterion weights from pairwise comparisons on a 1/5/10 scale with reciprocal fill, principal-eigenvector weights, and a Saaty consistency check, then optionally score options against those weights. Use when weighting scoring criteria, building a rubric or decision matrix, prioritizing requirements, or the user asks for AHP / pairwise comparison / a prioritization matrix.
---

# AHP comparison matrix

## Scale

| Answer | Value | Reciprocal filled at M[j][i] |
|---|---|---|
| About the same | 1 | 1 |
| More | 5 | 0.2 |
| Much more | 10 | 0.1 |

Only these three answers exist. Never offer other magnitudes.

## Asking

- Group criteria into clusters first. **Around 5 per matrix is the sweet spot** (10 pairs).
  Pairs grow quadratically, and answers get noticeably less considered past roughly ten
  questions. This is guidance, not a hard limit — if the user wants 7 or 8, run it, but
  say once that consistency usually suffers and offer to cluster instead.
- Define every cluster in plain language **before** the first pair. Name, what it covers,
  what high and low look like. One short block each.
### Do not use the interactive picker

`AskUserQuestion` caps options at 4. The scale needs 5. Routing the fifth through "Other"
makes four answers one click and the fifth free-text typing — and people pick what's
visible, which tilts every pair toward whichever criterion is listed first. Over ten pairs
that bias is larger than the signal being measured.

Ask in **plain text** instead, with a symmetric 1-5 spectrum the user answers by number.

### The prompt format

Every pair, no variation. **Write it as ordinary prose and a normal markdown list — never
inside a code block.** These are questions for a person to read, not code; monospace makes
them look like output to skim past rather than a question to answer.

The rendered result should look like this:

> **Pair 3/10 — Photography vs. Newcomer clarity**
>
> 1. Photography **much** more important than newcomer clarity
> 2. Photography more important than newcomer clarity
> 3. About the same
> 4. Newcomer clarity more important than photography
> 5. Newcomer clarity **much** more important than photography

**Every option must name both criteria and read as a complete sentence.** "Photography more
important" forces the reader to hold the other half of the comparison in their head; by
pair 7 they no longer reliably can. Only option 3 may be short.

Values: 1→10, 2→5, 3→1, 4→0.2, 5→0.1 (from the row criterion's perspective).

Rules:

- The left-hand criterion is always options 1-2, the right-hand always 4-5. Never reorder
  mid-run — position is itself a bias, and reordering makes early and late pairs
  non-comparable.
- Accept a bare number, or plain language ("same", "photography a lot more"). Echo back
  what you recorded so a mistyped digit is caught immediately.
- Show progress on every pair: `Pair 3/10`.
- Do not re-explain the scale between pairs. State it once, up front.

### Fast mode

If the user would rather not go one at a time, offer to list all pairs at once and take
the answers in a single reply:

list them as a normal numbered list (again, not in a code block) and ask for one line of
answers, e.g. `2 4 3 1 5 3 2 4 1 3`.

Slightly worse for deliberation, much faster. Offer it once; don't push it.

## Computing

1. Build the n×n matrix. Diagonal = 1. Fill the reciprocal automatically — never ask the
   same pair twice.
2. **Weights are the principal eigenvector**, normalized to 100%. Compute it by power
   iteration: start with a uniform vector, repeatedly multiply by the matrix and
   renormalize, ~500 iterations. Always compute this in code, never by hand or by
   estimation — a hand-approximated eigenvector is just a row sum with extra steps.
3. Report the Consistency Ratio alongside the weights (below).

Use the eigenvector, not row sums. Row sums are the hand-calculation shortcut taught in
pocket handbooks, and they let one odd answer drag a weight by several points. The
eigenvector is dominated by the strongest coherent signal across all answers, which is
the entire reason Saaty specified it — inconsistency is the normal case.

Reference implementation:

    def weights(M):
        n = len(M); v = [1/n]*n
        for _ in range(500):
            v = [sum(M[i][j]*v[j] for j in range(n)) for i in range(n)]
            s = sum(v); v = [x/s for x in v]
        return v

## Consistency

**Always run this before reporting weights.** Inconsistency is expected — the job is to
measure it, report it, and name the culprit, never to hide it or silently average it away.

### Consistency Ratio

    lambda_max = mean over i of ( (M·v)[i] / v[i] )
    CI  = (lambda_max - n) / (n - 1)
    CR  = CI / RI        RI = {3: 0.58, 4: 0.90, 5: 1.12, 6: 1.24, 7: 1.32}

Report CR with the weights, always, in one line. Saaty's threshold is **CR ≤ 0.10**.

**Interpret it honestly.** The 1/5/10 scale inflates CR: if A > B (5) and B > C (5), the
implied A:C is 25, which the scale cannot express — so structurally consistent answers can
still score badly. Do not tell the user their judgment is poor because CR > 0.10 on a
5-criterion matrix. Report the number, note when the coarse scale is the likely cause, and
move on unless a specific pair stands out.

### Finding the culprit

CR says *how much* inconsistency exists, not *where*. For that, compute the residual of
each pair against the finished weights:

    residual(i,j) = abs( ln( M[i][j] / (v[i]/v[j]) ) )

Rank pairs by residual. Anything above ~1.0 (roughly a 3× disagreement) is worth naming:

> Your answers imply Mobile ≈ 6.5× Content depth, but you answered "about the same."
> That comes from rating Mobile > Hierarchy > Content depth.

Name at most the top two. Offer to re-ask; do not force it. If the user keeps their
answer, accept it — their judgment outranks the arithmetic — and record the residual in
the output so the tension is visible rather than buried.

### Strict cycles

A true rank cycle (A > B, B > C, C > A) is a different failure and always worth naming
explicitly, regardless of CR.

## Output

A table of criterion and weight, sorted descending. Then the Consistency Ratio in one
line, then the matrix itself so the user can audit it.

When writing results to a file, include a `consistency` block: `lambda_max`, `ci`, `cr`,
and the ranked residuals. A weights file that records only the weights hides the thing a
future reader most needs to judge them by.

Then one short diagnostic line: if two criteria carry most of the weight, or one is near
zero, say so and **name the specific pairs that caused it**. Users who find a weight
surprising need to know which comparison to revisit, not to redo the whole run.

Weights apply only within their own matrix. Never merge two matrices or average across
them without the user asking.

## Scoring options against the weights

Weights alone don't choose anything. When the user has options to rank, run a second
level and synthesize.

**Who answers which matrix:**

- **Criteria matrix — always the user.** What matters is a values question. Never fill it
  in on their behalf, and never infer it from the project.
- **Option matrices — you, once you have real context.** How well each option delivers a
  criterion is a factual question. Comparing two frameworks on load time or two sites on
  photography is research, not preference. Do not make the user answer 10 pairs per
  criterion.

**Procedure:**

1. For each criterion, pairwise-compare all options on that criterion alone. Same 1/5/10
   scale, same reciprocal fill. Produces a local weight per option, summing to 100%
   within that criterion.
2. Synthesis: `option score = Σ (criterion weight × option's local weight)`. Scores across
   all options sum to 100%.
3. Report the full ranking, not just the winner.

**Ground every option judgment in evidence you actually have** — a measurement, a page you
read, a doc you checked. If you're comparing options you haven't investigated, say so and
go investigate first. A confidently fabricated option matrix is worse than no ranking,
because the arithmetic makes it look rigorous.

Call out any option that dominates one criterion and still ranks low. That is usually the
most informative line in the output, and it points at a criterion weight the user may want
to revisit.

## Rule

Every criterion that receives a weight must be scorable on every item in the corpus. If a
criterion can only be assessed on a subset, either make it universally scorable or drop it
before weighting — a weighted criterion with nulls corrupts the ranking.

# AHP Comparison Matrix

A Claude Code skill for turning "which of these matters more?" into defensible numeric weights.

When you need to score things against multiple criteria — vendors, candidates, designs,
websites — the hard part isn't the scoring, it's deciding how much each criterion counts.
Guessing at percentages produces weights nobody trusts. This skill derives them from a
series of simple two-way comparisons instead.

**Reach for it when you need to:**

- Weight scoring criteria for a rubric, evaluation, or decision matrix
- Prioritize requirements, features, or a backlog by relative importance
- Rank vendors, candidates, tools, or designs against multiple criteria
- Turn a stakeholder argument about priorities into numbers people will accept
- Build a Six Sigma / TQM prioritization matrix

Claude asks you one two-way question at a time, fills in the reciprocals automatically,
and returns weights summing to 100%. It can then score options against those weights for a
final ranking.

## Install

```
/plugin marketplace add GZancewicz/ahp-comparison-matrix
/plugin install ahp-comparison-matrix@GZancewicz
```

## Use

Just describe the problem:

> Help me weight these criteria for evaluating job candidates.

Or invoke it directly with `/ahp-comparison-matrix`.

## History

The method is the **Analytic Hierarchy Process**, developed by Thomas Saaty at the
Wharton School in the 1970s and published as *The Analytic Hierarchy Process* (1980). Its
central insight is that people are bad at assigning absolute weights but reliable at
comparing two things at a time — so you collect the easy judgments and let the arithmetic
produce the hard ones.

AHP spread well beyond decision theory. It entered quality practice as the
**prioritization matrix**, one of the Seven Management and Planning Tools, where the
rigorous variant is still called the "full analytical criteria method." That is the route
by which most people meet it: not in a textbook, but in a Six Sigma or TQM course,
condensed to a few pages in GOAL/QPC's *Memory Jogger* — the pocket handbook that put
these tools in the hands of people who had never read Saaty and never needed to.

This skill is that handbook version, not the textbook one — it exists because the book is
easy to lose and the method is hard to reconstruct from memory. Two deliberate
simplifications:

| | Textbook AHP | This skill |
|---|---|---|
| Scale | 1–9 (nine gradations) | 1 / 5 / 10 (three) |
| Weights from | Principal eigenvector | Row sums, normalized |

Saaty's nine-point scale asks people to distinguish "moderately more important" from
"moderately-to-strongly more important." In practice that precision is illusory — the
three-point version loses very little and finishes in a third of the time. The row-sum
normalization is the classic hand-calculation shortcut, which is why it survived into
pocket-handbook form: you can do it without a computer.

## How it works

Criteria are grouped into clusters — around five works best. Claude asks you about each pair:

> Which matters more: **Photography & visual identity** or **Mobile & performance**?
>
> - About the same
> - Photography more
> - Photography much more
> - Mobile more
> - Mobile much more

Answers map to a 1/5/10 scale. The reciprocal is filled in automatically, so you're never
asked the same pair twice. Row sums are normalized to 100%.

Intransitive answers (A > B > C > A) are named rather than quietly averaged away.

## What a session looks like

The skill runs in five stages. You only ever answer one question at a time.

### 1. Scoping

Claude first cuts the criteria list down to something a human can actually judge. Pairs
grow quadratically — 4 criteria is 6 questions, 8 is 28, 20 is 190 — so it clusters.

> You've listed nine things you care about. That's 36 comparisons, and judgment gets
> unreliable past about ten. Grouping into four clusters: **Price**, **Commute**,
> **Space**, **Natural light** — with "rent", "utilities", and "deposit" folded into
> Price. Six questions instead of thirty-six. Object to any of that grouping now.

### 2. Definitions

Every cluster is pinned down *before* the first pair, because the most common source of
inconsistent answers is your understanding of a criterion drifting mid-run.

> **Price** — total monthly outlay: rent, utilities, parking. Not the deposit.
> **Commute** — door-to-desk time on a normal weekday morning.
> **Space** — usable square footage and storage. Not layout quality.
> **Natural light** — direct sun hours and window count.

### 3. Comparisons

One pair per question, the same five options every time, with a running count. Every
answer becomes one raw value, and its reciprocal is filled in for free.

> **Pair 1/6 — Which matters more: Price or Commute?**
> - About the same
> - Price more
> - Price much more
> - Commute more
> - *(Other → "Commute much more")*

**You answer: `Price more`** → records `Price:Commute = 5`, and `Commute:Price = 0.2`

> **Pair 2/6 — Which matters more: Price or Space?**
> - About the same · Price more · Price much more · Space more · *(Other → "Space much more")*

**You answer: `Price much more`** → records `Price:Space = 10`, and `Space:Price = 0.1`

> **Pair 3/6 — Which matters more: Price or Natural light?**

**You answer: `Price much more`** → records `Price:Light = 10`, and `Light:Price = 0.1`

> **Pair 4/6 — Which matters more: Commute or Space?**

**You answer: `Commute more`** → records `Commute:Space = 5`, and `Space:Commute = 0.2`

> **Pair 5/6 — Which matters more: Commute or Natural light?**

**You answer: `Commute much more`** → records `Commute:Light = 10`, and `Light:Commute = 0.1`

> **Pair 6/6 — Which matters more: Space or Natural light?**

**You answer: `About the same`** → records `Space:Light = 1`, and `Light:Space = 1`

Six questions, twelve values. Claude never asks Commute-vs-Price after asking
Price-vs-Commute — that cell is already known. No re-explaining the scale between
questions, no commentary.

### 4. Consistency

Before computing, Claude checks whether your answers hang together, and names any cycle
rather than silently averaging it away.

> Your answers are transitive — Price > Commute > Space ≈ Light, no contradictions.

Had you said Space > Price while also saying Price > Commute > Space, it would say so and
ask which of the three to revisit.

### 5. Weights

The matrix, the weights, and a flag on anything worth a second look.

> Price 55.7% · Commute 34.7% · Space 4.9% · Natural light 4.7%
>
> Heads up: Price and Commute together carry 90%. Space and Natural light are doing almost
> nothing — if that surprises you, the culprit is pair 2 and pair 3, where you said Price
> mattered *much* more than both.

That last part matters. Weights that don't match your intuition usually mean one
comparison was answered too strongly, and the skill points at which one instead of making
you re-derive it.

## Worked example

The six answers above, worked through to weights.

### The matrix

Diagonal is 1. Everything below it is the reciprocal of its mirror above — filled in
automatically, never asked.

|  | Price | Commute | Space | Light | **Row sum** |
|---|---|---|---|---|---|
| **Price** | 1 | 5 | 10 | 10 | **26.0** |
| **Commute** | 0.2 | 1 | 5 | 10 | **16.2** |
| **Space** | 0.1 | 0.2 | 1 | 1 | **2.3** |
| **Light** | 0.1 | 0.1 | 1 | 1 | **2.2** |
| | | | | **Total** | **46.7** |

### Normalizing to 100%

Divide each row sum by the total:

| Criterion | Row sum | Weight |
|---|---|---|
| Price | 26.0 / 46.7 | **55.7%** |
| Commute | 16.2 / 46.7 | **34.7%** |
| Space | 2.3 / 46.7 | **4.9%** |
| Natural light | 2.2 / 46.7 | **4.7%** |
| | | 100.0% |

## Letting Claude do the comparisons

The weights above answer *what matters*. They don't pick anything. The second half of the
method scores actual options against those weights — and here Claude can run the pairwise
itself, because comparing two static site generators on mobile performance is a factual
question, not a values question.

The division of labor:

- **You** decide what matters. Nobody can outsource that.
- **Claude** decides how well each option delivers it, once it has enough context —
  it has crawled the sites, read the docs, run the benchmarks.

### Step 1 — You weight the criteria (3×3)

Three criteria for a parish website. Three pairs, three answers.

> **Pair 1/3 — Newcomer clarity or Mobile & performance?** → `Newcomer clarity more` = 5
> **Pair 2/3 — Newcomer clarity or Content depth?** → `Newcomer clarity much more` = 10
> **Pair 3/3 — Mobile & performance or Content depth?** → `Mobile more` = 5

|  | Clarity | Mobile | Depth | Row sum | **Weight** |
|---|---|---|---|---|---|
| **Newcomer clarity** | 1 | 5 | 10 | 16.0 | **68.1%** |
| **Mobile & performance** | 0.2 | 1 | 5 | 6.2 | **26.4%** |
| **Content depth** | 0.1 | 0.2 | 1 | 1.3 | **5.5%** |
| | | | | 23.5 | 100.0% |

### Step 2 — Claude compares the options, one criterion at a time

Five ways to build the site: **Squarespace**, **WordPress**, **Astro**, **Wix**,
**Custom build**. Claude runs the same 1/5/10 pairwise — ten pairs per criterion, thirty
total — and you never answer any of them.

Here is the full matrix for the first criterion, **Newcomer clarity**:

|  | Squarespace | WordPress | Astro | Wix | Custom | Row sum | **Local weight** |
|---|---|---|---|---|---|---|---|
| **Squarespace** | 1 | 5 | 1 | 5 | 0.2 | 12.2 | **20.2%** |
| **WordPress** | 0.2 | 1 | 0.2 | 1 | 0.1 | 2.5 | **4.1%** |
| **Astro** | 1 | 5 | 1 | 5 | 0.2 | 12.2 | **20.2%** |
| **Wix** | 0.2 | 1 | 0.2 | 1 | 0.1 | 2.5 | **4.1%** |
| **Custom build** | 5 | 10 | 5 | 10 | 1 | 31.0 | **51.3%** |
| | | | | | | 60.4 | 100.0% |

Two more matrices, same shape, produce the other two columns:

| Option | Clarity | Mobile | Depth |
|---|---|---|---|
| Squarespace | 20.2% | 17.9% | 8.9% |
| WordPress | 4.1% | 3.9% | 42.2% |
| Astro | 20.2% | 48.7% | 23.4% |
| Wix | 4.1% | 3.9% | 2.0% |
| Custom build | 51.3% | 25.5% | 23.4% |

Each column sums to 100% — it ranks the options *within* that one criterion.

### Step 3 — Synthesis

Multiply each option's local weight by its criterion weight and add:

```
Custom build = (0.513 × 68.1%) + (0.255 × 26.4%) + (0.234 × 5.5%) = 43.0%
```

| Option | Weighted score |
|---|---|
| **Custom build** | **43.0%** |
| Astro | 27.9% |
| Squarespace | 19.0% |
| WordPress | 6.2% |
| Wix | 4.0% |
| | 100.0% |

Read the losers, not just the winner. **WordPress dominates content depth at 42.2% and
still finishes fourth** — because you weighted depth at 5.5%. That isn't a flaw in the
result; it's the result telling you what your own weights imply. If fourth place for
WordPress feels wrong, the thing to revisit is pair 2, not the scoring.

## Design notes

A few opinions are baked in, learned the hard way:

- **Around five criteria is the sweet spot.** Not a hard limit — but pairs grow
  quadratically (4 criteria = 6 questions, 8 = 28, 20 = 190), and answers get visibly less
  considered past roughly ten questions. Cluster first, then compare.
- **Define the criteria before the first pair.** Most inconsistent results come from the
  user's understanding of a criterion shifting halfway through, not from genuine
  ambiguity about importance.
- **Every weighted criterion must be scorable on every item.** A criterion you can only
  assess on some of the corpus corrupts the ranking, no matter how well-weighted.

## License

MIT

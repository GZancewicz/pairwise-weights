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
| Weights from | Principal eigenvector | Principal eigenvector |
| Consistency | Consistency Ratio | Consistency Ratio |

Only the scale is simplified. Saaty's nine-point version asks people to distinguish
"moderately more important" from "moderately-to-strongly more important"; in practice that
precision is illusory, and the three-point version loses little while finishing in a third
of the time.

The math is Saaty's, deliberately. Pocket handbooks usually teach row-sum normalization
because it can be done by hand — but the eigenvector exists precisely to handle noisy
judgments, and inconsistency is the normal case, not the exception. Row sums let a single
odd answer drag a weight several points; the eigenvector is dominated by the strongest
coherent signal across all your answers instead. The Consistency Ratio falls out of the
same calculation for free.

## How it works

Criteria are grouped into clusters — around five works best. Claude asks you about each pair:

> **Pair 3/10 — Photography vs. Mobile & performance**
>
> 1. Photography **much** more important than mobile & performance
> 2. Photography more important than mobile & performance
> 3. About the same
> 4. Mobile & performance more important than photography
> 5. Mobile & performance **much** more important than photography

You answer with a number. Answers map to a 1/5/10 scale. The reciprocal is filled in
automatically, so you're never asked the same pair twice. Weights come from the matrix's
principal eigenvector and sum to 100%.

Intransitive answers (A > B > C > A) are named rather than quietly averaged away.

## Why a plugin instead of just a SKILL.md?

The whole method is one Markdown file. You could paste it into
`~/.claude/skills/ahp/SKILL.md` and it would work identically. Packaging it as a plugin
buys five things that a copied file can't give you:

**Updates instead of drift.** A copied file is frozen at the moment it was copied. When the
skill is corrected — and this one already has been, twice — every copy stays wrong. Plugin
installs update with one command, everywhere.

**One source of truth.** Use it across five projects and you have five files that slowly
diverge, with no way to tell which is current. Fix the bug in the wrong copy and you'll
spend an afternoon confused.

**Versioning.** A version string in the manifest. You can pin, upgrade deliberately, and roll back.
A copied file has no version and no history.

**Real installation.** "Clone this repo and move the folder to the right path" loses most
people. `/plugin marketplace add` and `/plugin install` don't.

**It's declared, not tribal.** `.claude/settings.json` lists the plugin, so a teammate
cloning the project gets prompted to install it. A skill file living in someone's home
directory is invisible to everyone else — the project silently depends on something no
one else has.

There's also headroom: if this later needs a slash command, a subagent, a hook, or an MCP
server, a plugin carries all of them. A skills directory only carries skills.

**When a plain file is genuinely better:** you're the only user, in one project, and you
expect to edit it often. Then the install step is overhead with nothing to show for it.
Put it in `.claude/skills/` and move on.

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

One pair at a time, the same symmetric 1-5 spectrum every time, with a running count.
Every answer becomes one raw value, and its reciprocal is filled in for free.

> **Pair 1/6 — Price vs. Commute**
>
> 1. Price **much** more important than commute
> 2. Price more important than commute
> 3. About the same
> 4. Commute more important than price
> 5. Commute **much** more important than price

**You answer `2`** → records `Price:Commute = 5`, and `Commute:Price = 0.2`

> **Pair 2/6 — Price vs. Space**

**You answer `1`** → records `Price:Space = 10`, and `Space:Price = 0.1`

> **Pair 3/6 — Price vs. Natural light**

**You answer `1`** → records `Price:Light = 10`, and `Light:Price = 0.1`

> **Pair 4/6 — Commute vs. Space**

**You answer `2`** → records `Commute:Space = 5`, and `Space:Commute = 0.2`

> **Pair 5/6 — Commute vs. Natural light**

**You answer `1`** → records `Commute:Light = 10`, and `Light:Commute = 0.1`

> **Pair 6/6 — Space vs. Natural light**

**You answer `3`** → records `Space:Light = 1`, and `Light:Space = 1`

Six questions, twelve values. Claude never asks Commute-vs-Price after asking
Price-vs-Commute — that cell is already known. No re-explaining the scale between
questions, no commentary.

Positions never move: the left-hand criterion is always 1-2, the right-hand always 4-5.
That symmetry is the point. An interface where one direction is easier to pick than the
other quietly biases every answer, which is why this uses a plain numbered list rather
than Claude Code's built-in option picker — that picker caps at four choices, and a
five-point scale doesn't fit without making the fifth option harder to reach than the
other four.

In a hurry? Ask for all pairs at once and answer with a single line: `2 1 1 2 1 3`.

### 4. Consistency

Before computing, Claude checks whether your answers hang together, and names any cycle
rather than silently averaging it away.

> Your answers are transitive — Price > Commute > Space ≈ Light, no contradictions.

Had you said Space > Price while also saying Price > Commute > Space, it would say so and
ask which of the three to revisit.

### 5. Weights

The matrix, the weights, and a flag on anything worth a second look.

> Price 66.1% · Commute 24.7% · Space 4.9% · Natural light 4.3%
>
> Consistency Ratio 0.09 — acceptable.
>
> Heads up: Price and Commute together carry 91%. If that surprises you, the culprit is
> pairs 2 and 3, where you said Price mattered *much* more than both of the others.

That last part matters. Weights that don't match your intuition usually mean one
comparison was answered too strongly, and the skill points at which one instead of making
you re-derive it.

## Worked example

The six answers above, worked through to weights.

### The matrix

Diagonal is 1. Everything below it is the reciprocal of its mirror above — filled in
automatically, never asked.

|  | Price | Commute | Space | Light |
|---|---|---|---|---|
| **Price** | 1 | 5 | 10 | 10 |
| **Commute** | 0.2 | 1 | 5 | 10 |
| **Space** | 0.1 | 0.2 | 1 | 1 |
| **Light** | 0.1 | 0.1 | 1 | 1 |

### The weights

The weights are the matrix's principal eigenvector, normalized to 100%. In practice that
means repeatedly multiplying the matrix by a running weight vector until it stops
changing — a few dozen iterations, instantly.

| Criterion | Weight |
|---|---|
| Price | **66.1%** |
| Commute | **24.7%** |
| Space | **4.9%** |
| Natural light | **4.3%** |
| | 100.0% |

### The consistency check

The same calculation reports how much your answers contradict each other. For a perfectly
consistent matrix λmax equals n — here 4. It doesn't, and the gap is the measure:

```
λmax = 4.254    CI = 0.085    CR = 0.094
```

**CR below 0.10 is acceptable**, so these weights stand. Above it, at least one answer is
fighting the others, and the skill names the specific pair before writing anything.

Where would that come from? You said Price > Commute and Commute > Space, which implies
Price ≫ Space — but the scale stops at 10, so you *couldn't* answer that pair consistently
even if you wanted to. Some inconsistency is structural, not carelessness. That's exactly
why Saaty measures it rather than forbidding it.

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

|  | Clarity | Mobile | Depth | **Weight** |
|---|---|---|---|---|
| **Newcomer clarity** | 1 | 5 | 10 | **74.3%** |
| **Mobile & performance** | 0.2 | 1 | 5 | **20.2%** |
| **Content depth** | 0.1 | 0.2 | 1 | **5.5%** |
| | | | | 100.0% |

CR 0.08 — acceptable.

### Step 2 — Claude compares the options, one criterion at a time

Five ways to build the site: **Squarespace**, **WordPress**, **Astro**, **Wix**,
**Custom build**. Claude runs the same 1/5/10 pairwise — ten pairs per criterion, thirty
total — and you never answer any of them.

Here is the full matrix for the first criterion, **Newcomer clarity**:

|  | Squarespace | WordPress | Astro | Wix | Custom | **Local weight** |
|---|---|---|---|---|---|---|
| **Squarespace** | 1 | 5 | 1 | 5 | 0.2 | **16.5%** |
| **WordPress** | 0.2 | 1 | 0.2 | 1 | 0.1 | **4.0%** |
| **Astro** | 1 | 5 | 1 | 5 | 0.2 | **16.5%** |
| **Wix** | 0.2 | 1 | 0.2 | 1 | 0.1 | **4.0%** |
| **Custom build** | 5 | 10 | 5 | 10 | 1 | **59.1%** |
| | | | | | | 100.0% |

Two more matrices, same shape, produce the other two columns:

| Option | Clarity | Mobile | Depth |
|---|---|---|---|
| Squarespace | 16.5% | 11.9% | 5.4% |
| WordPress | 4.0% | 3.7% | 56.8% |
| Astro | 16.5% | 56.7% | 17.7% |
| Wix | 4.0% | 3.7% | 2.3% |
| Custom build | 59.1% | 24.1% | 17.7% |

Each column sums to 100% — it ranks the options *within* that one criterion.

### Step 3 — Synthesis

Multiply each option's local weight by its criterion weight and add:

```
Custom build = (0.591 × 74.3%) + (0.241 × 20.2%) + (0.177 × 5.5%) = 49.8%
```

| Option | Weighted score |
|---|---|
| **Custom build** | **49.8%** |
| Astro | 24.7% |
| Squarespace | 14.9% |
| WordPress | 6.8% |
| Wix | 3.8% |
| | 100.0% |

Read the losers, not just the winner. **WordPress dominates content depth at 56.8% and
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

## Versioning

The version lives in one place: `version` in
[`.claude-plugin/plugin.json`](.claude-plugin/plugin.json). It's what `claude plugin list`
reports and what an install pins to.

Convention for changes:

| Change | Bump | Example |
|---|---|---|
| Method or output shape changes | major | dropping a calculation method |
| New capability | minor | adding option scoring |
| Correction, doc fix | patch | fixing a broken manifest |

Every version bump gets a matching git tag and a [CHANGELOG](CHANGELOG.md) entry, so a
specific version can be fetched from the repo rather than only from the plugin cache.

## Updating

```
claude plugin marketplace update GZancewicz
claude plugin update ahp-comparison-matrix@GZancewicz
```

Restart Claude Code to apply. Three things that trip people up:

- The `@GZancewicz` suffix is required — a bare plugin name reports "not found".
- User and project scopes update independently. Check `claude plugin list`; if it's
  installed at both, update both.
- `--scope project` resolves against your current directory, so run it from inside the
  project that declares the plugin.

## License

MIT

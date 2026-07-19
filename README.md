# pairwise-weights

A Claude Code skill for turning "which of these matters more?" into defensible numeric weights.

When you need to score things against multiple criteria — vendors, candidates, designs,
websites — the hard part isn't the scoring, it's deciding how much each criterion counts.
Guessing at percentages produces weights nobody trusts. This skill derives them from a
series of simple two-way comparisons instead.

It's a variant of the **Analytic Hierarchy Process** (AHP, Saaty 1977), simplified to a
three-point scale.

## How it works

Criteria are grouped into clusters (max 5 per matrix). Claude asks you about each pair:

> Which matters more: **Photography & visual identity** or **Mobile & performance**?
>
> - About the same
> - Photography more
> - Photography much more
> - Mobile more
> - Mobile much more

Answers map to a 1/5/10 scale. The reciprocal is filled in automatically, so you're never
asked the same pair twice. Row sums are normalized to 100%.

Claude reports two sets of weights — raw-sum and column-normalized — because the choice
materially changes the spread, and you should see it rather than inherit it silently.
Intransitive answers (A > B > C > A) are named rather than quietly averaged away.

## Install

```
/plugin marketplace add GZancewicz/pairwise-weights
/plugin install pairwise-weights@GZancewicz
```

## Use

Just describe the problem:

> Help me weight these criteria for evaluating job candidates.

Or invoke it directly with `/pairwise-weights`.

## Design notes

A few opinions are baked in, learned the hard way:

- **Five clusters maximum.** Ten pairs is about where careful judgment stops. Twenty
  criteria means 190 comparisons and the back half is noise.
- **Define the criteria before the first pair.** Most inconsistent results come from the
  user's understanding of a criterion shifting halfway through, not from genuine
  ambiguity about importance.
- **Every weighted criterion must be scorable on every item.** A criterion you can only
  assess on some of the corpus corrupts the ranking, no matter how well-weighted.

## License

MIT

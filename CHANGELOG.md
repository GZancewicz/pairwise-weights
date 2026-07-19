# Changelog

Versions follow [semver](https://semver.org/): breaking changes to the method or output
shape bump major, new capability bumps minor, corrections bump patch.

## 1.3.0

- Replaced the interactive option picker with a plain numbered 1-5 prompt. Claude Code's
  `AskUserQuestion` caps options at 4; a five-point scale only fit by pushing the fifth
  answer into free-text "Other", which made four answers one click and the fifth typing —
  biasing every pair toward the criterion listed first
- Options are now symmetric and positionally fixed: left criterion is always 1-2, right is
  always 4-5
- Added fast mode: list all pairs, answer with one line of numbers

## 1.2.0

- Renamed from `pairwise-weights` to `ahp-comparison-matrix`
- Added `.claude-plugin/marketplace.json` — required for `/plugin marketplace add`;
  installation was broken without it
- Removed `displayName` from the manifest — rejected by plugin validation
- Added History section documenting the AHP lineage and the GOAL/QPC *Memory Jogger* route
- Added "Why a plugin instead of just a SKILL.md?" section
- Front-loaded install instructions and use cases for discoverability

## 1.1.0

- Added option scoring and synthesis: Claude runs the option-level matrices, the user keeps
  the criteria matrix, results combine into a weighted ranking
- Criterion count softened from a hard cap of 5 to guidance (~5 works best)
- Dropped the column-normalized method — row sums normalized to 100% is now the only one
- Documented the five-stage session flow with all six comparison questions shown
- Added a diagnostic that names the specific pair behind a surprising weight

## 1.0.0

- Initial release: 1/5/10 pairwise scale, reciprocal fill, row sums normalized to 100%

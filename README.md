# devin-evals-action

GitHub Action that regression-gates a golden eval corpus with
[`devin-evals`](https://pypi.org/project/devin-evals/) — the
graders/corpus package from the devin-* ecosystem.

It runs `devin-evals corpus verify` on a corpus directory produced by
`devin-evals corpus generate` and fails when any case's actual verdict
diverges from its recorded `expected_status` — a **MISMATCH**, reported
per-case with its defect class so the CI error names the regression,
not just "evals failed". Documented `known_gap`s are tolerated unless
`strict` is on.

## Usage

```yaml
- uses: Icaro0310/devin-evals-action@v1
  with:
    corpus: packages/evals/corpus   # required
```

```yaml
- uses: Icaro0310/devin-evals-action@v1
  with:
    corpus: tests/corpus
    strict: "true"                    # also fail on known_gaps
    version: "devin-evals==0.2.0"
```

## The two distinct failures

| Situation | Message | Meaning |
|---|---|---|
| `corpus.json` unreadable/missing | precondition error | corpus not generated / wrong path — not a regression |
| cases diverge from `expected_status` | `MISMATCH <id> expected=X actual=Y` | a real grader regression, named per defect |

## Notes

- Cheap enough for PR triggers: the golden corpus replays in ~1 s
  (deterministic graders, synthetic sessions).
- Read-only by design — it replays recorded sessions, never modifies
  them, and never touches the repo beyond the corpus it reads.
- The satellite repo is only the Action distribution vehicle; all
  grading logic lives in
  [`devin-evals`](https://github.com/Icaro0310/devin-assure/tree/main/packages/evals).

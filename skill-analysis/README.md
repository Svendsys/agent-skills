# skill-analysis

Benchmark studies that measure whether a skill actually pays off, produced by the
[`benchmark-skill`](../claude/skills/benchmark-skill/SKILL.md).

**Running a benchmark means saving its results here.** Every benchmark run persists
its study under `skill-analysis/<skill-name>/` in this repo — that is part of
finishing the run, not an optional write-up. These are *our reports*: the durable
record of what each skill is worth, and the ruler a later re-run reproduces. A
benchmark whose numbers live only in a chat transcript hasn't been saved.

This repo intentionally does **not** vendor in studies from any consuming project
(e.g. a website that uses these skills); studies generated here stand on their own.

## Layout

```
skill-analysis/
├── README.md           # this file — the convention
├── TEMPLATE/           # the shape every study copies (read before designing one)
│   ├── README.md       # annotated report skeleton
│   ├── data.csv        # canonical column header, one row per agent run
│   ├── evidence.md     # verbatim transcript excerpts
│   └── harness/        # the committed, reproducible runner
└── <skill-name>/       # one directory per benchmarked skill
    ├── README.md
    ├── data.csv
    ├── evidence.md
    └── harness/
```

## How a study is structured

Each `<skill-name>/` study holds four things, described in full by the
`benchmark-skill` and templated under `TEMPLATE/`:

- **`README.md`** — the report: bottom-line verdict, scenario table, results grids
  (lead with the metric the skill exists to move), what the data says, why bare
  agents struggle, evidence link, method, and caveats.
- **`data.csv`** — one row per agent run, columns kept consistent across studies so
  they stay comparable.
- **`evidence.md`** — verbatim transcript excerpts contrasting the arms on the
  hardest scenario.
- **`harness/`** — the committed runner, fixtures, exact prompts, and pinned model
  IDs, so any later run reproduces the same benchmark byte-for-byte.

Start from `TEMPLATE/`, copy it to `<skill-name>/`, and fill it in.

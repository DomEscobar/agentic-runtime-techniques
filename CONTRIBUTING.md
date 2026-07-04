# Contributing

Contributions should make the catalog clearer, more grounded, or more useful.

## Add a Technique

Add a new entry to `data/techniques.yml` with:

- `name`
- `category`
- `summary`
- `loop_shape`
- `runtime_primitives`
- `solves`
- `tradeoffs`
- `sources`

If the technique is ambiguous, explain the ambiguity in `notes` instead of
pretending the naming is settled.

## Source Quality

Prefer primary sources:

- Papers
- Official docs
- Original blog posts
- GitHub repos with implementation code
- Talks or demos by the authors

Secondary summaries are useful, but mark them as secondary.

## Style

- Be concrete.
- Separate runtime primitives from agent behavior.
- Do not list a framework unless it demonstrates a reusable technique.
- Avoid hype words like autonomous, magical, or general unless quoting a source.

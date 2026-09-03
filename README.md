# DatasetDiscoveryWorkspace

Disposable execution workspace for DatasetDiscovery coding agents.

This repository contains only the stable workspace skeleton. The
DatasetDiscovery workspace manager creates and manages clones, symlinks the
configured six-table data layer at `data/tables`, stages the data catalog
(`catalog.md`), macros (`macros.sql`), agent prompt, tool shims, and OpenCode
config into place, records run state in `manifests/run_metadata.json`, and
backs up declared outputs to S3.

Runtime directories are intentionally untracked:

- `input/`: staged task inputs
- `data/`: table symlink and mutable derived data
- `scratch/`: temporary agent files
- `output/`: datasets and reports
- `logs/`: run logs
- `manifests/`: workspace and checkpoint metadata

The data catalog (`catalog.md`) and macros (`macros.sql`) are staged by the
manager at each `create`; they define the six tables (`comments`,
`submissions`, `author_events`, `subreddits`, `flairs`, `links`), their join
keys, and the DuckDB macros preloaded into every `query_tables` call.

Coding-agent instructions live in [`AGENTS.md`](AGENTS.md); Claude Code loads
[`CLAUDE.md`](CLAUDE.md), which points to the same instructions.

Experiment definitions, evaluation code, prompts, and orchestration remain in
the main DatasetDiscovery repository.

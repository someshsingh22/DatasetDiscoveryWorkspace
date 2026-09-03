# Workspace instructions

This repository is the complete execution workspace. Use files available here;
do not depend on the DatasetDiscovery source checkout.

## Paths

- `catalog.md`: the data catalog — six tables, schemas, join keys, macros, recipes. **Read first.**
- `macros.sql`: DuckDB macros preloaded into every `query_tables` connection.
- `data/tables/`: the read-only six-table Parquet layer. Query via the `query_tables` tool, not by reading files.
- `input/`: task-specific inputs.
- `scratch/`: temporary scripts and intermediate files.
- `output/`: final datasets, methodology, and reports.
- `logs/`: execution logs.
- `manifests/`: run and checkpoint metadata.

## Tools

Four native tools are staged into `.opencode/tool/`:

- `query_tables(sql, max_rows)` — one read-only SQL statement over the six tables (macros preloaded).
- `collect_threads(subreddits, …)` — thread-centric bulk collection; returns a `dataset_handle`.
- `collect_by_authors(authors, …)` — cross-community author-centric collection; returns a `dataset_handle`.
- `save_dataset(dataset_handle, research_statement, methodology, output_dir, phenomenon_label)` — final write.

Use SQL to explore and validate, the collect tools to pull, and `save_dataset` to persist. Keep temporary
work in `scratch/`; write declared deliverables to `output/`.

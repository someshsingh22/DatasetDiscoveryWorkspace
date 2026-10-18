# SQL workspace instructions

Read `catalog.md` first, then `sql_acquisition.md` and `run_policy.json`.
This is a disposable execution workspace for the SQL-only collector.

## Tools

The application generates these workspace-bound tools in `.opencode/tool/`:

- `query_tables`: bounded read-only SQL exploration over six registered tables.
- `materialize_query`: select source IDs and acquire canonical immutable records.
- `derive_artifact`: create a selection/annotation child with archived provenance.
- `compose_artifacts`: explicitly combine compatible parent artifacts.
- `finalize_artifact`: publish exactly the selected artifact and methodology.

No historical SQL macros or legacy collection tools are available. Use ordinary
SQL for author and thread selectors. Never modify acquired records or bypass the
tools to read source Parquet files. Extra source context requires acquisition.
Respect configured acquisition allowances and final-observation targets.

Optional tools and their instructions are staged by the application only when
the run policy enables observation search or training-output publication.
Follow those instructions when present; raw finalization alone may not complete
a task requiring training rows.

## Paths and integrity

- `catalog.md`: canonical six-table catalog and SQL acquisition contract.
- `input/`: task inputs; `scratch/`: temporary scripts and proposals.
- `output/`: immutable artifacts, final exports and SQL binding metadata.
- `logs/`: execution logs; `manifests/`: run and checkpoint metadata.
- `data/`: reserved local data directory; source tables are bound externally.

The runtime binds the configured source root and file/schema inventory; it does
not copy or symlink the corpus into this workspace. Instructions, runtime tools
and run policy are integrity-checked. Do not edit staged resources or bindings.
Local Python/model curation may read acquired artifacts in batches and write
proposals to scratch. Archive transformation and labeling provenance through the
tools. Python/bash access remains cooperative, not an OS security sandbox.

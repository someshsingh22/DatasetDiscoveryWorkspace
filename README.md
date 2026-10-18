# DatasetDiscoveryWorkspace

Canonical reusable workspace template for DatasetDiscovery's SQL-only collector.
The existing DatasetDiscovery submodule pins this repository; no separate template
repository or runtime clone is needed.

`workspace.yaml` declares template format 2, the `sql_v1` profile and runtime
directories. `AGENTS.md`, `CLAUDE.md`, and `catalog.md` are reusable instructions;
`catalog.md` is the authoritative agent-facing six-table catalog. No macros are
loaded. Read [AGENTS.md](AGENTS.md) for the five SQL/artifact tools.

DatasetDiscovery's Python WorkspaceManager creates a fresh local Git workspace,
stages these verified template resources, and adds its collector prompt, provider
config, acquisition policy and generated tool shims. It binds the external six-table
Parquet root and records template commit and resource hashes in the SQL binding.
It neither copies the source data nor installs a second execution environment.
Run metadata and existing checkpoints remain application responsibilities.

Runtime directories (`input/`, `data/`, `scratch/`, `output/`, `logs/`, `manifests/`)
are intentionally untracked. Tool implementations, provider/model settings,
training/evaluation implementations and experiment definitions belong in
DatasetDiscovery, not in this template. Optional training-output and observation
search instructions are supplied by that application when enabled.

From a compatible DatasetDiscovery checkout, initialize this pinned submodule:

```sh
git submodule update --init DatasetDiscoveryWorkspace
```

A missing template, missing required asset or incompatible manifest is a setup
error; the application does not fall back to its own catalog. Updating template
resources makes existing workspaces stale; create a fresh workspace afterward.

For coordinated releases, commit and publish this repository first, then update
the DatasetDiscovery gitlink to that published commit and publish DatasetDiscovery.
Local template edits can be tested before publication; resource hashes identify
those edits independently of the unchanged base commit.

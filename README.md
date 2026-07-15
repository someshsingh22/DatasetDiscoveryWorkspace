# DatasetDiscoveryWorkspace

Disposable execution workspace for DatasetDiscovery coding agents.

This repository contains only the stable workspace skeleton. The
DatasetDiscovery workspace manager creates and refreshes clones, exposes the
configured corpus at `data/corpus`, writes a workspace-specific `.env`, and
backs up declared outputs to S3.

Runtime directories are intentionally untracked:

- `input/`: staged task inputs
- `data/`: corpus link and mutable derived data
- `scratch/`: temporary agent files
- `output/`: datasets and reports
- `logs/`: run logs
- `manifests/`: workspace and checkpoint metadata

Stable workspace context is tracked under `context/`, including the corpus
schema in `context/dataset_layout.md`.

Experiment definitions, evaluation code, prompts, and orchestration remain in
the main DatasetDiscovery repository.

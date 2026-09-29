# Cylc 8 + Rose 2 Real-Project Training Scaffold

This project is designed to exercise the concepts from the official Cylc and Rose documentation in one workflow.

## Scenario

A weather workflow has three tasks:

1. `forecast` runs on `weather_hpc` (Slurm) and creates a forecast file.
2. `transfer_data` runs on `weather_hpc` and transfers the forecast to a separate Research HPC filesystem.
3. `analyse` runs on `research_hpc` (PBS) and analyses the transferred result.

All three tasks are Rose applications invoked with `rose task-run`.

## Source tree

```text
cylc_rose_real_project/
├── flow.cylc
├── app/
│   ├── forecast/
│   │   ├── rose-app.conf
│   │   ├── bin/run-forecast
│   │   └── file/model-input.txt
│   ├── transfer_data/
│   │   ├── rose-app.conf
│   │   └── bin/transfer-data
│   └── analyse/
│       ├── rose-app.conf
│       └── bin/analyse-forecast
├── bin/
│   └── project-status
├── conf/
│   ├── global.cylc.example
│   └── global.local.cylc.example
└── docs/
    └── RUNBOOK.md
```

## What this project teaches

- `global.cylc` vs `flow.cylc`
- Cylc platforms and install targets
- Slurm vs PBS job runners
- CPU, memory, and execution-time directives
- Rose `app/`, `rose-app.conf`, `bin/`, and `file/`
- `rose task-run`
- `$CYLC_TASK_WORK_DIR`
- `$CYLC_WORKFLOW_SHARE_DIR`
- `$CYLC_TASK_SHARE_CYCLE_DIR`
- explicit cross-filesystem data transfer
- `cylc validate`, `lint`, `install`, `play`, `reinstall`, `reload`, `pause`, `stop`
- source directory vs installed run directory

## Important

`conf/global.cylc.example` contains illustrative hostnames (`login01`, `login02`, `research01`). Replace these with real site platform definitions before attempting an HPC run.

The workflow is intentionally configured for the two named HPC platforms, so it will not run unchanged on a laptop with no such platform configuration.

See `docs/RUNBOOK.md` for the implementation sequence.

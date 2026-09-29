# Project Runbook

## Phase 1 — Verify the software

Recommended combined environment:

```bash
conda install -c conda-forge cylc-flow cylc-rose metomi-rose
```

Check:

```bash
cylc version --long
rose --version
rose check-software
```

You should see the Cylc Rose plugin in `cylc version --long`.

## Phase 2 — Inspect site platforms

Before editing any site configuration:

```bash
cylc config --platform-names
cylc config --platforms
```

If `weather_hpc` and `research_hpc` already exist, use the site definitions.

If you are responsible for user-level configuration, adapt `conf/global.cylc.example` to your real hosts, job runners, install targets, and filesystem paths.

Key concept:

- platform = execution environment
- hosts = login/submission hosts
- job runner = Slurm/PBS/etc.
- install target = filesystem identity

## Phase 3 — Validate the source

From the source directory:

```bash
cylc validate .
cylc lint --ruleset 728
```

For a new Cylc 8 workflow, the migration lint rules should not find Cylc 7 syntax.

## Phase 4 — Understand the Rose apps

`forecast`:
- Rose copies `app/forecast/file/model-input.txt` into the task work directory.
- `app/forecast/bin` is prepended to `PATH`.
- `rose task-run` reads `app/forecast/rose-app.conf`.
- `run-forecast` writes the final product under `$CYLC_TASK_SHARE_CYCLE_DIR/forecast/`.

`transfer_data`:
- runs after `forecast`.
- verifies the forecast file.
- uses SSH + rsync to move it to the Research HPC run tree.

`analyse`:
- runs on Research HPC after transfer succeeds.
- reads the transferred forecast from its own `$CYLC_TASK_SHARE_CYCLE_DIR`.
- writes `analysis/analysis.txt`.

## Phase 5 — Install

From the source directory:

```bash
cylc install .
```

Typical result:

```text
~/cylc-run/cylc_rose_real_project/run1/
```

The source and run directories are separate. Do not treat the installed run directory as your development copy.

## Phase 6 — Play

```bash
cylc play cylc_rose_real_project/run1
```

Observe with:

```bash
cylc tui cylc_rose_real_project/run1
```

or, if your UI server is available:

```bash
cylc gui cylc_rose_real_project/run1
```

## Phase 7 — Inspect runtime files

On the relevant job platforms, inspect:

```text
~/cylc-run/cylc_rose_real_project/run1/
├── work/
├── share/
│   └── cycle/
└── log/
```

The exact cycle-point directory depends on the workflow run.

Important variables used by this project:

```bash
CYLC_TASK_WORK_DIR
CYLC_WORKFLOW_SHARE_DIR
CYLC_TASK_SHARE_CYCLE_DIR
CYLC_TASK_CYCLE_POINT
CYLC_WORKFLOW_ID
```

## Phase 8 — Make a source change

For example edit:

```text
app/forecast/rose-app.conf
```

Change:

```ini
FORECAST_HOURS=48
```

to:

```ini
FORECAST_HOURS=72
```

Then update the existing installation:

```bash
cylc reinstall cylc_rose_real_project/run1
```

If workflow configuration itself has changed and the scheduler is running:

```bash
cylc reload cylc_rose_real_project/run1
```

Remember:

- reinstall = copy changed source into the existing installation
- reload = tell a running scheduler to reload workflow configuration

## Phase 9 — Pause, resume, stop

```bash
cylc pause cylc_rose_real_project/run1
cylc play cylc_rose_real_project/run1

cylc stop cylc_rose_real_project/run1
cylc play cylc_rose_real_project/run1
```

Pause prevents new submissions while keeping the scheduler alive.
Stop shuts down the scheduler.
Play resumes an existing run from persisted state.

## Phase 10 — Fresh run

To execute the workflow as a new run, install again:

```bash
cylc install .
cylc play cylc_rose_real_project/run2
```

Do not use `cylc play run1` expecting a fresh workflow; it resumes that existing run.

## Production adaptation checklist

Before a real HPC run, confirm:

- [ ] actual platform names from `cylc config --platform-names`
- [ ] Slurm directives accepted by your Weather HPC site
- [ ] PBS flavor and `select` syntax accepted by Research HPC
- [ ] account/partition/queue directives if required
- [ ] SSH connectivity from transfer host to Research HPC
- [ ] rsync availability
- [ ] remote run-directory path is correct
- [ ] install targets correctly represent filesystems
- [ ] storage quotas and permissions
- [ ] module/Conda environment setup on job hosts
- [ ] Rose is available on every job host running `rose task-run`

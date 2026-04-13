# ParlaCAP Analysis Tutorials Report

**Date:** 2026-04-13  
**Project:** `ParlaCAP-Analysis-Tutorials`

## Executive Summary

This report combines the current setup status, reproducibility notes, API usage notes, and the optimization benchmark for Tutorial 1.

Current practical baseline:
- Python `3.11.x` is the safest default for this repo.
- `requirements.txt` currently installs successfully in the working `venv_parlacap` environment.
- API access uses `PARLACAP_API_KEY`.
- Tutorial 1 loading can be reduced from about `165.9 s` to about `56.0 s` with the tested optimized configuration.

Important caveat:
- `env.yaml` is not aligned with the recommended baseline in this report. It appears to target a different environment snapshot and includes Python `3.14`, so it should not be treated as the main reproducible setup file without cleanup.

## Project Context

### Main Goal

The project goal is to prepare ParlaCAP analysis tutorials that:
- are understandable and reproducible
- run on an average laptop
- use a reasonably optimized workflow
- also have an API-based version that answers the same research questions as the original tutorials

### Mentor Requirements

The working requirements gathered for this task are:
- get familiar with the dataset and the research questions answered by the tutorials
- optimize existing tutorials where possible
- check obvious `for` loops and other weak or overly improvised code patterns
- test the setup instructions, especially Python versions and requirements
- create an API-based tutorial version with pagination using `limit=1000` and `offset=0, 1000, 2000, ...`
- keep the analytical logic aligned with the existing tutorials

### Reference Links

- original tutorial repo: `https://github.com/clarinsi/ParlaCAP-Analysis-Tutorials`
- fork in use: `https://github.com/leskovecg/ParlaCAP-Analysis-Tutorials`
- backend API docs: `https://parlacap.ipipan.waw.pl/api/v1/docs`
- auth docs: `https://parlacap.ipipan.waw.pl/auth/docs`
- minimal API example: `https://github.com/5roop/parlaCAPI/blob/main/demo.ipynb`

## Tutorial Scope

### Dataset Focus

The tutorials work with parliamentary speech data and related annotations, including:
- topic / CAP category
- sentiment
- time
- country / parliament
- coalition vs opposition status

### Tutorial Research Questions

The current tutorial flow covers:
1. first steps with the dataset
2. topic and sentiment distributions
3. cross-country comparison
4. sentiment and topic frequency over time
5. coalition vs opposition dynamics

### Technical Framing

The intended migration path is conceptually:

```text
original: data = load_local_data() -> analysis(data)
target:   data = fetch_from_api()   -> analysis(data)
```

The main objective is to preserve the research questions and analytical outputs while changing the data-access layer.

## Setup And Reproducibility

### Recommended Baseline

- Python `3.11.x` is the recommended default for this repository.
- Python `3.10` may fail for some pinned versions.
- The local environment `venv_parlacap` is currently working with Python `3.11`.

### Requirements Status

The `requirements.txt` installation issue that blocked setup was caused by a dependency conflict:
- `bleach[css]` required `tinycss2 < 1.5`
- the repo had `tinycss2==1.5.1`

Applied fix:
- `tinycss2==1.5.1` -> `tinycss2==1.4.0`

Current state:
- `python -m pip install -r requirements.txt` completes successfully in `venv_parlacap`
- remaining output is limited to a non-critical pip upgrade notice

### Version Notes

- A previous Python `3.10` attempt failed because of `contourpy`
- the current `requirements.txt` uses `contourpy==1.3.2`
- this is consistent with the current Python `3.11` recommendation

### Important Environment Warning

`env.yaml` currently does not match the practical setup guidance above:
- it includes Python `3.14`
- it looks like an exported environment snapshot rather than a clean cross-platform environment file
- it should be treated carefully until it is simplified and aligned with the repo's actual recommended setup

## API Notes

### Authentication

Recommended authentication variable:
- `PARLACAP_API_KEY`

### Pagination

The API returns at most 1000 rows per request, so pagination should use:
- `limit=1000`
- `offset=0, 1000, 2000, ...`

### Practical Guidance For Average Laptops

To keep runs manageable on typical laptops:
- start with a subset of parliaments
- use a bounded date range where possible
- scale up only after the workflow is confirmed to work end-to-end

## Workspace And Current State

### Local Workspace

Current relevant workspace context:
- repo is cloned locally on `D:` due to disk-space constraints
- active environment is `venv_parlacap`
- local speech data is available in `speeches_no_text`
- work is being done primarily in VS Code notebooks

### Repo Structure Notes

Relevant repo areas:
- `notebook_python/` contains tutorials `0-5`
- `requirements.txt` and `env.yaml` are part of the reproducibility task
- `parlacap_context.md` serves as a handover-style context file

### Work Already Completed

Known completed work includes:
- repository fork and local clone
- virtual environment creation
- setup debugging around `requirements.txt`
- critical fixes in Tutorial 0 for Windows / venv guidance
- optimization investigation for Tutorial 1

### Current Tutorial 1 Note

At the time captured in the context notes:
- Tutorial 1 had previously received some refactoring and optimization work
- the main loading cell was later returned to the original structure on request
- CAP filtering cells after the loading step remained functional

## Open Issues And Risks

The most relevant currently documented risks are:
- reproducibility is not yet fully validated from a clean setup
- large files can still create RAM pressure, so chunking remains important
- API migration needs careful pagination handling and result-parity checks against the local pipeline
- average-laptop usability still depends on bounded workloads and practical notebook defaults

## Definition Of Done For The Current Phase

The working definition of done captured in the project context is:
- Tutorial 0 and Tutorial 1 run stably in the local setup
- setup notes reflect the actually working environment
- there is a clear plan for API migration
- a first working API notebook exists for the same analytical logic

## Optimization Benchmark

### Scope

Focus:
- optimization of loading and filtering the ParlaMint dataset for Tutorial 1

Date of benchmark:
- `2026-03-30`

### Headline Result

| Metric | Value |
|---|---|
| Baseline time | `165.9 s` |
| Optimized time | `56.0 s` |
| Speedup | `2.96x` |
| Improvement | `66.2% faster` |
| Rows processed | `4,225,608` |
| Parliaments | `28` |

### Variant Comparison

| Variant | Engine | Chunksize | Filter | Time (s) | Improvement | Note |
|---|---|---:|---|---:|---:|---|
| `test_1_default_query_50k` | default | 50k | query | `56.05` | `66.2%` | best result |
| `test_2_default_mask_50k` | default | 50k | mask | `57.11` | `65.5%` | nearly identical |
| `test_4_default_mask_100k` | default | 100k | mask | `64.71` | `61.0%` | larger chunks were slower |
| `test_3_default_mask_25k` | default | 25k | mask | `66.64` | `59.8%` | smaller chunks added overhead |
| `baseline_python_query_50k` | python | 50k | query | `165.92` | baseline | much slower |

### Main Findings

1. The biggest win came from removing `engine="python"` and using the default C parser.
2. `query()` and boolean masks performed very similarly in this benchmark.
3. `chunksize=50_000` was the best tested setting.
4. Filtering remained the main bottleneck even after optimization.

### Best Tested Configuration

```python
reader_kwargs = {
    "sep": "\t",
    "usecols": cols_to_keep,
    "dtype": dtypes,
    "chunksize": 50_000,
    "na_values": ["-"],
    "keep_default_na": False,
}
reader = pd.read_csv(file_path, **reader_kwargs)
```

Avoid this slower pattern:

```python
reader = pd.read_csv(
    file_path,
    engine="python",
    chunksize=50_000,
)
```

### Benchmark Breakdown

| Component | Time (s) | Share |
|---|---:|---:|
| Read | `24.3` | `43.4%` |
| Filter | `28.9` | `51.6%` |
| Concatenation | `1.25` | `2.2%` |
| Total | `56.05` | `100%` |

### Resource Notes

- A naive all-at-once approach may require roughly `8-10 GB RAM`
- the chunked optimized approach is estimated at roughly `2-3 GB RAM`

### Country-Level Runtime Notes

The slowest country files in the benchmark were:
- `TR` about `4.94 s`
- `FR` about `4.36 s`
- `GB` about `3.87 s`
- `HR` about `3.51 s`
- `DK` about `2.58 s`

This suggests runtime is driven by a mix of fixed per-file overhead and dataset size.

## Recommended Next Steps

1. Keep Python `3.11.x` as the documented default baseline.
2. Treat `requirements.txt` as the current working setup source.
3. Clean up or replace `env.yaml` before presenting it as a reproducible environment file.
4. Use the optimized Tutorial 1 loading configuration as the default where appropriate.
5. Continue validating API-based tutorials on bounded workloads suitable for average laptops.
6. Run a clean baseline execution of Tutorial 1 and record runtime, RAM footprint, and any failing cells.
7. Align future API work with the requirement for a dedicated `notebook_api/` version if that remains the agreed structure.

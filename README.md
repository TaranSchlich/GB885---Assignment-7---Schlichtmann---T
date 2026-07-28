# Bramble Bean — Operations Waste-Rate Report

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/TaranSchlich/GB885---Assignment-7---Schlichtmann---T/blob/main/ops_report.ipynb)

A Google Colab notebook that computes the company-wide monthly coffee **waste rate**
for Bramble Bean, a regional coffee chain with 40 cafés — plus a worked example of using
**Git history to investigate and repair a silent reporting bug**.

**Author:** Taran Schlichtmann · Built for **GB885 — Python Fundamentals (Collaborative & Professional Programming)**

---

## Business case

Bramble Bean's Director of Operations, Priya, flagged a discrepancy in the monthly ops
report: April's waste rate came in noticeably lower than March's, even though nothing
operational had changed — no new cafés, no new equipment, and no change to how waste is
logged. She suspected the underlying calculation had been altered. The regular report
author was out on leave, so the task was to use the team's Git history to find out what
happened, correct it, and document the fix.

## The investigation

The whole project history lives in Git, so the answer was in the commit log. Reading the
history surfaced a suspicious commit:

> **`Simplify waste rate calc`** — Theo Brandt, 2026-04-01

The diff showed the commit did two things under a "cleanup" label:

1. **Changed the metric's denominator** from `roasted_lbs` to `sold_lbs`.
2. Renamed the internal variables (the cosmetic change the message advertised).

Waste is generated against what a café **roasts**, not what it **sells**. Because cafés
sell more than they roast in a given month, swapping in the larger `sold_lbs` denominator
quietly shrank the reported rate — making April look better than it really was. March was
reported *before* this commit, so the March figure was never affected.

## The fix

Following professional version-control practice, the repair was made on a dedicated
branch and merged through a pull request rather than committed straight to `main`:

- **Branch:** `fix/waste-rate-denominator`
- **Change:** restored `roasted_lbs` as the denominator in `waste_rate()` (keeping the
  cleaner variable names)
- **Pull request:** [#1 — Fix April waste rate: divide by pounds roasted, not pounds sold](../../pull/1),
  with a plain-English explanation of what was wrong, when it was introduced, and what changed

## Corrected results

| Month | Reported (broken: waste ÷ sold) | Corrected (waste ÷ roasted) |
|-------|:-------------------------------:|:---------------------------:|
| March | 4.1% *(unaffected)*             | **4.1%**                    |
| April | 3.6%                            | **4.2%**                    |

The "improvement" in April was an artifact of the code change, not a real operational
gain — the true April waste rate is **4.2%**, slightly higher than March.

## Repository audit & security note

During a review of the repository, a credentials file — `bramble_api_token.json` — was
found committed to the project. Secrets should never live in a repository. It has been
removed from the working tree and a `.gitignore` now prevents it (and similar files) from
being re-added.

> ⚠️ **Important:** Deleting a secret in a new commit does **not** remove it from Git
> history — earlier commits still contain it, and anyone with the repo link can recover
> it. In a real engagement the correct response is to **rotate (regenerate) the exposed
> token immediately** and, if required, purge it from history with a tool like
> `git filter-repo` or the BFG Repo-Cleaner. The token here is a placeholder from the
> course exercise, so the project history is intentionally left intact to preserve the
> documented investigation.

## Skills demonstrated

- Investigating a codebase's history with `git log` and reading commit **diffs**
- Diagnosing a data-reporting regression from version-control evidence
- Repairing shared code on a **branch** and finalizing via a **pull request** and merge
- Writing clear, best-practice commit messages (imperative mood; explain *what* and *why*)
- Recognizing and remediating a committed secret
- Data aggregation in **pandas** (filtering and ratio metrics)

## Repository structure

```
.
├── ops_report.ipynb    # the analysis notebook (waste-rate report)
├── cafe_log.csv        # monthly per-café roasted / sold / waste pounds
├── cafe_info.csv       # café id → name lookup
├── README.md           # this file
└── .gitignore          # excludes secrets and local cruft
```

## How to run

1. Open `ops_report.ipynb` in [Google Colab](https://colab.research.google.com/) (use the
   **Open in Colab** badge above, or File → Open notebook → GitHub tab).
2. Run the cells top to bottom (**Runtime → Run all**).
3. No manual input is required — the café log is read directly from a public raw GitHub
   URL, so the notebook reproduces end to end on any machine.

Each section prints its result inline: the March and April company-wide waste rates and a
one-line summary of cafés covered.

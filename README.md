# Case 1 — Customer-Retention Contact List

This repository contains the supplied analysis code, saved outputs, and the
student-authored analysis for Case 1. The assigned CSV is used locally but is
excluded from version control.

## Environment

Use Python 3.11. The completed runs used Python 3.11.3 with the versions pinned
in `VD1_requirements.txt`:

- NumPy 1.24.2
- pandas 2.0.0
- scikit-learn 1.2.2

Install the requirements in a Python 3.11 environment:

```bash
python -m pip install -r VD1_requirements.txt
```

## Obtain the course CSV

Obtain `churn.csv` from the instructor-supplied Case 1 assignment package and
place it in the same directory as `VD1_analysis.py`. Use the assigned copy so
the data fingerprint and results match the course extract. The CSV is listed in
`.gitignore` and should not be uploaded publicly unless the course explicitly
requires it.

## Run the analysis

Run the comparison first:

```bash
python VD1_analysis.py compare --csv churn.csv --out outputs
```

Review `outputs/validation.csv` and record the method choice before evaluating
the final test. The recorded choice for this analysis is **boosted trees**
(`trees`), recorded on September 27, 2026.

Run the final evaluation with that recorded choice:

```bash
python VD1_analysis.py evaluate --csv churn.csv --out outputs --choice trees
```

## Dataset source and license

The handout identifies the file as the public Telco customer-churn example
distributed through the
[scikit-learn/churn-prediction dataset mirror](https://huggingface.co/datasets/scikit-learn/churn-prediction).
The mirror declares the dataset under CC BY 4.0; the handout records that this
was checked on September 9, 2026. The analysis uses the instructor-supplied
extract rather than downloading a replacement.

The dataset is a classroom example, not verified Summit Telecom data. In the
source data, `Churn = Yes` means that the customer left.

## Generated outputs

The complete `outputs/` directory contains:

- `validation.csv` — validation metrics for all three methods
- `split_rows.csv` — source-row partition assignments
- `compare_run.json` — compare-stage data fingerprint, versions, and settings
- `test_metrics.csv` — final-test metrics for all three methods
- `intervals.csv` — AUC intervals and paired AUC-difference intervals
- `probability_groups.csv` — probability-accuracy groups for boosted trees
- `scenarios.csv` — hypothetical value scenarios for all three methods
- `test_predictions.csv` — final-test predictions and chosen-list indicator
- `evaluate_run.json` — evaluation-stage fingerprint, versions, settings, and
  recorded choice

## Repository materials

- `analysis.md` — student-authored responses under Q1–Q4 headings
- `VD1_analysis.py` — supplied analysis script, unchanged
- `VD1_requirements.txt` — supplied package requirements
- `outputs/` — complete generated output directory
- `AI_transcript.md` — verbatim transcript of the Codex assistance used for the project
- `AI_USE.md` — brief description of AI assistance and personal checks
- `memo.pdf` — add the student-authored one-page memo before submission

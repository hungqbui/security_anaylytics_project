# Project 15 — Final checkpoint

Report: `final_checkpoint.pdf` (source: `final_checkpoint.tex`).
Repository: https://github.com/hungqbui/security_anaylytics_project

Install packages **outside the notebooks** with `python -m pip install -r requirements.txt`.
The model experiments were executed on a Colab NVIDIA L4 with Python 3.13.16;
data generation and report analysis also run on Python 3.11 CPU.

`data_preparation.ipynb` contains shared downloads, audits, duplicate protection, data splits and validation roles. The pipeline notebooks call it with `%run`; it can also be executed independently.

Run these notebooks top to bottom, in order:

1. `perturbation_preparation.ipynb` — deterministic perturbations,
   GPT paraphrases, validation perturbations (`selection_perturbations.csv`), paired concept perturbations (`concept_perturbations.csv`) and automated semantic review.
2. `final_experiment.ipynb` — frozen probe, 20 epochs for each of three LoRA seeds, validation-based epoch selection, representation/concept metrics.
3. `zero_shot_nli.ipynb` — pretrained NLI detector using fixed class descriptions.
4. `final_analysis.ipynb` — intervals, verdicts, seed spread, error categories, figure.

The equivalent single command is:

```bash
jupyter execute --inplace --timeout=7200 perturbation_preparation.ipynb final_experiment.ipynb zero_shot_nli.ipynb final_analysis.ipynb
```

The generated CSVs make API regeneration optional. To regenerate them, configure
`API_URL` (the full OpenAI-compatible chat-completions URL) and `OPENAI_API_KEY`:

```bash
export API_URL="http://localhost:8080/v1/chat/completions"
export OPENAI_API_KEY="your-key"
```

Generation uses `gpt-6.1-sol`. These variables are unnecessary when reusing the supplied CSVs.
Delete only the generated paraphrase/review CSVs you intend to regenerate.
Generated text is nondeterministic; the supplied CSVs define the reported evaluation set.
No API credential, model checkpoint or embedding cache is required in the repository.

`logits.csv` caches raw detector logits, probabilities and thresholds for calibration
and test rows, keyed by row ID, method, seed, condition and split. Probe logits are
the logistic decision function; NLI also retains both class-description entailment
logits, with their difference as the binary logit. These permit metric and threshold
analysis without model inference, but cannot reconstruct representation vectors.

`concept_robustness.csv` compares the clean and perturbed mean concept directions
within each encoder. It includes matched-pair bootstrap intervals and paired changes
relative to frozen for the training and held-out concept pairs separately.

The final `scores.csv` uses the final-checkpoint schema. `midsemester_scores.csv` preserves
the original baseline scores. `project15_checkpoint.ipynb` is historical; running it would
replace `scores.csv` with the older schema.

Download `check_final_checkpoint.py` from Canvas and run it separately:

```bash
python check_final_checkpoint.py --split split.csv --scores scores.csv --errors errors.csv --metric auroc --reference frozen --direction higher
```

Its full output is in `final_checker_output.txt` and the PDF appendix. Compile the report
with `pdflatex final_checkpoint.tex` twice, with the supplied static table fragments.

Data sources: [deepset/prompt-injections](https://huggingface.co/datasets/deepset/prompt-injections)
and [xTRam1/safe-guard-prompt-injection](https://huggingface.co/datasets/xTRam1/safe-guard-prompt-injection).
The notebooks pin publisher and model revisions. Generated text retains publisher labels;
automated review is supplementary evidence, not independently verified ground truth.
AI assistance is disclosed in the report.

The original 269 validation rows are split into separate epoch-selection and threshold-calibration roles in `validation_roles.csv`. `epoch_history.csv` records all 20 epochs per seed, including representation drift on the fixed clean selection rows; the best epoch by mean AUROC over clean/zero-width/homoglyph/base64 selection copies is restored in memory before test scoring (clean log loss breaks ties). No model checkpoint files are written.

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
3. `concept_reanalysis.ipynb` — concept similarities from cached representations; CPU only.
4. `zero_shot_nli.ipynb` — pretrained NLI detector using fixed class descriptions.
5. `final_analysis.ipynb` — intervals, verdicts, seed spread, error categories, figure.

The equivalent single command is:

```bash
jupyter execute --inplace --timeout=7200 perturbation_preparation.ipynb final_experiment.ipynb concept_reanalysis.ipynb zero_shot_nli.ipynb final_analysis.ipynb
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
No API credential or model checkpoint is required. The supplied representation caches allow CPU-only concept reanalysis.

`logits.csv` caches raw detector logits, probabilities and thresholds for calibration
and test rows, keyed by row ID, method, seed, condition and split. Probe logits are
the logistic decision function; NLI also retains both class-description entailment
logits, with their difference as the binary logit. These permit metric and threshold
analysis without model inference, but cannot reconstruct representation vectors.

`random_concept_pairs.csv` fixes 20 randomly sampled injection–benign test pairs
(10 per dataset; 40 distinct source prompts; seed 17). The direction is the
difference between the publisher-positive and publisher-negative class means;
pairing order does not affect it. `random_concept_perturbations.csv` contains the
clean and deterministic variants. `random_concept_robustness.csv` compares clean
and perturbed directions within each encoder, with 1,000 bootstrap draws that
resample each class independently within dataset and preserve source identity
across conditions and encoders. This replaces the screened-pair direction
comparison as the report's primary concept-robustness diagnostic.

The earlier `concept_robustness.csv` and quoted-control pairs remain auxiliary
results. Their narrower override concept differs from the new publisher-label
contrast; their estimates should not be treated as results from the same sample.

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

`representations_frozen_15.npz` and `representations_lora_{15,16,17}.npz`
cache the 768-dimensional original and test-variant embeddings, with source IDs
and condition names as array keys. They contain no model weights and load with
`allow_pickle=False`. After changing the random test sample within the cached sources/conditions, run the preparation
notebook and `concept_reanalysis.ipynb`, then `final_analysis.ipynb`; those analyses
need no GPU or LoRA retraining. A full fresh run executes `concept_reanalysis.ipynb`
between `final_experiment.ipynb` and the final analysis.

`random_concept_distances.csv` adds raw Euclidean distance, distance relative to
the clean concept norm, and unit-normalized Euclidean distance, with the same
source bootstrap and changes relative to frozen. Lower distance means greater
clean/perturbed similarity; raw and relative distances also depend on scale.

The arrays use the reported last-layer pooling, 128-token budget and selected epochs. New input conditions or a different representation extraction rule still require model inference.

`concept_sample_size.ipynb` checks 10, 20, 40, 60, 80, 100 and 112 pairs with
100 nested random subsets per size, reusing the same sources across models and
conditions. It also computes a reference from all 315 test prompts, retaining
equal dataset weights. The notebook reproduces the original 20-pair cosines as
an implementation check. All computation uses the cached arrays on CPU.
Outputs are `concept_sample_size_results.csv`, `concept_sample_size_summary.csv`,
`concept_sample_size_changes.csv`, `concept_full_test_reference.csv` and
`concept_sample_size_stability.pdf/png`. The plotted bands describe variation
across subsets of the fixed test pool, not population confidence intervals or
additional independent model runs. The PDF includes this analysis in Appendix E.

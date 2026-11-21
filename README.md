# Project 15 — Final checkpoint

Report: [metrics_export/final_checkpoint.pdf](metrics_export/final_checkpoint.pdf)
(source: [final_checkpoint.tex](metrics_export/final_checkpoint.tex)).
Repository: https://github.com/hungqbui/security_anaylytics_project

The project is organized into three folders:

- `data_preparation/`: shared data audit/splitting and perturbation notebooks,
  prepared prompts, concept pairs, generated text/reviews, `split.csv`, and
  `validation_roles.csv`.
- `experiments/`: frozen/LoRA and NLI experiment notebooks, the historical
  midsemester notebook, and the four `representations_*.npz` caches.
- `metrics_export/`: CPU analysis notebooks, prediction/logit and metric CSVs,
  figures, checker output, and the LaTeX/PDF report and table fragments.

Install packages **outside the notebooks**, from the repository root:

```bash
python -m pip install -r requirements.txt
```

The model experiments were executed on a Colab NVIDIA L4 with Python 3.13.16;
data preparation and cached analysis also run on Python 3.11 CPU. Keep all three
folders and `requirements.txt` together when using Colab. Notebooks resolve the
project root automatically from either the repository root or their own folder.

Run these notebooks top to bottom, in order:

1. `data_preparation/perturbation_preparation.ipynb` — deterministic tests, GPT
   paraphrases/reviews, validation perturbations and concept samples. It runs
   `data_preparation/data_preparation.ipynb` for downloads, audits and splits.
2. `experiments/final_experiment.ipynb` — frozen probe and 20 epochs for each of
   three LoRA seeds, validation selection, cached logits and representations.
3. `metrics_export/concept_reanalysis.ipynb` — fixed-sample concept similarities
   and Euclidean distances from cached representations; CPU only.
4. `experiments/zero_shot_nli.ipynb` — pretrained NLI detector with fixed class
   descriptions, including cached entailment logits.
5. `metrics_export/final_analysis.ipynb` — intervals, verdicts, seed spread,
   error categories and figures.
6. `metrics_export/concept_sample_size.ipynb` — repeated concept sampling and
   full-test references; CPU only.

Equivalent command, from the repository root:

```bash
jupyter execute --inplace --timeout=7200 data_preparation/perturbation_preparation.ipynb experiments/final_experiment.ipynb metrics_export/concept_reanalysis.ipynb experiments/zero_shot_nli.ipynb metrics_export/final_analysis.ipynb metrics_export/concept_sample_size.ipynb
```

For cached concept reanalysis without model inference or LoRA retraining:

```bash
jupyter execute --inplace --timeout=600 metrics_export/concept_reanalysis.ipynb metrics_export/concept_sample_size.ipynb
```

Prepared CSVs in `data_preparation/` make API regeneration optional. Regeneration
uses `gpt-6.1-sol`; configure the full OpenAI-compatible chat-completions URL and key:

```bash
export API_URL="http://localhost:8080/v1/chat/completions"
export OPENAI_API_KEY="your-key"
```

These variables are unnecessary when reusing the supplied CSVs. Delete only the
paraphrase/review CSVs you intend to regenerate. Generated text is nondeterministic;
the supplied CSVs define the reported set. No API credential or model checkpoint
is included.

`data_preparation/random_concept_pairs.csv` fixes the original 20 random test
pairs (10 per dataset; 40 distinct sources; seed 17). Its perturbation texts are
in `random_concept_perturbations.csv` in the same folder. The direction is the
difference between publisher-positive and publisher-negative class means, so
pairing order does not affect it. `metrics_export/random_concept_robustness.csv`
contains its cosines and 1,000 class/dataset-stratified bootstrap draws;
`random_concept_distances.csv` adds raw, relative and unit Euclidean distances.
The earlier screened quoted-control pairs and their metrics remain auxiliary
results with a narrower override concept.

`metrics_export/concept_sample_size.ipynb` checks 10, 20, 40, 60, 80, 100 and 112
pairs with 100 nested random subsets per size, using identical sources across
models/conditions. It also computes the reference from all 315 test prompts with
equal dataset weights and reproduces the original 20-pair values as a check.
Its results, summaries, model changes, full-test references and plot are named
`concept_sample_size_*` and `concept_full_test_reference.csv` in `metrics_export/`.
Bands describe sampling spread within the fixed pool, not population confidence
intervals or independent training runs. Appendix E documents the exact procedure.

`metrics_export/logits.csv` stores calibration/test raw logits, probabilities
and thresholds keyed by source, method, seed and condition. Probe logits are the
logistic decision function; NLI retains both description-entailment logits and
their difference. These support metric/threshold analysis without model inference.

`experiments/representations_frozen_15.npz` and the three
`experiments/representations_lora_<seed>.npz` files retain 768-dimensional original
and test-condition embeddings with source IDs/condition names. They contain no
model weights and load with `allow_pickle=False`. After changing the sample within
cached sources/conditions, rerun preparation and the CPU concept notebooks.
Caches use the reported pooling, 128-token budget and selected epochs; new input
conditions or extraction rules still require model inference.

The 269 validation rows have separate selection/calibration roles in
`data_preparation/validation_roles.csv`. `metrics_export/epoch_history.csv` records
all 20 epochs per seed, including fixed-selection-text drift. The best mean
clean/zero-width/homoglyph/base64 validation AUROC selects an in-memory state;
clean log loss breaks ties. Thresholds use separate calibration rows. No model
checkpoint files are written.

The final prediction schema is in `metrics_export/scores.csv`;
`metrics_export/midsemester_scores.csv` retains the original baseline.
`experiments/project15_checkpoint.ipynb` is historical: running it overwrites
`scores.csv` in `metrics_export/` with the old schema.

Download `check_final_checkpoint.py` from Canvas and run it separately from the
repository root:

```bash
python check_final_checkpoint.py --split data_preparation/split.csv --scores metrics_export/scores.csv --errors metrics_export/errors.csv --metric auroc --reference frozen --direction higher
```

The unedited output is `metrics_export/final_checker_output.txt`, also included
in the report appendix. Compile the report from its folder:

```bash
cd metrics_export
pdflatex final_checkpoint.tex
pdflatex final_checkpoint.tex
```

Data sources: [deepset/prompt-injections](https://huggingface.co/datasets/deepset/prompt-injections)
and [xTRam1/safe-guard-prompt-injection](https://huggingface.co/datasets/xTRam1/safe-guard-prompt-injection).
Notebooks pin dataset/model revisions. Publisher labels and automated reviews are
not independently verified semantic ground truth. AI assistance is disclosed in
the report.

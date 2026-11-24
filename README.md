# Project 15 — Code and cached representations

Repository: https://github.com/hungqbui/security_anaylytics_project

The repo contains notebooks, cached embeddings, package requirements and the
three CSVs explicitly required by the final checkpoint:

- `data_preparation/split.csv` — fixed train/validation/test membership and labels.
- `metrics_export/scores.csv` — scored test rows for each method and condition.
- `metrics_export/errors.csv` — sampled errors and their categories.

Reports, figures, checker output and all other generated CSVs remain excluded by
`.gitignore`.

- `data_preparation/`: raw-data auditing/splits, perturbation generation, and
  input reconstruction for cached analysis.
- `experiments/`: frozen/LoRA and zero-shot NLI notebooks, the historical
  midsemester notebook, and four representation caches.
- `metrics_export/`: CPU notebooks that calculate and export metrics.

Install packages outside the notebooks, from the repository root:

```bash
python -m pip install -r requirements.txt
```

Keep the three folders and `requirements.txt` together. Notebooks find the project
root from either the root directory or their own folder.

## Cached concept analysis — no GPU or API needed

```bash
jupyter execute --inplace --timeout=600 metrics_export/concept_reanalysis.ipynb metrics_export/concept_sample_size.ipynb
```

The notebooks automatically run `data_preparation/cached_data_preparation.ipynb`
when their inputs are absent. It reconstructs source labels/splits and the fixed
20-pair source-ID sample from the embedding cache, writing local CSV inputs.

- `concept_reanalysis.ipynb`: clean/perturbed concept cosines and raw, relative,
  and unit Euclidean distances, with class/dataset-stratified bootstrap intervals.
- `concept_sample_size.ipynb`: 10–112 pairs, 100 nested random subsets per size,
  matched model comparisons, and a reference using all 315 test prompts with
  equal dataset weights. Its bands are finite-pool sampling spreads, not
  independent-test-dataset confidence intervals.

Each `experiments/representations_*.npz` file contains the 768-dimensional original
and test-condition embeddings, source IDs, condition names, and the original
source labels/datasets/splits. Load with `allow_pickle=False`. No model weights
are stored. Caches cover frozen DeBERTa and LoRA seeds 15/16/17 at their selected
epochs, with the reported 128-token budget and last-layer mean pooling. Changing
samples or concept metrics within these cached sources needs no model inference;
new inputs or extraction rules do.

## Full experiment pipeline

Prepared text datasets and additional metric exports are not bundled. The three
committed CSVs preserve the submitted results. To regenerate
paraphrases and reviews, configure the full OpenAI-compatible chat-completions
endpoint and its key:

```bash
export API_URL="http://localhost:8080/v1/chat/completions"
export OPENAI_API_KEY="your-key"
```

Generation uses `gpt-6.1-sol`. The full pipeline requires access to that endpoint;
newly generated text can differ from the text represented in the supplied caches.

```bash
jupyter execute --inplace --timeout=7200 data_preparation/perturbation_preparation.ipynb experiments/final_experiment.ipynb metrics_export/concept_reanalysis.ipynb experiments/zero_shot_nli.ipynb metrics_export/final_analysis.ipynb metrics_export/concept_sample_size.ipynb
```

Preparation runs `data_preparation/data_preparation.ipynb` to download pinned
publisher data, audit duplicates, and make fixed train/validation/test splits.
LoRA uses 20 epochs for each of three seeds, with separate epoch-selection and
threshold-calibration validation roles. Model runs used Colab NVIDIA L4; the
cached-analysis notebooks run on CPU. `final_analysis.ipynb` expects the CSVs
produced by the full pipeline, including training history and NLI scores.

The pipeline writes ignored runtime CSVs, including raw detector logits and
probabilities in `metrics_export/logits.csv`. The historical
`experiments/project15_checkpoint.ipynb` writes the old prediction schema and
should be run separately from the final pipeline.

Download the course checker separately and run it on the committed files:

```bash
python check_final_checkpoint.py --split data_preparation/split.csv --scores metrics_export/scores.csv --errors metrics_export/errors.csv --metric auroc --reference frozen --direction higher
```

Data sources: [deepset/prompt-injections](https://huggingface.co/datasets/deepset/prompt-injections)
and [xTRam1/safe-guard-prompt-injection](https://huggingface.co/datasets/xTRam1/safe-guard-prompt-injection).
Notebooks pin dataset/model revisions. Publisher labels and automated reviews are
not independently verified semantic ground truth. OpenAI Codex assisted with
implementation and analysis; GPT-6.1-sol generated the original paraphrases and
supplied automated intent reviews.

## Interactive presentation

[Open the presentation on GitHub Pages](https://hungqbui.github.io/security_anaylytics_project/).

The presentation uses animated SVG and includes the supplementary slides after the references. Click or press Space/Right to advance, Left to go back, F for fullscreen, and N for speaker notes.

GitHub Pages serves the self-contained `docs/index.html` from the `main` branch’s `/docs` folder.

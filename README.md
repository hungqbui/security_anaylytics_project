# Project 15 — Final checkpoint

Report: `final_checkpoint.pdf` (source: `final_checkpoint.tex`).
Repository: https://github.com/hungqbui/security_anaylytics_project

Install packages **outside the notebooks** with `python -m pip install -r requirements.txt`.
The model experiments were executed on a Colab Tesla T4 with Python 3.13.16;
data generation and report analysis also run on Python 3.11 CPU.

Run these notebooks top to bottom, in order:

1. `final_data.ipynb` — publisher downloads, original splits, deterministic perturbations,
   GPT paraphrases and automated semantic review.
2. `final_experiment.ipynb` — frozen probe, three LoRA seeds, representation/concept metrics.
3. `zero_shot_nli.ipynb` — pretrained NLI detector using fixed class descriptions.
4. `final_analysis.ipynb` — intervals, verdicts, seed spread, error categories, figure.

The equivalent single command is:

```bash
jupyter execute --inplace --timeout=1800 final_data.ipynb final_experiment.ipynb zero_shot_nli.ipynb final_analysis.ipynb
```

The generated CSVs make API regeneration optional. To regenerate them, configure
`OPENAI_API_KEY` and use the notebook's OpenAI-compatible endpoint at `localhost:8080`
with `gpt-6.1-sol`. Delete only the generated paraphrase/review CSVs you intend to regenerate.
Generated text is nondeterministic; the supplied CSVs define the reported evaluation set.
No API credential, model checkpoint or embedding cache is required in the repository.

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

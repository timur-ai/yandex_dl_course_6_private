## Relevant Files

- `notebooks/LLM_Pretrain_SFT.ipynb` - Target notebook where markdown annotations will be inserted.
- `assignment/assignment.md` - Source of requirements to map each code cell against.
- `tasks/0002-prd-notebook-markdown-annotations.md` - PRD defining template, mapping, and acceptance criteria.

### Notes

- Keep all inserted markdown in English to comply with the repo Language Policy.
- Do not alter code cells or execution order; insert markdown directly before each code cell only.
- Decision: For setup/infra-only cells, annotate using "closest step" with a brief justification.
- Decision: For cells touching multiple requirements, list all applicable steps and explain the relationship succinctly.

### Template

```
Assignment mapping

- Section and steps: <Pretrain|Post-train SFT> – Step(s) N[, M]
- Why it matters: <3–5 sentences explaining intent and how the code fulfills it>
- Requirement paraphrase (English): “<short paraphrase of the relevant requirement>” (ref: Lx–Ly in assignment.md)
- Notes: <“closest step” if mapping is approximate; mention multi‑step relationship if applicable>
```

### Reference Mappings (from PRD)

| Cell | Section/Steps | Assignment lines |
| --- | --- | --- |
| 0 | Closest: Pretrain–5; SFT–2 | L25–L29, L121–L125 |
| 1 | Closest: Pretrain–6; SFT–2 | L47 |
| 2 | Pretrain–1; SFT–1 | L19, L121–L124 |
| 3 | Pretrain–2 | L20–L24 |
| 4 | Pretrain–2 | L21, L24 |
| 5 | Pretrain–2 | L22 |
| 6 | Pretrain–2 | L23 |
| 7 | Pretrain–2 (chunking prep) | L24 |
| 8 | Pretrain–2 | L20–L24 |
| 9 | Pretrain–3 | L25–L27 |
| 10 | Pretrain–3 | L25–L27 |
| 11 | Pretrain–3 | L25–L27 |
| 12 | Pretrain–3 | L25–L27 |
| 13 | Pretrain–4 | L27 |
| 14 | Pretrain–4 | L27 |
| 15 | Pretrain–4; 6 | L27, L47 |
| 16 | Pretrain–5 | L28–L29 |
| 17 | Pretrain–6 | L47 |
| 18 | Pretrain–6 | L31–L48 |
| 19 | Pretrain–6 | L47 |
| 20 | Closest: Pretrain–7 | L48 |
| 21 | SFT–1 | L119–L124 |
| 22 | SFT–1 | L123 |
| 23 | SFT–2 (prep) | L52 |
| 24 | SFT–2 | L124–L125 |
| 25 | SFT–2 | L124–L125 |
| 26 | SFT–3 | L54–L63, L126–L135 |
| 27 | Closest: SFT–3 | L126–L135 |
| 28 | Closest: Pretrain–6 / SFT–2 | L47, L124–L125 |
| 29 | Pretrain–7; SFT–3 | L48, L126–L135 |
| 30 | Closest: Pretrain–7; SFT–3 | L48, L126–L135 |

### Drafts for cells 21–30

#### Draft – Cell 21
- Section and steps: Post‑train SFT Step 1
- Why it matters: Loads the Alpaca RU dataset from parquet shards to feed SFT. Confirms row count and column names, preventing schema surprises later. Serves as the entry point for converting raw instruction data into dialog format. Early inspection reduces downstream mapping errors.
- Requirement paraphrase (English): “Prepare the d0rj/alpaca-cleaned-ru dataset for training.” (ref: L119–L124)
- Notes: single‑step.

#### Draft – Cell 22
- Section and steps: Post‑train SFT Step 1
- Why it matters: Maps input→system, instruction→user, output→assistant, enforcing the dialog schema required by chat templates. Validates that required columns exist and removes unused fields. This normalization is critical for consistent prompt formatting. Keeps the dataset minimal and targeted.
- Requirement paraphrase (English): “Represent dialog data as transformers Dataset: input=system, instruction=user, output=assistant.” (ref: L123)
- Notes: single‑step.

#### Draft – Cell 23
- Section and steps: Post‑train SFT Step 2 (prep)
- Why it matters: Downloads the Qwen2.5‑0.5B base and tokenizer, ensuring pad/eos are configured for batching/generation. Establishes the model/dtype for the SFT stage. This is the foundation on which chat‑formatted sequences will be optimized. Correct token IDs avoid shape or decoding issues.
- Requirement paraphrase (English): “Use the Qwen 0.5B base model for SFT.” (ref: L52)
- Notes: preparation for training.

#### Draft – Cell 24
- Section and steps: Post‑train SFT Step 2
- Why it matters: Builds textual conversations with the tokenizer’s chat template and sets up SFTTrainer arguments and splits. This standardizes prompts across the dataset and configures optimization parameters. The trainer attaches the model, tokenizer, train/test splits, and text field. It is the central wiring for SFT.
- Requirement paraphrase (English): “Fine‑tune with TRL SFTTrainer on dialog data.” (ref: L124–L125)
- Notes: single‑step.

#### Draft – Cell 25
- Section and steps: Post‑train SFT Step 2
- Why it matters: Runs the SFT training loop and prints core metrics. Confirms that data formatting, optimization, and device settings are correctly integrated. Produces a model adapted to instruction following in Russian. Metrics provide a quick sanity check.
- Requirement paraphrase (English): “Run SFT on the prepared dataset.” (ref: L124–L125)
- Notes: single‑step.

#### Draft – Cell 26
- Section and steps: Post‑train SFT Step 3
- Why it matters: Generates answers for four evaluation prompts to qualitatively assess instruction‑following. Uses deterministic generation for comparability. Shows end‑to‑end readiness by formatting prompts via the same chat template used in training. Printed Q/A pairs facilitate reviewer judgment.
- Requirement paraphrase (English): “Check adequacy on four specified questions and show outputs.” (ref: L54–L63, L126–L135)
- Notes: single‑step.

#### Draft – Cell 27
- Section and steps: Closest – Post‑train SFT Step 3
- Why it matters: Saves the SFT‑tuned model and tokenizer to disk for reproducibility and reuse. While persistence is not explicitly mandated, it enables others to validate and deploy the fine‑tuned model. Artifacts are organized under `outputs/sft/final`. This supports grading and future experiments.
- Requirement paraphrase (English): “Present evaluation outputs; keep artifacts ready for review.” (ref: L126–L135)
- Notes: closest step.

#### Draft – Cell 28
- Section and steps: Closest – Pretrain Step 6 / SFT Step 2
- Why it matters: Reports environment and library versions, model parameter counts, and dataset sizes. Aids reproducibility and debugging by capturing runtime context. Although not explicitly listed, it is operationally useful for reviewers and future runs. Prevents silent drift across environments.
- Requirement paraphrase (English): “Train using the specified tools and datasets; ensure reproducibility.” (ref: L47, L124–L125)
- Notes: closest step.

#### Draft – Cell 29
- Section and steps: Pretrain Step 7; Post‑train SFT Step 3
- Why it matters: Prints the last epoch’s pretrain generations and the SFT evaluation answers in one place. This satisfies the requirement to leave generation outputs in the notebook for review. It consolidates qualitative evidence of both stages. Simplifies manual grading.
- Requirement paraphrase (English): “Generate and display results in the notebook.” (ref: L48, L126–L135)
- Notes: multi‑step mapping.

#### Draft – Cell 30
- Section and steps: Closest – Pretrain Step 7; Post‑train SFT Step 3
- Why it matters: Persists generations and evaluation outputs to JSON/CSV reports for external inspection and comparison. While optional, these artifacts are convenient for downstream analysis and archiving. They mirror the on‑screen displays with machine‑readable formats. Enhances traceability beyond the notebook.
- Requirement paraphrase (English): “Make the final results available for evaluation.” (ref: L48, L126–L135)
- Notes: closest step.

## Tasks

- [x] 1.0 Review PRD and confirm mapping approach
  - [x] 1.1 Read `tasks/0002-prd-notebook-markdown-annotations.md` sections 4, 6, 10, 11
  - [x] 1.2 Confirm English-only and 3–5 sentence requirement per template
  - [x] 1.3 Verify notebook code cell count matches PRD mapping table (cells 0–30)
  - [x] 1.4 Decide handling for “closest step” and multi-step mappings
- [x] 2.0 Prepare annotation template and references
  - [x] 2.1 Copy the markdown template from PRD for reuse
  - [x] 2.2 Extract per-cell section/step numbers and assignment line ranges
  - [x] 2.3 Draft explanations for cells 0–10 (3–5 sentences each, English)
  - [x] 2.4 Draft explanations for cells 11–20
  - [x] 2.5 Draft explanations for cells 21–30
- [x] 3.0 Insert mapping markdown before every code cell
  - [x] 3.1 Insert markdown before code cells 0–10 in `notebooks/LLM_Pretrain_SFT.ipynb`
  - [x] 3.2 Insert markdown before code cells 11–20
  - [x] 3.3 Insert markdown before code cells 21–30
  - [x] 3.4 Ensure no code modifications and original cell order preserved
- [x] 4.0 Verify coverage, consistency, and references
  - [x] 4.1 Confirm 100% of code cells are annotated (count check)
  - [x] 4.2 Check template/style consistency across all inserted markdown
  - [x] 4.3 Validate `assignment.md` line ranges exist and match intended steps
  - [x] 4.4 Visual inspection: open notebook and spot-check rendering of several cells
- [x] 5.0 Save and commit updated notebook and artifacts
  - [x] 5.1 Save updated `LLM_Pretrain_SFT.ipynb`
  - [x] 5.2 Commit changes with message: "feat: annotate LLM notebook cells with assignment mappings"
  - [x] 5.3 Share updated notebook for review



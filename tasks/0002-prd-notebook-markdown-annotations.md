## Notebook Markdown Annotations PRD

### 1. Introduction / Overview
This feature adds an explanatory markdown cell immediately before every code cell in `notebooks/LLM_Pretrain_SFT.ipynb`. Each inserted markdown explains which requirement(s) from `assignment/assignment.md` the following code cell fulfills and why. The goal is to make the notebook self-evident to reviewers and junior developers by explicitly mapping implementation to assignment steps for both Pretrain and Post-train SFT sections.

### 2. Goals
- Explicitly map every code cell to the relevant assignment step(s) with short rationale.
- Include a brief, human-readable reference to the assignment step(s) and a concise paraphrase of the relevant requirement text.
- Keep language clear and consistent in English per repository policy.
- Ensure 100% coverage and consistent formatting across the notebook.

### 3. User Stories
- As a reviewer, I want to see which assignment requirement a code cell implements so I can quickly verify scope and correctness.
- As a junior developer, I want a short explanation and cited step(s) so I can learn the flow and replicate the approach.
- As an instructor, I want the notebook to remain readable without digging into the assignment document.

### 4. Functional Requirements
1. Insert a markdown cell immediately before every code cell in `notebooks/LLM_Pretrain_SFT.ipynb`.
2. Each inserted markdown must contain 3–5 sentences that:
   - Name the assignment section and step number(s) (e.g., “Pretrain – Step 2”).
   - Briefly explain the intent of the step(s) and why this cell’s code implements them.
   - Provide a short English paraphrase of the relevant assignment text and include a line-number citation to `assignment/assignment.md` (e.g., “ref: L19–L27”).
   - For multi-step cells, list all applicable steps and explain the relationship succinctly.
   - For setup/infra cells not explicitly described in the assignment, reference the closest relevant step and label as “closest step”.
3. Do not modify any code cells or their execution order/content.
4. Do not insert markdown before existing markdown cells; annotate code cells only.
5. Language for all newly added markdown must be English (per repository Language Policy). Do not include non‑English text; use English paraphrases and cite line ranges from `assignment/assignment.md` instead of quoting non‑English text directly.
6. Maintain consistent style and template across all insertions.

### 5. Non‑Goals (Out of Scope)
- Writing or refactoring any code in the notebook.
- Automating cell insertion with scripts; this PRD specifies a one‑time manual edit.
- Adding external documentation links or citations to third‑party sources in the inserted markdown.

### 6. Design Considerations
- Keep each explanation compact (3–5 sentences). Prioritize clarity and traceability.
- Reference steps with clear labels: “Pretrain – Step N” or “Post‑train SFT – Step N”.
- Since `assignment/assignment.md` is written in Russian, comply with the repository’s English‑only policy by paraphrasing the requirement in English and citing the exact line range, e.g., “ref: L21–L27”.

#### Markdown Template (to use before every code cell)
```
Assignment mapping

- Section and steps: <Pretrain|Post-train SFT> – Step(s) N[, M]
- Why it matters: <3–5 sentences explaining intent and how the code fulfills it>
- Requirement paraphrase (English): “<short paraphrase of the relevant requirement>” (ref: Lx–Ly in assignment.md)
- Notes: <“closest step” if mapping is approximate; mention multi‑step relationship if applicable>
```

### 7. Technical Considerations
- Perform a one‑time manual edit in Jupyter to add markdown cells before each code cell. Do not change code cells.
- Use the exact cell ordering present at the time of this PRD. If cells are reordered later, re‑apply the mapping accordingly.
- Keep formatting minimal; no images or external links required.

### 8. Success Metrics
- 100% of code cells are preceded by a mapping markdown cell.
- No contradictions between explanations and `assignment/assignment.md`.
- Consistent template/style throughout.
- English language used throughout the inserted markdown.

### 9. Open Questions
- None identified.

### 10. Complete Cell‑to‑Assignment Mapping Guide
Use the following guide to apply the template to each code cell by index. Line references (Lx–Ly) refer to `assignment/assignment.md` as currently committed.

| Cell | Summary of Code | Section/Steps | Rationale and Reference |
| --- | --- | --- | --- |
| 0 | Imports (datasets, tokenizers, transformers, trl) | Closest step: Pretrain – Step 5; Post‑train SFT – Step 2 | Library setup required to build tokenizer/model/trainers. ref: L25–L29, L121–L125 |
| 1 | Seeding, device/dtype setup | Closest step: Pretrain – Step 6; Post‑train SFT – Step 2 | Reproducible training and device selection for training. ref: L47 |
| 2 | Validate data directories; list corpus/parquet | Pretrain – Step 1; Post‑train SFT – Step 1 | Confirms required data presence before preprocessing/SFT. ref: L19, L121–L124 |
| 3 | Scan corpus file stats | Pretrain – Step 2 | Initial corpus inspection to inform preprocessing. ref: L20–L24 |
| 4 | Read paragraphs, deduplicate lines/paragraphs | Pretrain – Step 2 | Deduplication and paragraph grouping. ref: L21, L24 |
| 5 | Filter non‑Cyrillic lines | Pretrain – Step 2 | Enforce Cyrillic‑only constraint. ref: L22 |
| 6 | Normalize punctuation/whitespace | Pretrain – Step 2 | Handle repeated punctuation and spacing. ref: L23 |
| 7 | Join paragraphs; defer chunking to tokens | Pretrain – Step 2 (chunking prep) | Prepare text for token‑level packing later. ref: L24 |
| 8 | Preprocessing summary | Pretrain – Step 2 | Summarize pre/post stats to verify preprocessing scope. ref: L20–L24 |
| 9 | Train BPE tokenizer (≈3k vocab) + BOS/EOS | Pretrain – Step 3 | Build domain tokenizer and add special tokens. ref: L25–L27 |
| 10 | Save/reload tokenizer; inspect IDs | Pretrain – Step 3 | Persist and verify tokenizer assets. ref: L25–L27 |
| 11 | Round‑trip encode/decode assertion | Pretrain – Step 3 | Sanity‑check tokenizer fidelity. ref: L25–L27 |
| 12 | Export special token IDs | Pretrain – Step 3 | Capture IDs for downstream use. ref: L25–L27 |
| 13 | Token‑level packing to 512; build Dataset | Pretrain – Step 4 | Prepare sequences and attention masks for LM. ref: L27 |
| 14 | Split train/validation | Pretrain – Step 4 | Establish evaluation split. ref: L27 |
| 15 | HF fast tokenizer + data collator | Pretrain – Step 4; Step 6 | Collation for causal LM training. ref: L27, L47 |
| 16 | Define ~150M Llama config/model | Pretrain – Step 5 | Matches suggested architecture scale. ref: L28–L29 |
| 17 | TrainingArguments + Trainer | Pretrain – Step 6 | Trainer, weight decay, effective batch size 64–128 via accumulation. ref: L47 |
| 18 | Test prompts + epoch‑end generation callback | Pretrain – Step 6 | Required prompts and validation callback logic. ref: L31–L48 |
| 19 | Train + evaluate (loss/ppl) | Pretrain – Step 6 | Execute training and report validation metrics. ref: L47 |
| 20 | Save pretrain artifacts | Closest step: Pretrain – Step 7 | Persist outputs; not explicitly required but supports review. ref: L48 |
| 21 | Load Alpaca RU parquet | Post‑train SFT – Step 1 | Load dataset for instruction tuning. ref: L119–L124 |
| 22 | Map fields to system/user/assistant | Post‑train SFT – Step 1 | Transform to dialog schema. ref: L123 |
| 23 | Load Qwen 0.5B + tokenizer; set pad | Post‑train SFT – Step 2 (prep) | Initialize base model for SFT. ref: L52 |
| 24 | Build chat texts; SFTTrainer + args | Post‑train SFT – Step 2 | Configure trainer for instruction tuning. ref: L124–L125 |
| 25 | Run SFT training | Post‑train SFT – Step 2 | Execute SFT phase. ref: L124–L125 |
| 26 | Generate answers for 4 questions | Post‑train SFT – Step 3 | Evaluate with specified prompts. ref: L54–L63, L126–L135 |
| 27 | Save SFT artifacts | Closest step: Post‑train SFT – Step 3 | Persist outputs; aids review and reuse. ref: L126–L135 |
| 28 | Environment summary | Closest step: Pretrain – Step 6 / SFT – Step 2 | Ancillary run metadata for reproducibility. ref: L47, L124–L125 |
| 29 | Consolidated pretrain gens + SFT answers | Pretrain – Step 7; Post‑train SFT – Step 3 | Display generations as required. ref: L48, L126–L135 |
| 30 | Persist reports to JSON/CSV | Closest step: Pretrain – Step 7; Post‑train SFT – Step 3 | Optional artifacts for grading/review. ref: L48, L126–L135 |

### 11. Acceptance Criteria
- a) 100% of code cells preceded by a mapping markdown block
- b) No contradictions with `assignment/assignment.md`
- c) Consistent style and template
- d) English‑only wording in inserted markdown



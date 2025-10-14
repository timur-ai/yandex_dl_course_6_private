## PRD: Russian LLM Pretraining and SFT Fine-tuning

### Introduction / Overview
This feature delivers two training pipelines for a small decoder-only language model focused on Russian literature domain. All implementation MUST be contained within a single Jupyter Notebook located at `notebooks/LLM_Pretrain_SFT.ipynb`. No standalone scripts or external configuration files are permitted; all steps must be implemented as notebook cells.
- Pretrain: learn language structure from a local corpus of Russian literature to produce coherent continuations.
- Post-train SFT: supervised fine-tune a stronger base model on a local instruction dataset to improve instruction following in Russian.

Inputs are local only (provided under `data/`). Outputs are trained artifacts, minimal generation demos, and logged metrics suitable for grading, all produced and displayed within the single notebook.

### Goals
- Reduce pretrain validation perplexity compared to an untrained baseline on a held-out split from the literature corpus.
- Generate coherent Russian continuations for a fixed set of 10 literature-style prompts (no garbled output, consistent topic and grammar).
- Train a custom BPE tokenizer with a small vocabulary appropriate for the corpus (~3k tokens) and integrate BOS/EOS tokens.
- Fine-tune Qwen2.5-0.5B on a local instruction dataset; improve response plausibility on 4 evaluation questions relative to base model outputs.
- Produce deterministic, reproducible runs (fixed seeds; logged configs; same results within noise on repeated runs).

### User Stories
- As a course student, I want clear, reproducible steps to train and evaluate both stages so my instructor can verify results quickly.
- As a reviewer/instructor, I want logged metrics and short generation samples to judge language quality without running long jobs.
- As an engineer, I want simple, minimal code and configurations that are easy to audit and modify.

### Functional Requirements
All functionality below MUST be implemented as cells in `notebooks/LLM_Pretrain_SFT.ipynb`.

1. Data ingestion (pretrain):
   - Read all plain-text literature files under `data/corpus/`.
   - Concatenate them into a single text stream; record basic statistics (num files, total chars, unique lines after dedup).

2. Preprocessing (pretrain):
   - Deduplicate exact duplicate lines/paragraphs.
   - Filter out lines containing non-Cyrillic letters; keep standard punctuation and whitespace.
   - Normalize repeated punctuation (e.g., sequences of `!`, `?`, `…`) to a single symbol; collapse excessive spaces.
   - Chunk text to fit context length 512 with room for BOS/EOS; join short fragments to minimize padding.

3. Tokenizer (pretrain):
   - Train a BPE tokenizer on the preprocessed corpus with vocabulary size ≈ 3,000.
   - Include special tokens: `<unk>`, `<pad>`, `<bos>`, `<eos>`.
   - Save the tokenizer as `tokenizer.json`; verify round-trip encode/decode on a sample.

4. Dataset objects (HF Datasets):
   - Represent tokenized pretrain data as `datasets.Dataset` with fields `input_ids`, `attention_mask`, `labels` (labels = input_ids shifted by 1 via data collator or mapping step).
   - Provide a small held-out validation split (e.g., 95/5 split by documents or stratified by file).

5. Model (pretrain):
   - Initialize a ~150M parameter decoder-only Transformer with a LLaMA-like config (example: `hidden_size=1024`, `intermediate_size=1536`, `num_hidden_layers=16`, `num_attention_heads=16`, `num_key_value_heads=8`, `max_position_embeddings=512`).
   - Tie embeddings and LM head; set `pad_token_id` and special tokens matching the tokenizer.

6. Training (pretrain):
   - Use Hugging Face `Trainer` for causal language modeling (MLM disabled).
   - Effective batch size between 64 and 128 via per-device batch size and gradient accumulation.
   - Use AdamW with `weight_decay` > 0 (e.g., 0.01), linear LR schedule with warmup (e.g., 3% of steps), mixed precision if GPU supports it.
   - Evaluate at least once per epoch; log training/validation loss; compute validation perplexity from loss.

7. Generation callback (pretrain):
   - After each epoch, run a deterministic generation pass on 10 fixed literature-style prompts (provided by the assignment; do not embed the text in the repo per language policy) and store the outputs alongside the epoch number.
   - The exact set of 10 test prompts from the assignment must be used at epoch end, and the generations must be displayed in a Jupyter Notebook cell for grading (prompts not embedded in this PRD per repository language policy).

8. Artifacts (pretrain):
   - Save final model weights, tokenizer, and training args from within the notebook cells; optionally keep the best checkpoint by validation loss.
   - The single notebook MUST include: (a) corpus stats, (b) a brief training log snapshot, and (c) final epoch-end generations for all 10 test prompts.

9. Data ingestion (SFT):
   - Load the local instruction dataset under `data/alpaca-cleaned-ru/` (Parquet provided) into `datasets.Dataset`.
   - Always include a 'system' field (empty string if unavailable). Map fields strictly as: input → system, instruction → user, output → assistant.
   - Maintain Russian text content in the dataset only (not duplicated in docs).

10. Base model (SFT):
   - Load `Qwen2.5-0.5B` base model and matching tokenizer from the Hugging Face Hub.
   - Set `pad_token_id` and EOS appropriately for generation.

11. Training (SFT):
   - Use TRL `SFTTrainer` with conversational format.
   - Effective batch size 64–128 via gradient accumulation; evaluate minimally (loss); enable mixed precision if available.
   - Keep training simple (no PEFT/LoRA; no RLHF) to satisfy course scope.

12. Evaluation (SFT):
   - Generate responses for 4 fixed evaluation questions (provided by the assignment; do not embed the text in the repo per language policy).
   - Qualitative criteria: fluent Russian, relevant, sensible; absence of obvious corruption, XML noise, or language drift.
   - Use the exact 4 evaluation questions from the assignment and show the generations in the Jupyter Notebook (Russian prompts/answers are allowed in the notebook; they are not embedded in this PRD per repository language policy).

13. Reproducibility & environment:
   - Fix random seeds across libraries.
   - Use `uv` for dependency management and `ruff` for linting as per repo standards.
   - Run the notebook via `uv run jupyter lab` or `uv run jupyter notebook` to ensure a consistent environment.
   - All user-facing logs and documentation in English; model input/output content remains Russian where appropriate.

### Non-Goals (Out of Scope)
- RLHF, DPO, or preference-based post-training.
- PEFT/LoRA or quantization; use full-precision or mixed precision only.
- Multi-node distributed training; assume single machine with optional GPU.
- External data downloads during grading; rely solely on provided `data/` contents.

### Design Considerations
- Small vocabulary tokenizer (~3k) fits the narrower language variety of classic literature and reduces model size; aligns with assignment guidance.
- Context length fixed to 512 for faster training and simpler memory planning.
- Generation samples are deterministic (fixed sampling configuration) for fair comparison across epochs.

### Technical Considerations
- Libraries: Hugging Face Transformers (Trainer, causal LM), Tokenizers (BPE), Datasets (local parquet/text), TRL (SFTTrainer), PyTorch backend.
- Data formats: pretrain uses tokenized `Dataset` with collator for shifting labels; SFT uses conversational records with roles `system`, `user`, `assistant`.
- Hardware: single GPU recommended for time; CPU fallback permitted with longer runtime.
- Compliance: repository language policy requires English docs; Russian strings remain in datasets and runtime-only outputs.
 - Deliverables include a single Jupyter Notebook executed on the course VM with visible generation outputs for pretrain (10 prompts) and SFT (4 questions).

### Success Metrics
- Pretrain: validation perplexity decreases across epochs; final perplexity lower than early-epoch baseline.
- Pretrain: qualitative generations on 10 prompts are coherent and on-topic.
- Tokenizer: trained artifact loads and round-trips without errors; special tokens behave as expected.
- SFT: qualitative improvement over base model on 4 evaluation questions (fewer artifacts, more relevant answers).
- Reproducibility: rerunning with same seed yields comparable metrics and identical saved configs.

### Deliverables
- A single Jupyter Notebook at `notebooks/LLM_Pretrain_SFT.ipynb` containing:
  - Pretrain: data stats, training/eval logs, and generation outputs for the 10 assignment prompts.
  - SFT: training/eval logs and generation outputs for the 4 assignment questions.
- Saved model/tokenizer artifacts for pretrain, and final SFT model output, produced by cells within the notebook.

### Acceptance Criteria
- One and only one implementation artifact: the notebook `notebooks/LLM_Pretrain_SFT.ipynb` (no external `.py` or `.yaml` files).
- Notebook shows the exact 10 pretrain prompts and 4 SFT questions with visible generations.
- Pretrain uses 512 context, BPE ~3k, Trainer with `weight_decay` > 0, batch size 64–128, and epoch-end generation callback.
- SFT uses Qwen2.5-0.5B and TRL `SFTTrainer` with strict role mapping: system/user/assistant (system non-optional).

### Open Questions
1. Minimum acceptable validation perplexity target for pretrain to pass grading?
2. Exact compute/time budget constraints (epochs/steps caps) for each stage on the grading VM?
3. Required artifact retention policy (keep all checkpoints vs. best-only)?
4. Should we freeze any layers during SFT to reduce overfitting, or fully fine-tune?

### References (via context7)
- Transformers: Trainer, CLM, generation, LLaMA config usage — /huggingface/transformers (retrieved 2025-10-14)
- Tokenizers: BPE training, special tokens, save/load — /huggingface/tokenizers (retrieved 2025-10-14)
- TRL: `SFTTrainer` API and dataset formats — /huggingface/trl (retrieved 2025-10-14)
- Datasets: local parquet/text loading, map/filter/split — /huggingface/datasets (retrieved 2025-10-14)



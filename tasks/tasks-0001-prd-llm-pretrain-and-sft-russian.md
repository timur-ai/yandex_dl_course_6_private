## Relevant Files

- `notebooks/LLM_Pretrain_SFT.ipynb` - Single notebook containing all preprocessing, tokenizer training, dataset creation, model training, epoch-end generations, SFT, evaluation, and artifact saving. No separate scripts/configs.
- `pyproject.toml` - Dependency and tooling context (uv, ruff, transformers, tokenizers, datasets, trl, torch).

### Notes

- Use `uv` exclusively:
  - Create/sync: `uv sync`
  - Lint: `uv run ruff check --fix --unsafe-fixes .`
  - Run notebook: `uv run jupyter lab` or `uv run jupyter notebook`
- Implement all steps as notebook cells; do not create `.py` or `.yaml` files.
- Keep logs/docs in English per repository language policy; dataset strings and model generations remain Russian where appropriate.
- Ensure determinism via fixed seeds across libraries; use mixed precision when available.

## Tasks

- [x] 1.0 Notebook setup and environment (uv, ruff, imports, fixed seeds)
  - [x] 1.1 Add intro markdown cell: goals, constraints (single notebook)
  - [x] 1.2 Add imports cell (datasets, tokenizers, transformers, trl, torch, numpy, random)
  - [x] 1.3 Add seed cell: set seeds across random/np/torch; print device/dtype and CUDA availability
  - [x] 1.4 Verify data paths exist: `data/corpus/`, `data/alpaca-cleaned-ru/`
  - [x] 1.5 Ensure `pyproject.toml` declares deps; run via `uv`

- [x] 2.0 Pretrain data ingestion and preprocessing cells (stats, dedup, Cyrillic filter, normalization, chunking to 512)
  - [x] 2.1 Read all `.txt` under `data/corpus/`; show counts and basic stats
  - [x] 2.2 Deduplicate exact lines/paragraphs; report uniques
  - [x] 2.3 Filter out lines with non-Cyrillic letters; keep punctuation/whitespace
  - [x] 2.4 Normalize repeated punctuation and collapse excessive spaces
  - [x] 2.5 Chunk text for context length 512 (reserve BOS/EOS); join short fragments
  - [x] 2.6 Display pre/post preprocessing corpus stats (files, chars, unique lines)

- [x] 3.0 Tokenizer training and validation cells (~3k BPE with special tokens)
  - [x] 3.1 Train BPE tokenizer (≈3,000 vocab) with `<unk>`, `<pad>`, `<bos>`, `<eos>`
  - [x] 3.2 Save `tokenizer.json` from the notebook; reload to verify
  - [x] 3.3 Round-trip encode/decode a sample; assert equality
  - [x] 3.4 Print special token IDs; set `pad_token_id`

- [x] 4.0 Pretrain dataset/model/training with epoch-end deterministic generations
  - [x] 4.1 Tokenize corpus; build `datasets.Dataset` with `input_ids`, `attention_mask`
  - [x] 4.2 Create 95/5 train/validation split
  - [x] 4.3 Configure CLM data collator (labels shifted by 1 via collator or map)
  - [x] 4.4 Define ~150M decoder-only config (LLaMA-like); tie embeddings and LM head
  - [x] 4.5 Set Trainer: AdamW (weight_decay>0), linear schedule with ~3% warmup, grad accumulation for effective 64–128, mixed precision if available
  - [x] 4.6 Implement epoch-end deterministic generation for the 10 fixed prompts; store/display outputs per epoch
  - [x] 4.7 Train for N epochs; log train/val loss and compute validation perplexity
  - [x] 4.8 Save final model weights and training args from within the notebook

- [x] 5.0 SFT dataset mapping and TRL `SFTTrainer` training/evaluation cells
  - [x] 5.1 Load Parquet under `data/alpaca-cleaned-ru/` into `datasets.Dataset`
  - [x] 5.2 Map fields strictly: input→system, instruction→user, output→assistant (ensure non-optional system with empty string default)
  - [x] 5.3 Load `Qwen2.5-0.5B` and tokenizer; set `pad_token_id`/EOS
  - [x] 5.4 Configure `SFTTrainer` with conversational format; effective batch 64–128 via accumulation; minimal eval (loss); mixed precision
  - [x] 5.5 Train and display training logs
  - [x] 5.6 Generate responses for the 4 fixed evaluation questions; display outputs

- [x] 6.0 Final notebook cells: save artifacts and display all required generations
  - [x] 6.1 Save artifacts: `tokenizer.json`, pretrain model, final SFT model
  - [x] 6.2 Print run config, seeds, environment summary for reproducibility
  - [x] 6.3 Display consolidated outputs: 10 pretrain generations and 4 SFT answers
  - [x] 6.4 (Optional) Persist per-epoch generation outputs to JSON/CSV for review
  - [x] 6.5 Add final markdown cell with rerun instructions (`uv run jupyter lab`)



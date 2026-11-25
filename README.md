# Prahlada Safety Audit — Datasets

> ⚠️ **Content warning.** This repository contains prompts that ask for harmful content
> (violence, illicit activity, drugs, hacking, fraud, harassment and more), and
> **model-written harmful completions** to those prompts. It is released for AI safety
> research only. Read the [intended use](#intended-use) section before using it.

Datasets from the [Prahlada Safety Audit](https://github.com/AaryaJain05/Prahlada-Safety-Audit),
a study of whether safety fine-tuning **removes** harmful representations from a language
model's internal activations or only **suppresses** them at the output. The study runs in two
registers, **English** and **Romanized Hindi** (Hindi written in Latin script), to test
whether safety training transfers across languages.

Code, experiments and findings live in the research repository. This repository holds the data only.

## Contents

| Path | Rows | What it is |
|---|---|---|
| [`v2/`](v2/) | 10,000 | **Main dataset.** 5,000 harmful items, each with a paired benign twin, plus a leakage-safe train/eval split |
| [`v1/`](v1/) | 5,000 + 200 | Original harmful source set, and a 200-record pilot set |
| [`audit_set/`](audit_set/) | 360 prompts, 720 rows × 3 | Representation-audit prompts across 12 harm categories, with responses from three model checkpoints |

Everything is UTF-8 JSON or JSONL (one JSON object per line).

---

## `v2/` — paired harmful / benign dataset (main)

| File | Rows | Description |
|---|---|---|
| `harmful_base.jsonl` | 2,796 | De-duplicated harmful items from the v1 source set (`src_index` points into `v1/harmful_eng_rh_data_5000.json`) |
| `harmful_new.jsonl` | 2,204 | Additional harmful items generated with Llama-3.3-70B-Instruct to reach 5,000 |
| `benign.jsonl` | 5,000 | Benign twins, paired 1:1 with a harmful item in the same category |
| `train.jsonl` | 8,000 | Training split: 4,000 harmful + 4,000 benign |
| `eval.jsonl` | 2,000 | Held-out split: 1,000 harmful + 1,000 benign, never used in training |
| `split.json` | — | `{item_id: "train" \| "eval"}`, the source of truth for the split |

**Categories:** `violence`, `gambling`, `illicit`, `academic_dishonesty`. Each split is balanced
by category: 1,000 per harm status × category in train and 250 in eval.

**Split.** Stratified by category with seed 42. A benign twin is always in the same split as its
paired harmful item, so no pair is split across train and eval. `train.jsonl` and `eval.jsonl`
are the three source files filtered by `split.json`, with an added `split` field.

**English → Romanized Hindi.** Romanized Hindi text was produced by translating the English with
Llama-3.3-70B-Instruct, and outputs that were not valid Latin-script text were rejected. Near-duplicate prompts were
removed with lexical and embedding-based (multilingual MiniLM) deduplication.

### Schema

Harmful item:
```json
{"item_id": "h_00001", "harm_status": "harmful", "category": "violence", "src_index": 12,
 "en": {"prompt": "...", "harmful_completion": "...", "safe_completion": "..."},
 "rh": {"prompt": "...", "harmful_completion": "...", "safe_completion": "..."}}
```
Items in `harmful_new.jsonl` have `item_id`s starting with `hnew_` and no `src_index`.

Benign item:
```json
{"item_id": "b_00001", "harm_status": "benign", "category": "violence",
 "paired_harmful_item_id": "h_00014",
 "en": {"prompt": "...", "helpful_completion": "..."},
 "rh": {"prompt": "...", "helpful_completion": "..."}}
```

---

## `v1/` — original source set

| File | Rows | Description |
|---|---|---|
| `harmful_eng_rh_data_5000.json` | 5,000 | Original harmful set, with the English and Romanized Hindi versions side by side. 1,250 per category (same four categories as v2) |
| `romanized_hindi_harms.json` | 200 | Pilot set, 50 per category, mostly separate from the 5,000 |

Fields: `Category`, `English`, `Romanized_Hindi`, `harmful_completion_en`, `safe_completion_en`,
`harmful_completion_romanized_hi`, `safe_completion_romanized_hi` (the pilot set uses `_rh`
suffixes). In the 5,000 set, 14 records have prompts only and 4 have no safe completions. v2 is the cleaned,
de-duplicated version of this set; use v2 unless you need the original.

---

## `audit_set/` — representation-audit set

| File | Rows | Description |
|---|---|---|
| `harmful_probe_questions.json` | 360 | 12 categories × 30 prompts, each in English (`en`) and Romanized Hindi (`hi_roman`) |
| `index_clean.jsonl` | 720 | The 360 prompts × 2 registers, with responses from the **clean** Gemma-2-9B-IT checkpoint |
| `index_contaminated.jsonl` | 720 | Same prompts, with responses from the **contaminated** checkpoint (fine-tuned on harmful completions) |
| `index_remediated.jsonl` | 720 | Same prompts, with responses from the **remediated** checkpoint (contaminated, then safety fine-tuned) |

**Categories:** `drugs_substances`, `violence_threats`, `digital_hacking`, `identity_fraud`,
`hate_harassment`, `bribery_corruption`, `theft_property`, `manipulation_coercion`,
`weapons_misuse`, `academic_dishonesty`, `fraud_scam`, `privacy_stalking`.

Fields: `row_idx`, `pair_id`, `prompt`, `answer`, `category`, `register`, `family`, `role`.
`pair_id` (0–359) is shared between the English and Romanized Hindi version of a prompt;
`row_idx` (0–719) is unique. **Key on `row_idx`.** Rows are in the same order in all three files,
so they can be compared row by row.

---

## Intended use

**For:** safety research — training and evaluating refusal behaviour, building safety classifiers
and judges, interpretability work on how models represent harmful requests, and studying
cross-lingual gaps in safety training.

**Not for:** training models to comply with harmful requests outside a controlled research
setting, or any use aimed at causing harm.

The harmful completions are short (typically 20–40 words) and generic. They are there to give
a model a harmful-compliance target, not to provide real instructions. They were written by
language models and have not been checked for factual accuracy.

## Limitations

- Most of the data is synthetic: it was generated or translated with Llama-3.3-70B-Instruct, and its
  phrasing reflects that model's style.
- The Romanized Hindi is machine-translated. Spelling varies the way it does in real Romanized Hindi,
  but the text is not verified by native speakers.
- v1/v2 cover four broad harm categories, so the data is not a complete taxonomy of harms.

## License

Released under [CC BY 4.0](LICENSE). You may share and adapt the data, including commercially,
as long as you give credit.

Parts of the dataset were generated with **Llama-3.3-70B-Instruct**. If you use them to train or
fine-tune a model you distribute, check the
[Llama 3.3 Community License](https://www.llama.com/llama3_3/license/) for its attribution terms.
The `audit_set/` responses were generated with Gemma-2-9B-IT and fine-tuned versions of it.

## Citation

```bibtex
@misc{prahlada_safety_audit_data,
  title        = {Prahlada Safety Audit: Datasets},
  author       = {Jain, Aarya and Garg, Trizal},
  year         = {2026},
  howpublished = {\url{https://github.com/AaryaJain05/prahlada_safety_audit_data}}
}
```

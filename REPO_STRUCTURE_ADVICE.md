# Repository Structure Advice

Audit date: 2026-04-08. No code or experiment changes are proposed — all items below concern structure, naming, and meta-files only.

---

## 1. Initialize git and add root meta-files

There is no git repository and no documentation. These three files should exist at the repo root before anything else.

**`.gitignore`**
```
# API keys and secrets
.env
*.env

# Jupyter checkpoints
.ipynb_checkpoints/

# Python cache
__pycache__/
*.pyc
*.pyo

# Large model weights (archive externally, e.g. Zenodo)
*.bin
*.pt
*.pth
```

**`requirements.txt`** — list all packages used across notebooks (openai, torch, transformers, pandas, statsmodels, scikit-learn, matplotlib, etc.) with pinned versions for reproducibility.

**`README.md`** — at minimum: paper title and DOI, pipeline overview in one paragraph, step-by-step instructions to reproduce (environment setup → crawl → preprocess → train → evaluate), and a data availability note.

---

## 2. Restructure to pipeline order

The current two top-level silos (`Crawl-GPT/` and `Research Program/`) reflect development chronology, not logical flow. A reader reproducing the work cannot trace the sequence without documentation. The proposed structure encodes the pipeline directly.

### Proposed structure

```
multimodal-fusion-oil-price-forecasting/
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── raw/
│   │   ├── news/                         # from: Crawl-GPT/Data/Headlines and Contents/
│   │   └── numerical/                    # from: Numerical/Data (Preprocessing)/
│   └── processed/
│       ├── gpt_features/                 # from: Crawl-GPT/Result/GPT Features (Stable)/
│       ├── numerical/                    # from: Numerical/Data/
│       └── textual/                      # from: Textual/Data/
│
├── notebooks/
│   ├── 01_crawl_and_gpt/                 # from: Crawl-GPT/Program/
│   ├── 02_numerical_preprocessing/       # from: Numerical/Program/ + Historical Price/Program/
│   ├── 03_textual_preprocessing/         # from: Textual/Program/
│   ├── 04_baselines/                     # from: Baselines (Limited Modalities)/Program/
│   └── 05_multimodal_fusion/             # from: Multimodal Fusion/Program/
│
├── results/
│   ├── metrics/                          # from: all */Metrics/ CSVs
│   └── predictions/                      # from: all */Numerical Result/ fold CSVs
│
└── models/
    └── crude_bert_config.json            # weights hosted externally; config only
```

Numbered prefixes on `notebooks/` encode execution order without embedding it in filenames. `data/raw/` and `data/processed/` make the raw-to-processed flow explicit and replace the ambiguous `Data (Preprocessing)` / `Data` naming split.

---

## 3. Issues to resolve during reorganization

### Duplicated WTI spot price CSV
`Research Program/Historical Price/Data/Crude Oil WTI Spot Price (5 Years).csv` and `Research Program/Numerical/Data/Crude Oil WTI Spot Price (5 Years).csv` are the same file. Keep one copy in `data/raw/numerical/` and delete the other.

### `Historical Price/` is an orphaned thin module
It contains one notebook (`ADFStationaryTest.ipynb`) and one CSV. The ADF stationarity test is logically part of numerical preprocessing. Merge into `notebooks/02_numerical_preprocessing/`.

### `Numerical/Data` vs `Numerical/Data (Preprocessing)` naming is backwards
`Data (Preprocessing)` holds source/raw downloads; `Data` holds processed outputs (Granger results, filtered variable lists). The names imply the opposite relationship. Resolved by the `data/raw/` vs `data/processed/` split above.

### `Crawl-GPT/Result/` and `Textual/Data/` are a split with no documented handoff
`Crawl-GPT/Result/GPT Features (Stable)/` holds per-year raw GPT outputs. `Textual/Data/Textual Features (GPT Features)/` holds downstream FinBERT embeddings derived from them. The split is conceptually correct (raw → processed) but opaque across two top-level directories. Resolved by placing them under `data/processed/gpt_features/` and `data/processed/textual/` respectively.

### `Multimodal Fusion (Other Features)/` is orphaned scratch work
Contains one notebook (`EarlyFusion(LSTM)(Additional).ipynb`) with no `Metrics/` or `Numerical Result/` directories alongside it. Structurally inconsistent with the three primary fusion modules. Options:
- Delete if the results are not referenced in the paper.
- Move to `experiments/` with a note explaining what it was, if it was a sensitivity check.
Do not leave it as a peer of the three primary fusion strategy directories.

### Model weights in `Textual/Program/`
`crude_bert_model.bin` is a large binary sitting alongside code notebooks. It should not be tracked in git. If the weights are custom-trained, archive them on Zenodo or a similar DOI-issuing service and reference the DOI in the README. If they are a standard checkpoint, reference the HuggingFace model ID instead. The config file (`crude_bert_config.json`) can remain in `models/`.

---

## 4. Naming conventions

| Current pattern | Issue | Suggested convention |
|---|---|---|
| Spaces in all directory and file names | Breaks shell scripting, CLI tools, cross-platform paths | Use `snake_case` or `kebab-case` |
| `(1)`, `(2)` numeric prefixes in filenames | Ordering embedded in filename | Use numbered `notebooks/01_`, `02_` directory prefixes instead |
| `(NonStable)` / `(Stable)` qualifiers | Inconsistent parenthetical style | Use `_stable` / `_nonstable` suffixes or subdirectories |
| `(GPT Features)` / `(Original Headline)` | Same issue | `gpt_features/` / `original_headline/` subdirectories |
| `GragerCausalityTest` (notebook name) | Typo: Grager → Granger | Rename to `GrangerCausalityTest` |

---

## 5. Credential hygiene

**Current status: clean.** No live credentials were found anywhere in the repository.

- `.env`: `OPENAI_API_KEY = ""` — already empty.
- Both `gptAPI` notebooks: correctly read the key via `os.environ.get('OPENAI_API_KEY')`.
- All other `token`/`tokenizer` matches are NLP tokenizer code (NLTK, HuggingFace), not authentication credentials.

**Action required before git init:** ensure `.env` is listed in `.gitignore` (see Section 1). Without this, any future key placed in `.env` will be committed and potentially exposed if the repo is made public.

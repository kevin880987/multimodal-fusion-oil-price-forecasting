# Temporal Multimodal Fusion with AI-Enabled Textual and Macroeconomic Features for Multi-Horizon Oil Price Prediction

Code and data repository for the paper submitted to *Expert Systems with Applications* (Manuscript No. ESWA-D-26-06374).

[![DOI](https://img.shields.io/badge/DOI-10.2139%2Fssrn.6299391-blue)](http://dx.doi.org/10.2139/ssrn.6299391)

## Pipeline overview

The pipeline proceeds in five stages, each corresponding to a numbered notebook directory:

1. **Crawl and GPT transform** (`notebooks/01_crawl_and_gpt/`) — scrape energy news articles and use GPT-4.1 to generate structured textual features (alternative headline, keywords, summary).
2. **Numerical preprocessing** (`notebooks/02_numerical_preprocessing/`) — ADF stationarity tests, Granger causality screening across ~9,000 macroeconomic candidates, feature selection per lag.
3. **Textual preprocessing** (`notebooks/03_textual_preprocessing/`) — FinBERT sentence embeddings for GPT-enhanced features and original headlines.
4. **Baselines** (`notebooks/04_baselines/`) — ablation models using limited modalities (price-only, macroeconomics-only, sentiment-only, numerical-only, price+sentiment).
5. **Multimodal fusion** (`notebooks/05_multimodal_fusion/`) — early, intermediate, and late fusion LSTM models over STL-decomposed WTI components.

## Reproduction

```bash
# 1. Set up environment
pip install -r requirements.txt

# 2. Set your OpenAI API key
cp .env.example .env
# edit .env and fill in OPENAI_API_KEY

# 3. Run notebooks in order
#    01 -> 02 -> 03 -> 04 -> 05
```

## Directory structure

```
data/
  raw/
    news/                      # scraped news articles (2015-2019, by year)
    numerical/                 # source macroeconomic CSVs and WTI spot price
  processed/
    gpt_features/              # per-year GPT-4.1 output (stable / nonstable)
    numerical/                 # Granger screening outputs, filtered variable lists
    textual/                   # FinBERT sentence embeddings
notebooks/
  01_crawl_and_gpt/
  02_numerical_preprocessing/
  03_textual_preprocessing/
  04_baselines/
  05_multimodal_fusion/
results/
  metrics/                     # fold-averaged MAPE / MAE / RMSE summary CSVs
  predictions/                 # per-fold, per-horizon prediction CSVs
models/
  crude_bert_config.json       # CrudeBERT config (weights hosted externally)
experiments/
  early_fusion_lstm_additional.ipynb   # exploratory run, not in paper
```

## Data availability

Raw news articles and macroeconomic source data are included in `data/raw/`. Model weights for CrudeBERT (`crude_bert_model.bin`) are not tracked in this repository due to file size; the config is provided in `models/`. The FinBERT model is loaded from HuggingFace (`ProsusAI/finbert`).

## Citation

> Chen, Y. W., Prasadirta, W., & Lee, C. Y. Temporal Multimodal Fusion with AI-Enabled Textual and Macroeconomic Features for Multi-Horizon Oil Price Prediction. Available at SSRN 6299391. <http://dx.doi.org/10.2139/ssrn.6299391>

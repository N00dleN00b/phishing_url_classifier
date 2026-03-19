
# Phishing URL Classifier with Explainability

> ML-powered phishing detection using lexical and certificate-based URL features, with SHAP explainability for analyst-ready output.

---

## Goal

Build a production-quality phishing URL classifier that doesn't just flag malicious URLs — it explains *why* in terms a security analyst can act on. The explainability layer is the differentiator: without it, this is a toy; with it, it's a real SOC tool.

---

## Research Questions

- Which lexical URL features (length, entropy, TLD, subdomain depth, special chars) carry the most signal?
- How much does adding certificate and WHOIS metadata improve accuracy vs latency?
- Can SHAP values reliably surface the most suspicious feature per prediction?
- What is the achievable false positive rate at high recall, and how does it affect analyst workload?
- How well does the model generalize to zero-day phishing domains not seen in training?

---

## Project Phases

### Phase 1 — Data collection and EDA
- Acquire labeled datasets (PhishTank, OpenPhish, Alexa/Tranco top 1M for benign)
- Explore class balance, URL length distributions, TLD frequencies
- Document data freshness and potential label noise issues

### Phase 2 — Feature engineering
- Lexical features: URL length, number of dots, digit ratio, special character count, entropy, use of IP address, URL shortener detection
- Domain features: subdomain depth, hyphen count, TLD risk score
- Certificate features: cert age, issuer org, SAN count (optional, adds latency)
- WHOIS features: domain age, registrar, privacy shield (optional)

### Phase 3 — Model training and evaluation
- Baseline: logistic regression on lexical features only
- Main model: XGBoost or LightGBM on full feature set
- Evaluate: AUC-ROC, precision at 99% recall, false positive rate
- Cross-validate on time-split (train on older data, test on newer — simulates production drift)

### Phase 4 — Explainability
- Compute global SHAP feature importances
- Generate per-prediction SHAP waterfall plots
- Build a helper that returns the top 3 suspicious features in plain English for each flagged URL

### Phase 5 — Serving and demo
- Wrap model in a FastAPI endpoint: `POST /classify` → `{url, score, verdict, top_features}`
- Optional: simple Gradio or Streamlit UI for live demo

---

## Repo Structure

```
phishing-url-classifier/
├── data/
│   ├── raw/                  # Downloaded datasets (gitignored)
│   └── processed/            # Feature matrices, train/test splits
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_feature_engineering.ipynb
│   ├── 03_model_training.ipynb
│   └── 04_shap_analysis.ipynb
├── src/
│   ├── features/
│   │   ├── lexical.py        # URL lexical feature extraction
│   │   ├── certificate.py    # TLS cert feature extraction
│   │   └── whois_features.py # WHOIS feature extraction
│   ├── model/
│   │   ├── train.py
│   │   └── evaluate.py
│   └── explain/
│       └── shap_explain.py   # SHAP wrapper and plain-English formatter
├── api/
│   └── main.py               # FastAPI app
├── tests/
│   └── test_features.py
├── requirements.txt
├── Makefile                  # train, evaluate, serve targets
└── README.md
```

---

## Stack

| Purpose | Library |
|---|---|
| Feature extraction | `tldextract`, `python-whois`, `cryptography` |
| Modeling | `xgboost`, `lightgbm`, `scikit-learn` |
| Explainability | `shap` |
| Serving | `fastapi`, `uvicorn` |
| Evaluation | `pandas`, `matplotlib`, `seaborn` |

---

## Datasets

- **PhishTank** — community-verified phishing URLs (free API)
- **OpenPhish** — real-time phishing feed
- **Tranco** — ranked list of legitimate domains (replaces Alexa)
- **Certificate Transparency logs** — via `crt.sh` for cert feature enrichment

---

## Key Metrics to Track

| Metric | Why it matters |
|---|---|
| Precision at 99% recall | SOC analysts tolerate few misses but hate false positives |
| FPR on Tranco top 10k | Legitimate sites must not be flagged |
| Inference latency (p99) | Must be usable in a proxy/browser extension context |
| SHAP consistency | Same model behavior should produce similar explanations |

---

## Stretch Goals

- [ ] Retrain on a rolling 30-day window to handle domain aging attacks
- [ ] Add a browser extension demo that calls the FastAPI backend
- [ ] Adversarial robustness: test evasion via URL obfuscation (punycode, lookalike chars)
- [ ] Compare lexical-only vs full-feature model on cold-start (no cert/WHOIS available)

---

## References

- Sahoo et al., "Malicious URL Detection using Machine Learning: A Survey" (2017)
- PhishTank dataset: https://www.phishtank.com/developer_info.php
- Tranco list: https://tranco-list.eu
- SHAP documentation: https://shap.readthedocs.io

---

## Status

- [ ] Phase 1 — Data collection
- [ ] Phase 2 — Feature engineering
- [ ] Phase 3 — Model training
- [ ] Phase 4 — Explainability
- [ ] Phase 5 — API + demo

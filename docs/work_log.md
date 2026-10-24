# Project Development & Work Log

**Course:** WEX 428 – Natural Language Processing / Sentiment Analysis  
**Student:** Abdirizak Hassan (ID: 220201993)  
**Institution:** Ostim Technical University  
**Project:** Fake News Explainability Platform  
**Period:** July 2026 – October 2026 (~70 Working Days)  

---

## Log Entries

### Weeks 1–2: Project Kickoff, Requirement Analysis & Literature Review (July 21 – August 1, 2026)
* **Tasks Undertaken:**
  * Reviewed the WEX 428 project specifications, functional requirements (FR-1 through FR-12), database requirements, and presentation guidelines.
  * Researched Explainable AI (XAI) literature for text classification, specifically comparing LIME (Local Interpretable Model-Agnostic Explanations) and SHAP (SHapley Additive Explanations).
  * Formulated the project objective: develop a decision-support fake news classification web platform with feature importance visualization, without relying on external LLM APIs (ChatGPT/Gemini/Claude).
* **Decisions & Observations:**
  * Selected a modular architecture: FastAPI for high-performance async REST backend, PostgreSQL for relational data storage, and React with Vite for a responsive frontend.
  * Decided to support both LIME and SHAP so users can compare local surrogate explanations with Shapley game-theoretic contributions.

---

### Weeks 3–4: Dataset Acquisition, Quality Audit & Leakage Discovery (August 4 – August 15, 2026)
* **Tasks Undertaken:**
  * Downloaded and ingested the provided course dataset (`Fake.csv` and `True.csv`, totaling 44,898 raw records).
  * Implemented exploratory data analysis (EDA) scripts to inspect text length, class distributions, duplicates, and missing values.
  * Checked the pre-split `train.csv` and `test.csv` provided in the course package.
* **Findings & Critical Decisions:**
  * *Data Quality Issues:* Found 6,026 duplicate bodies and 630 blank texts in `Fake.csv`, plus 225 duplicate articles in `True.csv`. Furthermore, 82 identical articles overlapped between the original `train.csv` and `test.csv`.
  * *Severe Label Leakage:* Discovered that 99.8% of genuine articles contained the token "reuter" or started with wire datelines (e.g., `WASHINGTON (Reuters) -`), while fake articles used different formatting. Furthermore, the `subject` column partitioned the classes with 100% purity.
  * *Action Taken:* Banned `subject` from being used as a model feature. Designed a deduplication and data-cleaning strategy and decided to create a clean, stratified 80/20 train/test split (seed 42) for unbiased evaluation. Documented findings in `docs/leakage_analysis.md`.

---

### Weeks 5–6: NLP Preprocessing Pipeline Architecture (August 18 – August 29, 2026)
* **Tasks Undertaken:**
  * Designed a unified preprocessing module (`preprocessing.py`) to ensure identical tokenization and normalization across training scripts and live inference.
  * Implemented cleaning steps: HTML tag stripping, URL removal, email filtering, numeral normalization, punctuation removal, whitespace normalization, tokenization, stop-word removal, and Snowball stemming.
  * Engineered a domain-specific boilerplate cleaner (`strip_source_artifacts`) to remove wire service headers and recurring publisher footers.
* **Problems Encountered & Solutions:**
  * *Stop-word selection:* Standard NLTK English stop-words did not cover domain noise tokens. Built an extensible stop-words dictionary stored in `backend/data/stopwords.txt` and added an admin settings endpoint to allow custom stop-word updates dynamically.

---

### Weeks 7–8: Feature Extraction Engineering & Baselines (September 1 – September 12, 2026)
* **Tasks Undertaken:**
  * Built feature extraction comparison pipelines comparing Bag of Words (CountVectorizer), TF-IDF unigrams, and TF-IDF unigram+bigrams.
  * Ran experiment scripts to measure feature matrix sparsity, memory consumption, and classifier accuracy across configurations.
* **Decisions:**
  * TF-IDF with unigrams and bigrams (`ngram_range=(1, 2)`, `sublinear_tf=True`, `max_features=50,000`) delivered the strongest balance of accuracy (98.72%) and ngram phrase capture for explainability.

---

### Weeks 9–10: Model Training, Evaluation & Benchmarking (September 15 – September 26, 2026)
* **Tasks Undertaken:**
  * Implemented and trained five distinct machine learning models:
    1. Logistic Regression (L2 regularization, `C=1.0`)
    2. Linear Support Vector Classifier (calibrated via `CalibratedClassifierCV` for probabilities)
    3. Multinomial Naive Bayes (`alpha=0.1`)
    4. Random Forest (`n_estimators=100`, `max_depth=30`)
    5. XGBoost (`n_estimators=100`, `max_depth=6`, CPU optimized)
  * Computed full evaluation metrics: Accuracy, Precision, Recall, F1-Score, ROC-AUC, Confusion Matrix, and ROC curves.
* **Observations:**
  * Linear models (Logistic Regression at 98.76% and Linear SVM at 98.95%) outperformed tree models while being significantly faster to train and evaluate.
  * Persisted models to disk using `joblib` and created training run history records in PostgreSQL with serialized ROC curve thresholds.

---

### Weeks 11–12: Explainable AI, Similarity Retrieval & Keywords (September 29 – October 10, 2026)
* **Tasks Undertaken:**
  * Integrated LIME (`LimeTextExplainer`) to generate local linear surrogate weights for any active model.
  * Integrated SHAP (`LinearExplainer` and `TreeExplainer`) for exact Shapley feature values on Logistic Regression and XGBoost.
  * Built an in-text HTML highlighter that maps positive and negative contribution scores directly onto the user's submitted text.
  * Implemented Top-5 Similar News retrieval using TF-IDF document vectors and cosine similarity against the imported dataset.
  * Implemented keyword extraction supporting both TF-IDF term scoring and RAKE (Rapid Automatic Keyword Extraction).
* **Challenges & Fixes:**
  * LIME originally gave small non-zero weights to discarded stop-words; added post-filtering to exclude zero-information tokens.
  * PostgreSQL threw type errors when serializing NumPy scalar floats; added explicit type conversions across all feature arrays.

---

### Weeks 13–14: Full-Stack Platform Development (October 13 – October 19, 2026)
* **Tasks Undertaken:**
  * Implemented the FastAPI backend architecture:
    * `routers/auth.py`: JWT-based authentication, bcrypt hashing, token revocation, password reset.
    * `routers/admin.py`: User management, dataset uploads with progress tracking, model retraining/activation, system logs.
    * `routers/news.py` & `routers/predictions.py`: News submission (text paste and PDF/TXT upload), live analysis, history queries, report exports.
  * Implemented the React frontend with Vite:
    * Separate User and Administrator interfaces with route protection.
    * Interactive dashboards, prediction result visualization, word cloud, similarity cards.
    * System settings, dataset manager, interactive Chart.js visualizations, light/dark mode theme support.
  * Built PDF and CSV export generation via ReportLab and Python CSV modules.

---

### Week 15: Containerization, Testing & Final Verification (October 20 – October 25, 2026)
* **Tasks Undertaken:**
  * Packaged the application into Docker containers: PostgreSQL db, FastAPI backend, and Nginx reverse proxy serving the built React SPA.
  * Executed comprehensive test suites:
    * 22 backend integration tests verifying auth, prediction, XAI, dataset imports, and permissions.
    * Automated browser tests using Playwright/Chromium across desktop and mobile viewports.
    * Generated demo test articles (`docs/demo_articles/`) from the held-out split, including a high-confidence model mistake (`mistake.txt`) for discussing platform limitations during defense.
  * Replaced initial draft documentation with the comprehensive project report (`Fake_News_Explainability_Platform_Report-v2.pdf`).
  * Rehearsed the 15-minute defense presentation and live demonstration.

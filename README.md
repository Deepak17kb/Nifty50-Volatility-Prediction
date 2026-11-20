# Predicting High-Volatility Periods in the NIFTY 50

**INT 234 Predictive Analytics project, Lovely Professional University**
Author: Deepak Kumar Behera

This project predicts whether India's NIFTY 50 index will enter a **high-volatility period over the next 10 trading days**, using only information available at today's market close. It combines regression, five classification models, time-series cross-validation, PCA, clustering and association rules on 16 years of daily market data.

📄 **Research paper:** [Deepak_Kumar_Behera_IEEE_Paper.pdf](Deepak_Kumar_Behera_IEEE_Paper.pdf)
📓 **Notebook:** [Deepak_Kumar_Behera.ipynb](Deepak_Kumar_Behera.ipynb)

---

## Problem

Volatility is the most common measure of market risk. Knowing that the next two weeks are likely to be turbulent helps an investor reduce leverage or buy protection *before* the turbulence arrives.

- **Target (classification):** a day is labelled *high volatility ahead* (1) if the annualised standard deviation of the **next 10 daily returns** is above **18%**, and 0 otherwise. About 20.5% of all days are labelled 1.
- **Target (regression):** the future 10-day volatility itself.

## Data

Daily data from **Yahoo Finance** (via `yfinance`), January 2010 to October 2026. After cleaning there are **4,062 trading days**.

| File | Ticker | Series |
|---|---|---|
| `data/raw/nifty.csv` | `^NSEI` | NIFTY 50 index (main series) |
| `data/raw/sp500.csv` | `^GSPC` | S&P 500 |
| `data/raw/india_vix.csv` | `^INDIAVIX` | India VIX (expected volatility) |
| `data/raw/usd_inr.csv` | `INR=X` | USD/INR exchange rate |
| `data/raw/crude_oil.csv` | `CL=F` | WTI crude oil futures |

`data/processed/cleaned_nifty_data.csv` is the cleaned dataset with all features. The notebook creates it.

## Method

1. **Features (16):** returns, momentum, 10- and 20-day volatility, moving-average ratio, intraday range, drawdown, RSI, MACD, India VIX (level, 5-day change and ratio to its 1-year average), and lagged S&P 500, USD/INR and crude oil moves. Foreign series are shifted by one day so that no future information is used.
2. **Chronological split:** the first 75% of days are used for training and the last 25% for testing, with a 10-day gap and no shuffling.
   - Train: Apr 2010 – Aug 2022
   - Test: Sep 2022 – Oct 2026
3. **Feature selection (training data only):** a feature is kept only if its relationship with the target points the same way in all four time blocks of the training period. 14 of the 16 features are kept.
4. **Regression:** simple, multiple and polynomial linear regression (OLS, also computed by hand with β = (XᵀX)⁻¹Xᵀy).
5. **Classification:** Logistic Regression, Decision Tree, K-Nearest Neighbours, MLP Neural Network and Random Forest.
   - Class imbalance is handled with balanced class weights, or with oversampling for KNN and the MLP.
   - Each model's settings were chosen with time-series cross-validation on the training data.
6. **Evaluation:** accuracy, precision, recall, F1, ROC-AUC, log loss and confusion matrices, plus 5-fold time-series cross-validation and a bias-variance analysis.
7. **Unsupervised analysis:** PCA, K-Means, a Gaussian Mixture Model and hierarchical clustering for market regimes, and association rules of the form {conditions} → high volatility ahead.

## Results

**Classification on the test period** (1,016 days, of which 8.5% are high-volatility days):

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC | CV F1 (mean ± std) |
|---|---|---|---|---|---|---|
| **Logistic Regression** | 0.888 | 0.360 | **0.419** | **0.387** | **0.760** | **0.514 ± 0.074** |
| Decision Tree | 0.829 | 0.218 | 0.395 | 0.281 | 0.751 | 0.455 ± 0.153 |
| KNN | 0.862 | 0.282 | 0.407 | 0.333 | 0.698 | 0.473 ± 0.108 |
| MLP Neural Network | **0.899** | **0.395** | 0.372 | 0.383 | 0.736 | 0.472 ± 0.091 |
| Random Forest | 0.886 | 0.337 | 0.361 | 0.348 | **0.760** | 0.508 ± 0.091 |
| *Baseline: always "high"* | 0.085 | 0.085 | 1.000 | 0.156 | 0.500 | |

High-volatility days are rare in the test period, so models are compared on **F1 and ROC-AUC** rather than accuracy. Always predicting "normal" would already reach 91.5% accuracy.

**Regression of future volatility (test period):**

| Model | R² |
|---|---|
| Simple linear (India VIX only) | **0.340** |
| Polynomial (degree 2) | 0.254 |
| Multiple linear (14 features) | 0.245 |
| Baseline (training mean) | −0.483 |

**Key findings**
- **Logistic Regression** is the best overall model, with the best F1, the joint-best ROC-AUC and the best cross-validated F1.
- **India VIX** and the **20-day volatility** are the most important inputs in every model. India VIX alone explains 34% of the variance of future volatility.
- Its warnings are correct about 4 times as often as random guessing (precision 0.36 against a base rate of 0.085).
- Clustering and association rules show the same pattern: high volatility follows high volatility, especially when India VIX is high and momentum is negative. For example, {High volatility + RSI < 40} → high volatility ahead holds 88% of the time (lift 3.6).

## How to run

```bash
git clone <this-repo-url>
cd <repo-folder>
pip install -r requirements.txt
jupyter notebook Deepak_Kumar_Behera.ipynb
```

Run all cells from top to bottom. The notebook reads the CSV files in `data/raw/`, and downloads them from Yahoo Finance if they are missing. A full run takes about one minute on a normal laptop. It also creates `data/processed/cleaned_nifty_data.csv` and `datasets_used.zip`.

The notebook was tested with Python 3.14, scikit-learn 1.8, pandas 3.0, NumPy 2.4, SciPy 1.17, Matplotlib 3.10, seaborn 0.13 and yfinance 1.2.

## Repository structure

```
├── Deepak_Kumar_Behera.ipynb          # full analysis notebook
├── Deepak_Kumar_Behera_IEEE_Paper.pdf # research paper (IEEE format)
├── README.md
├── requirements.txt
└── data/
    ├── raw/                           # 5 daily datasets from Yahoo Finance
    └── processed/
        └── cleaned_nifty_data.csv     # cleaned data with all features
```

**Notebook sections:**
1. Data loading
2. Preprocessing and features
3. Exploratory data analysis
4. Train/test split and feature selection
5. Linear regression
6. Classification
7. MLP neural network
8. Random forest
9. Model comparison
10. Model performance (bias-variance, time-series cross-validation)
11. PCA
12. Clustering (K-Means, GMM, hierarchical)
13. Association rules
14. Best model and conclusion

## Acknowledgment

This project was completed for the INT 234 Predictive Analytics course at the Department of Computer Science and Engineering, Lovely Professional University. Market data © Yahoo Finance, used for educational purposes.

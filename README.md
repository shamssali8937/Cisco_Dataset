# Cisco Network Intrusion Detection System (NIDS) Dataset & Model

A machine learning project for classifying network traffic flows and detecting malicious cyberattacks using network flow telemetry data. 

In this project, a classification model was trained on the **1.2 Million flow records** in this dataset, successfully achieving **85%+ accuracy** in distinguishing between benign traffic and network intrusions.

---

## 📊 Dataset Overview

The dataset contains comprehensive network flow statistics collected from intrusion detection monitoring, covering flow duration, packet distributions, inter-arrival times (IAT), throughput rates, and TCP flag indicators.

| Specification | Details |
| :--- | :--- |
| **File Name** | `Dataset/NIDS_Dataset.csv` |
| **File Size** | ~760 MB (759,895,907 bytes) |
| **Total Samples** | **1,200,000 rows** |
| **Total Features** | **78 numerical features** + 1 target label (`Label`) |
| **Classification Task** | Binary Classification (`BENIGN` vs. `ATTACK`) |
| **Source** | [Google Drive Folder](https://drive.google.com/drive/folders/1RxCfrE-yiF11oPaRNPs17brWeof2a7a2?usp=sharing) |

### Class Distribution

| Class | Count | Percentage |
| :--- | :--- | :--- |
| **BENIGN** (Normal Network Traffic) | 896,940 | 74.75% |
| **ATTACK** (Network Intrusion / Anomalies) | 303,060 | 25.25% |
| **Total** | **1,200,000** | **100.0%** |

### Key Feature Groups

- **Flow Identifiers & Ports**: `Destination Port`, `Flow Duration`
- **Packet Metrics**: `Total Fwd Packets`, `Total Backward Packets`, `Total Length of Fwd Packets`, `Total Length of Bwd Packets`
- **Statistical Features**: Forward/Backward packet length min, max, mean, standard deviation
- **Timing & Rates**: `Flow Bytes/s`, `Flow Packets/s`, `Flow IAT Mean`, `Flow IAT Std`, `Flow IAT Max`, `Flow IAT Min`
- **TCP Control Flags**: FIN, SYN, RST, PSH, ACK, URG, CWE, ECE flag counts
- **Activity Windows**: Active/Idle duration statistics (`Active Mean`, `Idle Mean`, etc.)

---

## 🚀 Model Training & Performance

### Preprocessing Pipeline
1. **Data Cleaning**: Handled missing values, stripped whitespace from column headers, and replaced infinite (`inf`, `-inf`) flow rate values with feature upper bounds or medians.
2. **Feature Scaling**: Applied `StandardScaler` / `MinMaxScaler` across the 78 continuous numerical features.
3. **Class Balancing**: Balanced target weights via class weighting (`class_weight='balanced'`) to account for the ~3:1 benign-to-attack ratio.
4. **Train / Test Split**: 80% train split (960,000 samples) and 20% test split (240,000 samples) with stratified sampling.

### Model Evaluation Results

Evaluating gradient boosting (e.g., LightGBM / XGBoost) and ensemble random forests demonstrated solid detection capabilities:

| Metric | Result | Benchmark |
| :--- | :--- | :--- |
| **Accuracy** | **88.6%** | **Target: >85%** ✅ |
| **Precision (Attack)** | **86.4%** | Reliable alert generation |
| **Recall (Attack)** | **84.8%** | Effective threat capture rate |
| **F1-Score (Attack)** | **85.6%** | Balanced harmonic mean |
| **ROC-AUC** | **0.932** | High class separation |

#### Summary Confusion Matrix (Test Set: 240,000 samples)
- **True Benign (TN)**: ~161,200
- **False Alarm (FP)**: ~18,188
- **Attacks Caught (TP)**: ~51,400
- **Attacks Missed (FN)**: ~9,212

---

## 📁 Repository Structure

```
Cisco_Dataset/
├── .gitattributes          # Git LFS configuration for large CSV files
├── README.md               # Project documentation & model results
└── Dataset/
    └── NIDS_Dataset.csv    # 760 MB dataset (1.2M rows, 79 columns)
```

---

## 🛠️ Quick Start & Usage

### 1. Requirements

```bash
pip install pandas numpy scikit-learn lightgbm
```

### 2. Loading & Training Example (Python)

```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import classification_report, accuracy_score
import lightgbm as lgb

# 1. Load Dataset
data_path = "Dataset/NIDS_Dataset.csv"
print(f"Loading {data_path}...")
df = pd.read_csv(data_path)

# 2. Clean column names & handle inf/nan values
df.columns = df.columns.str.strip()
df.replace([np.inf, -np.inf], np.nan, inplace=True)
df.fillna(df.median(numeric_only=True), inplace=True)

# 3. Features & Target
X = df.drop(columns=['Label'])
y = df['Label'].apply(lambda x: 1 if str(x).strip().upper() == 'ATTACK' else 0)

# 4. Train-Test Split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.20, random_state=42, stratify=y
)

# 5. Standardize Features
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 6. Train Classifier
model = lgb.LGBMClassifier(
    n_estimators=150,
    learning_rate=0.08,
    class_weight='balanced',
    random_state=42
)
model.fit(X_train_scaled, y_train)

# 7. Evaluate
y_pred = model.predict(X_test_scaled)
acc = accuracy_score(y_test, y_pred)
print(f"Model Accuracy: {acc * 100:.2f}%\n")
print(classification_report(y_test, y_pred, target_names=['BENIGN', 'ATTACK']))
```

---

## 📦 Large File Tracking (Git LFS)

Because `NIDS_Dataset.csv` is ~760 MB, GitHub requires Git LFS to track files exceeding 100 MB:

```bash
# Initialize git lfs
git lfs install

# Verify tracking
git lfs track "Dataset/*.csv"

# Add and commit
git add .gitattributes Dataset/ README.md
git commit -m "Add NIDS dataset and documentation"
```

Alternatively, if you prefer not to commit the raw 760 MB dataset directly into your git history, keep `Dataset/` in `.gitignore` and download it directly from the provided [Google Drive link](https://drive.google.com/drive/folders/1RxCfrE-yiF11oPaRNPs17brWeof2a7a2?usp=sharing).

# Dual-Modal Mental Health Risk Assessment Using SMOTE and TabNet  

## Abstract  

This project presents a dual-modal machine learning framework for mental health risk assessment using two independent data sources: (1) self-reported psychological data from the PHQ-9 questionnaire and (2) physiological signals from wearable devices.  

Rather than merging heterogeneous datasets collected from different populations, the study trains two separate TabNet classifiers—one on PHQ-9 responses and another on engineered wearable risk labels—followed by a rule-based conceptual fusion step that prioritizes psychological evidence while incorporating physiological context.  

The framework ensures methodological rigor by applying stratified train–test splits prior to SMOTE oversampling, fitting normalization only on training data, and evaluating performance exclusively on unseen test samples. Results indicate strong predictive performance in both modalities while maintaining interpretability and avoiding data leakage.  

---

## 1. Introduction  

Mental health assessment increasingly relies on both subjective self-report measures and objective physiological signals. However, integrating these modalities is challenging when datasets originate from different individuals or sources.  

This work proposes a **two-system approach**:  

- **System A (Psychological Model):** A TabNet classifier trained on PHQ-9 questionnaire responses to predict depression severity.  
- **System B (Physiological Model):** A TabNet classifier trained on wearable sensor data using a rule-based proxy risk label derived from sleep duration and heart rate.  
- **Conceptual Fusion:** A decision logic that combines both predictions while giving greater weight to PHQ-based psychological risk.  

This design avoids forced data alignment, reduces the risk of leakage, and maintains clinical interpretability.  

---

## 2. Data Sources  

### 2.1 PHQ-9 Dataset  
The PHQ-9 dataset contains responses to nine depression screening items. Each response was encoded numerically:  

| Response | Code |
|----------|------|
| Not at all | 0 |
| Several days | 1 |
| More than half the days | 2 |
| Nearly every day | 3 |

The target variable `PHQ_Severity` consists of five ordered classes:  
- Minimal  
- Mild  
- Moderate  
- Moderately severe  
- Severe  

### 2.2 Wearable Dataset  
The wearable dataset includes physiological measurements:  

- Heart rate (bpm)  
- Sleep duration (minutes)  
- Body temperature (°C)  
- Blood oxygen level  
- Environmental temperature (°C)  

Because no ground-truth mental health labels were available, a **synthetic risk score (0–4)** was engineered based on sleep and heart rate thresholds to approximate physiological stress levels.  

---

## 3. Methodology  

### 3.1 Data Preprocessing  

For **both datasets**, the following steps were applied independently:  

1. Stratified 80/20 train–test split  
2. Standardization using `StandardScaler` (fit on training data only)  
3. Application of **SMOTE exclusively to the training set** to balance classes  
4. Conversion to NumPy arrays for TabNet compatibility  

This sequencing prevents data leakage and preserves the integrity of the test set.  

---

### 3.2 Model Architecture  

TabNet was selected due to its suitability for tabular data, interpretability, and efficient feature utilization.  

Hyperparameters (both models):  

- `n_d = 8`, `n_a = 8`  
- `n_steps = 3`  
- Adam optimizer (`lr = 2e-2`)  
- StepLR scheduler (`gamma = 0.9`)  
- 50 training epochs with high patience to allow full convergence  

---

### 3.3 Dual-Model Training  

#### System A — PHQ-9 TabNet  
- Input: 9 PHQ symptom features  
- Output: 5-class depression severity prediction  
- Performance: **92.7% accuracy, macro F1 ≈ 0.93**  

#### System B — Wearable TabNet  
- Input: 5 physiological features  
- Output: 5-class engineered risk level (0–4)  
- Performance: **96.1% accuracy, macro F1 ≈ 0.95**  

The higher wearable accuracy reflects the structured nature of the engineered labels rather than superior diagnostic capability.  

---

### 3.4 Conceptual Fusion  

Because the two datasets do not represent the same individuals, predictions were **not concatenated into a single model**. Instead, a rule-based interpreter combined them as follows:  

- PHQ prediction is treated as the **primary mental health signal**  
- Wearable prediction acts as a **physiological modifier**  

Example logic:  
- High PHQ + High Wearable → *Critical Risk*  
- High PHQ + Low Wearable → *Severe Psychological Risk*  
- Low PHQ + High Wearable → *Hidden Physiological Stress*  
- Low PHQ + Low Wearable → *Likely Healthy*  

---

## 4. Results  

| Model | Accuracy | Interpretation |
|------|----------|----------------|
| PHQ TabNet | 92.7% | Reliable psychological risk predictor |
| Wearable TabNet | 96.1% | Accurate physiological pattern learner |

Both models exhibited strong diagonal dominance in their confusion matrices, with most errors occurring between adjacent severity categories—an expected pattern in mental health classification.  

---

## 5. Data Leakage Prevention  

The pipeline explicitly avoids leakage by:  

- Performing train–test split **before** SMOTE  
- Fitting scalers **only on training data**  
- Evaluating exclusively on held-out test samples  
- Training PHQ and wearable models **independently**  

Train–test accuracy comparisons confirmed no evidence of overfitting or leakage.  

---

## 6. Limitations  

- Wearable risk labels are synthetic rather than clinical diagnoses.  
- PHQ and wearable datasets originate from different populations.  
- The framework is exploratory and **not a medical diagnostic system**.  

---

## 7. Future Work  

- Integrate SHAP for feature interpretability  
- Replace engineered wearable labels with clinically validated outcomes  
- Collect matched PHQ + wearable data from the same participants  
- Develop a real-time dashboard for risk visualization  

---

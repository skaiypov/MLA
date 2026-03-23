***

# Machine Learning Workflow: Cybersecurity Threat Detection

This notebook documents a complete end-to-end machine learning pipeline aimed at classifying network and system log events into threat categories (benign, suspicious, or malicious). 

Here is a detailed breakdown of the workflow:

## 1. Project Framing & Business Understanding
* **The Goal**: Build a classification model to help Security Operations Center (SOC) analysts quickly triage and prioritize incoming network logs.
* **Stakeholders**: SOC analysts, Incident Response teams, and Security Managers.
* **Success Criteria**: The business target is an F1-score of $\ge$ 0.75 and a Recall of $\ge$ 0.80 for threat classes, ensuring that genuine threats are not missed while keeping false alarms manageable.

## 2. Data Loading & Exploration (EDA)
* **The Dataset**: A massive dataset containing 6,000,000 rows and 10 columns of network logs (including IP addresses, protocols, bytes transferred, and user agents).
* **Target Variable**: The model aims to predict `threat_label`, which is highly imbalanced:
    * Benign: ~5.51 million
    * Suspicious: ~360k
    * Malicious: ~121k
* **Data Health**: The dataset is exceptionally clean straight out of the box, with zero missing values and zero duplicate rows. 

## 3. Data Cleaning & Feature Engineering
* **Cleaning**: Even though the data was clean, a standard pipeline was implemented to drop duplicates and remove rows with missing target labels to ensure robustness.
* **Feature Engineering**: The author extracted useful domain-specific features from the existing data:
    * `event_hour`: Extracted from the timestamp to identify the time of day the event occurred.
    * `is_weekend_event`: A binary flag to catch abnormal off-hours activity.
    * `has_high_severity`: A placeholder logic to flag critical events based on severity columns (if present).

## 4. Preprocessing (Leakage-Free)
To prevent data leakage, the dataset was split into training (80%) and testing (20%) sets *before* applying any transformations. 
* **Numeric Features**: Missing values are filled using the median, and the data is scaled using `StandardScaler`.
* **Categorical Features**: Missing values are filled with the most frequent value, and the data is encoded using `OneHotEncoder`.
* **Pipeline**: These steps are neatly bundled using Scikit-Learn's `ColumnTransformer` and `Pipeline`.

## 5. Model Training
Two baseline classification models were trained on the preprocessed data. Both models used `class_weight='balanced'` to account for the heavy imbalance between benign and malicious logs.
1.  **Logistic Regression**: A linear model used as a simple, interpretable baseline.
2.  **Decision Tree Classifier**: A non-linear model (max depth = 10) that can capture complex routing rules or thresholds.

## 6. Evaluation & Results
The models were evaluated using Accuracy, Precision, Recall, and F1-score on the 20% test set (1.2 million rows).

* **Logistic Regression**: Achieved a perfect **1.00** across all metrics. *Note: In real-world scenarios, a perfect score usually indicates data leakage or that the synthetic dataset has a trivial, deterministic rule separating the classes.*
* **Decision Tree**: Achieved an Accuracy of ~79% and a weighted F1-score of ~0.86. However, looking at the classification report, it struggled with the minority classes:
    * *Malicious*: Perfect recall (1.00) but terrible precision (0.09), meaning it flagged way too many things as malicious.
    * *Suspicious*: Good precision (1.00) but poor recall (0.37), meaning it missed a lot of suspicious events.

## 7. Conclusion & Next Steps
The notebook successfully establishes a functional ML pipeline that meets the basic business criteria. However, the author wisely notes that these are just simple baselines. 

**Proposed Next Steps:**
* Add richer features (like session tracking or time-window aggregations).
* Test more advanced models like Random Forest or XGBoost.
* Implement cross-validation and tune prediction thresholds for better real-world deployment.

***


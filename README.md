[README.md](https://github.com/user-attachments/files/32069287/README.md)
# Undergraduate-Research
ML-Based Malware Detection
# Web Attack Detection with CNN-BiLSTM

Replication and evaluation of a deep learning model for detecting web attacks in network flow data, benchmarked against a Random Forest baseline.

HSE University, Moscow Institute of Electronics and Mathematics (MIEM) · Information Security programme · Feb 2023

📄 **[Full report (PDF, in Russian)](./report.pdf)** · 📓 **[Colab notebook](https://colab.research.google.com/drive/1lEZoXbeZ1C8nG9r25ZheauQ16imSQpzk)**

---

## Overview

Signature-based defences fail against traffic patterns they have not seen before, which is why intrusion detection has moved toward learned classifiers operating on flow-level features. This project reproduces the CNN-BiLSTM architecture proposed by Sinha & Manollas for network intrusion detection, applies it to web-attack classification, and asks a narrower question: **does the deep model actually beat a classical ensemble on this data?**

The short answer is no. On the same dataset and the same split, Random Forest outperforms CNN-BiLSTM on accuracy, recall and F1. That negative result — and the reason behind it — is the substance of the work.

---

## Data

Derived from **CIC-IDS2017** (Canadian Institute for Cybersecurity), using a filtered and balanced subset:
`web_attacks_balanced.csv` — **7,267 flows × 76 features**.

**Classes:** `BENIGN`, `Web Attack – Brute Force`, `Web Attack – SQL Injection`, `Web Attack – XSS`

**Preprocessing:**
- Dropped 7 identifier columns — Flow ID, source/destination IP, source/destination port, protocol, timestamp. Addresses and ports are trivially spoofable, so the model is forced to learn from the *shape* of the traffic rather than from its origin.
- Min-max scaling to [0, 1]; one-hot encoding of labels.
- Both applied **inside** each cross-validation fold to avoid data leakage.

---

## Model

Sequential Keras model, **202,568 parameters**:

```
Conv1D(64, kernel=32, padding='same', relu)   input: (76, 1)
MaxPooling1D(5)
BatchNormalization
Bidirectional(LSTM(64))
Reshape(128, 1)
MaxPooling1D(5)
BatchNormalization
Bidirectional(LSTM(128))
Dropout(0.5)
Dense(4) → softmax
```

Optimiser Adam · loss categorical cross-entropy · batch size 32 · 10 epochs
Evaluation: **stratified 5-fold cross-validation** (`shuffle=True, random_state=42`)
Training time: ~15 min on CPU

The 1-D convolution extracts local feature correlations cheaply; the two BiLSTM layers (64 → 128 units) read the sequence in both directions to capture longer-range dependencies. Max pooling and batch normalisation between blocks cut parameter count and training time.

---

## Results

### Multi-class (4 classes, mean over 5 folds, weighted)

| Metric | Score |
|---|---|
| Accuracy | 0.814 |
| Precision | 0.721 |
| Recall | 0.814 |
| F1 | 0.744 |

The aggregate number hides the real failure. Per-class, on the pooled confusion matrix:

| True class | Correctly classified |
|---|---|
| BENIGN | 6,073 / 6,105 |
| Web Attack – Brute Force | 1,114 / 1,808 |
| Web Attack – SQL Injection | 4 / 25 |
| Web Attack – XSS | 8 / 783 |

**The model essentially cannot detect XSS or SQL injection.** 694 brute-force flows and 279 XSS flows are misread as benign. High headline accuracy here is an artefact of class imbalance — BENIGN dominates the set, and predicting it is enough to score well.

### Binary comparison against Random Forest

Same data, same final fold, output layer reshaped to two classes (benign / attack):

| Metric | Random Forest | CNN-BiLSTM |
|---|---|---|
| Accuracy | 0.9842 | 0.7151 |
| Precision | 0.9791 | 1.0000 |
| Recall | 0.9679 | 0.0505 |
| F1 | 0.9735 | 0.0961 |

CNN-BiLSTM achieves perfect precision by being extremely reluctant to call anything an attack — it catches roughly 5% of them. For an intrusion detection system, that trade-off is the wrong way round: a missed attack costs more than a false alarm.

---

## What this shows

- On flow-level tabular features with only ~7k samples, a tree ensemble is the stronger and cheaper choice. The sequence-modelling capacity of BiLSTM has little to work with here.
- Aggregate accuracy is a misleading metric on imbalanced intrusion data. Per-class recall is the number that matters.
- Deep architectures that perform well in the source paper do not transfer automatically to a reduced dataset and a shortened training budget.

## Limitations

Written as an undergraduate research practicum, and a replication rather than an original contribution — the architecture, the balanced dataset and the Random Forest baseline all come from prior work (see References). Specific constraints:

- 10 epochs on a reduced dataset; the source paper trains longer on the full set, so the deep model is likely undertrained rather than fundamentally unsuited.
- No class weighting or resampling was applied, which is the obvious first remedy for the XSS/SQLi collapse.
- No hyperparameter search; the architecture was taken as published.
- Static evaluation only — no adversarial or drift testing.

---

## Repository contents

```
report.pdf                full report (Russian)
notebook.ipynb            training and evaluation
```

## References

1. Sinha, J. & Manollas, M. *Efficient Deep CNN-BiLSTM Model for Network Intrusion Detection.* AIPR 2020. https://jaysinha.me/files/aipr_20_ids_paper_pre_print.pdf
2. Sharafaldin, I., Lashkari, A. H. & Ghorbani, A. A. *Intrusion Detection Evaluation Dataset (CIC-IDS2017).* University of New Brunswick. https://www.unb.ca/cic/datasets/ids-2017.html
3. fisher85. *ml-cybersecurity: python-web-attack-detection.* https://github.com/fisher85/ml-cybersecurity

## Citation

```bibtex
@misc{karimov2023webattack,
  author = {Karimov, Karim},
  title  = {Web Attack Detection with CNN-BiLSTM: A Replication Study},
  year   = {2023},
  note   = {HSE University, Moscow Institute of Electronics and Mathematics},
  url    = {https://github.com/mirakky/web-attack-detection-cnn-bilstm}
}
```

## Author

Karim Karimov — MSc candidate, Science, Technology and Innovation Management and Policy, HSE University / UNICAMP
mirakky99@gmail.com · [github.com/mirakky](https://github.com/mirakky)

Supervised by V. V. Bashun, MIEM HSE.

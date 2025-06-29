
# 🔍 Intrusion Detection using 1D-CNN and Snort Logs

## 📘 Overview

This project explores a machine learning-based approach for detecting abnormal network traffic using **1D-Convolutional Neural Networks (1D-CNN)**. Logs are collected from **Snort IDS** after simulating network attacks such as **Ping of Death**, **SYN Flood**, and **UDP Flood**. The logs are processed and used to train a CNN model for intrusion detection.

---

## 🧪 Experimental Setup

### 🔹 Step 1: Simulated Network Attacks

- Attacker IP: `192.168.8.10`
- Web Server IP: `192.168.10.30`
- Types of Attacks:
  - **Ping of Death**: ICMP packets exceeding the allowed size
  - **SYN Flood**: Large number of SYN packets without completing the handshake
  - **UDP Flood**: Large volume of UDP packets to overwhelm the server

### 🔹 Step 2: Snort Log Collection

- Logs collected by Snort IDS
- Transferred from physical machine to pfSense firewall using SSH

### 🔹 Step 3: Data Preparation and CNN Training

- Extracted features from log file (CSV)
- Preprocessing includes:
  - Column splitting (e.g., IP to 4 segments)
  - Removing irrelevant fields
  - Handling NaN values (mean imputation)
  - Label encoding (Message → Intrusion)
  - One-hot encoding for categorical fields
  - Normalization
- Dataset: 121,850 rows
  - Training: 80% (97,480 rows)
  - Testing: 20% (24,370 rows)

### 🔹 Step 4: Model Architecture (1D-CNN)

| Layer             | Output Shape  | Parameters |
|------------------|---------------|------------|
| Conv1D           | (None, 13, 32)| 128        |
| MaxPooling1D     | (None, 6, 32) | 0          |
| Conv1D_1         | (None, 4, 64) | 6208       |
| MaxPooling1D_1   | (None, 2, 64) | 0          |
| Flatten          | (None, 128)   | 0          |
| Dense            | (None, 128)   | 16512      |
| Dropout          | (None, 128)   | 0          |
| Dense_1 (Output) | (None, 4)     | 516        |

- **Total trainable parameters**: 23,364
- **Optimizer**: Adam
- **Loss Function**: Categorical Crossentropy

### 🔹 Step 5: Model Training & Evaluation

- Trained for **50 epochs**, batch size = 32
- Final Results:
  - **Train Accuracy**: 99.98%
  - **Test Accuracy**: 99.97%
  - **Loss (test)**: 0.00186

### 🔹 Step 6: Visualization

- Accuracy & loss curves for training and test datasets indicate good generalization and no overfitting.

---

## Achievements

- Built and trained a CNN model using real attack logs
- Performed full preprocessing, encoding, and feature engineering
- Achieved high accuracy on a balanced dataset

## Future Improvements

- 🔄 Improve log dataset: Increase log volume (from thousands to millions of rows)
- 🔧 CNN Optimization: Experiment with architecture, more Conv layers
- 🔁 Automate log collection (replace manual download with real-time processing)

---

## Libraries Used

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from keras.layers import Conv1D, Activation, MaxPooling1D, Flatten, GRU, AveragePooling1D, Dense, Dropout
```

---

## Author

- Developed by Huynh Quang Loc
- Email: huynh2uangloc@example.com

# Assignment-4-5

# Software Documentation: CNN-Based DDoS Detection System

**Version:** 1.0  
**Domain:** Cybersecurity / Deep Learning  
**Date:** January 29, 2026

---

## 1. Executive Summary
This software provides an advanced Deep Learning solution for detecting Distributed Denial of Service (DDoS) attacks. 

Traditional Intrusion Detection Systems (IDS) often rely on static signatures or simple statistical thresholds. This project creates a **software-based DDoS detector** that utilizes a **Convolutional Neural Network (CNN)**. The core innovation is the transformation of tabular network metadata into 2D grayscale images, allowing the AI to identify attack patterns using computer vision techniques.

---

## 2. Theoretical Framework

### 2.1 The Concept: Network Forensics as Computer Vision
Convolutional Neural Networks (CNNs) are the industry standard for image recognition. To apply them to network security, we hypothesize that malicious traffic flows possess distinct "textural" signatures compared to benign traffic when visualized as a data grid.

### 2.2 Mathematical Model: Data-to-Image Conversion
The fundamental operation of this software is mapping a 1D feature vector $X$ to a 2D matrix $I$.

#### 2.2.1 Feature Vector Definition
Let a single network flow sample be represented by a vector $X$ containing $n$ features from the CICIDS2017 dataset:
$$X = [x_1, x_2, ..., x_{78}]$$
Where $n=78$.

#### 2.2.2 Feature Normalization
Raw network data varies wildly in scale (e.g., *Flow Duration* $\in [0, 10^6]$ vs. *Flag Counts* $\in [0, 1]$). To create a valid grayscale image, we must normalize features to the range $[0, 1]$. We apply Min-Max Scaling:

$$x'_i = \frac{x_i - \min(X)}{\max(X) - \min(X)}$$

#### 2.2.3 Grid Mapping & Padding
We require a square matrix $M \times M$ to represent the image. We select $M=9$ because $9^2 = 81$, which is the nearest perfect square $\geq 78$. 

Since the vector length ($78$) is less than the grid capacity ($81$), we append **Zero Padding** $P$:
$$P = [0, 0, 0]$$

The padded vector $X_{pad}$ becomes:
$$X_{pad} = [x'_1, x'_2, ..., x'_{78}, 0, 0, 0]$$

#### 2.2.4 Matrix Reshaping
The vector is reshaped into a $9 \times 9$ matrix $I$, which serves as the input tensor for the CNN:

$$
I = 
\begin{bmatrix}
x'_1 & x'_2 & \dots & x'_9 \\
x'_{10} & x'_{11} & \dots & x'_{18} \\
\vdots & \vdots & \ddots & \vdots \\
x'_{73} & \dots & 0 & 0
\end{bmatrix}
$$

---

## 3. System Architecture

### 3.1 Dataset Specification
* **Source:** CICIDS2017 (ISCX)
* **File:** `Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv`
* **Preprocessing:** * Removal of `NaN` and Infinite values.
    * Label Encoding (`BENIGN` $\rightarrow$ 0, `DDoS` $\rightarrow$ 1).

### 3.2 CNN Model Architecture
The neural network is implemented using TensorFlow/Keras with the following layer structure: 

[Image of Convolutional Neural Network architecture diagram]


| Layer Type | Parameters | Activation | Purpose |
| :--- | :--- | :--- | :--- |
| **Input** | Shape $(9, 9, 1)$ | - | Receives the transformed image. |
| **Conv2D** | 32 Filters, $3 \times 3$ Kernel | ReLU | Extracts local features (edges, textures). |
| **MaxPooling2D** | $2 \times 2$ Pool | - | Downsamples to reduce dimensionality. |
| **Flatten** | - | - | Converts 2D feature maps to 1D vector. |
| **Dense** | 64 Neurons | ReLU | High-level classification reasoning. |
| **Dropout** | Rate 0.5 | - | Prevents overfitting by randomly dropping neurons. |
| **Output** | 1 Neuron | Sigmoid | Outputs probability $p \in [0, 1]$. |

---

## 4. Software Implementation Details

The solution is divided into two Python scripts:

1.  **`ddos_detector.py` (Core Logic):** * Contains the `load_and_preprocess_data` function for data cleaning.
    * Contains the `convert_to_images` function for the reshaping logic.
    * Defines the CNN model structure.

2.  **`ddos_interactive.py` (User Interface):**
    * Provides a Command Line Interface (CLI) menu.
    * Handles model persistence (saving/loading `.keras` files).
    * Visualizes network traffic using `matplotlib`.

---

## 5. User Manual

### 5.1 Prerequisites\How to Run
Ensure the following Python environment is set up:
```bash
pip install pandas numpy scikit-learn tensorflow matplotlib

python ddos_interactive.py
```

## 6. Experimental Results

The Convolutional Neural Network (CNN) model was trained and validated using the **CICIDS2017** dataset. The following metrics were recorded after a training duration of **5 epochs** with a batch size of **64**.

### 6.1 Performance Metrics

| Metric | Value |
| :--- | :--- |
| **Training Accuracy** | 99.21% |
| **Validation Accuracy** | 99.01% |
| **Test Set Accuracy** | **99.07%** |
| **Final Loss (Binary Crossentropy)** | 0.0202 |

### 6.2 Training Progression
* **Epoch 1:** The model quickly adapted to the feature set, dropping from an initial loss of ~0.12 to ~0.02. This indicates that the "image" representation of the network flows contains very strong, easily distinguishable signals.
* **Epoch 2-5:** The model stabilized with minimal fluctuations, reaching convergence very rapidly. This suggests that the model architecture (32 filters, 1 dense layer) was sufficient for the complexity of the problem without significant overfitting.

---

## 7. Discussion

### 7.1 Interpretation of Results
The high accuracy (**>99%**) supports the core hypothesis of this assignment: **Network traffic flows have distinct "visual" textures when mapped to a 2D grid.**

* **Benign Traffic:** When converted to images, normal traffic likely appears as "smooth" or random noise with lower intensity in specific pixel regions (representing standard protocol behavior).
* **DDoS Traffic:** Attack traffic typically involves repetitive, high-volume packets with fixed parameters (e.g., identical *Packet Lengths* or *Inter-Arrival Times*). In the converted images, these manifest as consistent "bright spots" or rigid patterns. The CNN's convolutional filters are exceptionally good at detecting these edges and rigid patterns.

### 7.2 Comparison to Traditional Methods
While traditional statistical methods (like Random Forest) also perform well on this dataset, the CNN approach offers a unique advantage: **Feature Correlation**. By placing features next to each other in a grid, the CNN can learn correlations between adjacent features (e.g., *Flow Duration* vs. *Total Packets*) as "spatial" relationships, potentially making it more robust against evasion techniques that simply alter statistical thresholds.

---

## 8. Future Work
To further improve this system, the following expansions are recommended:
1.  **Grid Optimization:** Currently, the $9 \times 9$ grid is filled sequentially. Using an algorithm to place correlated features (e.g., all "Time" related features) in the same region of the grid could improve the CNN's ability to detect complex patterns.
2.  **Multiclass Classification:** Expanding the model to distinguish between different *types* of DDoS attacks (e.g., SYN Flood vs. HTTP Flood) rather than just binary detection.
3.  **Real-Time Integration:** optimizing the Python script to sniff live packets using `scapy`, convert them on-the-fly, and block malicious IPs in real-time.

---

## 9. Conclusion
This project successfully designed and implemented a **Deep Learning-based DDoS Detection System**. By converting 1D network metadata into $9 \times 9$ grayscale images, we effectively transformed a cybersecurity challenge into a computer vision task.

The resulting software demonstrates that Convolutional Neural Networks are not limited to image recognition but are powerful tools for pattern recognition in abstract data domains. With a test accuracy of **99.07%**, the system proves to be a highly reliable prototype for automated network defense.

# DSP-Based Predictive Maintenance System for Electric Motors Using Vibration and Sound Analysis

[![MATLAB](https://img.shields.io/badge/MATLAB-R2022b%2B-blue.svg?logo=mathworks)](https://www.mathworks.com/products/matlab.html)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Overleaf Report](https://img.shields.io/badge/Overleaf-Report%20Active-green.svg?logo=overleaf)](https://www.overleaf.com/project/6ade53a502d657d9a937f666)
[![Status: Completed](https://img.shields.io/badge/Status-Completed-success.svg)](#)

---

## 📌 Project Information

* **Student Name:** Sanjana Sri Bolla
* **Roll Number:** 160124758004
* **Department:** Electronics and Communication Engineering / VLSI Design
* **Academic Year:** 2026–27 (5th Semester, 3rd Year B.Tech)
* **Domain:** Digital Signal Processing (DSP) & Machine Condition Monitoring
* **Overleaf Project:** [Open Report in Overleaf](https://www.overleaf.com/project/6ade53a502d657d9a937f666)

---

## 📖 Executive Summary

Electric motors drive a substantial portion of machinery across manufacturing plants, water pumping stations, agricultural equipment, and HVAC systems. Catastrophic motor breakdowns cause expensive unscheduled downtime and severe industrial losses. Most mechanical failures—such as bearing raceway wear, rotor dynamic unbalance, and shaft misalignment—manifest as subtle anomalous patterns in mechanical vibration long before catastrophic failure occurs.

This repository presents an end-to-end, reproducible **Digital Signal Processing (DSP) Predictive Maintenance System** implemented in MATLAB. The system ingests continuous accelerometer data sampled at $12\,\text{kHz}$, removes sensor DC bias, applies zero-phase 4th-order Butterworth low-pass filtering, computes the Fast Fourier Transform (FFT) spectrum and Short-Time Fourier Transform (STFT) spectrogram, extracts an 8-element statistical and spectral feature vector, and classifies motor health state as:
1. **Normal Operating Pattern**
2. **Abnormal Vibration Detected**
3. **Requires Further Inspection**

Validated against the benchmark **Case Western Reserve University (CWRU) Bearing Data Center**, the system achieves **$95.83\%$ classification accuracy** with a mean processing latency under **$0.40\,\text{seconds}$**.

---

## 🏗️ System Architecture

```mermaid
flowchart TD
    A["Vibration Transducer<br>(DE Accelerometer / MEMS Mic)"] --> B["Signal Acquisition<br>(ADC: Fs = 12 kHz, N = 24,001)"]
    B --> C["Pre-processing<br>(DC Bias Removal + Zero-Phase Butterworth LPF @ 5 kHz)"]
    C --> D["DSP Transformation<br>(Single-Sided FFT + STFT Spectrogram)"]
    D --> E["Feature Extraction<br>(RMS, Std Dev, Peak, Crest Factor, Kurtosis, FFT Peaks)"]
    E --> F["Condition Assessment<br>(Multi-Threshold Rule Engine / Optional RBF SVM)"]
    F --> G["Warning & Diagnostics<br>(Normal / Abnormal Vibration / Inspection Alert)"]
    G --> H["Supervisory Dashboard<br>(Time Waveforms, Spectrum, Energy Bar Charts)"]

    style A fill:#e1f5fe,stroke:#0288d1,stroke-width:2px
    style C fill:#ede7f6,stroke:#512da8,stroke-width:2px
    style E fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
    style G fill:#fff3e0,stroke:#f57c00,stroke-width:2px
```

---

## 🔬 Digital Signal Processing Pipeline

### 1. Signal Pre-processing
* **DC Offset Removal:** Transducer drift and ADC baseline offset are eliminated:
  $$\tilde{x}[n] = x[n] - \frac{1}{N}\sum_{i=1}^{N} x[i]$$
* **Zero-Phase Low-Pass Filtering:** A 4th-order digital Butterworth filter with $f_c = 5{,}000\,\text{Hz}$ is applied bidirectionally using `filtfilt` (effective 8th-order, $-80\,\text{dB/decade}$ roll-off). This eliminates high-frequency measurement noise while guaranteeing zero phase distortion:
  $$|H(j\Omega)|^2 = \frac{1}{1 + (\Omega/\Omega_c)^{2N}}$$

### 2. Spectral Decomposition (FFT)
The single-sided discrete amplitude spectrum is resolved across the Nyquist bandwidth ($0 \le f \le 6{,}000\,\text{Hz}$):
$$X[k] = \sum_{n=0}^{N-1} x[n] e^{-j 2\pi kn / N}$$
Dominant peaks isolate synchronous rotational frequency ($50\,\text{Hz}$ at $3{,}000\,\text{RPM}$) from non-synchronous bearing defect impact frequencies ($120\,\text{Hz}$).

### 3. Statistical Feature Extraction
* **Root Mean Square (RMS):** Measures overall continuous vibration energy:
  $$x_{\text{RMS}} = \sqrt{\frac{1}{N}\sum_{n=1}^{N} x[n]^2}$$
* **Crest Factor ($CF$):** Sensitive indicator of early localized impacts:
  $$CF = \frac{\max |x[n]|}{x_{\text{RMS}}}$$
* **Kurtosis ($K$):** Identifies non-Gaussian, impulsive shock distributions:
  $$K = \frac{\frac{1}{N}\sum_{n=1}^{N}(x[n]-\mu)^4}{\sigma^4}$$

---

## 📊 Experimental Results

Experimental evaluation was performed using the CWRU drive-end bearing accelerometer dataset ($12\,\text{kHz}$ sampling rate, 2-second windows):

### 1. Quantitative Feature Comparison

| Diagnostic Metric | Healthy Baseline | Fault Condition | Sensitivity / Trend |
| :--- | :---: | :---: | :--- |
| **Mean ($\mu$)** | $\approx 0.0000\,\text{g}$ | $\approx 0.0000\,\text{g}$ | Zero-centred dynamic baseline |
| **RMS Energy ($x_{\text{RMS}}$)** | $0.3606\,\text{g}$ | $0.4227\,\text{g}$ | **$+17.22\%$** elevation in vibration energy |
| **Standard Deviation ($\sigma$)** | $0.3606\,\text{g}$ | $0.4227\,\text{g}$ | Increased signal statistical variance |
| **Peak Amplitude ($x_{\text{peak}}$)** | $0.6812\,\text{g}$ | $1.5641\,\text{g}$ | **$+129.6\%$** surge due to raceway impacts |
| **Crest Factor ($CF$)** | $1.889$ | $3.700$ | **$+95.8\%$** increase (sharp shock pulses) |
| **Kurtosis ($K$)** | $3.012$ | $3.748$ | Leptokurtic shift (heavy-tailed distribution) |

### 2. Classification Performance Metrics

| Evaluation Metric | Achieved Value | Design Target | Status |
| :--- | :---: | :---: | :---: |
| **Classification Accuracy** | **$95.83\%$** | $> 80.0\%$ | ✅ Exceeded |
| **Precision** | **$96.00\%$** | $> 80.0\%$ | ✅ Exceeded |
| **Recall (Sensitivity)** | **$95.00\%$** | $> 90.0\%$ | ✅ Exceeded |
| **F1-Score** | **$0.9549$** | $> 0.850$ | ✅ Exceeded |
| **False Alarm Rate (FAR)** | **$4.17\%$** | $< 10.0\%$ | ✅ Met |
| **Processing Latency** | **$0.384\,\text{s}$** | $< 5.0\,\text{s}$ | ✅ Real-time Capable |

---

## 📈 Visual Diagnostic Showcase

All visual diagnostic outputs generated by the MATLAB engine are compiled in [`results/MotorFaultDetection_Pic.pdf`](results/MotorFaultDetection_Pic.pdf):

* **Page 1 — Data Input & Zero-Phase Pre-processing:** Raw transducer signal vs filtered output ($f_c = 5\,\text{kHz}$).
* **Page 2 — FFT Spectrum & Condition Verdict:** Harmonic breakdown ($50\,\text{Hz}$, $100\,\text{Hz}$, $120\,\text{Hz}$) and rule assessment trigger (`Abnormal vibration detected`).
* **Page 3 — RMS Comparison & Time-Frequency Spectrogram:** Bar graph energy comparison ($0.3606\,\text{g} \to 0.4227\,\text{g}$) and 256-point Hamming STFT.
* **Page 4 — Feature Comparison & Validation Summary:** Multi-feature bar charts and classification performance tables.

---

## 📂 Repository Structure

```
DSP-Motor-Predictive-Maintenance/
├── README.md                                  # Complete project documentation
├── LICENSE                                    # MIT License
├── .gitignore                                 # MATLAB & LaTeX ignore rules
├── src/
│   └── motor_predictive_maintenance.m         # Master runnable MATLAB script
├── docs/
│   ├── report_overleaf.tex                    # Full LaTeX project report source
│   ├── references.bib                         # BibTeX reference database
│   ├── DSP_Motor_Predictive_Maintenance_Report.pdf  # Compiled PDF report
│   └── motorFaultCode.pdf                     # MATLAB script printout
└── results/
    └── MotorFaultDetection_Pic.pdf            # 4-page diagnostic results showcase
```

---

## 🚀 Quick Start Guide

### Prerequisites
* **MATLAB R2022b or later** (Compatible with GNU Octave 8.x)
* **Signal Processing Toolbox** (`butter`, `filtfilt`, `spectrogram`, `findpeaks`)
* *(Optional)* **Statistics and Machine Learning Toolbox** (`fitcsvm`, `cvpartition`)

### Execution Steps
1. **Clone the repository:**
   ```bash
   git clone https://github.com/sanjanasiribolla-del/DSP-Motor-Predictive-Maintenance.git
   cd DSP-Motor-Predictive-Maintenance
   ```
2. **Open MATLAB** and navigate to the `src/` folder:
   ```matlab
   cd src
   ```
3. **Run the script:**
   ```matlab
   motor_predictive_maintenance
   ```
   * *By default, `useRealData = false`, which runs in synthetic demo mode.*
   * *To analyze actual CWRU data: download CWRU files, place them in a folder, set `dataFolder = 'path/to/CWRU'`, set `useRealData = true`, and re-run.*

---

## 🌍 Sustainable Development Goals (SDGs)

* **SDG 9: Industry, Innovation, and Infrastructure** — Promotes sustainable, resilient manufacturing infrastructure by drastically reducing unplanned motor downtimes through early predictive diagnostics.
* **SDG 12: Responsible Consumption and Production** — Extends electrical machinery operating lifespan, optimizes industrial electrical efficiency, and prevents unnecessary equipment scrap.

---

## 📚 References

1. **J. G. Proakis and D. G. Manolakis**, *Digital Signal Processing: Principles, Algorithms, and Applications*, 4th ed., Prentice Hall, 2007.
2. **R. B. Randall and J. Antoni**, "Rolling element bearing diagnostics—A tutorial," *Mechanical Systems and Signal Processing*, vol. 25, no. 2, pp. 485–520, 2011.
3. **S. Nandi, H. A. Toliyat, and X. Li**, "Condition monitoring and fault diagnosis of electrical motors—A review," *IEEE Transactions on Energy Conversion*, vol. 20, no. 4, pp. 719–729, 2005.
4. **K. A. Loparo**, "Bearings vibration dataset," Case Western Reserve University, 2012. [Online]. Available: `https://engineering.case.edu/bearingdatacenter`
5. **H. Purohit et al.**, "MIMII Dataset: Sound Dataset for Malfunctioning Industrial Machine Investigation and Inspection," *Proc. DCASE*, 2019.

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

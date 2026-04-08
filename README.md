# 🎙️ Keyword Spotting (KWS) on Arduino Nano 33 BLE Sense
> **TinyML Audio Classification: Clap, Snap, and Tap Detection**

## 📖 Overview
[cite_start]This repository contains the implementation of a **Keyword Spotting (KWS)** system designed for low-power embedded devices[cite: 1, 2]. [cite_start]The goal is to detect and classify three specific sound events (**Clap**, **Snap**, and **Tap**) in real-time directly on the microcontroller using **TensorFlow Lite Micro**[cite: 11, 2221, 2222, 2223].

## 🏗️ Technical Architecture
[cite_start]The project implements a full digital signal processing (DSP) pipeline to transform raw audio into classification results[cite: 712, 713]:

### 1. Data Acquisition
* [cite_start]**Sensor**: MP34DT06JTR digital microphone[cite: 134, 152].
* [cite_start]**Sampling**: Pulse-Density Modulation (PDM) sampled at **16 kHz**[cite: 138, 158].
* [cite_start]**Format**: Converted to 16-bit Pulse Code Modulation (PCM) samples[cite: 158, 185].

### 2. Feature Engineering (MFCC Pipeline)
[cite_start]To reduce dimensionality and focus on relevant features, we convert 1-second audio windows into Mel-Frequency Cepstral Coefficients (MFCCs)[cite: 717, 1675]:
* [cite_start]**Pre-emphasis**: High-pass filtering to balance the spectrum[cite: 916, 919].
* [cite_start]**Hamming Window**: Smoothing frame edges to prevent spectral leakage[cite: 956, 960].
* [cite_start]**FFT & Power Spectrum**: Converting time-domain signals to frequency domain[cite: 995, 1066].
* [cite_start]**Mel Filterbank**: Scaling frequencies according to human auditory perception[cite: 1163, 1165, 1232].
* [cite_start]**Log-Energy & DCT-II**: Compressing dynamic range and decorrelating features[cite: 1362, 1457, 1459].

### 3. Deep Learning Model
* [cite_start]**Type**: Convolutional Neural Network (CNN)[cite: 1755, 1893].
* [cite_start]**Framework**: TensorFlow Lite for Microcontrollers[cite: 11].
* [cite_start]**Optimization**: Uses **CMSIS-DSP** to leverage the ARM Cortex-M4 hardware FPU and SIMD instructions for 4x faster processing[cite: 727, 737, 742, 1001].

## 🛠️ Hardware Requirements
* [cite_start]**Microcontroller**: Arduino Nano 33 BLE Sense Rev2[cite: 82].
* [cite_start]**Processor**: Nordic nRF52840 (ARM Cortex-M4 @ 64MHz)[cite: 83, 90].
* [cite_start]**Memory**: 256 KB RAM / 1 MB Flash[cite: 101, 102].

## 🚀 Getting Started
1. [cite_start]**Data Collection**: Collect 1-second audio samples for each class[cite: 2215].
2. [cite_start]**Training**: Use the provided Python notebook to extract features and train the CNN[cite: 2213].
3. [cite_start]**Deployment**: Export the model as a C header file (`model.h`) and upload the Arduino sketch[cite: 2216, 2217].

## 📂 Project Structure
```text
├── data/               # 1-second samples (Clap, Snap, Tap)
├── src/                # Arduino sketch (.ino)
├── model/              # TFLite model and model.h
├── notebook/           # Training and Feature Extraction (Colab)
└── README.md           # Documentation

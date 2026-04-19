# 🎙️ Keyword Spotting (KWS) on Arduino Nano 33 BLE Sense
> **TinyML Audio Classification: Clap, Snap, and Tap Detection**
<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/492805f8-6ee6-43be-8ea0-a7f41c6aa45e" />

## 📖 Overview
This repository contains the implementation of a **Keyword Spotting (KWS)** system designed for low-power embedded devices. The goal is to detect and classify three specific sound events (**Clap**, **Snap**, and **Tap**) in real-time directly on the microcontroller using **TensorFlow Lite Micro**.

## 🏗️ Technical Architecture
The project implements a full digital signal processing (DSP) pipeline to transform raw audio into classification results:

### 1. Data Acquisition
* **Sensor**: MP34DT06JTR digital microphone.
* **Sampling**: Pulse-Density Modulation (PDM) sampled at **16 kHz**.
* **Format**: Converted to 16-bit Pulse Code Modulation (PCM) samples.

### 2. Feature Engineering (MFCC Pipeline)
To reduce dimensionality and focus on relevant features, we convert 1-second audio windows into Mel-Frequency Cepstral Coefficients (MFCCs):
* **Pre-emphasis**: High-pass filtering to balance the spectrum.
* **Hamming Window**: Smoothing frame edges to prevent spectral leakage.
* **FFT & Power Spectrum**: Converting time-domain signals to frequency domain.
* **Mel Filterbank**: Scaling frequencies according to human auditory perception.
* **Log-Energy & DCT-II**: Compressing dynamic range and decorrelating features.

### 3. Deep Learning Model
* **Type**: Convolutional Neural Network (CNN).
* **Framework**: TensorFlow Lite for Microcontrollers.
* **Optimization**: Uses **CMSIS-DSP** to leverage the ARM Cortex-M4 hardware FPU and SIMD instructions for faster processing.

## 🛠️ Hardware Requirements
* **Microcontroller**: Arduino Nano 33 BLE Sense Rev2.
* **Processor**: Nordic nRF52840 (ARM Cortex-M4 @ 64MHz).
* **Memory**: 256 KB RAM / 1 MB Flash.

## 🚀 Getting Started
1. **Data Collection**: Collect 1-second audio samples for each class.
2. **Training**: Use the provided Python notebook to extract features and train the CNN.
3. **Deployment**: Export the model as a C header file (model.h) and upload the Arduino sketch.

## 📂 Project Structure
```text
├── data/               # 1-second samples (Clap, Snap, Tap)
├── src/                # Arduino sketch (.ino)
├── model/              # TFLite model and model.h
├── notebook/           # Training and Feature Extraction (Colab)
└── README.md           # Documentation


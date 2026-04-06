# Preventing Wrong Decisions in Smart Building Systems

## 🏢 IoT Sensor Attack Detection with LSTM Autoencoder

A comprehensive project for detecting and classifying attacks on IoT sensors (DHT11) in smart building systems using Deep Learning.

---

## 📋 Project Overview

This project focuses on detecting three critical types of attacks on IoT sensor networks:

### **Attack Types**

| Attack Type | Description | Detection Method |
|------------|-------------|------------------|
| **🔄 Replay Attack** | Attacker replays old valid sensor readings to the system | Pattern repetition analysis & temporal consistency check |
| **🔌 Injection Attack** | Attacker inserts fake/malicious data with extreme values | Statistical outlier detection (3σ threshold) |
| **❄️ Freeze Attack** | Sensor stops transmitting new data (stuck at constant value) | Variance analysis - detects zero variance |

---

## 🧠 Methodology

### **LSTM Autoencoder**
- **Training Data**: Normal DHT11 sensor readings only (unsupervised learning)
- **Architecture**: 
  - Encoder: LSTM(64) → LSTM(32) with Dropout(0.2)
  - Decoder: RepeatVector → LSTM(32) → LSTM(64) → Dense(2)
- **Loss Function**: Mean Squared Error (MSE) - reconstruction error
- **Threshold**: Mean + 3×Std of validation errors

### **Detection Pipeline**
1. Train autoencoder on normal data only
2. Calculate reconstruction error threshold
3. For test data:
   - Calculate MSE (reconstruction error)
   - Analyze temporal patterns
   - Classify attack type based on anomaly signature
   - Output confidence score

---

## 📂 Project Structure

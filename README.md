# Edge-to-Cloud Audio Capture and Streaming Framework

A real-time audio capture and streaming system that enables simultaneous connection to multiple ESP32 devices with automatic buffering during connection loss and secure cloud storage via AWS.

---

## 🚀 Features

- **Multi-Device Support:** Connect to multiple ESP32 BLE devices simultaneously  
- **Smart Buffering:** Circular buffer (3 clips × 32 KB = 96 KB) provides ~3 seconds of audio retention during disconnection  
- **Automatic Recovery:** Buffered audio clips are automatically transferred upon reconnection  
- **Secure Authentication:** AWS Cognito integration with email verification  
- **Private Cloud Storage:** User-scoped S3 storage with encryption  
- **Cross-Platform Ready:** iOS operational, Android deployment ready  
- **Real-Time Monitoring:** Live data transfer with progress tracking  

---

## 📋 Table of Contents

1. [Architecture](#-architecture)  
2. [Technical Specifications](#-technical-specifications)  
3. [Getting Started](#-getting-started)  
4. [Installation](#-installation)  
5. [Usage](#-usage)  
6. [Data Flow](#-data-flow)  
7. [Security](#-security)  
8. [Roadmap](#-roadmap)  
9. [Use Cases](#-use-cases)  
10. [Current Limitations](#-current-limitations)  
11. [Technical Risks & Mitigation](#-technical-risks--mitigation)  
12. [Project Structure](#-project-structure)  
13. [Contributing](#-contributing)  
14. [Acknowledgments](#-acknowledgments)  

---

## 🏗️ Architecture

The system uses a three-tier architecture:

### Layer 1: ESP32 Microcontrollers

- Continuous audio recording (currently simulated; I2S microphone ready)  
- Circular buffer with 3 slots × 32 KB each  
- BLE GATT service for data transmission  
- Automatic buffering when phone disconnects  
- Resends buffered clips on reconnection  

### Layer 2: Flutter Mobile Application

- BLE scanning and multi-device connection management  
- Per-device data buffers and state tracking  
- Protocol parser: header + chunks + end marker  
- Automatic S3 upload after file reception  
- AWS Amplify integration for auth and storage  

### Layer 3: AWS Cloud Services

- **Cognito User Pool:** Email-based authentication  
- **Cognito Identity Pool:** Temporary AWS credentials  
- **S3 Bucket:** Private user-scoped storage (`us-east-1`)  
- **Encryption:** At rest and in transit  

---

## 📊 Technical Specifications

### Core Specs

| Component      | Specification                                |
|---------------|----------------------------------------------|
| ESP32         | Single-core RISC-V @ 160 MHz, ~400 KB SRAM   |
| BLE Version   | 4.2, Custom GATT service                     |
| Audio Format  | WAV, 16 kHz sample rate                      |
| Buffer Size   | 96 KB total (3 clips × 32 KB)                |
| Clip Duration | ~1 second per clip                           |
| BLE Chunk Size| 512 bytes                                    |
| Transfer Rate | ~10–20 KB/s                                  |
| Flutter       | 3.x with Dart 3.0+                           |
| iOS Target    | iOS 12+                                      |
| Max Devices   | 7–10 simultaneous connections                |

### Performance Metrics

- **End-to-end latency:** ~2–5 seconds (capture to S3)  
- **BLE connection time:** ~3–5 seconds  
- **Upload success rate:** >99% with Amplify retry  
- **Buffer overflow protection:** 3-second disconnection tolerance  

---

## 🚦 Getting Started

### Prerequisites

**Hardware**

- ESP32 microcontroller(s)  
- (Optional) I2S microphone module (e.g., INMP441)  
- (Optional) SD card module for extended buffering  

**Software**

- Flutter 3.x+  
- Dart 3.0+  
- Arduino IDE or PlatformIO  
- AWS account with Cognito and S3 configured  
- iOS development environment (Xcode)  

---

## ☁️ AWS Setup

### 1. Create Cognito User Pool

- Region: `us-east-1`  
- Email-based authentication  
- Password: 8+ characters with basic complexity  

### 2. Create Cognito Identity Pool

- Link to the User Pool  
- Enable authenticated access  

### 3. Create S3 Bucket

- Private access configuration  
- Server-side encryption enabled  
- IAM policy path pattern: `private/{cognito-identity-id}/*`  

### 4. Configure IAM Policies

Example S3 access policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::your-bucket-name/private/${cognito-identity.amazonaws.com:sub}/*"
    }
  ]
}

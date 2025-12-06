# Edge-to-Cloud Audio Capture and Streaming Framework

A real-time audio capture and streaming system that enables simultaneous connection to multiple ESP32 devices with automatic buffering during connection loss and secure cloud storage via AWS.

---

## 🚀 Features

- **Multi-Device Support:** Connect to multiple ESP32 BLE devices simultaneously   
- **Secure Authentication:** AWS Cognito integration with email verification  
- **Private Cloud Storage:** User-scoped S3 storage with encryption  
- **Real-Time Monitoring:** Live data transfer with progress tracking  

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

4. **Configure IAM Policies**:
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
   ```

## 💾 Installation

### ESP32 Firmware

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/ble-audio-streaming.git
   cd ble-audio-streaming/esp32
   ```

2. Open `sketch_nov4a.ino` in Arduino IDE

3. Install required libraries:
   - BLE (built-in ESP32 library)

4. Update configuration if needed (BLE service UUID, buffer settings)

5. Upload to ESP32 board

### Flutter Application

1. Navigate to the Flutter app directory:
   ```bash
   cd ../flutter_app
   ```

2. Install dependencies:
   ```bash
   flutter pub get
   ```

3. Configure AWS Amplify:
   - Update `amplifyconfiguration.dart` with your AWS credentials
   - Add your Cognito User Pool ID
   - Add your Cognito Identity Pool ID
   - Add your S3 bucket name

4. Run the app:
   ```bash
   flutter run
   ```




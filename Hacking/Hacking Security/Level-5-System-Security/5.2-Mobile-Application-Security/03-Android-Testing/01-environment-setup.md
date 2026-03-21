# Unit 3: Android Testing — Topic 1: Environment Setup

## Overview

Setting up a proper **Android security testing environment** is the first step in mobile penetration testing. A well-configured lab allows you to decompile apps, intercept traffic, perform runtime analysis, and test on both emulators and physical devices. This topic covers everything needed to build a complete Android testing lab.

---

## 1. Testing Environment Components

```
ANDROID TESTING LAB:

┌─────────────────────────────────────────────────────┐
│                 TESTING MACHINE                      │
│  ┌────────────┐  ┌────────────┐  ┌──────────────┐  │
│  │ Android    │  │ Burp Suite │  │ Analysis     │  │
│  │ Studio/SDK │  │ / mitmproxy│  │ Tools        │  │
│  │ (emulator) │  │ (proxy)    │  │ (jadx, etc.) │  │
│  └──────┬─────┘  └──────┬─────┘  └──────────────┘  │
│         │               │                           │
│    ADB  │          WiFi │                           │
│         │          Proxy│                           │
│  ┌──────┴─────┐  ┌──────┴─────┐                    │
│  │  Android   │  │  Physical  │                    │
│  │  Emulator  │  │  Device    │                    │
│  │  (rooted)  │  │  (rooted)  │                    │
│  └────────────┘  └────────────┘                    │
└─────────────────────────────────────────────────────┘
```

---

## 2. Essential Tools Installation

```bash
# === ANDROID SDK / PLATFORM TOOLS ===
# Install Android Studio or standalone SDK tools
# Key components: adb, emulator, build-tools

# Verify ADB
adb version
adb devices  # List connected devices

# === EMULATOR SETUP ===
# Option 1: Android Studio AVD Manager
# Create virtual device → Choose Pixel → Select API level
# Use Google APIs image (NOT Google Play — can't root)

# Option 2: Genymotion (faster, pre-rooted)
# Download from genymotion.com
# Create device → Already rooted!

# === PROXY SETUP (Burp Suite) ===
# 1. Configure Burp to listen on all interfaces (0.0.0.0:8080)
# 2. On device/emulator: Settings → WiFi → Proxy → Manual
#    Host: <your-machine-IP>  Port: 8080
# 3. Export Burp CA certificate
# 4. Install on device (see traffic interception topic)

# === FRIDA ===
pip install frida-tools
# Download frida-server for device architecture
# Push to device:
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server &

# Verify Frida
frida-ps -U  # List processes on USB device

# === OBJECTION ===
pip install objection
# Runtime mobile exploration toolkit (built on Frida)

# === APKTOOL ===
# Download from ibotpeaches.github.io/Apktool/
apktool d target.apk -o output_dir  # Decompile
apktool b output_dir -o rebuilt.apk  # Rebuild

# === JADX ===
# Download from github.com/skylot/jadx
jadx-gui target.apk  # GUI decompiler
jadx target.apk -d output_dir  # CLI decompiler

# === MobSF (Mobile Security Framework) ===
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf
# Access at http://localhost:8000
# Upload APK for automated analysis
```

---

## 3. Device Preparation

```bash
# === ROOTING (for full testing) ===
# Option 1: Magisk (most common)
# Flash Magisk via custom recovery (TWRP)
# Provides root with MagiskSU
# Supports MagiskHide (hide root from apps)

# Option 2: Emulator (already rooted with certain images)
# Use Google APIs image (NOT Google Play Store)
# adb root  # Restart ADB as root

# === USB DEBUGGING ===
# Settings → About Phone → Tap Build Number 7 times
# Settings → Developer Options → Enable USB Debugging

# === ADB ESSENTIALS ===
adb devices              # List devices
adb shell                # Interactive shell
adb push local remote    # Upload file
adb pull remote local    # Download file
adb install app.apk      # Install app
adb uninstall com.pkg    # Uninstall app
adb logcat               # View device logs
adb shell pm list packages  # List installed apps

# === CERTIFICATE INSTALLATION (for traffic interception) ===
# Android 7+: User certs not trusted by apps targeting SDK 24+
# Must install as SYSTEM cert (requires root):
adb root
adb remount
adb push burp-cert.pem /system/etc/security/cacerts/9a5ba575.0
adb shell chmod 644 /system/etc/security/cacerts/9a5ba575.0
adb reboot
```

---

## 4. Testing Checklist

```
ENVIRONMENT VERIFICATION:
□ ADB connected and device recognized
□ Device rooted (Magisk or emulator root)
□ USB debugging enabled
□ Frida server running on device
□ Proxy configured and intercepting traffic
□ Burp CA certificate installed as system cert
□ Analysis tools installed (jadx, apktool, MobSF)
□ Target application installed
□ Sufficient storage space on device
□ Screen recording/screenshot capability
```

---

## Summary Table

| Tool | Purpose | Installation |
|------|---------|-------------|
| ADB | Device communication | Android SDK |
| Burp Suite | Traffic interception | portswigger.net |
| Frida | Runtime instrumentation | pip install frida-tools |
| Objection | Runtime exploration | pip install objection |
| jadx | Java decompilation | GitHub release |
| apktool | APK decompile/rebuild | GitHub release |
| MobSF | Automated analysis | Docker |
| Magisk | Root management | GitHub release |

---

## Revision Questions

1. What are the essential components of an Android testing lab?
2. How do you set up Frida on an Android device?
3. Why do you need root access for comprehensive testing?
4. How do you install a proxy CA certificate as a system cert on Android 7+?
5. What is the difference between Google APIs and Google Play emulator images?

---

*Previous: None (First topic in this unit) | Next: [02-static-analysis.md](02-static-analysis.md)*

---

*[Back to README](../README.md)*

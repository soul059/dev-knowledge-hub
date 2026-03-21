# Unit 4: OSCP Preparation — Topic 4: Report Writing

## Overview

The OSCP exam requires a **professional penetration testing report** submitted within 24 hours after the hacking portion. A poor report can fail you even if you hacked enough machines. This topic covers report structure, writing techniques, and common mistakes to avoid.

---

## 1. Report Structure

```
OSCP REPORT TEMPLATE:

1. COVER PAGE
   → Student name and OSID
   → Exam date
   → Report title
   → "Confidential" marking

2. TABLE OF CONTENTS
   → Auto-generated
   → All sections linked

3. EXECUTIVE SUMMARY
   → High-level overview
   → Number of machines compromised
   → Critical findings summary
   → Risk rating overview

4. METHODOLOGY
   → Tools used
   → Approach taken
   → Scanning methodology
   → Exploitation approach

5. FINDINGS (per machine)
   → Machine IP address
   → Service enumeration
   → Vulnerability identified
   → Exploitation steps
   → Post-exploitation
   → Proof screenshots
   → Remediation recommendations

6. APPENDICES
   → Tool output
   → Full scan results
   → Additional screenshots

PAGE ESTIMATES:
  → Cover + TOC: 2 pages
  → Executive summary: 1-2 pages
  → Methodology: 1-2 pages
  → Per machine: 5-15 pages
  → Total: 30-80+ pages
```

---

## 2. Writing Machine Sections

```
PER-MACHINE DOCUMENTATION:

SECTION A: SERVICE ENUMERATION
  "Initial port scanning revealed the following
   open ports and services:"
  
  → Include Nmap output (formatted)
  → List all discovered services
  → Note versions identified
  → Document any interesting findings

SECTION B: VULNERABILITY IDENTIFICATION
  "Through manual testing and research, the 
   following vulnerability was identified:"
  
  → CVE number (if applicable)
  → Vulnerability description
  → Affected service/version
  → CVSS score
  → Reference links

SECTION C: EXPLOITATION
  "The following steps were taken to exploit
   the identified vulnerability:"
  
  → Step 1: [Exact command]
  → Step 2: [Response/output]
  → Step 3: [Next command]
  → Screenshot after each significant step
  → Must be REPRODUCIBLE

SECTION D: POST-EXPLOITATION
  "After gaining initial access, privilege 
   escalation was performed:"
  
  → User context (whoami)
  → Enumeration performed
  → PrivEsc vector found
  → Steps to escalate
  → Final proof screenshot

SECTION E: PROOF
  → Screenshot showing:
    - IP address (ifconfig/ipconfig)
    - Hostname
    - User context (whoami)
    - Contents of proof.txt
    - Proof submitted via exam panel

SECTION F: REMEDIATION
  → Specific fix for vulnerability
  → Update/patch recommendation
  → Configuration hardening
  → Best practice suggestion
```

---

## 3. Screenshot Best Practices

```
SCREENSHOT REQUIREMENTS:

MANDATORY SCREENSHOTS:
  ✓ Proof.txt with IP address visible
  ✓ whoami/id output
  ✓ ifconfig/ipconfig output
  ✓ hostname display
  ✓ All in SAME screenshot/terminal

GOOD SCREENSHOT PRACTICES:
  → Full terminal window visible
  → Clear, readable text
  → No overlapping windows
  → Timestamp visible if possible
  → Command that produced output shown
  → Annotate screenshots when helpful

SCREENSHOT WORKFLOW:
  During exam:
  1. Take screenshot after every significant step
  2. Name files descriptively:
     machine1_nmap.png
     machine1_exploit.png
     machine1_proof.png
  3. Organize in folders per machine
  4. Verify proof screenshots immediately

TOOLS FOR SCREENSHOTS:
  → Flameshot (Linux): Best for annotations
  → Greenshot (Windows): Capture and annotate
  → Built-in: PrintScreen + paste
  → Terminal: script command for full log

PRO TIP: Over-screenshot during exam.
It's better to have too many than too few.
You can always remove extras from the report.
```

---

## 4. Report Writing During the Exam

```
REAL-TIME DOCUMENTATION:

PREPARE BEFORE EXAM:
  → Report template ready (LibreOffice/Word)
  → Sections pre-formatted
  → Machine placeholders created
  → Screenshot folder structure ready
  → Note-taking tool configured (CherryTree)

DURING EXAM WORKFLOW:
  Hour 0-1:   Initial scans on all machines
              Start notes per machine
  Hour 1-18:  Attack machines
              Document in real-time
              Screenshot every step
  Hour 18-20: Verify all proofs captured
              Fill in any documentation gaps
  Hour 20-24: Continue attacking if needed

DOCUMENTATION AS YOU GO:
  For each step, record:
  1. Exact command used
  2. Full output (copy/paste)
  3. Screenshot taken
  4. What you learned
  5. Next step decision

AFTER EXAM (24 hours):
  Hour 1-4:  Convert notes to report format
  Hour 4-8:  Write detailed per-machine sections
  Hour 8-12: Add screenshots, format properly
  Hour 12-16: Proofread and verify reproducibility
  Hour 16-20: Final review and export to PDF
  Hour 20-24: Buffer time (submit early)

SUBMIT EARLY:
  → Don't wait until deadline
  → Technical issues happen
  → Upload can take time
  → Verify upload succeeded
  → Keep a backup copy
```

---

## 5. Common Mistakes and Tips

```
REPORT FAILURES:

COMMON REASONS FOR FAILING THE REPORT:
  ✗ Missing proof screenshots
  ✗ Steps not reproducible
  ✗ Wrong file format (not PDF)
  ✗ Submitted late
  ✗ Missing machine sections
  ✗ Using automated tools without documenting
  ✗ Poor formatting/unreadable
  ✗ Not including IP address in proof

WRITING TIPS:
  → Write for someone who wasn't there
  → Assume reader has technical knowledge
  → Be specific (exact commands, exact versions)
  → Include error messages and how you resolved
  → Show thought process, not just final commands
  → Use consistent formatting throughout
  → Number all screenshots
  → Reference screenshots in text

FORMATTING GUIDELINES:
  → Use consistent heading styles
  → Code blocks for commands/output
  → Tables for organized data
  → Bullet points for lists
  → Bold for important items
  → Page numbers
  → Professional tone (no slang)

REPORT TEMPLATE TOOLS:
  → OffSec provides a template (USE IT)
  → LibreOffice Writer (free)
  → Microsoft Word
  → LaTeX (if comfortable)
  → Pandoc (Markdown to PDF)

QUALITY CHECKLIST:
  [ ] Cover page with OSID
  [ ] Table of contents
  [ ] Executive summary
  [ ] Each machine fully documented
  [ ] All proof screenshots included
  [ ] Steps are reproducible
  [ ] Remediation recommendations
  [ ] Proofread for errors
  [ ] Exported as PDF
  [ ] Uploaded successfully
```

---

## Summary Table

| Section | Purpose | Priority |
|---------|---------|----------|
| Executive Summary | High-level overview | Medium |
| Enumeration | What was found | High |
| Exploitation | How access was gained | Critical |
| Proof | Evidence of compromise | Critical |
| Remediation | Fix recommendations | Medium |

---

## Revision Questions

1. What sections must be included in the OSCP report?
2. What must be visible in a proof screenshot?
3. How should you manage documentation during the 24-hour exam?
4. What common mistakes lead to report failures?
5. When should you start writing the report relative to the exam?

---

*Previous: [03-methodology-development.md](03-methodology-development.md) | Next: [05-time-management.md](05-time-management.md)*

---

*[Back to README](../README.md)*

# Public Records

## Unit 2 - Topic 4 | OSINT (Open Source Intelligence)

---

## Overview

**Public records** are government and institutional documents available to the public. They contain a wealth of information about organizations including financial data, legal filings, property records, and regulatory compliance — all valuable for building a comprehensive target profile.

---

## 1. Types of Public Records

| Record Type | Source | Intelligence Value |
|------------|--------|:---:|
| **SEC filings** | sec.gov (EDGAR) | Financial data, executives, subsidiaries |
| **Business registrations** | State secretary offices | Officers, registered agents, addresses |
| **Patent filings** | USPTO, EPO | Technology development, R&D focus |
| **Court records** | PACER, state courts | Legal disputes, security incidents |
| **Property records** | County assessor offices | Office locations, property details |
| **FCC filings** | fcc.gov | Wireless spectrum, device approvals |
| **Government contracts** | usaspending.gov, SAM.gov | Vendors, contract details |
| **Trademark records** | USPTO, WIPO | Brand names, upcoming products |
| **Tax records** | IRS (nonprofits), state | Financial structure |
| **Domain registrations** | WHOIS databases | Contact info, registration details |

---

## 2. SEC Filings (US Public Companies)

```
SEC EDGAR (sec.gov/cgi-bin/browse-edgar):

KEY FILING TYPES:
├── 10-K: Annual report
│   └── Business overview, risk factors, IT systems
├── 10-Q: Quarterly report
│   └── Recent changes, financial updates
├── 8-K: Current events
│   └── Acquisitions, breaches, leadership changes
├── DEF 14A: Proxy statement
│   └── Executive compensation, board members
└── S-1: IPO registration
    └── Detailed company overview

INTELLIGENCE FROM SEC FILINGS:
├── Subsidiary companies (additional attack surface)
├── Technology vendors and systems used
├── Risk factors (mention of cybersecurity risks)
├── Key personnel with titles
├── Physical office locations
├── Revenue and financial health
└── Recent acquisitions (may have different security)
```

---

## 3. Business Registration Records

```bash
# STATE BUSINESS REGISTRATIONS:
# Each US state has a Secretary of State database
# Search by company name to find:

EXAMPLE OUTPUT:
Entity Name:      Target Corporation
Status:           Active
Formation Date:   03/15/2005
Entity Type:      Corporation
State:            Delaware
Registered Agent: CT Corporation
Agent Address:    1209 Orange St, Wilmington, DE

Officers:
├── CEO: John Smith
├── CFO: Jane Doe
├── Secretary: Robert Johnson
└── Director: Sarah Williams

# INTERNATIONAL:
# Companies House (UK) — companieshouse.gov.uk
# ABN Lookup (Australia) — abr.business.gov.au
# Handelsregister (Germany) — handelsregister.de
```

---

## 4. Document Metadata

```bash
# METADATA EXTRACTION FROM PUBLIC DOCUMENTS:

# Download public PDFs from target website
wget https://target.com/annual-report.pdf

# Extract metadata
exiftool annual-report.pdf

# OUTPUT:
# Author:          jsmith
# Creator:         Microsoft Word 2019
# Producer:        Adobe PDF Library 15.0
# Create Date:     2024:01:15 09:30:22
# Title:           Q4 Financial Report
# Company:         Target Corporation

# INTELLIGENCE FROM METADATA:
# ├── Author usernames (potential login names)
# ├── Software versions (potential vulnerabilities)
# ├── Internal paths (reveal directory structure)
# ├── Printer names (internal infrastructure)
# └── GPS coordinates (from images)

# FOCA — automated metadata extraction
# Downloads and analyzes documents from target domain
```

---

## 5. Using Public Records Effectively

| Record | What to Look For | How It Helps |
|--------|-----------------|-------------|
| **SEC 10-K** | IT vendors, subsidiaries | Expand attack surface |
| **Business reg** | Officer names | Social engineering targets |
| **Patents** | Technology details | Understand internal systems |
| **Court records** | Security incidents | Previous weaknesses |
| **Job postings** | Required skills | Technology identification |
| **Documents** | Metadata | Usernames, software versions |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Public records** | Government/institutional documents freely available |
| **SEC filings** | Financials, subsidiaries, risk factors |
| **Business records** | Officer names, addresses |
| **Metadata** | Hidden data in documents (usernames, software) |
| **Patents** | Technology details and R&D direction |
| **Legal value** | All publicly available — no authorization needed |

---

## Quick Revision Questions

1. **What types of public records are useful for OSINT?**
2. **What intelligence can SEC filings reveal?**
3. **How does document metadata help pen testing?**
4. **What tool extracts metadata from documents?**
5. **Why are business registration records valuable?**

---

[← Previous: Social Media Intelligence](03-social-media-intelligence.md) | [Next: OSINT Frameworks →](05-osint-frameworks.md)

---

*Unit 2 - Topic 4 of 6 | [Back to README](../README.md)*

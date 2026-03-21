# Functionality Mapping

## Unit 3: Web Reconnaissance — Topic 5

## 🎯 Overview

Functionality mapping goes beyond listing URLs — it involves understanding what the application does, how different features interact, what business logic governs workflows, and where security-critical operations occur. This deep understanding enables targeted testing of the highest-risk functionality rather than surface-level scanning.

---

## 1. Functionality Categories

```
┌──────────────────────────────────────────────────────────────┐
│              APPLICATION FUNCTIONALITY MAP                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Authentication          │  Registration, Login, Logout      │
│                          │  Password Reset, MFA, OAuth       │
│  ─────────────────────── │ ────────────────────────────────  │
│  User Management         │  Profile, Settings, Preferences   │
│                          │  Account Deletion, Data Export    │
│  ─────────────────────── │ ────────────────────────────────  │
│  Content Operations      │  Create, Read, Update, Delete     │
│                          │  Search, Filter, Sort             │
│  ─────────────────────── │ ────────────────────────────────  │
│  Financial/Transactions  │  Payments, Transfers, Refunds     │
│                          │  Cart, Checkout, Subscriptions    │
│  ─────────────────────── │ ────────────────────────────────  │
│  Communication           │  Messaging, Notifications, Email  │
│                          │  Comments, Reviews, Sharing       │
│  ─────────────────────── │ ────────────────────────────────  │
│  Administration          │  User admin, Config, Reports      │
│                          │  Logs, Backups, System Settings   │
│  ─────────────────────── │ ────────────────────────────────  │
│  File Operations         │  Upload, Download, Preview        │
│                          │  Convert, Share, Delete           │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Workflow Mapping

### Authentication Flow

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌──────────┐
│  Login  │──→│ Verify  │──→│  MFA    │──→│Dashboard │
│  Page   │   │ Creds   │   │ Check   │   │          │
└────┬────┘   └────┬────┘   └────┬────┘   └──────────┘
     │             │             │
     │ Failed      │ Failed      │ Failed
     ▼             ▼             ▼
┌─────────┐   ┌─────────┐   ┌──────────┐
│ Error   │   │ Lockout │   │ MFA Retry│
│ Message │   │ (5 tries)│   │ or Reset │
└─────────┘   └─────────┘   └──────────┘

Testing points:
• Credential validation (timing, error messages)
• Lockout bypass (account enumeration)
• MFA bypass (race condition, backup codes)
• Session creation (token strength, fixation)
```

### Payment Flow

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│   Cart   │→ │ Checkout │→ │ Payment  │→ │ Confirm  │
│ Add Item │  │ Address  │  │ Process  │  │ Receipt  │
│ Quantity │  │ Shipping │  │ Card/API │  │ Email    │
└──────────┘  └──────────┘  └──────────┘  └──────────┘

Testing points:
• Can price be modified between steps?
• Can items be added after payment calculation?
• Is quantity validated (negative numbers)?
• Can shipping address be changed after payment?
• Race condition: simultaneous checkout with same cart?
```

---

## 3. Role-Based Access Mapping

```
┌──────────────────────────────────────────────────────────────┐
│              ROLE-ACCESS MATRIX                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Function          │  Guest │  User  │ Premium │  Admin     │
│  ─────────────────┼────────┼────────┼─────────┼──────────  │
│  View public pages │   ✓   │   ✓   │    ✓    │    ✓      │
│  Register/Login    │   ✓   │   ─   │    ─    │    ─      │
│  View profile      │   ✗   │   ✓   │    ✓    │    ✓      │
│  Edit own profile  │   ✗   │   ✓   │    ✓    │    ✓      │
│  View others' data │   ✗   │   ✗   │    ✗    │    ✓      │
│  Premium features  │   ✗   │   ✗   │    ✓    │    ✓      │
│  Export data       │   ✗   │   ✗   │    ✓    │    ✓      │
│  Manage users      │   ✗   │   ✗   │    ✗    │    ✓      │
│  System config     │   ✗   │   ✗   │    ✗    │    ✓      │
│  View logs         │   ✗   │   ✗   │    ✗    │    ✓      │
│                                                              │
│  Testing: Access each function with each role               │
│  Look for: Horizontal (user→user) and Vertical (user→admin) │
│  privilege escalation                                        │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Input-Output Mapping

```bash
# For each endpoint, document:
# 1. Input format (params, headers, body)
# 2. Validation rules (client and server)
# 3. Output format (HTML, JSON, redirect)
# 4. Error handling (messages, codes)

# Example documentation:
# POST /api/search
# Input: {"query": "string", "limit": int, "category": "string"}
# Validation: query min 2 chars, limit max 100
# Output: {"results": [...], "total": int}
# Errors: 400 (bad input), 429 (rate limited)
# Notes: query reflected in response (test XSS)
#        limit not validated server-side (test large values)
```

---

## 5. Security-Critical Functions

```
┌──────────────────────────────────────────────────────────────┐
│          HIGH-RISK FUNCTIONALITY TO PRIORITIZE                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  🔴 Critical:                                                │
│  • Login/Authentication          • Payment processing        │
│  • Password reset                • File upload               │
│  • Account creation              • Admin panel               │
│  • API key management            • Data export               │
│                                                              │
│  🟡 High:                                                    │
│  • Profile updates               • Search functionality      │
│  • User-to-user messaging        • Content creation          │
│  • Permission changes            • Integration settings      │
│                                                              │
│  🟢 Medium:                                                  │
│  • Preference changes            • Notification settings     │
│  • Theme/display settings        • Help/support tickets      │
│  • Static content pages          • Documentation             │
└──────────────────────────────────────────────────────────────┘
```

---

## 6. Documentation Template

```markdown
## Endpoint: POST /api/users/{id}/transfer

**Description**: Transfer funds between user accounts
**Authentication**: Bearer token required
**Authorization**: User can only transfer from own account
**Rate Limit**: 5 requests per minute

**Parameters**:
| Name | Type | Required | Validation |
|------|------|----------|-----------|
| from_account | string | Yes | Must be user's account |
| to_account | string | Yes | Must exist |
| amount | float | Yes | > 0, <= balance |
| note | string | No | Max 255 chars |

**Test Cases**:
- [ ] Transfer from another user's account (IDOR)
- [ ] Negative amount (logic flaw)
- [ ] Amount > balance (insufficient funds handling)
- [ ] Concurrent transfers (race condition)
- [ ] XSS in note field
- [ ] SQL injection in account fields
- [ ] Transfer to non-existent account
- [ ] Rate limit bypass
```

---

## 📊 Summary Table

| Mapping Area | Security Focus | Common Findings |
|-------------|---------------|-----------------|
| Auth flows | Bypass, enumeration | Weak password reset, no MFA |
| Payment flows | Price manipulation | Race conditions, negative values |
| Role access | AuthZ bypass | Horizontal/vertical privesc |
| File operations | Upload/download | Unrestricted upload, path traversal |
| API endpoints | IDOR, injection | Missing auth, mass assignment |
| Admin functions | Access control | Default creds, exposed panels |

---

## ❓ Revision Questions

1. Why is functionality mapping more important than simple URL listing?
2. How do you create a role-access matrix for testing?
3. What makes payment flows particularly vulnerable to business logic attacks?
4. How should security-critical functions be prioritized during testing?
5. What should be documented for each discovered endpoint?
6. How does workflow mapping help identify race conditions?

---

*Previous: [04-parameter-discovery.md](04-parameter-discovery.md) | Next: [06-comment-metadata-analysis.md](06-comment-metadata-analysis.md)*

---

*[Back to README](../README.md)*

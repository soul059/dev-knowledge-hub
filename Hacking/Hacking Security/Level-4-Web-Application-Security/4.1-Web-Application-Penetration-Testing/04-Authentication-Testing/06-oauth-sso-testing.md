# OAuth and SSO Testing

## Unit 4: Authentication Testing — Topic 6

## 🎯 Overview

OAuth 2.0 and Single Sign-On (SSO) allow users to authenticate via third-party providers (Google, GitHub, Microsoft). While they reduce password fatigue, their complex flows introduce unique vulnerabilities including open redirect, token theft, and authorization bypass. This topic covers OAuth/OIDC flows, common misconfigurations, and testing methodologies.

---

## 1. OAuth 2.0 Flows

### Authorization Code Flow (Most Common)

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│   User   │    │  Client  │    │Auth Server│    │ Resource │
│ (Browser)│    │  (App)   │    │ (Google)  │    │  Server  │
│          │    │          │    │          │    │  (API)   │
│ 1. Click │    │          │    │          │    │          │
│ "Login   │    │          │    │          │    │          │
│  with    │──→│ 2. Redirect to Auth Server │    │          │
│  Google" │    │──────────────→│          │    │          │
│          │    │          │    │ 3. Login │    │          │
│          │←──────────────────│    Page   │    │          │
│ 4. Login │──────────────────→│          │    │          │
│ & Consent│    │          │    │ 5. Auth  │    │          │
│          │    │←─── 6. Redirect + Code ──│    │          │
│          │    │          │    │          │    │          │
│          │    │── 7. Exchange code ──────→│    │          │
│          │    │←── 8. Access Token ──────│    │          │
│          │    │          │    │          │    │          │
│          │    │── 9. API request + Token ────→│          │
│          │    │←── 10. Protected data ───────│          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

### Implicit Flow (Legacy, Insecure)

```
# Token returned directly in URL fragment
# https://client.com/callback#access_token=abc123&token_type=bearer

# ⚠ Token visible in browser history, Referer headers
# ⚠ No code exchange → easier to intercept
# ⚠ Deprecated by OAuth 2.1
```

---

## 2. OAuth Attack Vectors

### Open Redirect via redirect_uri

```bash
# Normal authorization request:
https://auth.google.com/authorize?
  client_id=APP_ID&
  redirect_uri=https://app.target.com/callback&
  response_type=code&
  scope=openid profile email

# Attack: Modify redirect_uri to attacker's site
https://auth.google.com/authorize?
  client_id=APP_ID&
  redirect_uri=https://evil.com/steal&
  response_type=code

# Bypass techniques for redirect_uri validation:
redirect_uri=https://evil.com@target.com/callback
redirect_uri=https://target.com.evil.com/callback
redirect_uri=https://target.com/callback?next=https://evil.com
redirect_uri=https://target.com/callback/../../../evil
redirect_uri=https://target.com/callback%0d%0aLocation:%20evil.com
```

### CSRF in OAuth Flow

```
┌──────────────────────────────────────────────────────────────┐
│              OAUTH CSRF ATTACK                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Attacker initiates OAuth flow on target app              │
│  2. Gets authorization code for ATTACKER's account           │
│  3. Doesn't use the code — sends callback URL to victim     │
│                                                              │
│  Victim clicks:                                              │
│  https://target.com/callback?code=ATTACKER_CODE              │
│                                                              │
│  4. Target app exchanges code → gets ATTACKER's profile     │
│  5. Links attacker's OAuth to VICTIM's account!             │
│                                                              │
│  Prevention: state parameter (CSRF token in OAuth)           │
│  Test: Remove state parameter → does flow still work?        │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Test for missing state parameter
# 1. Start OAuth flow, capture the authorization URL
# 2. Remove the state parameter
# 3. Complete the flow — if it works, CSRF is possible

# Check state parameter:
# • Is it present in authorization request?
# • Is it validated on callback?
# • Is it bound to user's session?
# • Is it unpredictable/random?
```

### Token Theft via Referer

```
┌──────────────────────────────────────────────────────────────┐
│  Implicit Flow token leaks via Referer header:               │
│                                                              │
│  1. User redirected to:                                      │
│     https://app.com/callback#access_token=SECRET_TOKEN       │
│                                                              │
│  2. Page loads external resource (image, script):            │
│     <img src="https://analytics.com/pixel.gif">              │
│                                                              │
│  3. Referer header sent to analytics.com:                    │
│     Referer: https://app.com/callback#access_token=SECRET    │
│                                                              │
│  ⚠ Token leaked to third party!                             │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. SSO Testing

### SAML-Based SSO

```
┌──────────────────────────────────────────────────────────────┐
│                    SAML FLOW                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. User → Service Provider (SP)                             │
│  2. SP redirects to Identity Provider (IdP)                  │
│  3. User authenticates at IdP                                │
│  4. IdP creates SAML Assertion (signed XML)                  │
│  5. User redirected back to SP with assertion                │
│  6. SP validates assertion → grants access                   │
│                                                              │
│  SAML Attack Vectors:                                        │
│  • XML Signature Wrapping (XSW)                              │
│  • Assertion manipulation                                    │
│  • Replay attacks                                            │
│  • Missing signature validation                              │
│  • Comment injection in NameID                               │
└──────────────────────────────────────────────────────────────┘
```

### SAML Attacks

```bash
# SAML assertion replay
# Capture valid SAML response, replay later
# If no timestamp/nonce validation → works!

# SAML comment injection
# Original: <NameID>user@target.com</NameID>
# Modified: <NameID>admin@target.com<!--user@target.com--></NameID>
# If parser reads "admin@target.com" → auth as admin

# SAML signature bypass
# Remove signature and see if SP still accepts assertion
# Some SPs don't validate signatures!

# Tools:
# SAMLRaider (Burp extension)
# saml2aws
```

---

## 4. OAuth Testing Checklist

```
┌──────────────────────────────────────────────────────────────┐
│              OAUTH TESTING CHECKLIST                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Authorization Endpoint:                                     │
│  □ Is redirect_uri strictly validated?                       │
│  □ Is state parameter present and validated?                 │
│  □ Can scope be escalated?                                   │
│  □ Is PKCE used for public clients?                          │
│                                                              │
│  Token Endpoint:                                             │
│  □ Is authorization code single-use?                         │
│  □ Is client authentication required?                        │
│  □ Are tokens properly scoped?                               │
│  □ Is token lifetime appropriate?                            │
│                                                              │
│  Callback Endpoint:                                          │
│  □ Is state validated before token exchange?                 │
│  □ Is error handling secure?                                 │
│  □ Are tokens stored securely?                               │
│                                                              │
│  General:                                                    │
│  □ Can account be linked to attacker's OAuth?                │
│  □ Is token revocation implemented?                          │
│  □ Are refresh tokens rotated?                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Account Takeover via OAuth

```bash
# Scenario 1: Email-based account linking
# App creates account with OAuth email
# If attacker registers with same email first → conflict
# Some apps merge accounts → attacker gains access

# Scenario 2: Missing email verification
# OAuth provider doesn't verify email
# Attacker creates OAuth account with victim's email
# App trusts OAuth email → creates session as victim

# Scenario 3: OAuth provider switching
# User has account via Google OAuth (user@gmail.com)
# Attacker creates GitHub account with user@gmail.com
# Links GitHub OAuth → takes over account
# Test: Can different OAuth providers link to same account?
```

---

## 📊 Summary Table

| Attack | OAuth Component | Prevention |
|--------|----------------|------------|
| Open Redirect | redirect_uri | Strict whitelist validation |
| CSRF | Missing state | Random state parameter |
| Token Theft | Implicit flow | Use Authorization Code + PKCE |
| Scope Escalation | scope parameter | Server-side scope enforcement |
| Account Linking | Email trust | Verify email ownership |
| Code Replay | Authorization code | Single-use codes |

---

## ❓ Revision Questions

1. How does the OAuth Authorization Code flow differ from Implicit flow?
2. What is the purpose of the state parameter in OAuth?
3. How can redirect_uri validation be bypassed?
4. What are SAML signature wrapping attacks?
5. How can OAuth flows lead to account takeover?
6. Why is PKCE important for public OAuth clients?

---

*Previous: [05-session-management.md](05-session-management.md)*

---

*[Back to README](../README.md)*

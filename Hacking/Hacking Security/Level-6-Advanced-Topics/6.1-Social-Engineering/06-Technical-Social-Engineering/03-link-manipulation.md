# Unit 6: Technical Social Engineering — Topic 3: Link Manipulation

## Overview

**Link manipulation** is the technique of disguising malicious URLs to appear legitimate, tricking users into clicking links that lead to phishing pages, malware downloads, or exploit kits. It combines technical URL tricks with social engineering context to bypass both human judgment and automated security filters.

---

## 1. URL Anatomy and Manipulation

```
URL STRUCTURE:

  https://username:password@subdomain.domain.tld:port/path?query=value#fragment
  ──────  ───────────────── ──────────────────────── ──── ──── ─────────── ────────
  scheme   userinfo          host                    port path  query      fragment

MANIPULATION TARGETS:
  → Domain: The most important trust indicator
  → Subdomain: Often overlooked by users
  → Path: Can mimic legitimate page structure
  → Userinfo: Abused to show fake domain
  → Fragment: Can contain executable code
```

---

## 2. Link Manipulation Techniques

```
TECHNIQUE 1: LOOKALIKE DOMAINS
  Legitimate:     microsoft.com
  Lookalike:      micr0soft.com (zero for 'o')
                  microsft.com  (missing letter)
                  microsoft-login.com (added word)
                  rnicrosoft.com (rn → m)
                  microsofft.com (double letter)

TECHNIQUE 2: SUBDOMAIN ABUSE
  Legitimate:     login.microsoft.com
  Malicious:      microsoft.com.evil-site.com
                  login.microsoft.com.attacker.com
                  microsoft-security.attacker.com
  → Users see "microsoft.com" and stop reading
  → Actual domain is at the end

TECHNIQUE 3: HOMOGRAPH ATTACKS (IDN)
  Legitimate:     apple.com
  Malicious:      аpple.com (Cyrillic 'а')
  Punycode:       xn--pple-43d.com
  → Characters from other alphabets look identical
  → a (Latin) vs а (Cyrillic)
  → o (Latin) vs о (Cyrillic)
  → e (Latin) vs е (Cyrillic)

TECHNIQUE 4: URL SHORTENERS
  Legitimate URL:  https://malicious-site.com/phishing
  Shortened:       https://bit.ly/3xY2mK
                   https://tinyurl.com/abc123
                   https://t.co/XyZ123
  → Hides true destination
  → Common in social media, SMS
  → Users can't preview destination

TECHNIQUE 5: DATA URIs
  data:text/html;base64,PHNjcmlwdD5...
  → Embeds entire page in the URL
  → No server needed
  → Can create phishing pages inline
  → Some browsers/apps execute directly

TECHNIQUE 6: OPEN REDIRECTS
  Legitimate:
    https://trusted-site.com/redirect?url=https://evil.com
  → Uses trusted site's redirect function
  → URL starts with trusted domain
  → Redirects to attacker's site
  → Bypasses URL filters

TECHNIQUE 7: @ SYMBOL ABUSE
  https://www.microsoft.com@attacker.com/login
  → Browser ignores everything before @
  → Actually navigates to attacker.com
  → User sees "microsoft.com" in link text
  → Most modern browsers now show warnings

TECHNIQUE 8: URL ENCODING
  https://attacker.com/%70%68%69%73%68
  → Hex encoding of path/parameters
  → Obfuscates true path
  → Bypasses simple URL filters
  → Decoded by browser automatically
```

---

## 3. Display vs Destination Tricks

```
HTML EMAIL MANIPULATION:

HYPERLINK MISMATCH:
  Display: https://www.paypal.com/account
  Actual:  https://paypa1.com/steal-creds
  
  HTML: <a href="https://paypa1.com/steal-creds">
        https://www.paypal.com/account</a>

  → User sees legitimate URL text
  → Actual link goes elsewhere
  → Hover shows real URL (desktop only)
  → Mobile: long-press to preview

BUTTON LINKS:
  [Verify Your Account]
  → Button hides URL completely
  → No visible URL to inspect
  → Common in legitimate emails too
  → Users trained to click buttons

IMAGE LINKS:
  → Screenshot of legitimate email with fake URL
  → Entire email is an image linking to phishing
  → Text-based filters can't read image content
  → Looks identical to legitimate email

CLICKJACKING:
  → Invisible iframe over legitimate page
  → User thinks they click trusted button
  → Actually clicking attacker's element
  → Can capture credentials, authorize actions
```

---

## 4. Social Engineering Context

```
MAKING LINKS CLICKABLE:

URGENCY:
  "Your account will be suspended in 24 hours.
   Click here to verify: [link]"

CURIOSITY:
  "Check out this photo of you: [link]"
  "Your name appears in this article: [link]"

AUTHORITY:
  "Required: Complete your compliance training: [link]"
  "IT Department: Update your password here: [link]"

GREED:
  "You've won a $500 gift card! Claim: [link]"
  "Exclusive discount — 80% off: [link]"

FEAR:
  "Unauthorized login detected. Secure your 
   account: [link]"
  "Your payment failed. Update billing: [link]"

SOCIAL PROOF:
  "John and 5 others shared this article: [link]"
  "Trending: See what everyone's talking about: [link]"
```

---

## 5. Defending Against Link Manipulation

```
TECHNICAL CONTROLS:
  → URL rewriting (Safe Links / URL Defense)
  → Real-time URL scanning
  → DNS filtering (block known malicious domains)
  → Web proxy with URL categorization
  → Browser phishing protection (Safe Browsing)
  → Homograph detection and blocking
  → URL shortener expansion before delivery

USER TRAINING:
  → Hover over links before clicking
  → Check the actual domain name
  → Read URLs right-to-left (domain first)
  → Be suspicious of shortened URLs
  → Don't click links in unexpected emails
  → Type URLs directly into browser
  → Use bookmarks for important sites

ORGANIZATIONAL:
  → Link click tracking and analysis
  → Phishing simulation with link testing
  → External email warnings
  → URL inspection in email gateway
  → Incident response for clicked phishing links
```

---

## Summary Table

| Technique | Example | Detection Difficulty |
|-----------|---------|---------------------|
| Lookalike domain | micr0soft.com | Medium |
| Subdomain abuse | microsoft.com.evil.com | Easy (trained users) |
| Homograph | аpple.com (Cyrillic) | Very Hard |
| URL shortener | bit.ly/abc123 | Hard (hidden) |
| Open redirect | trusted.com/redirect?url= | Hard |
| @ symbol abuse | microsoft.com@evil.com | Medium |
| Display mismatch | Link text ≠ href | Easy (hover) |

---

## Revision Questions

1. What are the most common link manipulation techniques?
2. How do homograph attacks exploit Unicode characters?
3. Why are open redirects dangerous in the context of phishing?
4. How does the display vs destination mismatch work in HTML emails?
5. What technical controls best protect against malicious links?

---

*Previous: [02-malicious-attachments.md](02-malicious-attachments.md) | Next: [04-qr-code-attacks.md](04-qr-code-attacks.md)*

---

*[Back to README](../README.md)*

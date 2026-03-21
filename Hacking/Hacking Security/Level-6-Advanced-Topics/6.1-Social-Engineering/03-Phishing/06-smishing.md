# Unit 3: Phishing — Topic 6: Smishing (SMS Phishing)

## Overview

**Smishing** (SMS Phishing) uses text messages to trick recipients into clicking malicious links, calling attacker-controlled numbers, or providing sensitive information. With SMS open rates of 98% (compared to ~20% for email), smishing is an increasingly effective attack vector, especially as mobile devices become primary computing platforms.

---

## 1. Why Smishing Is Effective

```
SMISHING ADVANTAGES:

HIGH ENGAGEMENT:
  → 98% SMS open rate (vs 20% email)
  → 90% read within 3 minutes
  → 45% response rate (vs 6% email)
  → Users trust SMS more than email

LIMITED SECURITY:
  → No spam filters like email
  → Can't hover over links to preview
  → Small screen hides full URLs
  → No sender authentication (SPF/DKIM)
  → Limited security awareness training covers SMS

MOBILE CONTEXT:
  → Users are often distracted (on the go)
  → Quick decisions on mobile
  → Auto-fill may pre-populate forms
  → Mobile browsers hide URL bars
  → Apps can be installed from links

TECHNICAL EASE:
  → Bulk SMS services are cheap
  → Sender ID spoofing is simple
  → Short URLs look normal in SMS
  → No need for email infrastructure
```

---

## 2. Common Smishing Messages

```
SMISHING TEMPLATES:

BANKING:
  "[BankName] ALERT: Unusual activity detected 
   on your account. Verify immediately: 
   https://bit.ly/bnk-verify"

DELIVERY:
  "USPS: Your package cannot be delivered. 
   Update delivery preferences: 
   https://usps-tracking.info/update"

ACCOUNT VERIFICATION:
  "Your [Service] account will be suspended. 
   Verify your identity now: 
   https://verify-account.co/login"

TAX/GOVERNMENT:
  "IRS: You are eligible for a tax refund of 
   $1,200. Claim here: https://irs-refund.co"

TWO-FACTOR CODE THEFT:
  "Your verification code is 847291. If you 
   did not request this, call 1-800-XXX-XXXX 
   immediately to secure your account."
  → Attacker then calls and asks for the code

PRIZE/REWARD:
  "Congratulations! You've won a $500 gift card 
   from [Store]. Claim before midnight: 
   https://claim-reward.info"

COVID/HEALTH:
  "Health Dept: You were in close contact with 
   a COVID case. Get testing info: 
   https://health-alert.info/test"

JOB OFFER:
  "Amazon is hiring! $35/hr work from home. 
   Apply now: https://amzn-careers.info/apply"
```

---

## 3. Smishing Technical Methods

```
SMS SPOOFING TECHNIQUES:

SENDER ID SPOOFING:
  → Set alphanumeric sender ID (e.g., "BankName")
  → Appears as coming from the business
  → Not available in all countries/carriers
  → Example: sender shows "Chase" instead of number

BULK SMS SERVICES:
  → Twilio, Vonage, Plivo (legitimate)
  → Underground SMS gateways
  → SIM farms for mass sending
  → Email-to-SMS gateways

SHORT URL MASKING:
  → bit.ly, tinyurl, is.gd
  → Custom short domains
  → URL shorteners hide actual destination
  → Preview features often unused on mobile

SIM SWAPPING (Advanced):
  → Social engineer carrier to port number
  → Receive victim's SMS/calls
  → Intercept 2FA codes
  → Access banking/email accounts

ATTACK INFRASTRUCTURE:
  Phone Number Sources:
    → Data breaches
    → Public records
    → Social media
    → War dialing
    → Purchased lists

  Landing Pages:
    → Mobile-optimized phishing pages
    → Clone of legitimate app login
    → Auto-detect mobile browser
    → App installation prompts
```

---

## 4. Smishing Campaign Analysis

```
REAL-WORLD SMISHING CAMPAIGNS:

FluBot CAMPAIGN (2021):
  → "You have a missed package delivery"
  → Link installs Android malware
  → Malware sends SMS from victim's phone
  → Self-propagating via contact list
  → Spread across 11 countries

ROAMING MANTIS (Ongoing):
  → DNS hijacking + smishing combo
  → Targets Android and iOS users
  → Android: installs malware APK
  → iOS: redirects to phishing page
  → Adapts to victim's OS automatically

TRICKBOT SMISHING:
  → "Your TrustedBank account locked"
  → Links to credential harvesting page
  → Captured creds used for TrickBot deployment
  → Led to ransomware infections

COMMON PATTERNS:
  → Short, urgent messages
  → Single URL link
  → Impersonate trusted brands
  → Time-sensitive language
  → No personalization usually
```

---

## 5. Defending Against Smishing

```
DEFENSE STRATEGIES:

INDIVIDUAL:
  → Never click links in unexpected texts
  → Go directly to official app/website
  → Don't respond to unknown numbers
  → Report spam texts (forward to 7726)
  → Enable carrier spam filtering
  → Be wary of texts creating urgency

ORGANIZATIONAL:
  → Include smishing in security awareness
  → SMS-based 2FA → authenticator app migration
  → Mobile device management (MDM)
  → Report suspicious texts to IT
  → Conduct smishing simulation exercises

TECHNICAL:
  → Carrier-level spam filtering
  → RCS Business Messaging verification
  → Mobile threat defense solutions
  → URL filtering on mobile devices
  → App installation restrictions
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Definition | Phishing via SMS/text messages |
| Open rate | 98% (vs 20% for email) |
| Read time | 90% within 3 minutes |
| Common pretexts | Banking, delivery, account, tax |
| Key advantage | High engagement, low suspicion |
| Primary defense | Never click links — use official apps |

---

## Revision Questions

1. Why is smishing so effective compared to email phishing?
2. What are the most common smishing message templates?
3. How does SMS sender ID spoofing work?
4. What real-world smishing campaigns have been significant?
5. What is the most effective individual defense against smishing?

---

*Previous: [05-vishing.md](05-vishing.md) | Next: [07-phishing-infrastructure.md](07-phishing-infrastructure.md)*

---

*[Back to README](../README.md)*

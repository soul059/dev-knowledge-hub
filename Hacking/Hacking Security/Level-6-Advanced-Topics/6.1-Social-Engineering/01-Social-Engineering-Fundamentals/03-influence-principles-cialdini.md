# Unit 1: Social Engineering Fundamentals — Topic 3: Influence Principles (Cialdini)

## Overview

Dr. Robert Cialdini's **six principles of influence** (later expanded to seven) form the theoretical framework most cited in social engineering. These principles explain why people comply with requests and how attackers weaponize them. Understanding Cialdini's model is essential for both executing and defending against social engineering attacks.

---

## 1. The Seven Principles of Influence

```
CIALDINI'S PRINCIPLES OF INFLUENCE:

┌─────────────────────────────────────────────┐
│                                              │
│  1. RECIPROCITY     ← "I owe you"           │
│  2. COMMITMENT      ← "I already said yes"  │
│  3. SOCIAL PROOF    ← "Everyone does it"    │
│  4. AUTHORITY        ← "The boss said so"    │
│  5. LIKING           ← "I like you"          │
│  6. SCARCITY         ← "Running out!"        │
│  7. UNITY            ← "We're the same"      │
│                                              │
└─────────────────────────────────────────────┘
```

---

## 2. Each Principle in Detail

```
1. RECIPROCITY — "I did something for you, now you owe me"

  Principle: People feel obligated to return favors
  
  LEGITIMATE USE:
    → Free samples at stores → feel obligated to buy
    → Birthday gift → feel obligated to reciprocate
  
  SOCIAL ENGINEERING USE:
    → "I helped you with your computer issue last week"
    → "I bought you coffee, can you let me into the server room?"
    → "I'm sharing this exclusive report with you"
    → Attacker provides "help" then asks for sensitive info
  
  DEFENSE: Recognize uninvited favors as potential manipulation

---

2. COMMITMENT & CONSISTENCY — "I already agreed, so I'll keep going"

  Principle: Once committed, people act consistently
  
  LEGITIMATE USE:
    → Signing a pledge → follow through
    → Public commitments increase compliance
  
  SOCIAL ENGINEERING USE:
    → Start with small harmless questions
    → Gradually escalate to sensitive requests
    → "You agreed security is important, right?"
    → "Then you'll want to verify your credentials here"
    → Written/verbal agreement used as leverage
  
  DEFENSE: Each request should be evaluated independently

---

3. SOCIAL PROOF — "Everyone else is doing it"

  Principle: People follow what others do, especially when uncertain
  
  LEGITIMATE USE:
    → Restaurant reviews influence choices
    → "Best-selling" labels increase purchases
  
  SOCIAL ENGINEERING USE:
    → "Everyone in your department has already updated"
    → "70% of employees completed the verification"
    → "Your colleague John already gave me this information"
    → Fake testimonials or endorsements
    → Mass phishing that implies many have responded
  
  DEFENSE: Verify claims independently, don't assume others checked

---

4. AUTHORITY — "The person in charge said to do it"

  Principle: People obey authority figures
  
  LEGITIMATE USE:
    → Doctor's advice → follow medical instructions
    → Police officer's directions → comply
  
  SOCIAL ENGINEERING USE:
    → Impersonate CEO: "This is urgent, wire the money now"
    → Impersonate IT: "I need your password for maintenance"
    → Impersonate law enforcement: "We're investigating..."
    → Use titles, uniforms, jargon to establish authority
    → Name-drop executives: "Sarah from the board asked me..."
  
  DEFENSE: Verify identity through official channels

---

5. LIKING — "I trust people I like"

  Principle: People say yes to those they like
  
  Factors that increase liking:
    → Physical attractiveness
    → Similarity ("We went to the same university!")
    → Compliments ("You're so knowledgeable about this")
    → Familiarity (multiple interactions build trust)
    → Association (connected to liked things/people)
  
  SOCIAL ENGINEERING USE:
    → Build rapport before making request
    → Find common ground (LinkedIn research)
    → Be friendly, complimentary, charming
    → Mirroring body language/speech patterns
    → "I'm a [same school] grad too!"
  
  DEFENSE: Separate liking from decision-making

---

6. SCARCITY — "It's running out, act now!"

  Principle: Things seem more valuable when scarce
  
  LEGITIMATE USE:
    → "Limited edition" products
    → "Only 3 seats remaining"
  
  SOCIAL ENGINEERING USE:
    → "This security update expires in 1 hour"
    → "Only the first 50 people get the bonus"
    → "This access will be revoked at midnight"
    → Time-limited offers create FOMO
    → Artificial deadlines prevent verification
  
  DEFENSE: Artificial urgency = red flag. Take time to verify.

---

7. UNITY — "We're part of the same group"

  Principle: People favor those in their "in-group"
  
  LEGITIMATE USE:
    → Team loyalty, family bonds
    → Professional association membership
  
  SOCIAL ENGINEERING USE:
    → "As a fellow employee of [company]..."
    → "We're both alumni of [university]"
    → "As parents, you understand..."
    → Creating shared identity with target
    → Tribal affiliation exploited
  
  DEFENSE: Group membership doesn't verify legitimacy
```

---

## 3. Principles Combined in Attacks

```
COMBINED ATTACK EXAMPLE:

BUSINESS EMAIL COMPROMISE:

Step 1 (AUTHORITY):
  "Hi, this is Mark from the CFO's office"

Step 2 (URGENCY/SCARCITY):
  "We need an urgent wire transfer before 3 PM today"

Step 3 (SOCIAL PROOF):
  "Finance already approved it on their end"

Step 4 (COMMITMENT):
  "You'll handle this, right?" → "Yes"
  "Great, here are the wire details..."

Step 5 (LIKING):
  "Thanks so much, I really appreciate your help with this.
   I'll make sure the CFO knows you stepped up."

Result: $200,000 wired to attacker's account
```

---

## Summary Table

| Principle | Trigger Phrase | Red Flag |
|-----------|---------------|----------|
| Reciprocity | "I did this for you..." | Unsolicited favors |
| Commitment | "You agreed earlier..." | Escalating requests |
| Social Proof | "Everyone else did it..." | Unverified claims |
| Authority | "The CEO needs this..." | Unverified identity |
| Liking | "We have so much in common..." | Excessive flattery |
| Scarcity | "This expires in 1 hour..." | Artificial urgency |
| Unity | "As fellow employees..." | Assumed membership |

---

## Revision Questions

1. List and explain Cialdini's seven principles of influence.
2. How is the authority principle used in Business Email Compromise?
3. Give an example of how reciprocity is weaponized in social engineering.
4. Why does scarcity/urgency make people more vulnerable?
5. How can an attacker combine multiple principles in a single attack?

---

*Previous: [02-psychology-of-manipulation.md](02-psychology-of-manipulation.md) | Next: [04-attack-lifecycle.md](04-attack-lifecycle.md)*

---

*[Back to README](../README.md)*

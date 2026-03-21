# Unit 10: Other Web Vulnerabilities - Topic 5: Business Logic Flaws

## Overview

Business logic flaws are vulnerabilities that arise from flawed assumptions or inadequate validation of the application's business rules and workflows. Unlike technical vulnerabilities (SQLi, XSS), business logic flaws exploit the **intended functionality** of an application in unintended ways. These vulnerabilities cannot be detected by automated scanners because they require understanding the application's purpose, context, and business rules. They are often the most impactful and hardest-to-find vulnerabilities in web applications.

---

## 1. Understanding Business Logic Vulnerabilities

### Business Logic vs Technical Vulnerabilities

```
Technical Vulnerability:                    Business Logic Flaw:
┌──────────────────────────┐               ┌──────────────────────────┐
│ Input: ' OR 1=1--        │               │ Input: quantity=-5       │
│ Expected: Username       │               │ Expected: Positive int   │
│ Result: SQL Injection    │               │ Result: Negative charge  │
│                          │               │   (money credited!)      │
│ The CODE is broken       │               │ The LOGIC is broken      │
│ Scanner CAN detect       │               │ Scanner CANNOT detect    │
│ Standard fix available   │               │ Custom fix needed        │
└──────────────────────────┘               └──────────────────────────┘
```

### Categories of Business Logic Flaws

| Category | Description | Example |
|----------|-------------|---------|
| **Workflow Bypass** | Skipping steps in a process | Skip payment in checkout |
| **Price Manipulation** | Altering prices or calculations | Negative quantity for refund |
| **Privilege Abuse** | Using features beyond intended scope | Unlimited free trial accounts |
| **Assumption Exploitation** | Breaking implicit assumptions | Race conditions on balance |
| **Feature Misuse** | Using legitimate features maliciously | Password reset to takeover |
| **Validation Gap** | Missing server-side business rules | Client-only price validation |
| **State Manipulation** | Altering application state | Modifying cart after discount |

---

## 2. Price and Payment Manipulation

### Negative Quantity/Amount Attacks

```http
# Normal purchase
POST /api/checkout
{
    "item_id": 1001,
    "quantity": 2,
    "price": 49.99
}
# Total: $99.98

# Attack: Negative quantity
POST /api/checkout
{
    "item_id": 1001,
    "quantity": -2,
    "price": 49.99
}
# Total: -$99.98 (CREDIT to attacker!)

# Attack: Zero price
POST /api/checkout
{
    "item_id": 1001,
    "quantity": 2,
    "price": 0.00
}
# Total: $0.00

# Attack: Fractional quantity
POST /api/checkout
{
    "item_id": 1001,
    "quantity": 0.001,
    "price": 49.99
}
# Total: $0.05 (for a $49.99 item)
```

### Currency and Rounding Exploitation

```
Attack: Exploit rounding errors in currency conversion

Step 1: Account has $1.00
Step 2: Convert $1.00 to EUR at rate 0.85 = €0.85
Step 3: Convert €0.85 to GBP at rate 0.87 = £0.7395 → rounded to £0.74
Step 4: Convert £0.74 to USD at rate 1.38 = $1.0212 → rounded to $1.02
Step 5: Profit: $0.02 per cycle
Step 6: Automate 10,000 cycles = $200 profit

This exploits rounding in each conversion step.
```

### Discount and Coupon Abuse

```http
# Apply same coupon multiple times
POST /api/apply-coupon
{"code": "SAVE20"}    # 20% off
# Apply again
POST /api/apply-coupon
{"code": "SAVE20"}    # Another 20% off (now 40% off)
# Apply again...

# Apply multiple different coupons
POST /api/apply-coupon
{"code": "SAVE20"}    # 20% off
POST /api/apply-coupon
{"code": "WELCOME10"} # 10% off (stacking = 30%)

# Coupon on already-discounted items
# Item: $100, Sale price: $50
# Apply 50% coupon → $25 (50% off the sale price)
# But intended: coupon shouldn't stack with sale

# Negative coupon values
POST /api/apply-coupon
{"code": "CUSTOM", "discount": -50}
# Adds $50 to the order? Or creates negative discount?
```

### Gift Card / Store Credit Manipulation

```
Attack Flow: Gift card balance exploitation

1. Buy $100 gift card
2. Split into two $50 orders
3. Use gift card for Order A ($50)
4. Simultaneously use same gift card for Order B ($50)
   (Race condition - both check balance before deducting)
5. Result: Spent $100 gift card for $100 worth of goods
   but only paid $100 for the gift card
   
# More advanced:
1. Buy $100 gift card with gift card (circular)
2. Return items bought with gift card (get cash refund?)
3. Transfer gift card balance between accounts
4. Apply gift card to subscription (recurring charge on finite balance?)
```

---

## 3. Workflow and Process Bypass

### Multi-Step Process Manipulation

```
Intended Checkout Flow:
┌────────┐   ┌──────────┐   ┌─────────┐   ┌──────────┐   ┌──────────┐
│ Step 1 │──►│ Step 2   │──►│ Step 3  │──►│ Step 4   │──►│ Step 5   │
│ Cart   │   │ Shipping │   │ Payment │   │ Confirm  │   │ Complete │
└────────┘   └──────────┘   └─────────┘   └──────────┘   └──────────┘

Attack: Skip payment step
┌────────┐   ┌──────────┐                  ┌──────────┐   ┌──────────┐
│ Step 1 │──►│ Step 2   │─────────────────►│ Step 4   │──►│ Step 5   │
│ Cart   │   │ Shipping │   SKIP STEP 3!   │ Confirm  │   │ Complete │
└────────┘   └──────────┘                  └──────────┘   └──────────┘
```

**Testing Approach:**
```http
# Map the entire workflow first
GET /checkout/cart           # Step 1
POST /checkout/shipping      # Step 2  
POST /checkout/payment       # Step 3
POST /checkout/confirm       # Step 4
GET /checkout/complete        # Step 5

# Try skipping steps:
# Go directly from Step 1 to Step 4
POST /checkout/confirm
{"order_id": "12345"}
# Does it require payment to have been processed?

# Try going backwards:
# After payment, go back to cart, add items, then confirm
POST /checkout/cart/add
{"item_id": 2002, "quantity": 1}
POST /checkout/confirm
# New item added after payment!

# Try repeating steps:
# Submit payment confirmation multiple times
POST /checkout/confirm
POST /checkout/confirm
POST /checkout/confirm
# Multiple orders for single payment?
```

### Registration and Verification Bypass

```http
# Email verification bypass
# Normal flow: Register → Verify Email → Access Account

# Attack 1: Access features before verification
POST /api/register
{"email": "user@example.com", "password": "pass123"}
# Skip email verification
GET /api/dashboard  # Does it work without verified email?

# Attack 2: Change email after verification
POST /api/register
{"email": "legitimate@example.com"}
# Verify legitimate email
# Then change to unverified email
PUT /api/profile
{"email": "attacker@evil.com"}  # Maintains verified status?

# Attack 3: Registration token reuse
GET /verify?token=abc123  # Verify account
# Try using same token for different account
GET /verify?token=abc123&email=other@example.com
```

---

## 4. Authentication Logic Flaws

### Password Reset Flaws

```http
# Flaw 1: Predictable reset tokens
GET /reset?token=1001    # User ID as token!
GET /reset?token=1002    # Try other users

# Flaw 2: Token in response
POST /forgot-password
{"email": "victim@example.com"}
# Response includes:
{"status": "sent", "token": "abc123def456"}  # Token leaked!

# Flaw 3: No rate limiting on reset
# Brute force 4-digit PIN
for pin in range(0000, 9999):
    POST /reset-verify {"email": "victim@example.com", "pin": pin}

# Flaw 4: Host header manipulation
POST /forgot-password HTTP/1.1
Host: evil.com
{"email": "victim@example.com"}
# Reset link sent to victim: http://evil.com/reset?token=xxx
# Victim clicks → token sent to attacker's server

# Flaw 5: Reset token doesn't expire
# Tokens remain valid indefinitely
# Old intercepted tokens can be reused

# Flaw 6: Password reset poisoning via multiple parameters
POST /forgot-password HTTP/1.1
Host: target.com
Host: evil.com                    # Second Host header
{"email": "victim@example.com"}
```

### Account Lockout Bypass

```http
# Flaw: Lockout based on username only
POST /login {"username": "admin", "password": "guess1"}  # Attempt 1
POST /login {"username": "admin", "password": "guess2"}  # Attempt 2
POST /login {"username": "admin", "password": "guess3"}  # Attempt 3
# LOCKED! But...
POST /login {"username": "Admin", "password": "guess4"}  # Different case!
POST /login {"username": "admin ", "password": "guess5"} # Trailing space!
POST /login {"username": " admin", "password": "guess6"} # Leading space!

# Flaw: Lockout based on IP
# Use X-Forwarded-For rotation
POST /login
X-Forwarded-For: 1.1.1.1
{"username": "admin", "password": "guess1"}

POST /login
X-Forwarded-For: 1.1.1.2
{"username": "admin", "password": "guess2"}

# Flaw: Lockout reset on successful login
# Attacker tries 4 passwords, then logs in with known credentials
# Counter resets, try 4 more...
```

### Two-Factor Authentication (2FA) Bypass

```http
# Flaw 1: Skip 2FA step entirely
POST /login {"username": "admin", "password": "password"}
# Instead of going to /2fa, go directly to /dashboard
GET /dashboard  # 2FA bypassed?

# Flaw 2: 2FA code in response
POST /2fa/send
# Response: {"code": "123456"}  # Code leaked!

# Flaw 3: Reusable 2FA codes
POST /2fa/verify {"code": "123456"}  # Valid
POST /2fa/verify {"code": "123456"}  # Still valid! (should be one-time)

# Flaw 4: No rate limiting on 2FA
# Brute force 6-digit code (1,000,000 combinations)
# With 100 req/s = 10,000 seconds ≈ 2.7 hours max

# Flaw 5: 2FA only checked on login, not on sensitive actions
# Login with 2FA → change password without 2FA
```

---

## 5. Feature Misuse and Abuse

### Referral System Abuse

```
Intended: Invite friends → both get $10 credit

Attack:
1. Create account A (attacker)
2. Generate referral link
3. Create fake accounts B, C, D, E... using temp emails
4. Sign up each with referral link
5. Transfer credits or purchase with credits
6. Repeat with new accounts

Automation:
- Use temp email services (guerrillamail, 10minutemail)
- Script account creation
- Use different IPs/proxies
```

### File/Data Export Abuse

```http
# Export feature designed for user's own data
GET /api/export?user_id=1001
# Returns: CSV with user 1001's data

# Attack: Export other users' data (IDOR + business logic)
GET /api/export?user_id=1002
GET /api/export?user_id=1003

# Attack: Export too much data (no pagination limit)
GET /api/export?limit=999999
# Server tries to export millions of records → DoS

# Attack: Export formats with formulas
# CSV injection: If exported data contains user input
# User sets name to: =cmd|'/C calc'!A0
# When admin opens CSV in Excel → command execution
```

### Rate Limit Bypass for Feature Abuse

```http
# Free tier: 100 API calls/month
# Attack: Rotate API keys
POST /api/register → get key1 (100 calls)
POST /api/register → get key2 (100 calls)
POST /api/register → get key3 (100 calls)
# Effectively unlimited API access

# Free trial: 30 days
# Attack: Register new accounts every 30 days
# Or: Change account email to reset trial
# Or: Use virtual credit cards for new trials
```

---

## 6. Testing Methodology for Business Logic Flaws

### Think Like a Malicious User, Not a Hacker

```
Business Logic Testing Framework:
┌─────────────────────────────────────────────────────────┐
│ 1. UNDERSTAND THE APPLICATION                           │
│    - What does it do?                                   │
│    - What are the business rules?                       │
│    - What assumptions are made?                         │
│    - Who are the users and what are their roles?        │
├─────────────────────────────────────────────────────────┤
│ 2. MAP ALL WORKFLOWS                                    │
│    - Registration, login, checkout, etc.                │
│    - Identify every step and transition                 │
│    - Document expected vs actual behavior               │
├─────────────────────────────────────────────────────────┤
│ 3. IDENTIFY ASSUMPTIONS                                 │
│    - "Users will always go step 1 → 2 → 3"            │
│    - "Prices are always positive"                      │
│    - "Users can only see their own data"               │
│    - "Coupons can only be used once"                   │
├─────────────────────────────────────────────────────────┤
│ 4. CHALLENGE EACH ASSUMPTION                            │
│    - What if I skip steps?                             │
│    - What if I use negative values?                    │
│    - What if I repeat an action?                       │
│    - What if I do things in a different order?         │
│    - What if I do two things simultaneously?           │
├─────────────────────────────────────────────────────────┤
│ 5. TEST BOUNDARY CONDITIONS                             │
│    - Maximum/minimum values                            │
│    - Zero quantities                                   │
│    - Very large numbers                                │
│    - Special characters in business fields             │
├─────────────────────────────────────────────────────────┤
│ 6. TEST TIMING AND RACE CONDITIONS                      │
│    - Simultaneous identical requests                   │
│    - Actions during state transitions                  │
│    - Expired token/session usage                       │
└─────────────────────────────────────────────────────────┘
```

### Key Questions to Ask

| Area | Questions |
|------|-----------|
| **Pricing** | Can I change prices? Use negative values? Get items for free? |
| **Checkout** | Can I skip payment? Add items after payment? Reuse payment? |
| **Coupons** | Can I stack coupons? Reuse codes? Create my own? |
| **Accounts** | Can I access others' data? Create unlimited accounts? |
| **Limits** | Are there rate limits? Can I exceed quotas? Bypass restrictions? |
| **Workflows** | Can I skip steps? Go backwards? Repeat steps? |
| **Roles** | Can I escalate privileges? Access admin features? |
| **Data** | Can I export others' data? Manipulate reports? |
| **Timing** | Race conditions? Expired token use? Time-of-check issues? |
| **Integration** | Payment gateway manipulation? Third-party API abuse? |

---

## 7. Real-World Bug Bounty Examples

### Example 1: Uber — Free Rides
```
Vulnerability: Negative fare adjustment
- Uber allowed drivers to adjust fares for toll/route changes
- Attacker adjusted fare to negative amount
- Resulted in Uber paying the rider instead of charging
- Impact: Financial loss
```

### Example 2: Starbucks — Infinite Gift Card Balance
```
Vulnerability: Race condition in gift card transfer
- Transfer balance from Card A to Card B simultaneously
- Both transfers succeed before balance deduction
- Card B gets double the intended amount
- Repeat to multiply balance indefinitely
- Impact: Unlimited store credit
```

### Example 3: E-commerce — Free Premium Subscriptions
```
Vulnerability: Subscription logic flaw
- Sign up for free trial (7 days)
- Cancel before trial ends
- Re-register with same email
- Another free trial granted (no check for previous trials)
- Automate to get perpetual free access
- Impact: Revenue loss
```

### Example 4: Hotel Booking — Price Manipulation
```
Vulnerability: Client-side price calculation
- Room price calculated in JavaScript
- Modify price in intercepted request
- POST /booking {"room_id": 501, "price": 1.00, "nights": 3}
- Server didn't validate price against database
- Booked $500/night room for $1/night
- Impact: Revenue loss
```

---

## 8. Prevention Techniques

### Secure Design Principles

```python
# 1. Server-side validation of ALL business rules
@app.route('/checkout', methods=['POST'])
def checkout():
    cart = get_cart(session['user_id'])
    
    # Never trust client-supplied prices
    total = 0
    for item in cart.items:
        # Get price from DATABASE, not from request
        product = Product.query.get(item.product_id)
        if product is None:
            abort(400, "Invalid product")
        
        # Validate quantity
        if item.quantity <= 0 or item.quantity > product.stock:
            abort(400, "Invalid quantity")
        
        total += product.price * item.quantity
    
    # Apply coupons with server-side validation
    if cart.coupon:
        coupon = Coupon.query.filter_by(code=cart.coupon).first()
        if coupon and coupon.is_valid() and not coupon.is_used():
            total = apply_discount(total, coupon)
            coupon.mark_used()
    
    # Validate total is positive
    if total <= 0:
        abort(400, "Invalid order total")
    
    return process_payment(total)

# 2. Enforce workflow sequence
@app.route('/checkout/confirm', methods=['POST'])
def confirm():
    order = Order.query.get(request.json['order_id'])
    
    # Verify ALL previous steps completed
    if not order.shipping_set:
        abort(400, "Shipping address required")
    if not order.payment_processed:
        abort(400, "Payment required")
    if order.confirmed:
        abort(400, "Already confirmed")
    
    order.confirmed = True
    db.session.commit()
```

### Prevention Checklist

| Control | Implementation |
|---------|---------------|
| **Server-side price validation** | Always get prices from database, never from client |
| **Workflow enforcement** | Check that all previous steps are completed |
| **Idempotency** | Prevent duplicate processing of same action |
| **Rate limiting** | Limit coupon usage, API calls, account creation |
| **Input range validation** | Reject negative, zero, and unreasonably large values |
| **State machine** | Model workflows as state machines with valid transitions |
| **Atomic operations** | Use database transactions for financial operations |
| **Audit logging** | Log all business-critical operations for review |
| **Code review** | Peer review business logic for assumption gaps |
| **Threat modeling** | Identify abuse cases during design phase |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Definition** | Exploiting intended functionality in unintended ways |
| **Price Manipulation** | Negative quantities, zero prices, rounding exploits |
| **Workflow Bypass** | Skip steps, reorder steps, repeat steps |
| **Coupon Abuse** | Stacking, reuse, negative discounts |
| **Auth Logic Flaws** | Password reset, 2FA bypass, lockout bypass |
| **Feature Misuse** | Referral abuse, data export, rate limit bypass |
| **Testing Approach** | Understand business rules, challenge assumptions |
| **Prevention** | Server-side validation, workflow enforcement, atomic operations |

---

## Revision Questions

1. Why can't automated scanners detect business logic vulnerabilities? What makes manual testing essential?
2. Describe three different price manipulation techniques and explain how to prevent each.
3. How would you test a multi-step checkout process for workflow bypass vulnerabilities?
4. What is a race condition in the context of business logic? Give an example involving gift card transfers.
5. Explain the difference between technical vulnerabilities and business logic flaws with examples.
6. Design a secure checkout system that prevents price manipulation, coupon abuse, and workflow bypass.

---

*Previous: [04-deserialization-vulnerabilities.md](04-deserialization-vulnerabilities.md) | Next: [06-race-conditions.md](06-race-conditions.md)*

---

*[Back to README](../README.md)*

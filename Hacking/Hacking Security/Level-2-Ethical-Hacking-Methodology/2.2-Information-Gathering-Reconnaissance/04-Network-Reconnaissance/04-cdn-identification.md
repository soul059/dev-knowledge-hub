# CDN Identification

## Unit 4 - Topic 4 | Network Reconnaissance

---

## Overview

**CDN (Content Delivery Network) identification** determines whether a target uses a CDN and which provider. CDNs like Cloudflare, Akamai, and AWS CloudFront can mask the target's real IP address and provide WAF protection, requiring special techniques to bypass and reach the origin server.

---

## 1. Common CDN Providers

| CDN Provider | Detection Clues |
|-------------|----------------|
| **Cloudflare** | NS: *.ns.cloudflare.com, Headers: cf-ray, server: cloudflare |
| **Akamai** | CNAME: *.akamaiedge.net, Headers: X-Akamai-* |
| **AWS CloudFront** | CNAME: *.cloudfront.net, Headers: x-amz-cf-* |
| **Fastly** | Headers: X-Served-By, via: *.fastly.net |
| **Azure CDN** | CNAME: *.azureedge.net |
| **Google Cloud CDN** | Headers: via: google |
| **KeyCDN** | CNAME: *.kxcdn.com |
| **StackPath** | CNAME: *.stackpathcdn.com |

---

## 2. CDN Detection Methods

```bash
# === CHECK DNS RECORDS ===
dig target.com A +short
# If IP belongs to CDN range → behind CDN

dig target.com NS +short
# ns1.cloudflare.com → Cloudflare

dig target.com CNAME +short
# target.com.cdn.cloudflare.net → Cloudflare

# === CHECK HTTP HEADERS ===
curl -I https://target.com

# Cloudflare indicators:
# server: cloudflare
# cf-ray: 7a1234abcdef-LAX
# cf-cache-status: HIT

# Akamai indicators:
# X-Akamai-Transformed: ...
# Server: AkamaiGHost

# CloudFront indicators:
# x-amz-cf-id: ...
# x-amz-cf-pop: LAX50-C1
# via: 1.1 abc123.cloudfront.net

# === WHATWEB ===
whatweb https://target.com
# Identifies CDN in output

# === WAFW00F (WAF Detection) ===
wafw00f https://target.com
# Detects: Cloudflare, Akamai, Incapsula, ModSecurity, etc.
```

---

## 3. Finding Origin IP Behind CDN

```bash
# === METHOD 1: Historical DNS ===
# Check DNS before CDN adoption
# SecurityTrails, ViewDNS.info IP History

# === METHOD 2: Subdomains ===
# Not all subdomains go through CDN
dig mail.target.com A +short       # Mail servers often direct
dig ftp.target.com A +short        # FTP may be direct
dig vpn.target.com A +short        # VPN direct
dig direct.target.com A +short     # Sometimes explicitly named

# === METHOD 3: Email Headers ===
# Send email to target, check "Received:" headers
# May contain origin server IP

# === METHOD 4: SSL Certificate Search ===
shodan search "ssl.cert.subject.cn:target.com"
censys search "services.tls.certificates.leaf.names:target.com"
# Find IPs serving target's SSL cert that aren't CDN IPs

# === METHOD 5: Censys/Shodan Direct ===
shodan search "hostname:target.com"
# May show origin IP alongside CDN IPs

# === METHOD 6: DNS Leak Tools ===
# crimeflare.org — database of Cloudflare origin IPs
# CloudFlair tool — find origin behind Cloudflare
```

---

## 4. CDN Impact on Testing

| Aspect | Impact |
|--------|--------|
| **Port scanning** | Scanning CDN IP only shows CDN ports (80, 443) |
| **WAF protection** | CDN WAF blocks many attack payloads |
| **IP blocking** | CDN may rate-limit or block scanning IPs |
| **Origin hidden** | Can't directly test origin server |
| **Shared IP** | CDN IPs serve many customers |

```
TESTING STRATEGY WHEN CDN DETECTED:

1. Identify origin IP using methods above
2. If found → add to /etc/hosts and test directly
   echo "203.0.113.10 target.com" >> /etc/hosts
3. Test application through CDN (normal path)
4. Test application directly to origin (bypass CDN)
5. Compare results — origin may lack WAF protection!
```

---

## 5. CDN Detection Checklist

| Check | Command | CDN Indicator |
|-------|---------|:---:|
| DNS nameservers | `dig NS` | cloudflare, akamai |
| HTTP headers | `curl -I` | cf-ray, x-amz-cf |
| WAF detection | `wafw00f` | WAF product name |
| IP range lookup | `whois IP` | CDN org name |
| Technology detect | `whatweb` | CDN in results |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **CDN** | Masks origin IP, provides WAF |
| **Detection** | DNS, HTTP headers, wafw00f |
| **Origin finding** | Historical DNS, mail headers, Shodan |
| **Impact** | Can't directly scan CDN IPs |
| **Bypass** | Test origin directly if found |
| **Common CDNs** | Cloudflare, Akamai, CloudFront, Fastly |

---

## Quick Revision Questions

1. **How does a CDN affect penetration testing?**
2. **How do you detect which CDN a target uses?**
3. **Name 3 methods to find the origin IP behind a CDN.**
4. **Why test the origin server directly if possible?**
5. **What is wafw00f used for?**

---

[← Previous: Network Mapping](03-network-mapping.md) | [Next: Cloud Infrastructure Discovery →](05-cloud-infrastructure-discovery.md)

---

*Unit 4 - Topic 4 of 5 | [Back to README](../README.md)*

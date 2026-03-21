# Robots.txt and Sitemap Analysis

## Unit 5 - Topic 4 | Web Reconnaissance

---

## Overview

**Robots.txt** and **sitemap.xml** are standard web files that help search engines crawl websites. For penetration testers, these files often inadvertently reveal hidden directories, sensitive paths, and the complete URL structure of a web application.

---

## 1. Robots.txt

```bash
# === ACCESSING ROBOTS.TXT ===
curl https://target.com/robots.txt

# === EXAMPLE OUTPUT ===
User-agent: *
Disallow: /admin/
Disallow: /backup/
Disallow: /api/internal/
Disallow: /config/
Disallow: /old-site/
Disallow: /staging/
Disallow: /wp-admin/
Disallow: /user/profile?debug=true

Sitemap: https://target.com/sitemap.xml

# === WHAT THIS REVEALS ===
# ├── /admin/          → Admin panel exists!
# ├── /backup/         → Backup directory!
# ├── /api/internal/   → Internal API endpoints!
# ├── /config/         → Configuration files!
# ├── /old-site/       → Legacy site (often unpatched)!
# ├── /staging/        → Staging environment!
# └── debug=true       → Debug mode parameter!
```

---

## 2. Why Robots.txt is Valuable

| What It Shows | Security Implication |
|--------------|---------------------|
| **Disallowed paths** | Directories the org wants hidden |
| **Admin panels** | Targets for brute-force attacks |
| **API endpoints** | Internal APIs not meant for public |
| **Old/staging sites** | Often unpatched and vulnerable |
| **Debug parameters** | Additional attack surface |
| **Sitemap location** | Complete URL inventory |

```
⚠️ IMPORTANT: robots.txt is a REQUEST, not enforcement.
Search engines VOLUNTARILY follow it.
Attackers are NOT bound by robots.txt!

Disallow: /secret/     ← Does NOT prevent access
                        ← Actually tells you it exists!
```

---

## 3. Sitemap Analysis

```bash
# === ACCESSING SITEMAP ===
curl https://target.com/sitemap.xml
curl https://target.com/sitemap_index.xml  # Multiple sitemaps

# === EXAMPLE SITEMAP ===
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://target.com/products/</loc>
    <lastmod>2024-01-15</lastmod>
  </url>
  <url>
    <loc>https://target.com/api/v2/docs/</loc>
    <lastmod>2024-01-10</lastmod>
  </url>
  <url>
    <loc>https://target.com/portal/login/</loc>
    <lastmod>2023-12-20</lastmod>
  </url>
</urlset>

# === EXTRACT ALL URLS FROM SITEMAP ===
curl -s https://target.com/sitemap.xml | grep -oP '<loc>\K[^<]+'

# === CHECK FOR MULTIPLE SITEMAPS ===
curl -s https://target.com/sitemap_index.xml | grep -oP '<loc>\K[^<]+'
# May list: sitemap-posts.xml, sitemap-pages.xml, sitemap-users.xml
```

---

## 4. Additional Discovery Files

```bash
# === SECURITY.TXT ===
curl https://target.com/.well-known/security.txt
# Reveals: security contact, bug bounty info, PGP key

# === HUMANS.TXT ===
curl https://target.com/humans.txt
# Reveals: developer names, technologies used, CMS

# === CROSSDOMAIN.XML (Flash — legacy) ===
curl https://target.com/crossdomain.xml
# May allow cross-domain requests from any origin

# === CLIENTACCESSPOLICY.XML (Silverlight — legacy) ===
curl https://target.com/clientaccesspolicy.xml

# === .WELL-KNOWN DIRECTORY ===
curl https://target.com/.well-known/
# May contain: openid-configuration, jwks.json, etc.

# === OPENAPI/SWAGGER SPEC ===
curl https://target.com/swagger.json
curl https://target.com/openapi.json
curl https://target.com/api-docs
# Full API documentation with all endpoints!
```

---

## 5. Automated Analysis

```bash
# === PARSERO (robots.txt analyzer) ===
parsero -u https://target.com
# Automatically checks all Disallowed paths
# Reports which return 200 (accessible!)

# === CUSTOM SCRIPT ===
# Extract and test all disallowed paths:
curl -s https://target.com/robots.txt \
  | grep "Disallow:" \
  | awk '{print $2}' \
  | while read path; do
      code=$(curl -o /dev/null -s -w "%{http_code}" "https://target.com${path}")
      echo "$code - $path"
    done

# Output:
# 200 - /admin/          ← ACCESSIBLE!
# 403 - /backup/         ← Forbidden but exists
# 404 - /old-site/       ← Removed
# 301 - /staging/        ← Redirects somewhere
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **robots.txt** | Reveals hidden paths orgs want unlisted |
| **sitemap.xml** | Complete URL inventory of the site |
| **Disallow** | NOT enforcement — just a suggestion |
| **security.txt** | Security contact and bug bounty info |
| **swagger.json** | Full API endpoint documentation |
| **Parsero** | Automated robots.txt analysis |

---

## Quick Revision Questions

1. **Why is robots.txt valuable for reconnaissance?**
2. **Does robots.txt prevent access to listed paths?**
3. **What information can sitemap.xml provide?**
4. **Where do you find the security.txt file?**
5. **How do you automate testing of disallowed paths?**

---

[← Previous: Directory Enumeration](03-directory-enumeration.md) | [Next: Web Archives →](05-web-archives.md)

---

*Unit 5 - Topic 4 of 6 | [Back to README](../README.md)*

# Unit 5: A05 - Security Misconfiguration — Topic 1: Default Configurations

## Overview

Security misconfiguration is the **most commonly seen vulnerability** across web applications and infrastructure. Default configurations — including default credentials, default settings, unnecessary features enabled, and insecure default permissions — account for a significant portion of security breaches. Attackers routinely scan for default configurations because they require zero skill to exploit.

---

## 1. The Default Configuration Problem

```
┌─────────────────────────────────────────────────────────────────┐
│              DEFAULT CONFIGURATION RISKS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Software ships with CONVENIENCE as priority, not SECURITY     │
│                                                                 │
│  Defaults typically include:                                   │
│  ├── Default admin credentials (admin/admin, root/root)        │
│  ├── Debug mode enabled                                        │
│  ├── Verbose error messages                                    │
│  ├── Sample/test data and pages                                │
│  ├── Unnecessary services enabled                              │
│  ├── Weak encryption settings                                  │
│  ├── Open permissions (777, Everyone:Full Control)             │
│  ├── Default ports and paths                                   │
│  └── Unnecessary HTTP methods enabled                          │
│                                                                 │
│  Attacker's perspective:                                       │
│  "Did the admin change the defaults?" → Often NO               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Common Default Credentials

| Product | Default Username | Default Password | Risk |
|---------|-----------------|------------------|------|
| **Apache Tomcat** | tomcat | tomcat (or s3cret) | Deploy webshells via Manager |
| **Jenkins** | admin | (initial setup wizard) | Code execution |
| **phpMyAdmin** | root | (empty) | Full DB access |
| **MongoDB** | (none) | (no auth by default) | Full DB access |
| **Elasticsearch** | (none) | (no auth < 8.0) | Data exposure |
| **Redis** | (none) | (no auth by default) | RCE via SLAVEOF |
| **PostgreSQL** | postgres | postgres | Full DB access |
| **Cisco Devices** | cisco / admin | cisco / admin | Network takeover |
| **WordPress** | admin | admin | Site takeover |
| **Grafana** | admin | admin | Dashboard access |
| **RabbitMQ** | guest | guest | Message queue access |
| **Portainer** | admin | (set on first login) | Container management |

### Scanning for Default Credentials

```bash
# Nmap default credential scripts
nmap --script http-default-accounts -p 80,8080,8443 target.com
nmap --script mysql-empty-password -p 3306 target.com
nmap --script mongodb-info -p 27017 target.com

# Hydra brute force with default lists
hydra -L /usr/share/seclists/Passwords/Default-Credentials/tomcat-betterdefaultpasslist.txt \
      -P /usr/share/seclists/Passwords/Default-Credentials/tomcat-betterdefaultpasslist.txt \
      target.com http-get /manager/html

# Metasploit default credential modules
use auxiliary/scanner/http/tomcat_mgr_login
use auxiliary/scanner/http/jenkins_login
use auxiliary/scanner/mysql/mysql_login
```

---

## 3. Web Server Default Configurations

### Apache HTTPD

```apache
# ❌ DEFAULT — Insecure
# Server version exposed
ServerTokens Full
ServerSignature On

# Directory listing enabled
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
</Directory>

# All HTTP methods allowed
# No security headers

# ✅ HARDENED
ServerTokens Prod
ServerSignature Off

<Directory "/var/www/html">
    Options -Indexes -FollowSymLinks
    AllowOverride None
</Directory>

# Disable unnecessary methods
<LimitExcept GET POST>
    Require all denied
</LimitExcept>

# Security headers
Header set X-Content-Type-Options "nosniff"
Header set X-Frame-Options "DENY"
Header set Content-Security-Policy "default-src 'self'"
Header set Strict-Transport-Security "max-age=31536000"

# Disable TRACE method
TraceEnable Off
```

### Nginx

```nginx
# ❌ DEFAULT — Insecure
server {
    listen 80;
    server_name example.com;
    
    # Version exposed in headers
    # server_tokens on; (default)
    
    # autoindex on; (if set)
}

# ✅ HARDENED
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Hide version
    server_tokens off;
    
    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header Content-Security-Policy "default-src 'self'" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Disable unnecessary methods
    if ($request_method !~ ^(GET|POST|HEAD)$) {
        return 405;
    }
    
    # TLS configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    
    # Disable directory listing
    autoindex off;
}
```

---

## 4. Application Framework Defaults

### Django

```python
# ❌ DEFAULT / DEV SETTINGS — Insecure
DEBUG = True                          # Exposes stack traces, settings
ALLOWED_HOSTS = ['*']                 # Accepts any hostname
SECRET_KEY = 'django-insecure-...'    # Weak/known key

# ✅ PRODUCTION SETTINGS
DEBUG = False
ALLOWED_HOSTS = ['www.example.com']
SECRET_KEY = os.environ['DJANGO_SECRET_KEY']  # From environment variable

# Security middleware
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
X_FRAME_OPTIONS = 'DENY'
```

### Express.js (Node.js)

```javascript
// ❌ DEFAULT — Insecure
const app = express();
// X-Powered-By: Express header exposed by default
// No security headers
// No rate limiting
// Stack traces in errors

// ✅ HARDENED
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const app = express();

// Remove X-Powered-By header
app.disable('x-powered-by');

// Add security headers via Helmet
app.use(helmet());
app.use(helmet.contentSecurityPolicy({
    directives: { defaultSrc: ["'self'"] }
}));

// Rate limiting
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));

// Secure error handling (no stack traces)
app.use((err, req, res, next) => {
    console.error(err.stack);  // Log internally
    res.status(500).json({ error: 'Internal server error' });  // Generic response
});
```

### Spring Boot (Java)

```yaml
# ❌ DEFAULT — Potentially insecure
# application.yml
spring:
  devtools:
    restart:
      enabled: true          # Hot reload
  h2:
    console:
      enabled: true          # H2 database console exposed

management:
  endpoints:
    web:
      exposure:
        include: "*"          # ALL actuator endpoints exposed

# ✅ HARDENED
spring:
  devtools:
    restart:
      enabled: false
  h2:
    console:
      enabled: false

management:
  endpoints:
    web:
      exposure:
        include: health,info  # Only health and info
  endpoint:
    health:
      show-details: never     # Don't expose health details

server:
  error:
    include-message: never
    include-stacktrace: never
    include-binding-errors: never
```

---

## 5. Database Default Configurations

```
┌──────────────────┬────────────────────────────┬──────────────────────────┐
│ Database         │ Default Risk               │ Hardening                │
├──────────────────┼────────────────────────────┼──────────────────────────┤
│ MySQL            │ Remote root login allowed  │ Run mysql_secure_install │
│                  │ Test database exists        │ Remove test DB, anon users│
│                  │                            │ Bind to 127.0.0.1        │
├──────────────────┼────────────────────────────┼──────────────────────────┤
│ PostgreSQL       │ trust auth in pg_hba.conf  │ Use scram-sha-256       │
│                  │ Listens on all interfaces  │ Bind to localhost        │
│                  │                            │ Require SSL connections   │
├──────────────────┼────────────────────────────┼──────────────────────────┤
│ MongoDB          │ No authentication enabled  │ Enable auth, create users│
│                  │ Binds to all interfaces    │ Bind to 127.0.0.1        │
│                  │                            │ Enable TLS               │
├──────────────────┼────────────────────────────┼──────────────────────────┤
│ Redis            │ No authentication          │ Set requirepass          │
│                  │ Binds to all interfaces    │ Bind to 127.0.0.1        │
│                  │ Dangerous commands allowed │ Rename CONFIG, FLUSHALL  │
├──────────────────┼────────────────────────────┼──────────────────────────┤
│ Elasticsearch    │ No auth (pre-8.0)          │ Enable X-Pack security   │
│                  │ HTTP API publicly open     │ TLS, RBAC               │
└──────────────────┴────────────────────────────┴──────────────────────────┘
```

### MongoDB Hardening

```javascript
// Enable authentication in mongod.conf
security:
  authorization: enabled

// Create admin user
use admin
db.createUser({
    user: "adminUser",
    pwd: passwordPrompt(),
    roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
})

// Create application user with minimum privileges
use myapp
db.createUser({
    user: "appUser",
    pwd: passwordPrompt(),
    roles: [{ role: "readWrite", db: "myapp" }]
})

// Bind to localhost only
net:
  bindIp: 127.0.0.1
  tls:
    mode: requireTLS
    certificateKeyFile: /etc/ssl/mongodb.pem
```

---

## 6. Default Configuration Checklist

```
□ All default credentials changed on ALL systems
□ Default/sample pages removed (index.html, phpinfo.php)
□ Debug mode disabled in production
□ Server version headers suppressed
□ Directory listing disabled
□ Unnecessary HTTP methods disabled (TRACE, PUT, DELETE)
□ Default ports changed where appropriate
□ Test/demo databases removed
□ Admin panels restricted to internal network or VPN
□ Unnecessary services and modules disabled
□ File permissions set to minimum required
□ Default error pages replaced with custom generic pages
□ SSL/TLS configured with strong ciphers only
□ Logging configured and monitored
```

---

## Summary Table

| Component | Default Risk | Primary Mitigation |
|-----------|-------------|-------------------|
| Web Servers | Version exposure, directory listing | Server hardening guide |
| Databases | No auth, public binding | Enable auth, bind localhost |
| Frameworks | Debug mode, verbose errors | Production configuration |
| Network Devices | Default credentials | Change on deployment |
| Cloud Services | Public buckets, open APIs | Least privilege policies |

---

## Revision Questions

1. Why are default configurations the most commonly exploited vulnerability?
2. List default credentials for five common services and explain the risk of each.
3. Compare the default vs hardened configuration for Apache and Nginx.
4. What are the critical Django/Express/Spring Boot settings that must be changed for production?
5. Write a database hardening checklist for MongoDB and Redis.
6. How would you automate the detection of default configurations across an organization?

---

*Previous: None (First topic in this unit) | Next: [02-unnecessary-features.md](02-unnecessary-features.md)*

---

*[Back to README](../README.md)*

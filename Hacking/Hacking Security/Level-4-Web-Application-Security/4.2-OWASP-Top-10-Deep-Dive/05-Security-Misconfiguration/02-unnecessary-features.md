# Unit 5: A05 - Security Misconfiguration — Topic 2: Unnecessary Features

## Overview

Every unnecessary feature, service, port, page, account, or privilege represents an **expanded attack surface** that can be exploited. The principle of **least functionality** dictates that systems should be configured to provide only the capabilities required for their intended purpose. This topic covers identifying and removing unnecessary features across all layers of the technology stack.

---

## 1. Categories of Unnecessary Features

```
┌─────────────────────────────────────────────────────────────────┐
│              UNNECESSARY FEATURES TO REMOVE                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SERVICES & PORTS:                                             │
│  ├── FTP (21) — Use SFTP/SCP instead                           │
│  ├── Telnet (23) — Use SSH instead                             │
│  ├── SNMP v1/v2 (161) — Use v3 or remove                      │
│  ├── RDP (3389) — VPN + MFA or remove                         │
│  └── Debug ports (various)                                     │
│                                                                 │
│  WEB SERVER FEATURES:                                          │
│  ├── Directory listing / autoindex                             │
│  ├── WebDAV module                                             │
│  ├── Server status pages (/server-status, /server-info)        │
│  ├── CGI/PHP info pages                                        │
│  ├── Default/sample applications                               │
│  └── TRACE/TRACK HTTP methods                                  │
│                                                                 │
│  APPLICATION FEATURES:                                         │
│  ├── Debug/development endpoints                               │
│  ├── Test accounts                                             │
│  ├── Admin consoles exposed publicly                           │
│  ├── API documentation on production                           │
│  ├── Unused API versions (v1, v2 when v3 is current)           │
│  └── Sample/demo data                                          │
│                                                                 │
│  SYSTEM FEATURES:                                              │
│  ├── Compiler/development tools on servers                     │
│  ├── Package managers on production                            │
│  ├── Unused kernel modules                                     │
│  ├── Unused user accounts                                      │
│  └── Unnecessary scheduled tasks/cron jobs                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Web Server Unnecessary Modules

### Apache Modules to Disable

```apache
# Check loaded modules
apachectl -M

# ❌ Modules to disable in production:
LoadModule status_module     # /server-status page
LoadModule info_module       # /server-info page
LoadModule autoindex_module  # Directory listing
LoadModule cgi_module        # CGI execution (if not needed)
LoadModule dav_module        # WebDAV
LoadModule dav_fs_module     # WebDAV filesystem
LoadModule userdir_module    # ~/username directories
LoadModule proxy_module      # Proxy (if not needed)

# Disable on Debian/Ubuntu:
a2dismod status info autoindex cgi dav dav_fs userdir

# Disable on CentOS/RHEL:
# Comment out LoadModule lines in httpd.conf
```

### Nginx Unnecessary Features

```nginx
# Remove default site
rm /etc/nginx/sites-enabled/default

# Disable autoindex globally
autoindex off;

# Remove stub_status if not monitored
# location /nginx_status {
#     stub_status on;    ← REMOVE
# }

# Disable unused upstream modules
# Only compile with needed modules:
# ./configure --without-http_autoindex_module \
#             --without-http_ssi_module
```

---

## 3. Application-Level Unnecessary Features

### Debug Endpoints

```python
# ❌ DANGEROUS — Debug/dev endpoints in production

# Flask debug routes (MUST REMOVE)
@app.route('/debug/config')
def show_config():
    return jsonify(dict(app.config))  # Exposes secrets!

@app.route('/debug/routes')
def show_routes():
    return jsonify([str(rule) for rule in app.url_map.iter_rules()])

@app.route('/debug/env')
def show_env():
    return jsonify(dict(os.environ))  # Exposes credentials!

# Django debug toolbar (MUST DISABLE in prod)
# settings.py
INSTALLED_APPS = [
    'debug_toolbar',  # ← REMOVE in production
]

# Spring Boot Actuator endpoints
# /actuator/env       — Environment variables
# /actuator/configprops — Configuration properties
# /actuator/heapdump  — JVM heap dump
# /actuator/threaddump — Thread dump
# /actuator/mappings  — URL mappings

# ✅ SECURE — Only expose health in production
management.endpoints.web.exposure.include=health
```

### Swagger/API Documentation

```
❌ RISK: Swagger UI exposed on production
   https://api.production.com/swagger-ui.html
   → Attackers can see ALL API endpoints, parameters, data types
   → Essentially a roadmap for attacking the application

✅ MITIGATION:
   Option 1: Remove entirely from production build
   Option 2: Restrict to authenticated users only
   Option 3: Restrict to internal network / VPN only
```

---

## 4. Sample and Default Applications

```
Common default apps to remove:

┌──────────────────────────┬───────────────────────────────────┐
│ Platform                 │ Default Apps to Remove            │
├──────────────────────────┼───────────────────────────────────┤
│ Apache Tomcat            │ /examples, /docs, /manager,      │
│                          │ /host-manager, ROOT application   │
├──────────────────────────┼───────────────────────────────────┤
│ IIS                      │ Default Web Site, iisstart.htm    │
│                          │ ISAPI filters, sample apps        │
├──────────────────────────┼───────────────────────────────────┤
│ WordPress                │ Sample page, Hello Dolly plugin,  │
│                          │ unused themes                     │
├──────────────────────────┼───────────────────────────────────┤
│ PHP                      │ phpinfo.php, test.php,            │
│                          │ adminer.php, phpmyadmin            │
├──────────────────────────┼───────────────────────────────────┤
│ ColdFusion               │ /CFIDE/administrator, /CFIDE/,    │
│                          │ getting_started pages             │
├──────────────────────────┼───────────────────────────────────┤
│ Jenkins                  │ /script console, /manage          │
│                          │ (restrict access)                 │
└──────────────────────────┴───────────────────────────────────┘
```

### Tomcat Hardening

```bash
# Remove default/sample applications
rm -rf $CATALINA_HOME/webapps/ROOT
rm -rf $CATALINA_HOME/webapps/docs
rm -rf $CATALINA_HOME/webapps/examples
rm -rf $CATALINA_HOME/webapps/host-manager

# Restrict Manager app (if needed)
# In tomcat-users.xml — use strong credentials
# In manager/META-INF/context.xml — restrict to internal IPs
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
       allow="127\.0\.0\.1|10\.0\.0\.\d+" />
```

---

## 5. Operating System Unnecessary Features

```bash
# List running services
systemctl list-units --type=service --state=running

# Disable unnecessary services
systemctl disable --now cups        # Printing (not needed on server)
systemctl disable --now avahi-daemon # mDNS (not needed on server)
systemctl disable --now bluetooth   # Bluetooth
systemctl disable --now nfs-server  # NFS (if not used)
systemctl disable --now rpcbind     # RPC (if not used)
systemctl disable --now postfix     # Mail (if not used)

# Remove unnecessary packages
apt remove --purge gcc make curl wget  # Dev tools (if not needed)
apt autoremove                          # Clean up dependencies

# Disable unused kernel modules
echo "install cramfs /bin/true" >> /etc/modprobe.d/disable-fs.conf
echo "install freevxfs /bin/true" >> /etc/modprobe.d/disable-fs.conf
echo "install jffs2 /bin/true" >> /etc/modprobe.d/disable-fs.conf
echo "install udf /bin/true" >> /etc/modprobe.d/disable-fs.conf

# Remove unnecessary SUID binaries
find / -perm -4000 -type f 2>/dev/null
# Evaluate each and remove SUID where not needed:
chmod u-s /usr/bin/unnecessary-suid-binary
```

---

## 6. Container Unnecessary Features

```dockerfile
# ❌ INSECURE — Full Ubuntu image with everything
FROM ubuntu:latest
RUN apt-get update && apt-get install -y python3 python3-pip
COPY . /app
CMD ["python3", "/app/main.py"]

# ✅ SECURE — Minimal Alpine-based image
FROM python:3.11-alpine
RUN adduser -D appuser  # Non-root user
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt && \
    rm -rf /root/.cache    # Remove pip cache
COPY --chown=appuser:appuser . .
USER appuser              # Run as non-root
CMD ["python3", "main.py"]

# ✅ MOST SECURE — Distroless image (Google)
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

FROM gcr.io/distroless/python3-debian12
COPY --from=builder /app /app
WORKDIR /app
CMD ["main.py"]
# No shell, no package manager, no unnecessary tools
```

---

## 7. Feature Audit Checklist

```
WEB SERVER:
□ Default pages removed
□ Directory listing disabled
□ Unnecessary modules disabled
□ Server status/info pages disabled
□ TRACE method disabled

APPLICATION:
□ Debug endpoints removed from production
□ Test accounts removed
□ API documentation restricted
□ Unused API versions decommissioned
□ Sample data removed
□ Admin consoles restricted to internal network

SYSTEM:
□ Unnecessary services stopped and disabled
□ Unnecessary packages removed
□ Development tools not on production servers
□ Unused user accounts disabled
□ SUID/SGID binaries audited
□ Scheduled tasks reviewed

CLOUD:
□ Unused Lambda/Functions removed
□ Unused storage buckets removed
□ Unused IAM roles removed
□ Default VPC security groups tightened
□ Unused API Gateway endpoints removed
```

---

## Summary Table

| Layer | Common Unnecessary Features | Impact if Left |
|-------|---------------------------|----------------|
| Network | Open ports, unused protocols | Direct entry points |
| Web Server | Default pages, status modules | Info disclosure, access |
| Application | Debug endpoints, test accounts | Code execution, auth bypass |
| OS | Dev tools, unused services | Privilege escalation |
| Container | Full OS image, root user | Container escape, RCE |

---

## Revision Questions

1. What is the principle of "least functionality" and why is it important for security?
2. List five Apache/Nginx modules that should be disabled in production and explain the risk of each.
3. Why are Swagger/API documentation pages dangerous on production systems?
4. What default Tomcat applications should be removed and why?
5. Compare a regular Docker image vs Alpine vs Distroless for security.
6. Design a quarterly feature audit process for a production web application.

---

*Previous: [01-default-configurations.md](01-default-configurations.md) | Next: [03-error-handling.md](03-error-handling.md)*

---

*[Back to README](../README.md)*

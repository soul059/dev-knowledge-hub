# Unit 5: A05 - Security Misconfiguration — Topic 6: Hardening Procedures

## Overview

Security hardening is the **systematic process of reducing the attack surface** by applying secure configurations, removing unnecessary components, and enforcing security policies across all layers of the technology stack. Hardening transforms a default-configured system into a production-ready, security-optimized system. This topic covers hardening procedures with reference to industry benchmarks and automation.

---

## 1. Hardening Frameworks and Benchmarks

```
┌─────────────────────────────────────────────────────────────────┐
│             INDUSTRY HARDENING STANDARDS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CIS Benchmarks (Center for Internet Security):                │
│  ├── Most widely used hardening guides                         │
│  ├── Available for 100+ platforms                              │
│  ├── Two levels: L1 (basic) and L2 (defense-in-depth)         │
│  └── Free PDF downloads: cisecurity.org                        │
│                                                                 │
│  DISA STIGs (Security Technical Implementation Guides):        │
│  ├── US Department of Defense standards                        │
│  ├── More strict than CIS                                      │
│  ├── Required for government systems                           │
│  └── Available at: public.cyber.mil                            │
│                                                                 │
│  NIST SP 800-123 (Guide to General Server Security):           │
│  ├── Federal standard                                          │
│  ├── General hardening guidelines                              │
│  └── Available at: nist.gov                                    │
│                                                                 │
│  Vendor-Specific Guides:                                       │
│  ├── Microsoft Security Baselines                              │
│  ├── AWS Well-Architected Framework                            │
│  ├── Red Hat Hardening Guide                                   │
│  └── Docker Bench for Security                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Linux Server Hardening

```bash
# ═══════════════════════════════════════════════════
# LINUX HARDENING SCRIPT (Based on CIS Benchmarks)
# ═══════════════════════════════════════════════════

# 1. UPDATE SYSTEM
apt update && apt upgrade -y
# Enable automatic security updates
apt install unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades

# 2. USER AND AUTHENTICATION
# Set password policy
cat >> /etc/security/pwquality.conf << EOF
minlen = 14
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
EOF

# Lock down root
passwd -l root              # Lock root password
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# Set login timeout
echo "TMOUT=900" >> /etc/profile    # 15 min idle timeout

# 3. SSH HARDENING
cat > /etc/ssh/sshd_config.d/hardening.conf << EOF
Protocol 2
PermitRootLogin no
PasswordAuthentication no        # Key-based only
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
AllowTcpForwarding no
PermitEmptyPasswords no
Banner /etc/issue.net
AllowUsers deploy admin          # Whitelist users
EOF
systemctl restart sshd

# 4. FIREWALL
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp                 # SSH (restrict to IP later)
ufw allow 443/tcp                # HTTPS
ufw enable

# 5. DISABLE UNNECESSARY SERVICES
systemctl disable --now cups avahi-daemon bluetooth rpcbind nfs-server

# 6. FILE PERMISSIONS
chmod 600 /etc/shadow
chmod 644 /etc/passwd
chmod 600 /boot/grub/grub.cfg
chmod 700 /root

# 7. LOGGING
# Ensure rsyslog/auditd is running
systemctl enable --now rsyslog
systemctl enable --now auditd

# Add audit rules
cat >> /etc/audit/rules.d/hardening.rules << EOF
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k sudo_changes
-w /var/log/ -p wa -k log_changes
EOF

# 8. KERNEL HARDENING (sysctl)
cat >> /etc/sysctl.d/99-hardening.conf << EOF
# Disable IP forwarding
net.ipv4.ip_forward = 0
# Disable ICMP redirect acceptance
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
# Enable SYN flood protection
net.ipv4.tcp_syncookies = 1
# Disable source routing
net.ipv4.conf.all.accept_source_route = 0
# Enable reverse path filtering
net.ipv4.conf.all.rp_filter = 1
# Restrict dmesg
kernel.dmesg_restrict = 1
# Restrict kernel pointer access
kernel.kptr_restrict = 2
# Enable ASLR
kernel.randomize_va_space = 2
EOF
sysctl --system
```

---

## 3. Web Server Hardening

### Nginx Hardening Checklist

```nginx
# /etc/nginx/conf.d/security.conf

# Hide version
server_tokens off;

# Security headers
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header Content-Security-Policy "default-src 'self'" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;

# TLS configuration
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384';
ssl_prefer_server_ciphers off;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:10m;
ssl_stapling on;
ssl_stapling_verify on;

# Request limits
client_max_body_size 10m;
client_body_timeout 12;
client_header_timeout 12;
keepalive_timeout 15;
send_timeout 10;

# Disable unnecessary methods
if ($request_method !~ ^(GET|HEAD|POST)$) {
    return 405;
}

# Block sensitive files
location ~ /\. {
    deny all;
}
location ~ \.(git|svn|env|bak|sql|log) {
    deny all;
}
```

---

## 4. Database Hardening

```sql
-- MySQL/MariaDB Hardening

-- Run secure installation
-- mysql_secure_install

-- 1. Remove anonymous users
DELETE FROM mysql.user WHERE User='';

-- 2. Remove test database
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';

-- 3. Disable remote root login
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');

-- 4. Set strong password policy
SET GLOBAL validate_password.length = 14;
SET GLOBAL validate_password.mixed_case_count = 1;
SET GLOBAL validate_password.number_count = 1;
SET GLOBAL validate_password.special_char_count = 1;

-- 5. Create application user with minimum privileges
CREATE USER 'webapp'@'10.0.2.%' IDENTIFIED BY 'StrongP@ssword!';
GRANT SELECT, INSERT, UPDATE ON appdb.* TO 'webapp'@'10.0.2.%';

-- 6. Enable audit logging
INSTALL PLUGIN audit_log SONAME 'audit_log.so';

FLUSH PRIVILEGES;
```

```ini
# my.cnf hardening
[mysqld]
# Bind to private IP only
bind-address = 10.0.2.50

# Disable local file access
local-infile = 0

# Disable symbolic links
symbolic-links = 0

# Enable error log
log-error = /var/log/mysql/error.log

# Enable slow query log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log

# TLS for client connections
require_secure_transport = ON
ssl-ca = /etc/mysql/ssl/ca-cert.pem
ssl-cert = /etc/mysql/ssl/server-cert.pem
ssl-key = /etc/mysql/ssl/server-key.pem
```

---

## 5. Container Hardening

```yaml
# Docker daemon hardening (/etc/docker/daemon.json)
{
    "icc": false,                    # Disable inter-container communication
    "no-new-privileges": true,       # Prevent privilege escalation
    "userland-proxy": false,         # Use iptables instead
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "live-restore": true,
    "default-ulimits": {
        "nofile": { "Name": "nofile", "Hard": 64000, "Soft": 64000 }
    }
}
```

```dockerfile
# Hardened Dockerfile
FROM python:3.11-slim

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY --chown=appuser:appuser . /app
WORKDIR /app

# Drop capabilities
# Set read-only filesystem where possible
# Run as non-root
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:8080/health || exit 1

EXPOSE 8080
CMD ["python", "app.py"]
```

```bash
# Docker Bench for Security (CIS Docker Benchmark)
docker run --net host --pid host --userns host --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /var/lib:/var/lib:ro \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    -v /etc:/etc:ro \
    docker/docker-bench-security
```

---

## 6. Automated Hardening with Configuration Management

### Ansible Hardening Playbook

```yaml
---
- name: Server Hardening
  hosts: all
  become: yes
  tasks:
    - name: Update all packages
      apt:
        update_cache: yes
        upgrade: dist

    - name: Disable root SSH login
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PermitRootLogin'
        line: 'PermitRootLogin no'
      notify: restart sshd

    - name: Disable password authentication
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PasswordAuthentication'
        line: 'PasswordAuthentication no'
      notify: restart sshd

    - name: Set firewall rules
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - '22'
        - '443'

    - name: Enable firewall
      ufw:
        state: enabled
        default: deny

    - name: Disable unnecessary services
      systemd:
        name: "{{ item }}"
        state: stopped
        enabled: no
      loop:
        - cups
        - avahi-daemon
        - bluetooth

    - name: Apply sysctl hardening
      sysctl:
        name: "{{ item.key }}"
        value: "{{ item.value }}"
        sysctl_set: yes
      loop:
        - { key: 'net.ipv4.ip_forward', value: '0' }
        - { key: 'net.ipv4.tcp_syncookies', value: '1' }
        - { key: 'kernel.randomize_va_space', value: '2' }

  handlers:
    - name: restart sshd
      systemd:
        name: sshd
        state: restarted
```

### Infrastructure as Code (IaC) Security

```hcl
# Terraform with security defaults
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = var.vpc_id

  # Only allow HTTPS inbound
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Restrict outbound
  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Environment = "production"
    ManagedBy   = "terraform"
  }
}

# S3 bucket with security defaults
resource "aws_s3_bucket" "data" {
  bucket = "company-data"
}

resource "aws_s3_bucket_public_access_block" "data" {
  bucket = aws_s3_bucket.data.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "data" {
  bucket = aws_s3_bucket.data.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}
```

---

## 7. Hardening Verification

```bash
# CIS-CAT (CIS Configuration Assessment Tool)
# Commercial tool that checks against CIS Benchmarks

# Lynis — Open source security auditing
apt install lynis
lynis audit system
# Generates hardening index score (0-100)
# Lists warnings and suggestions

# OpenSCAP — Compliance scanner
apt install libopenscap8
oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis \
    /usr/share/xml/scap/ssg/content/ssg-ubuntu2004-ds.xml

# Nessus — Vulnerability and compliance scanning
# Commercial tool with CIS Benchmark audit templates
```

---

## Summary Table

| Layer | Hardening Focus | Benchmark |
|-------|----------------|-----------|
| OS | Users, services, kernel, firewall | CIS Linux/Windows Benchmark |
| Web Server | Headers, TLS, modules, methods | CIS Apache/Nginx Benchmark |
| Database | Auth, access, encryption, logging | CIS MySQL/PostgreSQL Benchmark |
| Container | Image, runtime, daemon | CIS Docker Benchmark |
| Cloud | IAM, network, storage, logging | CIS AWS/Azure/GCP Benchmark |
| Application | Config, deps, error handling | OWASP ASVS |

---

## Revision Questions

1. What are CIS Benchmarks and DISA STIGs? How do they differ?
2. Write a Linux hardening script covering SSH, firewall, kernel parameters, and service management.
3. Create an Nginx security configuration file with all recommended hardening settings.
4. What are the essential MySQL hardening steps? Write the SQL commands.
5. How do you harden a Docker container? Cover Dockerfile, daemon, and runtime settings.
6. Design an automated hardening pipeline using Ansible or Terraform with compliance verification.

---

*Previous: [05-cloud-misconfigurations.md](05-cloud-misconfigurations.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

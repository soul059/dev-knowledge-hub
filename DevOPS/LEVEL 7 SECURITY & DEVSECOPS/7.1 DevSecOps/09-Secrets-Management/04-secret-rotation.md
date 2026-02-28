# Chapter 9.4: Secret Rotation

## Overview

Secret rotation is the practice of regularly replacing secrets (passwords, keys, tokens) with new values to limit the window of exposure if a secret is compromised. Automated rotation eliminates human error, ensures compliance, and reduces the blast radius of credential theft. This chapter covers rotation strategies, automation patterns, and zero-downtime techniques.

---

## Why Rotate Secrets?

```
THE RISK OF STATIC SECRETS:

  Day 0          Day 30        Day 90        Day 365
  ┌─────┐       ┌─────┐       ┌─────┐       ┌─────┐
  │ NEW │       │     │       │     │       │     │
  │PASS │       │SAME │       │SAME │       │SAME │
  │WORD │       │PASS │       │PASS │       │PASS │
  └─────┘       └─────┘       └─────┘       └─────┘
  Created    Still valid   Still valid   Still valid!
                             ▲
                             │
                     Compromised here
                     (unknown to team)
                             │
                             ▼
                     Attacker has access
                     for 275+ days!

  WITH ROTATION (every 30 days):

  Day 0     Day 30     Day 60     Day 90
  ┌─────┐  ┌─────┐   ┌─────┐   ┌─────┐
  │Pass1│  │Pass2│   │Pass3│   │Pass4│
  └─────┘  └─────┘   └─────┘   └─────┘
  Created  Rotated   Rotated   Rotated
                      ▲
                      │
               Compromised here
                      │ only valid
                      │ for <30 days
                      ▼
               Automatically
               invalidated!
```

---

## Rotation Strategies

```
ROTATION APPROACHES:

  1. SINGLE-USER ROTATION
  ┌────────────────────────────────────────────┐
  │  Same user, new password                    │
  │  • Simple but has brief downtime risk       │
  │  • Window where old pass invalid,           │
  │    app not yet updated                      │
  └────────────────────────────────────────────┘

  2. DUAL-USER ROTATION (Recommended)
  ┌────────────────────────────────────────────┐
  │  Two users alternate (user_a / user_b)      │
  │                                              │
  │  Phase 1: user_a active, user_b standby     │
  │  Phase 2: Rotate user_b password            │
  │  Phase 3: Switch app to user_b              │
  │  Phase 4: user_b active, user_a standby     │
  │  Phase 5: Rotate user_a password            │
  │                                              │
  │  ✅ Zero downtime — always one valid user   │
  └────────────────────────────────────────────┘

  3. DYNAMIC SECRETS (Best)
  ┌────────────────────────────────────────────┐
  │  Unique credentials per session             │
  │  • Vault generates creds on demand          │
  │  • Auto-expire via TTL (e.g., 1 hour)       │
  │  • No rotation needed — always fresh        │
  │  • Full audit trail per credential          │
  └────────────────────────────────────────────┘
```

---

## Zero-Downtime Rotation Pattern

```
ZERO-DOWNTIME DUAL-USER ROTATION:

  Step 1: Create two DB users
  ┌──────────┐     ┌──────────┐
  │ user_a   │     │ user_b   │
  │ active ✅│     │ standby  │
  └──────────┘     └──────────┘

  Step 2: App uses user_a
  App ──── user_a ────▶ Database

  Step 3: Rotate user_b (safe — not in use)
  ┌──────────┐     ┌──────────┐
  │ user_a   │     │ user_b   │
  │ active ✅│     │ NEW PASS │
  └──────────┘     └──────────┘

  Step 4: Switch app to user_b
  App ──── user_b ────▶ Database

  Step 5: Rotate user_a (safe — not in use)
  ┌──────────┐     ┌──────────┐
  │ user_a   │     │ user_b   │
  │ NEW PASS │     │ active ✅│
  └──────────┘     └──────────┘

  Result: No downtime at any step!
```

---

## AWS Secrets Manager Rotation

```python
# Dual-user rotation Lambda for RDS
import boto3
import json
import pymysql
import secrets
import string

def lambda_handler(event, context):
    step = event['Step']
    secret_id = event['SecretId']
    token = event['ClientRequestToken']
    
    sm_client = boto3.client('secretsmanager')
    
    if step == "createSecret":
        create_secret(sm_client, secret_id, token)
    elif step == "setSecret":
        set_secret(sm_client, secret_id, token)
    elif step == "testSecret":
        test_secret(sm_client, secret_id, token)
    elif step == "finishSecret":
        finish_secret(sm_client, secret_id, token)

def create_secret(client, secret_id, token):
    """Generate new password, store as AWSPENDING."""
    current = client.get_secret_value(
        SecretId=secret_id, VersionStage='AWSCURRENT'
    )
    current_dict = json.loads(current['SecretString'])
    
    # Generate strong password
    alphabet = string.ascii_letters + string.digits + "!@#$%^&*"
    new_password = ''.join(secrets.choice(alphabet) for _ in range(32))
    
    current_dict['password'] = new_password
    
    client.put_secret_value(
        SecretId=secret_id,
        ClientRequestToken=token,
        SecretString=json.dumps(current_dict),
        VersionStages=['AWSPENDING']
    )

def set_secret(client, secret_id, token):
    """Update password in the actual database."""
    pending = client.get_secret_value(
        SecretId=secret_id, VersionStage='AWSPENDING'
    )
    pending_dict = json.loads(pending['SecretString'])
    
    # Use admin credentials to change user password
    conn = pymysql.connect(
        host=pending_dict['host'],
        user='admin',
        password=get_admin_password(),
        database='mysql'
    )
    with conn.cursor() as cursor:
        cursor.execute(
            f"ALTER USER '{pending_dict['username']}'@'%' "
            f"IDENTIFIED BY '{pending_dict['password']}'"
        )
    conn.commit()
    conn.close()

def test_secret(client, secret_id, token):
    """Verify new credentials work."""
    pending = client.get_secret_value(
        SecretId=secret_id, VersionStage='AWSPENDING'
    )
    pending_dict = json.loads(pending['SecretString'])
    
    conn = pymysql.connect(
        host=pending_dict['host'],
        user=pending_dict['username'],
        password=pending_dict['password']
    )
    conn.close()  # Success!

def finish_secret(client, secret_id, token):
    """Promote AWSPENDING to AWSCURRENT."""
    metadata = client.describe_secret(SecretId=secret_id)
    
    for version_id, stages in metadata['VersionIdsToStages'].items():
        if 'AWSCURRENT' in stages and version_id != token:
            client.update_secret_version_stage(
                SecretId=secret_id,
                VersionStage='AWSCURRENT',
                MoveToVersionId=token,
                RemoveFromVersionId=version_id
            )
            break
```

```bash
# Configure rotation schedule
aws secretsmanager rotate-secret \
    --secret-id prod/myapp/database \
    --rotation-lambda-arn arn:aws:lambda:us-east-1:123:function:rotate-db \
    --rotation-rules '{"ScheduleExpression":"rate(30 days)"}'

# Trigger immediate rotation
aws secretsmanager rotate-secret \
    --secret-id prod/myapp/database
```

---

## Vault Dynamic Secret Rotation

```bash
# With Vault dynamic secrets, rotation is built in:

# Configure role with TTL
vault write database/roles/webapp \
    db_name=mydb \
    creation_statements="CREATE USER '{{name}}'@'%' \
      IDENTIFIED BY '{{password}}'; \
      GRANT SELECT, INSERT, UPDATE ON mydb.* TO '{{name}}'@'%';" \
    default_ttl="1h" \
    max_ttl="4h"

# App requests credentials — always fresh
vault read database/creds/webapp
# username: v-webapp-abc123  (unique per request)
# password: A1B2C3-random    (unique per request)
# ttl:      1h               (auto-expires)

# After 1 hour:
# - Vault revokes the lease
# - Database user is dropped
# - App requests new credentials
# No manual rotation needed!
```

---

## Application-Side Secret Refresh

```python
# Python — Application with automatic secret refresh
import boto3
import json
import time
from datetime import datetime, timedelta
from threading import Lock

class SecretCache:
    """Cache secrets with automatic refresh."""

    def __init__(self, secret_id, refresh_interval=300):
        self.secret_id = secret_id
        self.refresh_interval = refresh_interval  # seconds
        self.client = boto3.client('secretsmanager')
        self._secret = None
        self._last_refresh = None
        self._lock = Lock()

    def get_secret(self):
        """Get secret, refreshing if stale."""
        with self._lock:
            now = datetime.utcnow()
            if (self._secret is None or
                self._last_refresh is None or
                (now - self._last_refresh).seconds > self.refresh_interval):
                self._refresh()
            return self._secret

    def _refresh(self):
        """Fetch latest secret from Secrets Manager."""
        response = self.client.get_secret_value(
            SecretId=self.secret_id
        )
        self._secret = json.loads(response['SecretString'])
        self._last_refresh = datetime.utcnow()
        print(f"[{self._last_refresh}] Secret refreshed")

# Usage
db_creds = SecretCache("prod/myapp/database", refresh_interval=300)

def get_db_connection():
    """Get DB connection with latest credentials."""
    creds = db_creds.get_secret()
    return create_connection(
        host=creds['host'],
        user=creds['username'],
        password=creds['password']
    )
```

---

## Certificate Rotation

```bash
# Vault PKI — Automatic certificate rotation
vault secrets enable pki

# Configure CA
vault write pki/root/generate/internal \
    common_name="example.com" \
    ttl=87600h  # 10 years

# Create role for short-lived certs
vault write pki/roles/webserver \
    allowed_domains="example.com" \
    allow_subdomains=true \
    max_ttl=720h   # 30 days

# Issue certificate
vault write pki/issue/webserver \
    common_name="app.example.com" \
    ttl=72h  # 3 days
# Returns: certificate, private_key, ca_chain

# Vault Agent auto-renews certificates:
# vault-agent-config.hcl
template {
  source      = "/etc/vault/templates/cert.tpl"
  destination = "/etc/ssl/app-cert.pem"
  command     = "systemctl reload nginx"
}
```

---

## Rotation Schedule Guidelines

| Secret Type | Rotation Frequency | Method |
|-------------|-------------------|--------|
| **Database passwords** | 30-90 days | Automated (SM/Vault) |
| **API keys** | 90 days | Automated with key pairs |
| **TLS certificates** | 30-90 days (short-lived) | Vault PKI or ACME |
| **SSH keys** | 90 days or per-session | Vault SSH engine |
| **Cloud access keys** | Don't use! | Use IAM roles / OIDC instead |
| **Encryption keys** | Annually (or per policy) | Cloud KMS auto-rotation |
| **OAuth tokens** | Hours (short-lived) | Token refresh flow |
| **Service account keys** | 90 days | Workload identity instead |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| App crashes after rotation | App caches old credentials | Implement secret refresh (polling or event-driven); use connection pooling with retry logic |
| Rotation Lambda fails silently | Lambda timeout or missing permissions | Check CloudWatch Logs; ensure Lambda has `secretsmanager:*` and DB network access |
| Dual-user rotation creates extra DB users | Previous users not cleaned up | Ensure rotation function drops inactive user; audit database user list |
| Certificate rotation causes TLS errors | App doesn't reload certificate | Use file watch (inotify) to trigger reload; or use Vault Agent template with reload command |
| Rotation causes connection pool exhaustion | All connections invalidated at once | Use graceful connection drain; rotate during low-traffic periods; implement gradual rollout |

---

## Summary Table

| Strategy | Downtime Risk | Complexity | Best For |
|----------|--------------|------------|----------|
| **Single-user rotation** | Brief window | Low | Simple apps, non-critical |
| **Dual-user rotation** | Zero downtime | Medium | Database credentials, production |
| **Dynamic secrets (Vault)** | None | Higher setup | High-security, microservices |
| **Short-lived tokens** | None | Low-Medium | API access, OIDC/OAuth |
| **Certificate rotation** | Risk if not auto-reloaded | Medium | TLS, mTLS |

---

## Quick Revision Questions

1. **Why is secret rotation important?**
   > Rotation limits the window of exposure if a secret is compromised. Without rotation, a stolen credential can be used indefinitely. With 30-day rotation, an attacker has at most 30 days before the credential is automatically invalidated. Rotation also satisfies compliance requirements (PCI-DSS, SOC2, HIPAA).

2. **What is dual-user rotation and why is it preferred?**
   > Dual-user rotation uses two database users that alternate as active/standby. When rotating, the standby user's password is changed while the active user remains in use. Then the app switches to the updated standby user. This ensures zero downtime because there's always a valid user available during rotation.

3. **How do dynamic secrets eliminate the need for rotation?**
   > Dynamic secrets (via Vault) generate unique, short-lived credentials for each request. A database user is created on-demand with a TTL (e.g., 1 hour). When the TTL expires, Vault revokes the credentials and drops the database user. Since credentials are never shared and auto-expire, traditional rotation is unnecessary.

4. **How should applications handle secret rotation?**
   > Applications should implement a secret cache with periodic refresh (e.g., every 5 minutes), use connection pools with retry logic that re-fetches credentials on authentication failure, and avoid caching secrets indefinitely. Libraries like AWS SDK caching or Vault Agent sidecar patterns automate this.

5. **What is the recommended rotation frequency for different secret types?**
   > Database passwords: 30-90 days automated. API keys: 90 days with key pair rotation. TLS certificates: 30-90 days via ACME/Vault PKI. Cloud access keys: avoid entirely (use IAM roles). Encryption keys: annually via KMS auto-rotation. OAuth tokens: hours (use refresh tokens).

---

[← Previous: 9.3 Cloud Secrets Managers](03-cloud-secrets-managers.md) | [Next: 9.5 Integration Patterns →](05-integration-patterns.md)

[Back to Table of Contents](../README.md)

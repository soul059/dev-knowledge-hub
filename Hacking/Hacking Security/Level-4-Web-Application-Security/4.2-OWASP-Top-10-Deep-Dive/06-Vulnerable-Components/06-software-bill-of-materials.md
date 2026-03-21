# Unit 6: A06 - Vulnerable and Outdated Components — Topic 6: Software Bill of Materials (SBOM)

## Overview

A Software Bill of Materials (SBOM) is a **formal, machine-readable inventory** of all components, libraries, and dependencies that make up a software application. Just as a food label lists ingredients, an SBOM lists every software component — enabling vulnerability tracking, license compliance, and supply chain transparency. SBOMs are increasingly required by regulations (Executive Order 14028, EU Cyber Resilience Act).

---

## 1. What is an SBOM?

```
┌─────────────────────────────────────────────────────────────────┐
│                SBOM CONTENTS                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  For EACH component:                                           │
│  ├── Component name                                            │
│  ├── Version                                                   │
│  ├── Supplier/Author                                           │
│  ├── Unique identifier (PURL, CPE)                             │
│  ├── Dependency relationship (direct/transitive)               │
│  ├── License                                                   │
│  ├── Hash/checksum (integrity)                                 │
│  └── Timestamp (when SBOM was generated)                       │
│                                                                 │
│  NTIA Minimum Elements:                                        │
│  1. Supplier Name                                              │
│  2. Component Name                                             │
│  3. Version                                                    │
│  4. Unique Identifier                                          │
│  5. Dependency Relationship                                    │
│  6. Author of SBOM Data                                        │
│  7. Timestamp                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. SBOM Formats

### CycloneDX

```json
{
    "bomFormat": "CycloneDX",
    "specVersion": "1.5",
    "serialNumber": "urn:uuid:3e671687-395b-41f5-a30f-a58921a69b79",
    "version": 1,
    "metadata": {
        "timestamp": "2024-01-15T10:00:00Z",
        "tools": [{"name": "trivy", "version": "0.48.0"}],
        "component": {
            "type": "application",
            "name": "my-web-app",
            "version": "2.1.0"
        }
    },
    "components": [
        {
            "type": "library",
            "name": "express",
            "version": "4.18.2",
            "purl": "pkg:npm/express@4.18.2",
            "licenses": [{"license": {"id": "MIT"}}],
            "hashes": [
                {"alg": "SHA-256", "content": "abc123..."}
            ]
        },
        {
            "type": "library",
            "name": "lodash",
            "version": "4.17.21",
            "purl": "pkg:npm/lodash@4.17.21",
            "licenses": [{"license": {"id": "MIT"}}]
        }
    ],
    "dependencies": [
        {
            "ref": "pkg:npm/express@4.18.2",
            "dependsOn": [
                "pkg:npm/body-parser@1.20.1",
                "pkg:npm/cookie@0.5.0"
            ]
        }
    ]
}
```

### SPDX (Software Package Data Exchange)

```json
{
    "spdxVersion": "SPDX-2.3",
    "dataLicense": "CC0-1.0",
    "SPDXID": "SPDXRef-DOCUMENT",
    "name": "my-web-app-sbom",
    "documentNamespace": "https://example.com/sbom/my-web-app",
    "creationInfo": {
        "created": "2024-01-15T10:00:00Z",
        "creators": ["Tool: trivy-0.48.0"]
    },
    "packages": [
        {
            "SPDXID": "SPDXRef-Package-express",
            "name": "express",
            "versionInfo": "4.18.2",
            "downloadLocation": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
            "licenseConcluded": "MIT",
            "externalRefs": [
                {
                    "referenceCategory": "PACKAGE-MANAGER",
                    "referenceType": "purl",
                    "referenceLocator": "pkg:npm/express@4.18.2"
                }
            ]
        }
    ],
    "relationships": [
        {
            "spdxElementId": "SPDXRef-DOCUMENT",
            "relationshipType": "DESCRIBES",
            "relatedSpdxElement": "SPDXRef-Package-my-web-app"
        }
    ]
}
```

### Format Comparison

| Feature | CycloneDX | SPDX |
|---------|-----------|------|
| Focus | Security and risk | License compliance |
| Maintained by | OWASP | Linux Foundation |
| Formats | JSON, XML | JSON, XML, RDF, Tag-Value |
| Vulnerability data | ✅ VEX support | ✅ (via extensions) |
| Dependency graph | ✅ | ✅ |
| Best for | Security operations | Legal/compliance |

---

## 3. Generating SBOMs

```bash
# ═══════════════════════════════════════
# Trivy (recommended — multi-format)
# ═══════════════════════════════════════
# CycloneDX format
trivy fs --format cyclonedx --output sbom-cyclonedx.json .

# SPDX format
trivy fs --format spdx-json --output sbom-spdx.json .

# For container images
trivy image --format cyclonedx --output image-sbom.json myapp:latest

# ═══════════════════════════════════════
# Syft (Anchore — dedicated SBOM tool)
# ═══════════════════════════════════════
# Install
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh

# Generate from directory
syft dir:. -o cyclonedx-json > sbom.json
syft dir:. -o spdx-json > sbom-spdx.json

# Generate from container
syft myapp:latest -o cyclonedx-json

# ═══════════════════════════════════════
# Snyk
# ═══════════════════════════════════════
snyk sbom --format cyclonedx1.4+json > sbom.json

# ═══════════════════════════════════════
# Microsoft sbom-tool
# ═══════════════════════════════════════
sbom-tool generate -b . -bc . -pn MyApp -pv 1.0.0 -nsb https://example.com
```

---

## 4. SBOM in CI/CD Pipeline

```yaml
# GitHub Actions — Generate and publish SBOM
name: SBOM Generation
on:
  release:
    types: [published]

jobs:
  sbom:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate SBOM with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          format: 'cyclonedx'
          output: 'sbom.json'

      - name: Scan SBOM for vulnerabilities
        run: |
          trivy sbom sbom.json --severity HIGH,CRITICAL

      - name: Upload SBOM as release asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ github.event.release.upload_url }}
          asset_path: ./sbom.json
          asset_name: sbom-${{ github.event.release.tag_name }}.json
          asset_content_type: application/json

      - name: Upload to Dependency Track
        run: |
          curl -X POST \
            -H "X-Api-Key: ${{ secrets.DTRACK_API_KEY }}" \
            -F "project=${{ env.PROJECT_UUID }}" \
            -F "bom=@sbom.json" \
            "https://dtrack.example.com/api/v1/bom"
```

---

## 5. SBOM Management with Dependency-Track

```
Dependency-Track (OWASP):
- Open source SBOM management platform
- Ingests CycloneDX and SPDX SBOMs
- Continuous vulnerability monitoring
- Policy engine for compliance
- Dashboard for risk visibility

Workflow:
1. CI/CD generates SBOM on every build
2. SBOM uploaded to Dependency-Track
3. Dependency-Track maps components to vulnerabilities
4. Alerts sent when new CVEs affect your components
5. Dashboard shows risk across all projects
```

---

## 6. Regulatory Requirements

```
┌─────────────────────────────────────────────────────────────────┐
│          SBOM REGULATORY LANDSCAPE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  US Executive Order 14028 (May 2021):                          │
│  → Software sold to federal government MUST include SBOM       │
│  → Must use NTIA minimum elements                              │
│  → Applies to all federal software procurement                 │
│                                                                 │
│  EU Cyber Resilience Act (2024):                               │
│  → Products with digital elements must have SBOMs              │
│  → Manufacturers must document all components                  │
│  → Vulnerability handling process required                     │
│                                                                 │
│  FDA Cybersecurity Guidance (2023):                            │
│  → Medical devices must include SBOM                           │
│  → SBOM must be machine-readable                               │
│  → Must include all commercial/open-source/off-the-shelf       │
│                                                                 │
│  NIST SP 800-218 (SSDF):                                      │
│  → Secure Software Development Framework                      │
│  → Requires maintaining component inventory                   │
│  → SBOM is the recommended implementation                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | CycloneDX | SPDX | SWID |
|--------|-----------|------|------|
| Format | JSON/XML | JSON/XML/RDF | XML |
| Focus | Security | Legal/License | Installed software |
| OWASP endorsed | ✅ | - | - |
| ISO standard | - | ✅ ISO 5962 | ✅ ISO 19770-2 |
| Best tool | Trivy, Syft | Syft, SPDX tools | Microsoft |

---

## Revision Questions

1. What is an SBOM and why is it important for software supply chain security?
2. Compare CycloneDX and SPDX formats — their focus, structure, and use cases.
3. Generate an SBOM for a Node.js project using Trivy and explain the output fields.
4. Set up a CI/CD pipeline that generates and publishes SBOMs on every release.
5. What regulatory requirements mandate SBOMs? Which standards must they follow?
6. How does OWASP Dependency-Track use SBOMs for continuous vulnerability monitoring?

---

*Previous: [05-update-strategies.md](05-update-strategies.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

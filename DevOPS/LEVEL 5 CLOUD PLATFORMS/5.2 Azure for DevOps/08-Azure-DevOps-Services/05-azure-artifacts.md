# Chapter 5: Azure Artifacts

## Overview
**Azure Artifacts** is a package management service in Azure DevOps. It hosts NuGet, npm, Maven, Python (PyPI), Cargo, and Universal Packages. Teams can publish, consume, and share packages across projects and organisations with upstream sources and feed permissions.

---

## 5.1 Azure Artifacts Architecture

```
┌──────────── AZURE ARTIFACTS ──────────────────┐
│                                                 │
│  Azure DevOps Organization                      │
│  └── Project: MyApp                             │
│      └── Feed: "my-packages"                    │
│          │                                      │
│          ├── NuGet:                              │
│          │   ├── MyApp.Common 1.2.0             │
│          │   └── MyApp.Auth 2.0.1               │
│          │                                      │
│          ├── npm:                                │
│          │   ├── @myapp/ui-components 3.1.0     │
│          │   └── @myapp/utils 1.0.5             │
│          │                                      │
│          ├── Python (PyPI):                      │
│          │   └── myapp-sdk 1.0.0                │
│          │                                      │
│          └── Universal:                          │
│              └── deployment-scripts 2.0.0       │
│                                                 │
│  Upstream Sources:                               │
│  ├── nuget.org                                   │
│  ├── npmjs.com                                   │
│  └── pypi.org                                    │
│  (Cached locally on first use)                   │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## 5.2 Feed Types and Scopes

| Feed Scope | Visibility | Use Case |
|-----------|------------|----------|
| **Project-scoped** | Visible within the project | Team-specific packages |
| **Organisation-scoped** | Visible across all projects | Shared libraries |

```
┌────── UPSTREAM FLOW ──────────────────────┐
│                                            │
│  Developer                                 │
│     │ npm install lodash                   │
│     ▼                                      │
│  Azure Artifacts Feed                      │
│     │ lodash found in cache? ──► Yes ──►   │
│     │                            Return    │
│     ▼ No                                   │
│  Upstream: npmjs.com                       │
│     │ Download lodash                      │
│     ▼                                      │
│  Cache in Feed (future installs faster)    │
│     │                                      │
│     ▼                                      │
│  Return to Developer                       │
│                                            │
└────────────────────────────────────────────┘
```

---

## 5.3 Publishing Packages

### NuGet (.NET)

```bash
# Create NuGet package
dotnet pack --configuration Release --output ./nupkgs

# Add Azure Artifacts source
dotnet nuget add source \
  "https://pkgs.dev.azure.com/myorg/MyApp/_packaging/my-packages/nuget/v3/index.json" \
  --name "AzureArtifacts" \
  --username "az" \
  --password "<PAT>"

# Publish package
dotnet nuget push ./nupkgs/MyApp.Common.1.2.0.nupkg \
  --source "AzureArtifacts" \
  --api-key "az"
```

### npm (Node.js)

```bash
# Create .npmrc file in project root
echo "@myapp:registry=https://pkgs.dev.azure.com/myorg/MyApp/_packaging/my-packages/npm/registry/" > .npmrc
echo "//pkgs.dev.azure.com/myorg/MyApp/_packaging/my-packages/npm/registry/:_authToken=<PAT>" >> .npmrc

# Publish package
npm publish
```

### Python (PyPI)

```bash
# Build package
python -m build

# Upload to Azure Artifacts
twine upload \
  --repository-url https://pkgs.dev.azure.com/myorg/MyApp/_packaging/my-packages/pypi/upload/ \
  -u "az" \
  -p "<PAT>" \
  dist/*
```

---

## 5.4 Using Packages in Pipelines

```yaml
# azure-pipelines.yml — consuming packages
steps:
  # Authenticate with Azure Artifacts feed (NuGet)
  - task: NuGetAuthenticate@1

  - task: DotNetCoreCLI@2
    inputs:
      command: 'restore'
      feedsToUse: 'select'
      vstsFeed: 'my-packages'

  # Publish package to feed
  - task: DotNetCoreCLI@2
    inputs:
      command: 'push'
      packagesToPush: '$(Build.ArtifactStagingDirectory)/*.nupkg'
      nuGetFeedType: 'internal'
      publishVstsFeed: 'my-packages'
```

```yaml
# npm authentication in pipeline
steps:
  - task: npmAuthenticate@0
    inputs:
      workingFile: '.npmrc'

  - script: npm install
    displayName: 'Install dependencies'

  - script: npm publish
    displayName: 'Publish to feed'
```

---

## 5.5 Views and Versioning

```
┌────── FEED VIEWS (PROMOTION) ─────────────┐
│                                            │
│  @Local (default) ──► All published pkgs   │
│       │                                    │
│       ▼ Promote                            │
│  @Prerelease ──────► Tested packages       │
│       │                                    │
│       ▼ Promote                            │
│  @Release ─────────► Production-ready      │
│                                            │
│  Consumers can point to a specific view:   │
│  Feed URL + @Release = only stable pkgs    │
│                                            │
└────────────────────────────────────────────┘
```

### Semantic Versioning

| Version | Meaning |
|---------|---------|
| `1.0.0` | Initial release |
| `1.0.1` | Patch (bug fix) |
| `1.1.0` | Minor (new feature, backward compatible) |
| `2.0.0` | Major (breaking change) |
| `1.0.0-beta.1` | Pre-release |

---

## 5.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | PAT expired or wrong scope | Regenerate PAT with Packaging (Read/Write) scope |
| Package not found | Wrong feed URL or scope | Verify feed URL and project scope |
| Version conflict | Package version already exists | Bump version before publishing |
| Upstream not caching | Upstream source disabled on feed | Enable upstream sources in feed settings |
| Pipeline can't restore | Feed not authorized for pipeline | Use `NuGetAuthenticate@1` or `npmAuthenticate@0` |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Azure Artifacts** | Host NuGet, npm, Maven, PyPI, Universal packages |
| **Feeds** | Project-scoped or Organisation-scoped package stores |
| **Upstream Sources** | Proxy public registries (nuget.org, npmjs) with caching |
| **Views** | @Local → @Prerelease → @Release for package promotion |
| **Pipeline Auth** | Use `NuGetAuthenticate` / `npmAuthenticate` tasks |
| **Versioning** | Semantic Versioning (SemVer): Major.Minor.Patch |

---

## Quick Revision Questions

1. **What is Azure Artifacts?**
   > A package management service in Azure DevOps that hosts NuGet, npm, Maven, PyPI, and Universal packages with feed-based access control.

2. **What are upstream sources?**
   > Upstream sources proxy public registries (like npmjs.com or nuget.org). When a package is requested, it's fetched from upstream and cached in the feed.

3. **What are feed views?**
   > Views (@Local, @Prerelease, @Release) allow package promotion from development to production, so consumers only see stable versions.

4. **How do you authenticate a pipeline with Azure Artifacts?**
   > Use the `NuGetAuthenticate@1` task (for NuGet) or `npmAuthenticate@0` task (for npm) before restore/install steps.

5. **What is semantic versioning?**
   > A versioning convention: MAJOR.MINOR.PATCH. Major = breaking changes, Minor = new features (backward compatible), Patch = bug fixes.

---

[⬅ Previous: Azure Test Plans](04-azure-test-plans.md) | [⬆ Back to Table of Contents](../README.md) | [Next: YAML Pipelines In-Depth ➡](06-yaml-pipelines.md)

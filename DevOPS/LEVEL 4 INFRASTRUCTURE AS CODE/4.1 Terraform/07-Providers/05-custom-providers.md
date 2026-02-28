# Chapter 5: Custom Providers

[← Previous: Provider Aliases](04-provider-aliases.md) | [Back to README](../README.md) | [Next: Unit 8 - Provisioners →](../08-Advanced-Features/01-provisioners.md)

---

## Overview

When no existing provider supports your target API, you can build a **custom Terraform provider**. Custom providers use the same plugin architecture as official providers and are written in Go using the Terraform Plugin Framework or SDK.

---

## 5.1 When to Build a Custom Provider

```
┌──────────────────────────────────────────────────────────┐
│        WHEN TO BUILD A CUSTOM PROVIDER                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ✓ BUILD when:                                           │
│  ├── Internal API with no existing provider              │
│  ├── Custom service/platform to manage                   │
│  ├── Wrapping a REST API for Terraform                   │
│  └── Need custom CRUD operations                         │
│                                                           │
│  ✗ DON'T BUILD when:                                     │
│  ├── An official/partner/community provider exists       │
│  ├── Can use the HTTP provider or external data source   │
│  ├── A shell script via provisioner would suffice        │
│  └── terraform_data resource covers the use case         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 Provider Architecture

```
┌──────────────────────────────────────────────────────────┐
│           CUSTOM PROVIDER ARCHITECTURE                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Terraform Core ◄────► Provider Plugin (Go binary)       │
│                  gRPC                                    │
│                        ┌──────────────────────────┐      │
│                        │  Provider                 │      │
│                        │  ├── Configure()          │      │
│                        │  │   └── Auth, base URL   │      │
│                        │  │                        │      │
│                        │  ├── Resources            │      │
│                        │  │   ├── Create()         │      │
│                        │  │   ├── Read()           │      │
│                        │  │   ├── Update()         │      │
│                        │  │   └── Delete()         │      │
│                        │  │                        │      │
│                        │  └── Data Sources         │      │
│                        │      └── Read()           │      │
│                        └──────────────────────────┘      │
│                                   │                      │
│                                   ▼                      │
│                         ┌──────────────────┐             │
│                         │   Your API       │             │
│                         │   (REST, gRPC)   │             │
│                         └──────────────────┘             │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 5.3 Plugin Framework vs SDK

```
┌──────────────────────────────────────────────────────────────┐
│     TERRAFORM PLUGIN FRAMEWORK vs PLUGIN SDK                  │
├─────────────────────────┬────────────────────────────────────┤
│ Plugin Framework (v2)   │ Plugin SDK (v2) [Legacy]           │
├─────────────────────────┼────────────────────────────────────┤
│ Recommended for new     │ Used by older providers            │
│ providers               │                                    │
│                         │                                    │
│ Type-safe schemas       │ Schema using map[string]*Schema    │
│                         │                                    │
│ Better error handling   │ diag.Diagnostics                   │
│                         │                                    │
│ Import: terraform-      │ Import: terraform-plugin-sdk       │
│ plugin-framework        │                                    │
│                         │                                    │
│ Uses context.Context    │ Uses *schema.ResourceData          │
│                         │                                    │
│ Active development      │ Maintenance mode                   │
└─────────────────────────┴────────────────────────────────────┘
```

> **Recommendation:** Use the **Plugin Framework** for all new providers.

---

## 5.4 Building a Provider (Overview)

### Project Structure

```
terraform-provider-myapi/
├── main.go                     ← Entry point
├── go.mod
├── go.sum
├── internal/
│   └── provider/
│       ├── provider.go         ← Provider definition
│       ├── resource_item.go    ← Resource: myapi_item
│       ├── data_source_item.go ← Data source: myapi_item
│       └── models.go           ← Data models
├── examples/
│   └── main.tf                 ← Usage examples
├── docs/                       ← Documentation
└── Makefile
```

### Minimal Provider Code

```go
// ─── main.go ─────────────────────────────────────
package main

import (
    "context"
    "flag"
    "log"

    "github.com/hashicorp/terraform-plugin-framework/providerserver"
    "terraform-provider-myapi/internal/provider"
)

func main() {
    var debug bool
    flag.BoolVar(&debug, "debug", false, "enable debug mode")
    flag.Parse()

    opts := providerserver.ServeOpts{
        Address: "registry.terraform.io/myorg/myapi",
        Debug:   debug,
    }

    err := providerserver.Serve(context.Background(), provider.New, opts)
    if err != nil {
        log.Fatal(err.Error())
    }
}

// ─── internal/provider/provider.go ───────────────
package provider

import (
    "context"

    "github.com/hashicorp/terraform-plugin-framework/datasource"
    "github.com/hashicorp/terraform-plugin-framework/provider"
    "github.com/hashicorp/terraform-plugin-framework/provider/schema"
    "github.com/hashicorp/terraform-plugin-framework/resource"
)

type myapiProvider struct {
    version string
}

type myapiProviderModel struct {
    Endpoint types.String `tfsdk:"endpoint"`
    ApiKey   types.String `tfsdk:"api_key"`
}

func New(version string) func() provider.Provider {
    return func() provider.Provider {
        return &myapiProvider{version: version}
    }
}

func (p *myapiProvider) Schema(_ context.Context,
    _ provider.SchemaRequest, resp *provider.SchemaResponse) {
    resp.Schema = schema.Schema{
        Attributes: map[string]schema.Attribute{
            "endpoint": schema.StringAttribute{Required: true},
            "api_key":  schema.StringAttribute{Required: true, Sensitive: true},
        },
    }
}

func (p *myapiProvider) Resources(_ context.Context) []func() resource.Resource {
    return []func() resource.Resource{
        NewItemResource,
    }
}

func (p *myapiProvider) DataSources(_ context.Context) []func() datasource.DataSource {
    return []func() datasource.DataSource{
        NewItemDataSource,
    }
}
```

---

## 5.5 Alternatives to Custom Providers

```hcl
# ─── Option 1: HTTP Provider (simple REST calls) ─
terraform {
  required_providers {
    http = {
      source = "hashicorp/http"
    }
  }
}

data "http" "api_status" {
  url = "https://api.example.com/status"

  request_headers = {
    Authorization = "Bearer ${var.api_token}"
  }
}

output "api_status" {
  value = jsondecode(data.http.api_status.response_body)
}

# ─── Option 2: External Data Source ───────────────
data "external" "my_data" {
  program = ["python3", "${path.module}/scripts/get_data.py"]

  query = {
    environment = var.environment
  }
}

# Returns JSON: {"result": "value"}
output "data" {
  value = data.external.my_data.result
}

# ─── Option 3: terraform_data with provisioners ──
resource "terraform_data" "api_call" {
  triggers_replace = [var.config_version]

  provisioner "local-exec" {
    command = <<-EOT
      curl -X POST https://api.example.com/deploy \
        -H "Authorization: Bearer ${var.api_token}" \
        -d '{"version": "${var.app_version}"}'
    EOT
  }
}

# ─── Option 4: Shell Provider (community) ────────
terraform {
  required_providers {
    shell = {
      source  = "scottwinkler/shell"
      version = "~> 1.7"
    }
  }
}

resource "shell_script" "api_resource" {
  lifecycle_commands {
    create = "curl -X POST https://api.example.com/items -d '{\"name\": \"${var.name}\"}'"
    read   = "curl https://api.example.com/items/${self.output.id}"
    update = "curl -X PUT https://api.example.com/items/${self.output.id} -d '{\"name\": \"${var.name}\"}'"
    delete = "curl -X DELETE https://api.example.com/items/${self.output.id}"
  }

  environment = {
    API_TOKEN = var.api_token
  }
}
```

---

## 5.6 Publishing a Custom Provider

```
┌──────────────────────────────────────────────────────────┐
│       PUBLISHING TO TERRAFORM REGISTRY                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  1. Name repo: terraform-provider-<NAME>                 │
│  2. Create GitHub release with Go binaries               │
│  3. Sign with GPG key                                    │
│  4. Register at registry.terraform.io                    │
│  5. Users install via required_providers                  │
│                                                           │
│  For private use:                                         │
│  ├── Private Terraform Registry                          │
│  ├── Network mirror                                      │
│  ├── Filesystem mirror                                   │
│  └── dev_overrides in .terraformrc                       │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── Local development: .terraformrc ──────────────
provider_installation {
  dev_overrides {
    "myorg/myapi" = "/home/user/go/bin"
  }
  direct {}
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `provider not found` locally | Binary not in PATH | Set `dev_overrides` in `.terraformrc` |
| Plugin crash | Nil pointer in CRUD | Add null checks; use debug mode |
| Schema mismatch | State vs resource schema differ | Implement state migration |
| `external` data source errors | Script doesn't output JSON | Ensure script outputs valid JSON to stdout |
| `http` provider limitations | Can't do CRUD operations | Use shell provider or build custom provider |

---

## Summary Table

| Approach | Complexity | CRUD Support | Best For |
|----------|------------|-------------|----------|
| **HTTP data source** | Low | Read only | Simple API queries |
| **External data source** | Low | Read only | Script-based data lookup |
| **terraform_data** | Low | Create/Replace | One-off API calls |
| **Shell provider** | Medium | Full CRUD | Quick REST API wrapping |
| **Custom provider (Go)** | High | Full CRUD | Production-grade integration |

---

## Quick Revision Questions

1. **When should you build a custom Terraform provider?**
   > When managing an internal API or service with no existing provider, and you need full CRUD lifecycle management through Terraform.

2. **What language are Terraform providers written in?**
   > Go. They compile to a standalone binary that communicates with Terraform Core via gRPC.

3. **What is the Terraform Plugin Framework?**
   > The recommended library for building new providers. It provides type-safe schemas, better error handling, and uses `context.Context` patterns.

4. **What alternatives exist to building a custom provider?**
   > HTTP data source (read-only), external data source (script-based), terraform_data with provisioners, or the community shell provider.

5. **How do you use a custom provider during development?**
   > Add `dev_overrides` in `.terraformrc` pointing to the directory containing the compiled provider binary.

---

[← Previous: Provider Aliases](04-provider-aliases.md) | [Back to README](../README.md) | [Next: Unit 8 - Provisioners →](../08-Advanced-Features/01-provisioners.md)

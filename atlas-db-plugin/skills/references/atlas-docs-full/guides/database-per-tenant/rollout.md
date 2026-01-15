Staged Rollout Strategies for Multi-Tenant Schema Migrations | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

In the previous sections, we learned how to define target groups and deploy migrations to multiple tenant databases. In this section, we will explore **deployment rollout strategies** - a powerful feature that gives you fine-grained control over how migrations are applied across your tenant databases.

## Overview[​](#overview "Direct link to Overview")


When deploying schema migrations to multiple tenant databases, you often need more control than simply applying migrations to all targets at once. Common requirements include:

*   **Canary deployments** - Validate changes on a small subset of tenants before a broader rollout
*   **Regional rollouts** - Deploy to US-West first, then US-East, then EU, minimizing blast radius
*   **Priority ordering** - Migrate enterprise customers before SMB, or process tenants alphabetically
*   **Parallel execution** - Speed up deployments by running migrations concurrently within each stage
*   **Error resilience** - Log failures and continue with remaining tenants instead of stopping entirely

Atlas's `deployment` block addresses all these needs by organizing targets into **groups** with configurable execution order, parallelism, and error handling.

## The `deployment` Block[​](#the-deployment-block "Direct link to the-deployment-block")


The `deployment` block defines a rollout strategy that can be referenced by one or more environments.

### Basic Syntax[​](#basic-syntax "Direct link to Basic Syntax")

```codeBlockLines_AdAo
deployment "<name>" {  // Variables passed from the env block  variable "<var_name>" {    type    = <type>       // string, bool, number, etc.    default = <value>      // Optional default value  }  // Groups define execution stages  group "<group_name>" {    match      = <expr>                  // Boolean expression to filter targets    order_by   = <expr>                  // Expression to sort targets within group    parallel   = <number>                // Max concurrent executions (default: 1)    on_error   = FAIL | CONTINUE         // Error handling mode    depends_on = [group.<other_group>]   // Groups that must complete first  }}
```

### Connecting to an Environment[​](#connecting-to-an-environment "Direct link to Connecting to an Environment")


To use a deployment strategy, reference it in your `env` block using the `rollout` block:
```codeBlockLines_AdAo
env "prod" {  for_each = toset(var.tenants)  url      = urlsetpath(var.url, each.value)  rollout {    deployment = deployment.staged    vars = {      name = each.value    }  }}
```

## Group Matching Behavior[​](#group-matching-behavior "Direct link to Group Matching Behavior")


By default, groups are evaluated in the order they appear in the configuration file. When a target matches multiple groups, the **first matching group wins** - the target is assigned to it and skipped by subsequent groups. You can override the execution order using the `depends_on` attribute.

This allows you to define specific groups first (e.g., canary, internal) followed by a catch-all group for remaining targets:
```codeBlockLines_AdAo
deployment "staged" {  variable "name" {    type = string  }  // First: Internal tenants (matched first by position)  group "internal" {    match = startswith(var.name, "internal-")  }  // Second: Canary tenants  group "canary" {    match      = startswith(var.name, "canary-")    parallel   = 10    depends_on = [group.internal]  }  // Last: Catch-all for remaining targets (no match = all unmatched)  group "rest" {    depends_on = [group.canary]  }}
```

## Group Attributes[​](#group-attributes "Direct link to Group Attributes")


### `match`[​](#match "Direct link to match")


A boolean expression that determines which targets belong to this group. Targets matching multiple groups are assigned to the first matching group by file position.
```codeBlockLines_AdAo
group "internal" {  match = startswith(var.name, "my-company-") || var.name == "internal-test"}
```

### `order_by`[​](#order_by "Direct link to order_by")


Controls the execution order of targets within a group. Targets are sorted by this expression in ascending order. You can use a single expression or an array for multi-level sorting.
```codeBlockLines_AdAo
group "alphabetical" {  match    = var.tier == "FREE"  order_by = var.name  // Execute tenants alphabetically}group "by_region_then_name" {  order_by = [var.region, var.name]  // Sort by region first, then by name}
```

### `parallel`[​](#parallel "Direct link to parallel")


Maximum number of concurrent migrations within the group. Default is `1` (sequential execution).
```codeBlockLines_AdAo
group "free_tier" {  match    = var.tier == "FREE"  parallel = 10  // Run up to 10 migrations concurrently}
```

### `on_error`[​](#on_error "Direct link to on_error")


Defines behavior when a migration fails within the group:

*   `FAIL` (default) - Stop the group's execution immediately on first error
*   `CONTINUE` - Log the error and proceed with remaining targets in the group
```codeBlockLines_AdAo
group "non_critical" {  match    = var.tier == "FREE"  on_error = CONTINUE  // Log failures but continue with other tenants}
```

## Practical Examples[​](#practical-examples "Direct link to Practical Examples")


### Canary Deployment Pattern[​](#canary-deployment-pattern "Direct link to Canary Deployment Pattern")


Deploy to a single canary tenant first, then roll out to everyone else:
```codeBlockLines_AdAo
deployment "canary" {  variable "name" {    type = string  }  group "canary" {    match = var.name == "canary-tenant"  }  group "rest" {    parallel   = 5    depends_on = [group.canary]  }}
```

### Tiered Rollout by Customer Plan[​](#tiered-rollout-by-customer-plan "Direct link to Tiered Rollout by Customer Plan")


Roll out to internal tenants first, then free tier (with high parallelism), then paid customers (more carefully):
```codeBlockLines_AdAo
data "sql" "tenants" {  url   = var.management_url  query = <<SQL    SELECT name, tier    FROM tenants  SQL}env "prod" {  for_each = toset(data.sql.tenants.values)  url      = urlsetpath(var.url, each.value.name)  migration {    dir = "atlas://my-app"  }  rollout {    deployment = deployment.tiered    vars = {      name = each.value.name      tier = each.value.tier    }  }}deployment "tiered" {  variable "name" {    type = string  }  variable "tier" {    type = string  }  // Internal tenants first - one at a time  group "internal" {    match = startswith(var.name, "my-company-")  }  // Free tier next - parallelize aggressively.  group "free" {    match      = var.tier == "FREE"    parallel   = 10    on_error   = CONTINUE    depends_on = [group.internal]  }  // Paid customers last - more conservative  group "paid" {    parallel   = 3    on_error   = FAIL    depends_on = [group.free]  }}
```
*   [Overview](#overview)
*   [The `deployment` Block](#the-deployment-block)
    *   [Basic Syntax](#basic-syntax)
    *   [Connecting to an Environment](#connecting-to-an-environment)
*   [Group Matching Behavior](#group-matching-behavior)
*   [Group Attributes](#group-attributes)
    *   [`match`](#match)
    *   [`order_by`](#order_by)
    *   [`parallel`](#parallel)
    *   [`on_error`](#on_error)
*   [Practical Examples](#practical-examples)
    *   [Canary Deployment Pattern](#canary-deployment-pattern)
    *   [Tiered Rollout by Customer Plan](#tiered-rollout-by-customer-plan)
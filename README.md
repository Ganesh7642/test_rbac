# Love's Development — Snowflake Provision Toolkit

Infrastructure-as-code project for managing Snowflake resources in the **Love's Development** account using the [phData Toolkit CLI](https://toolkit.phdata.io) Provision tool (v0.90.0).

## What This Project Provisions

| Group File | Resources |
|------------|-----------|
| `service-accounts` | Service account `SVC_DBT_PROVISION_APP` (SERVICE type, default warehouse: `DBT_PROVISION_APP_WH`, default role: `FR_DBT_PROVISION_APP`) |
| `functional-role` | Functional role `FR_PROVISION_APP` (owned by SECURITYADMIN) |
| `object_roles` | Object roles: `OBJ_DB_DEVELOPMENT_USE`, `OBJ_DB_DEVELOPMENT_SELECT` (owned by SECURITYADMIN) |
| `functional-role-membership` | Grants OBJ roles into `FR_PROVISION_APP`; USAGE + OPERATE on `PROVISION_APP_WH` (each privilege independently destroy-able) |
| `warehouse-membership` | Grants USAGE on `PROVISION_APP_WH` to `FR_PROVISION_APP` |
| `resource-monitor` | `DBT_PROVISION_APP_WH_MONITOR` — monthly, notify at 75%, suspend at 100% |
| `warehouse` | `DBT_PROVISION_APP_WH` — XSMALL, Standard Gen2, initially suspended |
| `functional-role-user-membership` | Assigns `FR_DBT_PROVISION_APP` to `SVC_DBT_PROVISION_APP` |

## Prerequisites

- Java 17+
- [phData Toolkit CLI](https://toolkit.phdata.io) v0.90.0
- Snowflake key-pair authentication configured
- Environment variables set (see below)

## Environment Variables

| Variable | Description |
|----------|-------------|
| `SNOWFLAKE_USER` | Snowflake provisioning user |
| `SNOWFLAKE_ROLE` | Role used for provisioning |
| `SNOWFLAKE_WAREHOUSE` | Warehouse for provisioning operations |
| `SNOWFLAKE_PRIVATE_KEY_FILE` | Path to RSA private key (.p8) |
| `SNOWFLAKE_PRIVATE_KEY_PASSPHRASE` | Private key passphrase |

## Quick Start

```bash
# Install the Toolkit CLI
export TRAM_ACCOUNT_TOKEN=<your-token>
./bin/install-toolkit-cli

# Validate stack offline
toolkit provision apply --local

# Preview changes (read-only, connects to Snowflake)
toolkit provision apply --plan

# Apply changes
toolkit provision apply

# Apply without confirmation
toolkit provision apply --approve
```

## CI/CD

A GitHub Actions workflow (`.github/workflows/pipeline.yaml`) runs a **dry-run** (`--plan`) on every push and PR to `main`. It:

1. Checks out the repo
2. Installs Java 17 and the Toolkit CLI
3. Reconstructs the Snowflake private key from a base64-encoded secret
4. Runs `toolkit provision apply --plan`

## Configuration

The Snowflake connection and provisioning settings are defined in `toolkit.conf` (HOCON format):

- **Account**: `LOVES-LOVES_DEVELOPMENT`
- **Metadata database**: `SFADMIN_PROVISION_DB.provision`
- **Threads**: 16
- **Max destroys**: 200
- **User destroy type**: DISABLE (users are disabled, not dropped)

### Owner Roles

| Object Type | Owner Role |
|-------------|------------|
| Roles, Users, Network Policies | SECURITYADMIN |
| Tags, Network Rules, Resource Monitors | ACCOUNTADMIN |
| Databases, Warehouses, Schemas (default) | SYSADMIN |

## Adding New Resources

1. Create or update a **group file** in `stack/groups/` with new entries
2. Create or update the corresponding **template** in `stack/templates/`
3. Run `toolkit provision apply --local` to validate
4. Run `toolkit provision apply --plan` to preview
5. Run `toolkit provision apply` to apply

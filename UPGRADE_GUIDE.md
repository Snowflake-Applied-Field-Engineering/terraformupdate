# Snowflake Terraform Provider Upgrade Guide

This guide helps you upgrade your existing Snowflake Terraform configurations from older provider versions to the latest version. This is especially important if you're using provider versions older than v0.90.0.

## Table of Contents

- [Overview](#overview)
- [Before You Begin](#before-you-begin)
- [Version Compatibility](#version-compatibility)
- [Breaking Changes by Version](#breaking-changes-by-version)
- [Step-by-Step Upgrade Process](#step-by-step-upgrade-process)
- [Common Migration Scenarios](#common-migration-scenarios)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

## Overview

The Snowflake Terraform provider has undergone significant changes to align with Snowflake's best practices and improve resource management. The most significant changes are documented in the [Snowflake BCR Migration Guide](https://github.com/snowflakedb/terraform-provider-snowflake/blob/main/SNOWFLAKE_BCR_MIGRATION_GUIDE.md#bundle-2025_04).

### Key Changes in Recent Versions

- **Bundle 2025_04**: Major breaking changes to resource schemas and behavior
- **v0.90.0+**: Introduction of new grant resources and deprecation of legacy grant patterns
- **v0.80.0+**: Changes to warehouse, database, and schema resource attributes
- **v0.70.0+**: Updates to role and user management

## Before You Begin

### Prerequisites

1. **Backup Your State File**
   ```bash
   # Local state
   cp terraform.tfstate terraform.tfstate.backup
   
   # Remote state (S3 example)
   aws s3 cp s3://your-bucket/path/terraform.tfstate ./terraform.tfstate.backup
   ```

2. **Review Current Configuration**
   ```bash
   terraform state list
   terraform show
   ```

3. **Check Current Provider Version**
   ```bash
   terraform version
   grep -A 5 "required_providers" *.tf
   ```

4. **Create a Test Environment**
   - Test the upgrade in a non-production environment first
   - Use a separate workspace or state file

### Important Warnings

- **Do not skip major versions** - Upgrade incrementally (e.g., 0.70 → 0.80 → 0.90 → 0.94)
- **Review deprecation warnings** - Run `terraform plan` after each version upgrade
- **Test thoroughly** - Validate changes in dev/staging before production
- **Communicate with your team** - Coordinate upgrades to avoid conflicts

## Version Compatibility

| Provider Version | Terraform Version | Snowflake Compatibility | Status | Notes |
|-----------------|-------------------|------------------------|---------|-------|
| v2.3.x+ | >= 1.5.0 | All current versions | Current | BCR bundle fixes |
| v2.0.x - v2.2.x | >= 1.5.0 | All current versions | Supported | Update to 2.3+ for BCR |
| v1.0.x - v1.9.x | >= 1.4.0 | All current versions | Supported | Upgrade to 2.x |
| v0.94.x | >= 1.5.0 | All current versions | Deprecated | Legacy 0.x series |
| v0.90.x | >= 1.4.0 | All current versions | Deprecated | Legacy 0.x series |
| v0.80.x | >= 1.3.0 | All current versions | Deprecated | Legacy 0.x series |
| v0.70.x | >= 1.2.0 | All current versions | Deprecated | Legacy 0.x series |
| < v0.70.0 | >= 1.0.0 | All current versions | End of Life | Not supported |

**Key Version Milestones:**
- **v2.3.0+**: BCR bundle compatibility fixes (especially for SHOW FUNCTIONS/PROCEDURES)
- **v2.0.0**: Major version with breaking changes, new resource patterns
- **v1.0.0**: Grant system overhaul, deprecation of legacy grant resources
- **v0.90.0**: Introduction of `snowflake_grant_privileges_to_role`

## Breaking Changes by Version

### v2.3.x+ (Current)

**BCR Bundle Compatibility**
- Fixed parsing for SHOW FUNCTIONS/PROCEDURES with new argument format
- Resolves issues with function and procedure resources being removed from state
- Critical for BCR bundle compatibility

**Improvements**
- Better error messages
- Enhanced state management

### v2.0.x - v2.2.x

**Major Version 2.0 Changes**
- Significant refactoring of resource schemas
- New resource patterns and naming conventions
- Enhanced validation and error handling
- Improved state management

**Grant System Refinements**
- Further improvements to `snowflake_grant_privileges_to_role`
- Better handling of grant dependencies
- Enhanced privilege validation

### v1.0.x - v1.9.x

**Major Grant System Overhaul (v1.0)**
- Introduction of `snowflake_grant_privileges_to_role` (unified grant resource)
- Deprecation of legacy grant resources:
  - `snowflake_database_grant`
  - `snowflake_schema_grant`
  - `snowflake_warehouse_grant`
  - `snowflake_table_grant`
  - `snowflake_view_grant`
- New grant ownership model

**Resource Improvements**
- More strict validation on resource names
- Case sensitivity enforced
- Better dependency handling

### v0.94.x (Legacy)

**Grant Resources Refactored**
- `snowflake_database_grant` → Use specific grant resources
- `snowflake_schema_grant` → Use specific grant resources
- `snowflake_warehouse_grant` → Use specific grant resources

**Resource Attribute Changes**
- Warehouse: `warehouse_size` validation stricter
- Database: `data_retention_time_in_days` range updated
- Schema: `is_managed` behavior clarified

### v0.90.x (Legacy)

**Early Grant System Changes**
- Initial work on new grant patterns
- Preparation for v1.0 grant overhaul

### v0.80.x (Legacy)

**Schema Changes**
- `snowflake_user`: Password management changes
- `snowflake_role`: Comment field now required for tracking
- `snowflake_warehouse`: Auto-suspend/resume defaults changed

## Step-by-Step Upgrade Process

### Step 1: Assess Current State

```bash
# List all resources
terraform state list > current_resources.txt

# Check for deprecated resources
grep -E "(database_grant|schema_grant|warehouse_grant)" current_resources.txt
```

### Step 2: Update Provider Version (Incremental)

Edit your `versions.tf` or `main.tf`:

**If upgrading from 0.x to 1.x:**
```hcl
terraform {
  required_providers {
    snowflake = {
      source  = "Snowflake-Labs/snowflake"
      version = "~> 1.0"  # First upgrade to 1.x
    }
  }
}
```

**If upgrading from 1.x to 2.x:**
```hcl
terraform {
  required_providers {
    snowflake = {
      source  = "Snowflake-Labs/snowflake"
      version = "~> 2.0"  # Then upgrade to 2.x
    }
  }
}
```

**Current recommended version:**
```hcl
terraform {
  required_providers {
    snowflake = {
      source  = "Snowflake-Labs/snowflake"
      version = "~> 2.3"  # Latest with BCR fixes
    }
  }
}
```

**Important**: Always upgrade incrementally through major versions (0.x → 1.x → 2.x)

### Step 3: Initialize and Plan

```bash
# Reinitialize with new provider version
terraform init -upgrade

# Review the plan
terraform plan -out=upgrade.tfplan

# Save the plan output for review
terraform show upgrade.tfplan > upgrade_plan.txt
```

### Step 4: Address Deprecation Warnings

Review the plan output for warnings like:

```
Warning: Deprecated Resource
The resource type "snowflake_database_grant" is deprecated.
```

### Step 5: Migrate Deprecated Resources

See [Common Migration Scenarios](#common-migration-scenarios) below for specific examples.

### Step 6: Apply Changes

```bash
# Apply the upgrade
terraform apply upgrade.tfplan

# Verify the state
terraform state list
terraform show
```

### Step 7: Validate in Snowflake

```sql
-- Check databases
SHOW DATABASES;

-- Check warehouses
SHOW WAREHOUSES;

-- Check roles and grants
SHOW ROLES;
SHOW GRANTS TO ROLE your_role_name;

-- Check users
SHOW USERS;
```

## Common Migration Scenarios

### Scenario 1: Migrating Database Grants

**Old Configuration (v0.80.x and earlier):**
```hcl
resource "snowflake_database_grant" "grant" {
  database_name = "MY_DATABASE"
  privilege     = "USAGE"
  roles         = ["MY_ROLE"]
}
```

**New Configuration (v0.90.x+):**
```hcl
resource "snowflake_grant_privileges_to_role" "database_usage" {
  role_name  = "MY_ROLE"
  privileges = ["USAGE"]
  
  on_database = "MY_DATABASE"
}
```

**Migration Steps:**
```bash
# 1. Remove old resource from state
terraform state rm snowflake_database_grant.grant

# 2. Update configuration to new format

# 3. Import the existing grant
terraform import snowflake_grant_privileges_to_role.database_usage "MY_ROLE|false|USAGE|OnDatabase|MY_DATABASE"

# 4. Verify
terraform plan
```

### Scenario 2: Migrating Schema Grants

**Old Configuration:**
```hcl
resource "snowflake_schema_grant" "grant" {
  database_name = "MY_DATABASE"
  schema_name   = "MY_SCHEMA"
  privilege     = "USAGE"
  roles         = ["MY_ROLE"]
}
```

**New Configuration:**
```hcl
resource "snowflake_grant_privileges_to_role" "schema_usage" {
  role_name  = "MY_ROLE"
  privileges = ["USAGE"]
  
  on_schema {
    schema_name = "MY_DATABASE.MY_SCHEMA"
  }
}
```

### Scenario 3: Migrating Warehouse Grants

**Old Configuration:**
```hcl
resource "snowflake_warehouse_grant" "grant" {
  warehouse_name = "MY_WAREHOUSE"
  privilege      = "USAGE"
  roles          = ["MY_ROLE"]
}
```

**New Configuration:**
```hcl
resource "snowflake_grant_privileges_to_role" "warehouse_usage" {
  role_name  = "MY_ROLE"
  privileges = ["USAGE"]
  
  on_account_object {
    object_type = "WAREHOUSE"
    object_name = "MY_WAREHOUSE"
  }
}
```

### Scenario 4: Updating Warehouse Configuration

**Old Configuration:**
```hcl
resource "snowflake_warehouse" "warehouse" {
  name           = "MY_WAREHOUSE"
  warehouse_size = "XSMALL"  # Old format
}
```

**New Configuration:**
```hcl
resource "snowflake_warehouse" "warehouse" {
  name           = "MY_WAREHOUSE"
  warehouse_size = "X-SMALL"  # New format with hyphen
}
```

### Scenario 5: User Password Management

**Old Configuration:**
```hcl
resource "snowflake_user" "user" {
  name     = "MY_USER"
  password = "hardcoded_password"  # Deprecated
}
```

**New Configuration:**
```hcl
resource "snowflake_user" "user" {
  name     = "MY_USER"
  # Password should be managed outside Terraform or use must_change_password
  must_change_password = true
}
```

## Automated Migration Script

Use this helper script to identify resources that need migration:

```bash
#!/bin/bash
# migration_checker.sh

echo "Checking for deprecated resources..."

# Check for old grant resources
echo -e "\n=== Deprecated Grant Resources ==="
terraform state list | grep -E "(database_grant|schema_grant|warehouse_grant|table_grant)" || echo "None found"

# Check for old warehouse size format
echo -e "\n=== Checking Warehouse Sizes ==="
terraform show | grep -i "warehouse_size.*XSMALL\|warehouse_size.*XXSMALL" || echo "No issues found"

# Check for hardcoded passwords
echo -e "\n=== Checking for Hardcoded Passwords ==="
grep -r "password.*=" *.tf | grep -v "must_change_password" | grep -v "#" || echo "No issues found"

# Generate migration report
echo -e "\n=== Migration Report ==="
echo "Total resources: $(terraform state list | wc -l)"
echo "Deprecated grants: $(terraform state list | grep -E '(database_grant|schema_grant|warehouse_grant)' | wc -l)"

echo -e "\nReview complete. Check output above for items requiring migration."
```

Save as `migration_checker.sh` and run:
```bash
chmod +x migration_checker.sh
./migration_checker.sh
```

## Troubleshooting

### Issue: "Error: Invalid provider version"

**Solution:**
```bash
# Clear provider cache
rm -rf .terraform
rm .terraform.lock.hcl

# Reinitialize
terraform init -upgrade
```

### Issue: "Resource not found" after upgrade

**Solution:**
The resource may have been renamed. Check the state:
```bash
terraform state list
terraform state show <resource_name>

# If needed, rename in state
terraform state mv old_resource_name new_resource_name
```

### Issue: "Cycle error" in dependencies

**Solution:**
This often happens with grant resources. Break the cycle:
```bash
# Remove problematic resource from state
terraform state rm problematic_resource

# Reimport after fixing dependencies
terraform import resource_type.name "import_id"
```

### Issue: "Provider configuration has changed"

**Solution:**
```bash
# Reconfigure provider
terraform init -reconfigure

# If using workspaces
terraform workspace select default
terraform init -reconfigure
```

### Issue: Grants not applying correctly

**Solution:**
Ensure proper role hierarchy and dependencies:
```hcl
resource "snowflake_role" "role" {
  name = "MY_ROLE"
}

resource "snowflake_grant_privileges_to_role" "grant" {
  role_name = snowflake_role.role.name  # Use reference
  # ... rest of configuration
  
  depends_on = [snowflake_role.role]
}
```

## Best Practices for Future Upgrades

1. **Pin Provider Versions**: Use `~>` for minor version flexibility
   ```hcl
   version = "~> 0.94.0"  # Allows 0.94.x but not 0.95.0
   ```

2. **Monitor Deprecation Warnings**: Review `terraform plan` output regularly

3. **Use Modules**: Encapsulate common patterns to simplify updates

4. **Document Custom Configurations**: Comment non-standard setups

5. **Automate Testing**: Use Terratest or similar for validation

6. **Subscribe to Updates**: Watch the [provider repository](https://github.com/snowflakedb/terraform-provider-snowflake) for changes

## Additional Resources

- [Snowflake BCR Migration Guide](https://github.com/snowflakedb/terraform-provider-snowflake/blob/main/SNOWFLAKE_BCR_MIGRATION_GUIDE.md#bundle-2025_04) - Official migration guide for Bundle 2025_04
- [Terraform Snowflake Provider Documentation](https://registry.terraform.io/providers/Snowflake-Labs/snowflake/latest/docs)
- [Snowflake Terraform Provider GitHub](https://github.com/snowflakedb/terraform-provider-snowflake)
- [Snowflake Terraform Provider Discussions](https://github.com/snowflakedb/terraform-provider-snowflake/discussions)
- [Terraform Upgrade Guide](https://www.terraform.io/upgrade-guides)
- [Snowflake Documentation](https://docs.snowflake.com/)

## Getting Help

If you encounter issues during the upgrade:

1. **Check GitHub Issues**: Search for similar problems in the [provider issues](https://github.com/snowflakedb/terraform-provider-snowflake/issues)
2. **Review Discussions**: Check [GitHub Discussions](https://github.com/snowflakedb/terraform-provider-snowflake/discussions)
3. **Consult Documentation**: Review the latest provider documentation
4. **Contact Support**: Reach out to Snowflake support if needed

## Version History

- **2025-04**: Bundle 2025_04 breaking changes
- **2024-Q4**: v0.94.x release with grant refactoring
- **2024-Q3**: v0.90.x major grant system overhaul
- **2024-Q2**: v0.80.x schema and attribute updates

---

**Note**: Always test upgrades in a non-production environment first. This guide is maintained as a reference and should be used in conjunction with the official Snowflake Terraform provider documentation.


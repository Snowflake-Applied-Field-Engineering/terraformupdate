# Terraform Snowflake Template

A comprehensive Terraform template for managing Snowflake infrastructure as code. This template provides a modular and scalable approach to managing databases, schemas, warehouses, roles, users, and permissions in Snowflake.

## Features

- **Database Management**: Create and manage multiple databases with configurable retention policies
- **Schema Management**: Define schemas with transient and managed options
- **Warehouse Management**: Configure compute warehouses with auto-suspend and scaling policies
- **Role-Based Access Control**: Create roles and manage role hierarchies
- **User Management**: Provision users with default settings
- **Permission Management**: Grant privileges on databases, schemas, and warehouses
- **Network Policies**: Define IP-based access controls

## Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) >= 1.5.0
- Snowflake account with appropriate permissions (ACCOUNTADMIN or SECURITYADMIN role)
- Snowflake credentials (account, user, password)

## Upgrading from Older Versions

If you're upgrading from an older version of the Snowflake Terraform provider (especially versions before v0.90.0), please see the [UPGRADE_GUIDE.md](UPGRADE_GUIDE.md) for detailed migration instructions. The guide includes:

- Step-by-step upgrade process
- Breaking changes by version
- Common migration scenarios
- Automated migration checker script

**Quick Migration Check:**
```bash
./migration_checker.sh
```

For official migration documentation, see the [Snowflake BCR Migration Guide](https://github.com/snowflakedb/terraform-provider-snowflake/blob/main/SNOWFLAKE_BCR_MIGRATION_GUIDE.md#bundle-2025_04).

## Quick Start

### 1. Clone or Copy the Template

```bash
cd terraform-snowflake
```

### 2. Configure Your Variables

Copy the example variables file and customize it:

```bash
cp terraform.tfvars.example terraform.tfvars
```

Edit `terraform.tfvars` with your Snowflake credentials and desired configuration:

```hcl
snowflake_account  = "your-account-identifier"
snowflake_user     = "your-username"
snowflake_password = "your-password"
snowflake_role     = "ACCOUNTADMIN"
```

**Security Note**: Never commit `terraform.tfvars` to version control. Use environment variables or a secrets manager for production deployments.

### 3. Initialize Terraform

```bash
terraform init
```

### 4. Review the Plan

```bash
terraform plan
```

### 5. Apply the Configuration

```bash
terraform apply
```

## Configuration Guide

### Using Environment Variables (Recommended for Production)

Instead of storing credentials in `terraform.tfvars`, use environment variables:

```bash
export TF_VAR_snowflake_account="your-account"
export TF_VAR_snowflake_user="your-user"
export TF_VAR_snowflake_password="your-password"
export TF_VAR_snowflake_role="ACCOUNTADMIN"
```

### Configuring Resources

#### Databases

```hcl
databases = {
  "MY_DATABASE" = {
    comment             = "My database description"
    data_retention_days = 7
  }
}
```

#### Schemas

```hcl
schemas = {
  "MY_SCHEMA" = {
    database            = "MY_DATABASE"
    comment             = "My schema description"
    is_transient        = false
    is_managed          = false
    data_retention_days = 7
  }
}
```

#### Warehouses

```hcl
warehouses = {
  "MY_WAREHOUSE" = {
    comment             = "My warehouse"
    size                = "SMALL"           # X-SMALL, SMALL, MEDIUM, LARGE, X-LARGE, etc.
    auto_suspend        = 60                # seconds
    auto_resume         = true
    initially_suspended = true
    max_cluster_count   = 3
    min_cluster_count   = 1
    scaling_policy      = "STANDARD"        # STANDARD or ECONOMY
  }
}
```

#### Roles

```hcl
roles = {
  "MY_ROLE" = {
    comment = "Custom role description"
  }
}
```

#### Users

```hcl
users = {
  "MY_USER" = {
    comment           = "User description"
    login_name        = "my_user"
    display_name      = "My User"
    email             = "user@example.com"
    disabled          = false
    default_warehouse = "MY_WAREHOUSE"
    default_role      = "MY_ROLE"
    default_namespace = "MY_DATABASE.MY_SCHEMA"
  }
}
```

#### Grants

**Database Grants:**
```hcl
database_grants = {
  "grant_name" = {
    database_name     = "MY_DATABASE"
    privilege         = "USAGE"              # USAGE, CREATE SCHEMA, MODIFY, MONITOR, etc.
    roles             = ["MY_ROLE"]
    with_grant_option = false
  }
}
```

**Schema Grants:**
```hcl
schema_grants = {
  "grant_name" = {
    database_name     = "MY_DATABASE"
    schema_name       = "MY_SCHEMA"
    privilege         = "USAGE"              # USAGE, CREATE TABLE, CREATE VIEW, etc.
    roles             = ["MY_ROLE"]
    with_grant_option = false
  }
}
```

**Warehouse Grants:**
```hcl
warehouse_grants = {
  "grant_name" = {
    warehouse_name    = "MY_WAREHOUSE"
    privilege         = "USAGE"              # USAGE, OPERATE, MONITOR, MODIFY
    roles             = ["MY_ROLE"]
    with_grant_option = false
  }
}
```

## Common Snowflake Privileges

### Database Privileges
- `USAGE` - Use the database
- `CREATE SCHEMA` - Create schemas in the database
- `MODIFY` - Modify database settings
- `MONITOR` - Monitor database usage

### Schema Privileges
- `USAGE` - Use the schema
- `CREATE TABLE` - Create tables in the schema
- `CREATE VIEW` - Create views in the schema
- `CREATE STAGE` - Create stages in the schema
- `CREATE PIPE` - Create pipes in the schema

### Warehouse Privileges
- `USAGE` - Use the warehouse for queries
- `OPERATE` - Start, stop, suspend, or resume the warehouse
- `MONITOR` - Monitor warehouse usage
- `MODIFY` - Modify warehouse settings

## Remote State Management

For team collaboration, configure remote state storage. Uncomment and configure the backend block in `main.tf`:

```hcl
backend "s3" {
  bucket = "your-terraform-state-bucket"
  key    = "snowflake/terraform.tfstate"
  region = "us-east-1"
}
```

Or use Terraform Cloud:

```hcl
backend "remote" {
  organization = "your-org"
  workspaces {
    name = "snowflake-infrastructure"
  }
}
```

## Best Practices

1. **Use Environment Variables**: Store sensitive credentials in environment variables or a secrets manager
2. **Enable Remote State**: Use remote state storage for team collaboration
3. **Version Control**: Commit your `.tf` files but never commit `terraform.tfvars` or `.tfstate` files
4. **Modular Approach**: Organize resources logically using the provided variable maps
5. **Least Privilege**: Grant only the minimum required privileges to roles
6. **Resource Naming**: Use consistent naming conventions (e.g., `ENV_RESOURCE_TYPE`)
7. **Comments**: Document your resources with meaningful comments
8. **Auto-Suspend**: Configure warehouses to auto-suspend to save costs
9. **Data Retention**: Set appropriate data retention policies based on your needs
10. **Plan Before Apply**: Always run `terraform plan` before `terraform apply`

## Useful Commands

```bash
# Initialize Terraform
terraform init

# Format Terraform files
terraform fmt

# Validate configuration
terraform validate

# Plan changes
terraform plan

# Apply changes
terraform apply

# Destroy resources (use with caution!)
terraform destroy

# Show current state
terraform show

# List resources
terraform state list

# Import existing resource
terraform import snowflake_database.databases["EXISTING_DB"] "EXISTING_DB"
```

## Importing Existing Resources

If you have existing Snowflake resources, you can import them:

```bash
# Import a database
terraform import snowflake_database.databases["MY_DB"] "MY_DB"

# Import a warehouse
terraform import snowflake_warehouse.warehouses["MY_WH"] "MY_WH"

# Import a role
terraform import snowflake_role.roles["MY_ROLE"] "MY_ROLE"
```

## Troubleshooting

### Authentication Issues

If you encounter authentication errors:
1. Verify your account identifier is correct
2. Check that your user has the required role
3. Ensure your password is correct
4. Try using environment variables instead of `terraform.tfvars`

### Permission Errors

If you get permission errors:
1. Ensure you're using a role with sufficient privileges (ACCOUNTADMIN or SECURITYADMIN)
2. Check that the role has the necessary grants
3. Verify resource dependencies are correct

### State Lock Issues

If state is locked:
```bash
terraform force-unlock <lock-id>
```

## Security Considerations

1. **Never commit credentials** to version control
2. **Use environment variables** or a secrets manager for sensitive data
3. **Enable MFA** on your Snowflake account
4. **Implement network policies** to restrict access by IP
5. **Use least privilege** when granting permissions
6. **Rotate credentials** regularly
7. **Audit access** using Snowflake's ACCOUNT_USAGE schema

## Additional Resources

- [Terraform Snowflake Provider Documentation](https://registry.terraform.io/providers/Snowflake-Labs/snowflake/latest/docs)
- [Snowflake Documentation](https://docs.snowflake.com/)
- [Terraform Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)

## Support

For issues or questions:
- Check the [Terraform Snowflake Provider Issues](https://github.com/Snowflake-Labs/terraform-provider-snowflake/issues)
- Review [Snowflake Community](https://community.snowflake.com/)
- Consult [Terraform Documentation](https://www.terraform.io/docs/)

## License

This template is provided as-is for use in your Snowflake infrastructure management.


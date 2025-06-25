# Naming Conventions

This Helm chart uses the installation name (release name) directly for naming Kubernetes resources, making them simple and predictable.

## Naming Structure

Resources follow the pattern: `{release-name}`

### Example 1: Basic Installation

```bash
helm install mlflow-postgresql .
```

**Configuration:**
- Release name: `mlflow-postgresql`

**Created Resources:**
- Deployment: `mlflow-postgresql`
- Service: `mlflow-postgresql`
- Secret: `mlflow-postgresql-secret`
- PVC: `mlflow-postgresql-pvc`
- ServiceAccount: `mlflow-postgresql`

### Example 2: Different Application

```bash
helm install ecommerce-db .
```

**Configuration:**
- Release name: `ecommerce-db`

**Created Resources:**
- Deployment: `ecommerce-db`
- Service: `ecommerce-db`
- Secret: `ecommerce-db-secret`
- PVC: `ecommerce-db-pvc`
- ServiceAccount: `ecommerce-db`

### Example 3: Multiple Environments

```bash
# Development environment
helm install myapp-dev-db .

# Production environment  
helm install myapp-prod-db .
```

**Resources for development:**
- Deployment: `myapp-dev-db`
- Service: `myapp-dev-db`
- Secret: `myapp-dev-db-secret`

**Resources for production:**
- Deployment: `myapp-prod-db`
- Service: `myapp-prod-db`
- Secret: `myapp-prod-db-secret`

## Additional Labels

Besides the name, resources include labels that facilitate identification:

```yaml
labels:
  app.kubernetes.io/name: mlflow-postgresql    # Release name
  app.kubernetes.io/instance: mlflow-postgresql # Release name
  app.kubernetes.io/component: database        # Component type
  app.kubernetes.io/database: mydb            # Database name
```

## Useful Commands to Identify Resources

```bash
# List all resources for a specific installation
oc get all -l app.kubernetes.io/instance=mlflow-postgresql

# List all PostgreSQL databases in the cluster
oc get all -l app.kubernetes.io/component=database

# Find resources by release name
oc get all -l app.kubernetes.io/name=ecommerce-db

# List all Helm-managed resources
oc get all -l app.kubernetes.io/managed-by=Helm
```

## Advantages of This Convention

1. **Simplicity**: Resource names match exactly with the Helm release name
2. **Predictability**: Easy to guess resource names from installation command
3. **No conflicts**: Different release names ensure no resource name conflicts
4. **Clean naming**: No complex prefixes or suffixes
5. **Standard practice**: Follows common Helm naming conventions

## Practical Examples

### Scenario 1: Microservices Architecture

```bash
# Authentication service database
helm install auth-db .

# User service database
helm install user-db .

# Order service database
helm install order-db .
```

**Result:**
- `auth-db` (deployment, service, etc.)
- `user-db` (deployment, service, etc.)
- `order-db` (deployment, service, etc.)

### Scenario 2: Multi-Environment Setup

```bash
# Development
helm install webapp-dev-postgres . --namespace development

# Staging
helm install webapp-staging-postgres . --namespace staging

# Production
helm install webapp-prod-postgres . --namespace production
```

### Scenario 3: Application-Specific Naming

```bash
# MLflow tracking server database
helm install mlflow-postgresql .

# Airflow metadata database
helm install airflow-postgres .

# Grafana database
helm install grafana-db .
```

## Resource Management by Installation

### Operations by Release Name

```bash
# Scale specific installation
oc scale deployment mlflow-postgresql --replicas=2

# Restart specific installation
oc rollout restart deployment mlflow-postgresql

# Delete all resources from specific installation
oc delete all -l app.kubernetes.io/instance=mlflow-postgresql
```

### Monitoring by Installation

```bash
# View logs for specific installation
oc logs -f deployment/mlflow-postgresql

# Check resource usage for specific installation
oc top pod -l app.kubernetes.io/instance=mlflow-postgresql

# Monitor events for specific installation
oc get events --field-selector involvedObject.name=mlflow-postgresql
```

## Override Names

If you need different resource names, you can use:

### Option 1: Custom full name

```bash
helm install my-app . --set fullnameOverride=custom-postgres
```

**Result:** All resources will be named `custom-postgres`

### Option 2: Custom name override

```bash
helm install my-release . --set nameOverride=my-database
```

**Result:** Resources will be named `my-database` instead of `my-release`

## Best Practices

1. **Use descriptive names**: Choose release names that clearly identify the purpose
2. **Include environment**: Consider including environment in the release name
3. **Be consistent**: Use the same naming pattern across your organization
4. **Avoid conflicts**: Ensure release names are unique within the namespace
5. **Document naming**: Keep a record of your naming conventions

### Recommended Naming Pattern

```bash
# Format: {application}-{environment}-{component}
helm install webapp-prod-postgres . --namespace production
helm install api-dev-db . --namespace development
helm install analytics-staging-postgresql . --namespace staging
```

This pattern provides clear identification of:
- The application (`webapp`, `api`, `analytics`)
- The environment (`prod`, `dev`, `staging`)
- The component (`postgres`, `db`, `postgresql`)
# PostgreSQL Helm Chart for OpenShift

This Helm chart deploys PostgreSQL on OpenShift 4.18+ in a simple and secure way, fully compatible with OpenShift Security Context Constraints (SCC).

## 📋 Features

- ✅ Compatible with OpenShift 4.18+
- ✅ Security configuration compatible with SCC
- ✅ Configurable persistent storage
- ✅ Parameterizable credentials (user, password, database)
- ✅ Integrated health checks
- ✅ Configurable resources
- ✅ Official Red Hat PostgreSQL 15 image
- ✅ Simple resource naming using installation name

## 🏗️ Chart Structure

```
postgresql-openshift/
├── Chart.yaml                 # Chart metadata
├── values.yaml               # Default values
├── templates/
│   ├── deployment.yaml       # PostgreSQL deployment
│   ├── service.yaml          # ClusterIP service
│   ├── secret.yaml           # Secret for credentials
│   ├── pvc.yaml              # PersistentVolumeClaim
│   ├── serviceaccount.yaml   # ServiceAccount
│   ├── scc.yaml              # Security Context Constraint (optional)
│   ├── _helpers.tpl          # Helper functions
│   └── NOTES.txt             # Post-installation instructions
├── README.md                 # This file (English)
├── NAMING.md                 # Naming conventions (English)
├── docs/
│   ├── INSTALL.md            # Installation guide (English)
│   └── es/
│       ├── README.md         # Spanish documentation
│       ├── INSTALL.md        # Spanish installation guide
│       └── NAMING.md         # Spanish naming conventions
```

## 🚀 Quick Start

### Prerequisites
- OpenShift 4.18+
- Helm 3.x
- Permissions to create resources in the namespace

### Basic Installation

```bash
# Navigate to the chart directory
cd postgresql-openshift

# Install with default values
helm install my-postgresql .

# Install with custom parameters
helm install my-postgresql . \
  --set postgresql.database=mydatabase \
  --set postgresql.username=myuser \
  --set postgresql.password=mypassword123

# Install for specific application (example: MLflow)
helm install mlflow-postgresql . \
  --set postgresql.database=mlflow \
  --set postgresql.username=mlflow_user \
  --set postgresql.password=secure_password
```

### Installation Verification

```bash
# Verify that the pod is running
oc get pods

# View deployment logs
oc logs deployment/my-postgresql

# Check all created resources
oc get all -l app.kubernetes.io/instance=my-postgresql

# Check services
oc get svc
```

## ⚙️ Configuration

### Main Parameters

| Parameter | Description | Default Value |
|-----------|-------------|---------------|
| `postgresql.database` | Database name | `mydb` |
| `postgresql.username` | PostgreSQL user | `postgres` |
| `postgresql.password` | User password | `postgres123` |
| `postgresql.persistence.enabled` | Enable persistent storage | `true` |
| `postgresql.persistence.size` | Volume size | `8Gi` |
| `postgresql.persistence.storageClass` | Storage class | `""` (default) |
| `postgresql.resources.requests.memory` | Requested memory | `256Mi` |
| `postgresql.resources.requests.cpu` | Requested CPU | `250m` |
| `postgresql.resources.limits.memory` | Memory limit | `512Mi` |
| `postgresql.resources.limits.cpu` | CPU limit | `500m` |
| `serviceAccount.create` | Create ServiceAccount | `true` |
| `replicaCount` | Number of replicas | `1` |

### Custom Values File

Create a `custom-values.yaml` file:

```yaml
postgresql:
  database: "production_db"
  username: "app_user"
  password: "super_secure_password_123"
  
  persistence:
    enabled: true
    size: "20Gi"
    storageClass: "fast-ssd"
  
  resources:
    requests:
      memory: "512Mi"
      cpu: "500m"
    limits:
      memory: "1Gi"
      cpu: "1000m"

replicaCount: 1

serviceAccount:
  create: true
  annotations:
    description: "PostgreSQL service account"
```

Install with the custom file:

```bash
helm install my-postgresql . -f custom-values.yaml
```

## 🔐 Security Configuration for OpenShift

### Method 1: Standard Installation (Recommended)

This is the simplest approach and works with default security policies:

```bash
helm install my-postgresql . \
  --set postgresql.database=myapp \
  --set postgresql.username=appuser \
  --set postgresql.password=secretpassword123
```

### Method 2: Use anyuid SCC (Requires Administrator Permissions)

If you have administrator permissions and need to use the `anyuid` SCC:

```bash
# 1. Install the chart
helm install my-postgresql .

# 2. Assign anyuid SCC to ServiceAccount
oc adm policy add-scc-to-user anyuid \
  system:serviceaccount:$(oc project -q):my-postgresql

# 3. Restart the deployment to apply changes
oc rollout restart deployment/my-postgresql
```

### Method 3: Custom SCC (Cluster Admin Required)

To create a custom SCC (requires cluster-admin permissions):

```bash
helm install my-postgresql . \
  --set securityContextConstraint.create=true \
  --set postgresql.securityContext.runAsUser=26 \
  --set postgresql.securityContext.fsGroup=26
```

## 🔌 Connecting to PostgreSQL

### Get Database Credentials

```bash
# Get the password from the secret
POSTGRES_PASSWORD=$(oc get secret my-postgresql-secret -o jsonpath="{.data.password}" | base64 -d)
echo "Database Password: $POSTGRES_PASSWORD"

# Get connection information
echo "Host: my-postgresql"
echo "Port: 5432"
echo "Database: mydb"
echo "Username: postgres"
```

### Connect from a Temporary Pod

```bash
oc run postgresql-client --rm --tty -i --restart='Never' \
  --image registry.redhat.io/rhel8/postgresql-15 \
  --env="PGPASSWORD=$POSTGRES_PASSWORD" \
  --command -- psql --host my-postgresql \
  -U postgres -d mydb -p 5432
```

### Port Forward for External Connection

```bash
# Create port forward
oc port-forward svc/my-postgresql 5432:5432

# In another terminal, connect locally
psql --host 127.0.0.1 -U postgres -d mydb -p 5432
```

### Connect from Application

Use these connection parameters in your application:

```yaml
host: my-postgresql
port: 5432
database: mydb
username: postgres
password: <from-secret>
```

## 📊 Monitoring and Troubleshooting

### Check Application Status

```bash
# Check deployment status
oc get deployment my-postgresql

# Check pod status
oc get pods -l app.kubernetes.io/instance=my-postgresql

# View detailed logs
oc logs -f deployment/my-postgresql

# Describe pod for troubleshooting
oc describe pod -l app.kubernetes.io/instance=my-postgresql
```

### Common Issues

#### Security Context Constraint Errors

If you encounter SCC-related errors:

```bash
# Check which SCC is being used
oc describe pod <pod-name> | grep -i scc

# List available SCCs
oc get scc

# Check ServiceAccount permissions
oc describe scc restricted-v2

# View pod security context
oc get pod <pod-name> -o yaml | grep -A 10 securityContext
```

#### Storage and Persistence Issues

```bash
# Check PVC status
oc get pvc my-postgresql-pvc

# Describe PVC for error details
oc describe pvc my-postgresql-pvc

# Check available StorageClasses
oc get storageclass

# Check node storage capacity
oc describe nodes | grep -A 5 "Allocated resources"
```

#### Network and Connectivity Issues

```bash
# Test internal connectivity
oc run test-connection --rm -i --tty --image postgres:15 -- \
  psql -h my-postgresql -U postgres -d mydb

# Check service endpoints
oc get endpoints my-postgresql

# Verify service configuration
oc describe svc my-postgresql
```

## 🔄 Chart Management

### Update and Upgrade

```bash
# Update with new password
helm upgrade my-postgresql . \
  --set postgresql.password=new_secure_password

# Update with values file
helm upgrade my-postgresql . -f updated-values.yaml

# Update with reuse of values
helm upgrade my-postgresql . --reuse-values \
  --set postgresql.persistence.size=50Gi
```

### Rollback and History

```bash
# View installed releases
helm list

# View upgrade history
helm history my-postgresql

# Rollback to previous version
helm rollback my-postgresql

# Rollback to specific revision
helm rollback my-postgresql 2
```

### Uninstall

```bash
# Uninstall the chart (keeps PVC by default)
helm uninstall my-postgresql

# Also delete the PVC if needed
oc delete pvc my-postgresql-pvc

# Verify all resources are cleaned up
oc get all -l app.kubernetes.io/instance=my-postgresql
```

## 📈 Production and Scalability

### Production-Ready Configuration

```yaml
# production-values.yaml
postgresql:
  database: "production_db"
  username: "app_user"
  password: "ultra_secure_production_password_2024"
  
  persistence:
    enabled: true
    size: "100Gi"
    storageClass: "fast-ssd"
  
  resources:
    requests:
      memory: "2Gi"
      cpu: "1000m"
    limits:
      memory: "4Gi"
      cpu: "2000m"

serviceAccount:
  create: true
  annotations:
    description: "Production PostgreSQL service account"
    environment: "production"

# Optional: Node affinity for production workloads
nodeSelector:
  kubernetes.io/os: linux
  node-type: database
```

### Backup and Restore Operations

```bash
# Create database backup
oc exec deployment/my-postgresql -- \
  pg_dump -U postgres mydb > backup-$(date +%Y%m%d-%H%M%S).sql

# Create backup with compression
oc exec deployment/my-postgresql -- \
  pg_dump -U postgres mydb | gzip > backup-$(date +%Y%m%d).sql.gz

# Restore from backup
oc exec -i deployment/my-postgresql -- \
  psql -U postgres mydb < backup-20241225.sql

# List all databases
oc exec deployment/my-postgresql -- \
  psql -U postgres -c "\l"
```

### Monitoring and Metrics

```bash
# Monitor resource usage
oc top pod -l app.kubernetes.io/instance=my-postgresql

# Check database size
oc exec deployment/my-postgresql -- \
  psql -U postgres -c "SELECT pg_size_pretty(pg_database_size('mydb'));"

# Monitor connections
oc exec deployment/my-postgresql -- \
  psql -U postgres -c "SELECT count(*) FROM pg_stat_activity;"
```

## 🔧 Advanced Configuration

### Resource Naming Convention

This chart uses the installation name directly for resource naming:

- **Installation**: `helm install mlflow-postgresql .`
- **Resources created**:
  - Deployment: `mlflow-postgresql`
  - Service: `mlflow-postgresql`
  - Secret: `mlflow-postgresql-secret`
  - PVC: `mlflow-postgresql-pvc`

### Multiple Database Instances

```bash
# Deploy multiple PostgreSQL instances
helm install app1-db . --set postgresql.database=app1_db
helm install app2-db . --set postgresql.database=app2_db
helm install analytics-db . --set postgresql.database=analytics

# Each creates independent resources with unique names
```

### Environment-Specific Deployments

```bash
# Development environment
helm install webapp-dev-db . \
  --namespace development \
  --set postgresql.database=webapp_dev \
  --set postgresql.persistence.size=5Gi

# Production environment
helm install webapp-prod-db . \
  --namespace production \
  --set postgresql.database=webapp_prod \
  --set postgresql.persistence.size=100Gi
```

## 🤝 Contributing

We welcome contributions to improve this Helm chart:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/awesome-feature`)
3. **Commit** your changes (`git commit -am 'Add awesome feature'`)
4. **Push** to the branch (`git push origin feature/awesome-feature`)
5. **Create** a Pull Request

### Development Guidelines

- Follow Helm best practices
- Test changes on OpenShift 4.18+
- Update documentation for new features
- Ensure backward compatibility

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support and Help

If you encounter issues or need help:

### Troubleshooting Steps
1. **Check the logs**: `oc logs deployment/my-postgresql`
2. **Verify OpenShift security**: Review SCC configurations
3. **Check resource limits**: Ensure sufficient CPU/memory
4. **Review storage**: Verify PVC and StorageClass settings

### Getting Help
- **Documentation**: Check the installation and naming guides
- **Issues**: Open an issue in the project repository
- **OpenShift Support**: Consult Red Hat OpenShift documentation

### Useful Commands for Support
```bash
# Gather diagnostic information
oc describe deployment my-postgresql
oc describe pod -l app.kubernetes.io/instance=my-postgresql
oc get events --sort-by='.lastTimestamp'
helm status my-postgresql
```

## 📚 Documentation

- **[Installation Guide](docs/INSTALL.md)** - Detailed installation instructions
- **[Naming Conventions](NAMING.md)** - Resource naming guidelines
- **[Documentación en Español](docs/es/README.md)** - Spanish documentation

## 🏷️ Tags and Labels

Resources created by this chart include standard Kubernetes labels:

```yaml
labels:
  app.kubernetes.io/name: my-postgresql
  app.kubernetes.io/instance: my-postgresql
  app.kubernetes.io/component: database
  app.kubernetes.io/managed-by: Helm
  app.kubernetes.io/version: "15"
```

---

**Chart Version:** 0.1.0  
**PostgreSQL Version:** 15  
**Compatible with:** OpenShift 4.18+  
**Last Updated:** December 2024
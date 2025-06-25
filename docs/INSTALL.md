# PostgreSQL Installation on OpenShift

## Method 1: Standard Installation (Recommended)

This installation uses OpenShift's default security policies:

```bash
# Basic installation
helm install my-postgresql .

# With custom parameters
helm install my-postgresql . \
  --set postgresql.database=mydatabase \
  --set postgresql.username=myuser \
  --set postgresql.password=mypassword123
```

## Method 2: With Custom SCC (Requires Administrator Permissions)

If you need to use specific UIDs/GIDs and have cluster-admin permissions:

```bash
# 1. Install with custom SCC
helm install my-postgresql . \
  --set securityContextConstraint.create=true \
  --set postgresql.securityContext.runAsUser=26 \
  --set postgresql.securityContext.fsGroup=26

# 2. Grant permissions to ServiceAccount to use the SCC
oc adm policy add-scc-to-user my-postgresql-mydatabase-scc \
  system:serviceaccount:$(oc project -q):my-postgresql-mydatabase
```

## Method 3: Use Existing anyuid SCC

If you have permissions to use the `anyuid` SCC:

```bash
# 1. Install normally
helm install my-postgresql .

# 2. Assign anyuid SCC to ServiceAccount
oc adm policy add-scc-to-user anyuid \
  system:serviceaccount:$(oc project -q):my-postgresql-mydb

# 3. Restart deployment to apply changes
oc rollout restart deployment/my-postgresql-mydb
```

## Verification

After installation, verify that the pod is running:

```bash
oc get pods
oc logs deployment/my-postgresql-mydb
```

## Database Connection

```bash
# Get the password
POSTGRES_PASSWORD=$(oc get secret my-postgresql-mydb-secret -o jsonpath="{.data.password}" | base64 -d)

# Connect from a temporary pod
oc run postgresql-client --rm --tty -i
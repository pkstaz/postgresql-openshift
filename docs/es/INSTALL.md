# Instalación de PostgreSQL en OpenShift

## Método 1: Instalación estándar (Recomendado)

Esta instalación utiliza las políticas de seguridad predeterminadas de OpenShift:

```bash
# Instalación básica
helm install my-postgresql .

# Con parámetros personalizados
helm install my-postgresql . \
  --set postgresql.database=mibasededatos \
  --set postgresql.username=miusuario \
  --set postgresql.password=micontraseña123
```

## Método 2: Con SCC personalizado (Requiere permisos de administrador)

Si necesitas usar UIDs/GIDs específicos y tienes permisos de cluster-admin:

```bash
# 1. Instalar con SCC personalizado
helm install my-postgresql . \
  --set securityContextConstraint.create=true \
  --set postgresql.securityContext.runAsUser=26 \
  --set postgresql.securityContext.fsGroup=26

# 2. Dar permisos al ServiceAccount para usar el SCC
oc adm policy add-scc-to-user my-postgresql-mibasededatos-scc \
  system:serviceaccount:$(oc project -q):my-postgresql
```

## Método 3: Usar anyuid SCC existente

Si tienes permisos para usar el SCC `anyuid`:

```bash
# 1. Instalar normalmente
helm install my-postgresql .

# 2. Asignar anyuid SCC al ServiceAccount
oc adm policy add-scc-to-user anyuid \
  system:serviceaccount:$(oc project -q):my-postgresql

# 3. Reiniciar el deployment para aplicar los cambios
oc rollout restart deployment/my-postgresql
```

## Verificación

Después de la instalación, verifica que el pod esté ejecutándose:

```bash
oc get pods
oc logs deployment/my-postgresql
```

## Conexión a la base de datos

```bash
# Obtener la contraseña
POSTGRES_PASSWORD=$(oc get secret my-postgresql-secret -o jsonpath="{.data.password}" | base64 -d)

# Conectarse desde un pod temporal
oc run postgresql-client --rm --tty -i --restart='Never' \
  --image registry.redhat.io/rhel8/postgresql-15 \
  --env="PGPASSWORD=$POSTGRES_PASSWORD" \
  --command -- psql --host my-postgresql \
  -U postgres -d mydb -p 5432
```

## Troubleshooting

Si obtienes errores de SCC, verifica:

1. Que el pod use el ServiceAccount correcto:
   ```bash
   oc describe pod <pod-name> | grep "Service Account"
   ```

2. Qué SCCs están disponibles para el ServiceAccount:
   ```bash
   oc describe scc | grep -A 20 "Name:"
   ```

3. Los logs del pod para más detalles:
   ```bash
   oc logs <pod-name>
   ```

## Configuraciones Avanzadas

### Instalación con valores personalizados

Crea un archivo `valores-personalizados.yaml`:

```yaml
postgresql:
  database: "mi_aplicacion"
  username: "admin_app"
  password: "contraseña_super_segura_123"
  
  persistence:
    enabled: true
    size: "50Gi"
    storageClass: "gp2"
  
  resources:
    requests:
      memory: "1Gi"
      cpu: "500m"
    limits:
      memory: "2Gi"
      cpu: "1000m"
```

Instalar con el archivo:

```bash
helm install mi-postgres . -f valores-personalizados.yaml
```

### Múltiples bases de datos

Para instalar múltiples instancias de PostgreSQL:

```bash
# Base de datos para usuarios
helm install db-usuarios . \
  --set postgresql.database=usuarios \
  --set postgresql.username=user_admin \
  --set postgresql.password=usuarios_123

# Base de datos para productos
helm install db-productos . \
  --set postgresql.database=productos \
  --set postgresql.username=prod_admin \
  --set postgresql.password=productos_456
```

### Configuración para diferentes entornos

#### Desarrollo
```bash
helm install postgres-dev . \
  --set postgresql.database=desarrollo \
  --set postgresql.persistence.size=5Gi \
  --set postgresql.resources.requests.memory=128Mi
```

#### Producción
```bash
helm install postgres-prod . \
  --set postgresql.database=produccion \
  --set postgresql.persistence.size=100Gi \
  --set postgresql.resources.requests.memory=2Gi \
  --set postgresql.resources.limits.memory=4Gi
```

## Comandos Útiles Post-Instalación

### Monitoreo
```bash
# Ver el estado de todos los recursos
oc get all -l app.kubernetes.io/instance=my-postgresql

# Monitorear logs en tiempo real
oc logs -f deployment/my-postgresql

# Ver uso de recursos
oc top pod -l app.kubernetes.io/instance=my-postgresql
```

### Mantenimiento
```bash
# Escalar temporalmente (para mantenimiento)
oc scale deployment my-postgresql --replicas=0

# Restaurar el servicio
oc scale deployment my-postgresql --replicas=1

# Reiniciar el pod
oc rollout restart deployment/my-postgresql
```

### Backup
```bash
# Crear backup manual
oc exec deployment/my-postgresql -- \
  pg_dump -U postgres mydb > backup-$(date +%Y%m%d).sql

# Verificar el backup
ls -la backup-*.sql
```

## Solución de Problemas Comunes

### Pod en estado Pending
```bash
# Verificar eventos
oc describe pod <nombre-pod>

# Verificar recursos disponibles
oc describe node

# Verificar storage classes
oc get storageclass
```

### Errores de conectividad
```bash
# Probar conectividad interna
oc run test-connection --rm -i --tty --image postgres:15 -- \
  psql -h my-postgresql -U postgres -d mydb

# Verificar el servicio
oc get svc my-postgresql
oc describe svc my-postgresql
```

### Problemas de permisos
```bash
# Verificar SCC actual
oc get pod <pod-name> -o yaml | grep scc

# Listar SCCs disponibles
oc get scc

# Verificar permisos del ServiceAccount
oc describe sa my-postgresql
```
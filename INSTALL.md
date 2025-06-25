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
oc adm policy add-scc-to-user my-postgresql-postgresql-openshift-scc \
  system:serviceaccount:$(oc project -q):my-postgresql-postgresql-openshift
```

## Método 3: Usar anyuid SCC existente

Si tienes permisos para usar el SCC `anyuid`:

```bash
# 1. Instalar normalmente
helm install my-postgresql .

# 2. Asignar anyuid SCC al ServiceAccount
oc adm policy add-scc-to-user anyuid \
  system:serviceaccount:$(oc project -q):my-postgresql-postgresql-openshift

# 3. Reiniciar el deployment para aplicar los cambios
oc rollout restart deployment/my-postgresql-postgresql-openshift
```

## Verificación

Después de la instalación, verifica que el pod esté ejecutándose:

```bash
oc get pods
oc logs deployment/my-postgresql-postgresql-openshift
```

## Conexión a la base de datos

```bash
# Obtener la contraseña
POSTGRES_PASSWORD=$(oc get secret my-postgresql-postgresql-openshift-secret -o jsonpath="{.data.password}" | base64 -d)

# Conectarse desde un pod temporal
oc run postgresql-client --rm --tty -i --restart='Never' \
  --image registry.redhat.io/rhel8/postgresql-15 \
  --env="PGPASSWORD=$POSTGRES_PASSWORD" \
  --command -- psql --host my-postgresql-postgresql-openshift \
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
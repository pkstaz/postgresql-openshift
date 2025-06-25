# PostgreSQL Helm Chart para OpenShift

Este Helm chart despliega PostgreSQL en OpenShift 4.18+ de manera sencilla y compatible con las Security Context Constraints (SCC) de OpenShift.

## 📋 Características

- ✅ Compatible con OpenShift 4.18+
- ✅ Configuración de seguridad compatible con SCC
- ✅ Almacenamiento persistente configurable
- ✅ Credenciales parametrizables (usuario, contraseña, base de datos)
- ✅ Health checks integrados
- ✅ Recursos configurables
- ✅ Imagen oficial de Red Hat PostgreSQL 15

## 🏗️ Estructura del Chart

```
postgresql-openshift/
├── Chart.yaml                 # Metadatos del chart
├── values.yaml               # Valores por defecto
├── templates/
│   ├── deployment.yaml       # Deployment de PostgreSQL
│   ├── service.yaml          # Servicio ClusterIP
│   ├── secret.yaml           # Secret para credenciales
│   ├── pvc.yaml              # PersistentVolumeClaim
│   ├── serviceaccount.yaml   # ServiceAccount
│   ├── scc.yaml              # Security Context Constraint (opcional)
│   ├── _helpers.tpl          # Funciones helper
│   └── NOTES.txt             # Instrucciones post-instalación
├── README.md                 # Este archivo
└── INSTALL.md               # Instrucciones detalladas
```

## 🚀 Instalación Rápida

### Prerequisitos
- OpenShift 4.18+
- Helm 3.x
- Permisos para crear recursos en el namespace

### Instalación Básica

```bash
# Clonar o descargar los archivos del chart
cd postgresql-openshift

# Instalación con valores por defecto
helm install my-postgresql .

# Instalación con parámetros personalizados
helm install my-postgresql . \
  --set postgresql.database=mibasededatos \
  --set postgresql.username=miusuario \
  --set postgresql.password=micontraseña123
```

### Verificación de la Instalación

```bash
# Verificar que el pod esté ejecutándose
oc get pods

# Ver logs del deployment
oc logs deployment/my-postgresql-postgresql-openshift

# Verificar servicios
oc get svc
```

## ⚙️ Configuración

### Parámetros Principales

| Parámetro | Descripción | Valor por defecto |
|-----------|-------------|-------------------|
| `postgresql.database` | Nombre de la base de datos | `mydb` |
| `postgresql.username` | Usuario de PostgreSQL | `postgres` |
| `postgresql.password` | Contraseña del usuario | `postgres123` |
| `postgresql.persistence.enabled` | Habilitar almacenamiento persistente | `true` |
| `postgresql.persistence.size` | Tamaño del volumen | `8Gi` |
| `postgresql.persistence.storageClass` | Clase de almacenamiento | `""` (por defecto) |
| `postgresql.resources.requests.memory` | Memoria solicitada | `256Mi` |
| `postgresql.resources.requests.cpu` | CPU solicitada | `250m` |
| `postgresql.resources.limits.memory` | Límite de memoria | `512Mi` |
| `postgresql.resources.limits.cpu` | Límite de CPU | `500m` |

### Archivo de Valores Personalizados

Crea un archivo `custom-values.yaml`:

```yaml
postgresql:
  database: "produccion_db"
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
```

Instalar con el archivo personalizado:

```bash
helm install my-postgresql . -f custom-values.yaml
```

## 🔐 Configuración de Seguridad para OpenShift

### Método 1: Instalación Estándar (Recomendado)

Esta es la forma más sencilla y funciona con las políticas de seguridad predeterminadas:

```bash
helm install my-postgresql . \
  --set postgresql.database=miapp \
  --set postgresql.username=appuser \
  --set postgresql.password=secretpassword123
```

### Método 2: Usar anyuid SCC (Requiere permisos)

Si tienes permisos de administrador y necesitas usar el SCC `anyuid`:

```bash
# 1. Instalar el chart
helm install my-postgresql .

# 2. Asignar anyuid SCC al ServiceAccount
oc adm policy add-scc-to-user anyuid \
  system:serviceaccount:$(oc project -q):my-postgresql-postgresql-openshift

# 3. Reiniciar el deployment
oc rollout restart deployment/my-postgresql-postgresql-openshift
```

### Método 3: SCC Personalizado (Cluster Admin)

Para crear un SCC personalizado (requiere permisos de cluster-admin):

```bash
helm install my-postgresql . \
  --set securityContextConstraint.create=true \
  --set postgresql.securityContext.runAsUser=26 \
  --set postgresql.securityContext.fsGroup=26
```

## 🔌 Conexión a PostgreSQL

### Obtener Credenciales

```bash
# Obtener la contraseña
POSTGRES_PASSWORD=$(oc get secret my-postgresql-postgresql-openshift-secret -o jsonpath="{.data.password}" | base64 -d)
echo "Contraseña: $POSTGRES_PASSWORD"
```

### Conectarse desde un Pod Temporal

```bash
oc run postgresql-client --rm --tty -i --restart='Never' \
  --image registry.redhat.io/rhel8/postgresql-15 \
  --env="PGPASSWORD=$POSTGRES_PASSWORD" \
  --command -- psql --host my-postgresql-postgresql-openshift \
  -U postgres -d mydb -p 5432
```

### Port Forward para Conexión Externa

```bash
# Hacer port-forward
oc port-forward svc/my-postgresql-postgresql-openshift 5432:5432

# En otra terminal, conectarse localmente
psql --host 127.0.0.1 -U postgres -d mydb -p 5432
```

## 📊 Monitoreo y Troubleshooting

### Verificar Estado

```bash
# Estado del deployment
oc get deployment my-postgresql-postgresql-openshift

# Estado del pod
oc get pods -l app.kubernetes.io/name=postgresql-openshift

# Logs detallados
oc logs -f deployment/my-postgresql-postgresql-openshift

# Describir pod para troubleshooting
oc describe pod <nombre-del-pod>
```

### Problemas Comunes

#### Error de Security Context Constraint

Si ves errores relacionados con SCC:

```bash
# Verificar qué SCC está usando
oc describe pod <pod-name> | grep -i scc

# Listar SCCs disponibles
oc get scc

# Verificar permisos del ServiceAccount
oc describe scc restricted-v2
```

#### Problemas de Almacenamiento

```bash
# Verificar PVC
oc get pvc

# Describir PVC para ver errores
oc describe pvc my-postgresql-postgresql-openshift-pvc

# Verificar StorageClasses disponibles
oc get storageclass
```

## 🔄 Gestión del Chart

### Actualizar

```bash
# Actualizar con nuevos valores
helm upgrade my-postgresql . \
  --set postgresql.password=nueva_contraseña

# Actualizar con archivo de valores
helm upgrade my-postgresql . -f custom-values.yaml
```

### Desinstalar

```bash
# Desinstalar el chart (mantiene PVC)
helm uninstall my-postgresql

# Para eliminar también el PVC
oc delete pvc my-postgresql-postgresql-openshift-pvc
```

### Ver Historial

```bash
# Ver releases instalados
helm list

# Ver historial de versiones
helm history my-postgresql

# Rollback a versión anterior
helm rollback my-postgresql 1
```

## 📈 Escalabilidad y Producción

### Configuración para Producción

```yaml
# production-values.yaml
postgresql:
  database: "production_db"
  username: "app_user"
  password: "ultra_secure_production_password"
  
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
    # Añadir anotaciones específicas si es necesario
```

### Backup y Restore

```bash
# Crear backup
oc exec deployment/my-postgresql-postgresql-openshift -- \
  pg_dump -U postgres mydb > backup.sql

# Restaurar backup
oc exec -i deployment/my-postgresql-postgresql-openshift -- \
  psql -U postgres mydb < backup.sql
```

## 🤝 Contribuir

Para contribuir a este chart:

1. Fork el repositorio
2. Crea una rama para tu feature (`git checkout -b feature/nueva-caracteristica`)
3. Commit tus cambios (`git commit -am 'Añadir nueva característica'`)
4. Push a la rama (`git push origin feature/nueva-caracteristica`)
5. Crea un Pull Request

## 📄 Licencia

Este proyecto está bajo la Licencia MIT - ver el archivo LICENSE para detalles.

## 🆘 Soporte

Si encuentras problemas:

1. Revisa la sección de troubleshooting
2. Verifica los logs: `oc logs deployment/my-postgresql-postgresql-openshift`
3. Comprueba la configuración de seguridad de OpenShift
4. Abre un issue en el repositorio del proyecto

---

**Versión del Chart:** 0.1.0  
**Versión de PostgreSQL:** 15  
**Compatible con:** OpenShift 4.18+
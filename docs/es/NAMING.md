# Convención de Nombres

Este Helm chart utiliza el nombre de la instalación (release name) directamente para nombrar los recursos de Kubernetes, haciéndolos simples y predecibles.

## Estructura de Nombres

Los recursos siguen el patrón: `{nombre-instalacion}`

### Ejemplo 1: Instalación Básica

```bash
helm install mlflow-postgresql .
```

**Configuración:**
- Nombre de instalación: `mlflow-postgresql`

**Recursos creados:**
- Deployment: `mlflow-postgresql`
- Service: `mlflow-postgresql`
- Secret: `mlflow-postgresql-secret`
- PVC: `mlflow-postgresql-pvc`
- ServiceAccount: `mlflow-postgresql`

### Ejemplo 2: Aplicación Diferente

```bash
helm install ecommerce-db .
```

**Configuración:**
- Nombre de instalación: `ecommerce-db`

**Recursos creados:**
- Deployment: `ecommerce-db`
- Service: `ecommerce-db`
- Secret: `ecommerce-db-secret`
- PVC: `ecommerce-db-pvc`
- ServiceAccount: `ecommerce-db`

### Ejemplo 3: Múltiples Entornos

```bash
# Entorno de desarrollo
helm install miapp-dev-db .

# Entorno de producción
helm install miapp-prod-db .
```

**Recursos para desarrollo:**
- Deployment: `miapp-dev-db`
- Service: `miapp-dev-db`
- Secret: `miapp-dev-db-secret`

**Recursos para producción:**
- Deployment: `miapp-prod-db`
- Service: `miapp-prod-db`
- Secret: `miapp-prod-db-secret`

## Etiquetas Adicionales

Además del nombre, los recursos incluyen etiquetas que facilitan su identificación:

```yaml
labels:
  app.kubernetes.io/name: mlflow-postgresql    # Nombre de instalación
  app.kubernetes.io/instance: mlflow-postgresql # Nombre de instalación
  app.kubernetes.io/component: database        # Tipo de componente
  app.kubernetes.io/database: mydb            # Nombre de la base de datos
  app.kubernetes.io/managed-by: Helm          # Gestor de despliegue
```

## Comandos Útiles para Identificar Recursos

```bash
# Listar todos los recursos de una instalación específica
oc get all -l app.kubernetes.io/instance=mlflow-postgresql

# Listar todas las bases de datos PostgreSQL del cluster
oc get all -l app.kubernetes.io/component=database

# Encontrar recursos por nombre de instalación
oc get all -l app.kubernetes.io/name=ecommerce-db

# Listar todos los recursos gestionados por Helm
oc get all -l app.kubernetes.io/managed-by=Helm
```

## Ventajas de esta Convención

1. **Simplicidad**: Los nombres de recursos coinciden exactamente con el nombre de instalación de Helm
2. **Predictibilidad**: Es fácil adivinar los nombres de recursos desde el comando de instalación
3. **Sin conflictos**: Nombres de instalación diferentes aseguran que no hay conflictos de nombres
4. **Nomenclatura limpia**: No hay prefijos o sufijos complejos
5. **Práctica estándar**: Sigue las convenciones comunes de nomenclatura de Helm

## Ejemplos Prácticos de Uso

### Escenario 1: Arquitectura de Microservicios

```bash
# Base de datos del servicio de autenticación
helm install auth-db .

# Base de datos del servicio de usuarios
helm install user-db .

# Base de datos del servicio de pedidos
helm install order-db .
```

**Resultado:**
- `auth-db` (deployment, service, etc.)
- `user-db` (deployment, service, etc.)
- `order-db` (deployment, service, etc.)

### Escenario 2: Configuración Multi-Entorno

```bash
# Desarrollo
helm install webapp-dev-postgres . --namespace desarrollo

# Testing
helm install webapp-staging-postgres . --namespace staging

# Producción
helm install webapp-prod-postgres . --namespace produccion
```

### Escenario 3: Nomenclatura Específica por Aplicación

```bash
# Base de datos del servidor de seguimiento MLflow
helm install mlflow-postgresql .

# Base de datos de metadatos de Airflow
helm install airflow-postgres .

# Base de datos de Grafana
helm install grafana-db .
```

## Gestión de Recursos por Instalación

### Operaciones por Nombre de Instalación

```bash
# Escalar instalación específica
oc scale deployment mlflow-postgresql --replicas=2

# Reiniciar instalación específica
oc rollout restart deployment mlflow-postgresql

# Eliminar todos los recursos de instalación específica
oc delete all -l app.kubernetes.io/instance=mlflow-postgresql
```

### Monitoreo por Instalación

```bash
# Ver logs de instalación específica
oc logs -f deployment/mlflow-postgresql

# Verificar uso de recursos de instalación específica
oc top pod -l app.kubernetes.io/instance=mlflow-postgresql

# Monitorear eventos de instalación específica
oc get events --field-selector involvedObject.name=mlflow-postgresql
```

## Sobreescribir Nombres

Si necesitas nombres de recursos diferentes, puedes usar:

### Opción 1: Nombre completo personalizado

```bash
helm install mi-app . --set fullnameOverride=postgres-personalizado
```

**Resultado:** Todos los recursos se llamarán `postgres-personalizado`

### Opción 2: Sobreescribir nombre base

```bash
helm install mi-release . --set nameOverride=mi-base-datos
```

**Resultado:** Los recursos se llamarán `mi-base-datos` en lugar de `mi-release`

## Mejores Prácticas

1. **Usar nombres descriptivos**: Elige nombres de instalación que identifiquen claramente el propósito
2. **Incluir entorno**: Considera incluir el entorno en el nombre de instalación
3. **Ser consistente**: Usa el mismo patrón de nomenclatura en toda tu organización
4. **Evitar conflictos**: Asegúrate de que los nombres de instalación sean únicos dentro del namespace
5. **Documentar nomenclatura**: Mantén un registro de tus convenciones de nomenclatura

### Patrón de Nomenclatura Recomendado

```bash
# Formato: {aplicacion}-{entorno}-{componente}
helm install webapp-prod-postgres . --namespace produccion
helm install api-dev-db . --namespace desarrollo
helm install analytics-staging-postgresql . --namespace staging
```

Este patrón proporciona identificación clara de:
- La aplicación (`webapp`, `api`, `analytics`)
- El entorno (`prod`, `dev`, `staging`)
- El componente (`postgres`, `db`, `postgresql`)

## Gestión Avanzada de Recursos

### Operaciones Masivas por Etiquetas

```bash
# Escalar todos los deployments de PostgreSQL
oc scale deployment -l app.kubernetes.io/component=database --replicas=2

# Reiniciar todas las aplicaciones de un entorno específico
oc rollout restart deployment -l environment=production

# Ver uso de recursos de todas las bases de datos
oc top pod -l app.kubernetes.io/component=database
```

### Limpieza de Recursos

```bash
# Eliminar todos los recursos de una instalación específica
helm uninstall mlflow-postgresql

# Eliminar manualmente todos los recursos relacionados
oc delete all,pvc,secret,sa -l app.kubernetes.io/instance=mlflow-postgresql

# Verificar que no queden recursos
oc get all -l app.kubernetes.io/instance=mlflow-postgresql
```

Esta convención de nomenclatura simplificada hace que la gestión de recursos sea mucho más intuitiva y eficiente.
# Redis Deployment Guide

Esta guía describe cómo desplegar Redis en Kubernetes para la aplicación.

## 📋 Prerrequisitos

- Kubernetes cluster (v1.20+)
- `kubectl` configurado para acceder al cluster
- Namespace `eca-services` creado (o el namespace que uses para la aplicación)

## 🚀 Pasos de Despliegue

### 1. Crear el Secret de Redis

Primero, crea el secret que contiene la contraseña de Redis:

```bash
kubectl create secret generic redis-creai-cs \
  --from-literal=REDIS_PASSWORD="STRONG_PASS" \
  -n eca-services
```

**Nota**: Reemplaza `STRONG_PASS` con una contraseña segura. Si cambiaste el namespace, reemplaza `eca-services` con tu namespace en todos los comandos.

**Alternativa**: Si prefieres usar un archivo YAML:

1. Edita `k8s-secrets-redis.yaml` (si existe) con tus valores codificados en base64:
   ```bash
   echo -n "STRONG_PASS" | base64
   ```
2. Aplica el secret:
   ```bash
   kubectl apply -f k8s-secrets-redis.yaml
   ```

### 2. Aplicar el Deployment de Redis

```bash
# Aplicar el deployment y servicio de Redis
kubectl apply -f k8s-deployment.yaml
```

### 3. Verificar el Despliegue

```bash
# Verificar el estado del pod
kubectl get pods -n eca-services -l app=redis

# Verificar el servicio
kubectl get svc -n eca-services -l app=redis

# Ver logs de Redis
kubectl logs -f deployment/redis-deployment -n eca-services
```

## 🔧 Configuración

### Variables de Entorno en la Aplicación

Para que la aplicación se conecte a Redis, configura las siguientes variables de entorno en el deployment del backend:

- **REDIS_URL**: URL de conexión a Redis con autenticación
  - Formato: `redis://:PASSWORD@HOST:PORT`
  - Ejemplo: `redis://:STRONG_PASS@127.0.0.1:6379`
  - **Nota**: Esta variable debe obtenerse de un Secret de Kubernetes

### Conexión desde Otros Pods

Para conectarse a Redis desde otros pods en el mismo namespace:

- **Host**: `redis-service.eca-services.svc.cluster.local` (o simplemente `redis-service` si estás en el mismo namespace)
- **Puerto**: `6379`
- **URL completa**: `redis://:STRONG_PASS@redis-service:6379`

## 🧪 Pruebas

### Probar la Conexión a Redis

```bash
# Port forward al servicio Redis
kubectl port-forward -n eca-services svc/redis-service 6379:6379

# En otra terminal, probar con redis-cli
redis-cli -h localhost -p 6379 -a STRONG_PASS ping
# Debería responder: PONG
```

### Probar desde un Pod Temporal

```bash
# Crear un pod temporal con redis-cli
kubectl run -it --rm redis-test --image=redis:7-alpine --restart=Never -n eca-services -- \
  redis-cli -h redis-service -a STRONG_PASS ping
```

## 🐛 Troubleshooting

### Pod no inicia

```bash
# Ver logs del pod
kubectl logs <pod-name> -n eca-services

# Ver descripción del pod para más detalles
kubectl describe pod <pod-name> -n eca-services
```

### Errores de autenticación

Verifica que el secret esté configurado correctamente:

```bash
kubectl get secret redis-creai-cs -n eca-services
```

### Health checks fallando

Verifica que Redis esté respondiendo:

```bash
# Port forward y probar
kubectl port-forward -n eca-services svc/redis-service 6379:6379
redis-cli -h localhost -p 6379 -a STRONG_PASS ping
```

## 📊 Puertos

- **6379**: Puerto de Redis (protocolo RESP)

## 🔒 Seguridad

- La contraseña de Redis nunca debe ser commiteada al repositorio
- Usa `kubectl create secret` en lugar de archivos YAML con valores reales
- El contenedor se ejecuta como usuario no-root (UID 999)
- Todas las capacidades están deshabilitadas por defecto
- Considera usar PersistentVolumeClaim para producción en lugar de emptyDir

## 💾 Persistencia de Datos

Por defecto, este deployment usa `emptyDir` para almacenar datos, lo que significa que los datos se perderán si el pod se reinicia. Para producción, considera:

1. Crear un PersistentVolumeClaim
2. Modificar el deployment para usar el PVC en lugar de `emptyDir`

Ejemplo de PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-pvc
  namespace: eca-services
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

Luego, actualiza el deployment para usar el PVC en lugar de `emptyDir`.

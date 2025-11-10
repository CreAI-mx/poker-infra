# Kubernetes Deployment Guide

Esta guía describe cómo desplegar y probar la aplicación en Kubernetes.

## 📋 Prerrequisitos

- Kubernetes cluster (v1.20+)
- `kubectl` configurado para acceder al cluster
- Acceso a GitHub Container Registry (GHCR) para obtener imágenes
- Secret `ghcr-secret` configurado en el cluster para autenticación de imágenes

**Nota sobre el Namespace**: El namespace `eca-services` es el valor por defecto configurado en los manifiestos de Kubernetes. Si deseas usar un namespace diferente, puedes cambiarlo en `k8s-deployment.yaml` (busca `namespace: eca-services` y reemplázalo con el nombre deseado). Asegúrate de usar el mismo namespace en todos los comandos `kubectl`.

## 🚀 Pasos de Despliegue

### 1. Crear el Secret de Kubernetes

Primero, crea el secret que contiene las credenciales sensibles:

```bash
kubectl create secret generic eca-backend-secrets \
  --from-literal=SMTP_PASS="tu_contraseña_smtp" \
  -n eca-services
```
```bash
kubectl create secret generic mongodb-creai-cs \
  --from-literal=connectionString.standard="uri_mongo" \
  -n eca-services
```

**Nota**: Si cambiaste el namespace, reemplaza `eca-services` con tu namespace en todos los comandos.

**Alternativa**: Si prefieres usar el archivo YAML:

1. Edita `k8s-secrets-backend.yaml` con tus valores codificados en base64:
   ```bash
   echo -n "tu_contraseña" | base64
   ```
2. Aplica el secret:
   ```bash
   kubectl apply -f k8s-secrets-backend.yaml
   ```

### 2. Configurar Variables de Entorno

Edita el archivo `k8s-deployment.yaml` y configura todas las variables de entorno necesarias. Busca las secciones `env:` en los deployments y establece los valores correspondientes.

A continuación se muestra una tabla con las variables de entorno necesarias para el backend y el frontend. La tercera columna muestra un valor de ejemplo.

| Variable                        | Descripción                                                                     | Valor de Ejemplo                     |
|----------------------------------|---------------------------------------------------------------------------------|--------------------------------------|
| **MONGODB_HOST**                | Host (dirección IP o nombre) del servidor de MongoDB                            | `127.0.0.1`                          |
| **MONGODB_PORT**                | Puerto de conexión del servidor de MongoDB                                       | `27017`                              |
| **MONGODB_DATABASE**            | Nombre de la base de datos a utilizar en MongoDB                                 | `my_app_db`                          |
| **MONGODB_USER**                | Usuario para autenticación en MongoDB                                            | `admin`                              |
| **MONGODB_PASSWORD**            | Contraseña de MongoDB (obtenida del secret `eca-backend-secrets`)                | _(from secret)_                      |
| **ORIGINS**                     | Lista de dominios permitidos para peticiones CORS (incluir dominio del frontend) | `http://creai-frontend.net`          |
| **CHROMA_HOST**                 | Host (dirección IP o nombre) del servidor de ChromaDB                            | `127.0.0.1`                          |
| **CHROMA_PORT**                 | Puerto para ChromaDB                                                             | `8000`                               |
| **CHROMADB_COLLECTION**         | Nombre de colección en ChromaDB                                                  | `vector_collection`                  |
| **CHROMADB_DEFAULT_URI**        | URL base de ChromaDB                                                             | `http://127.0.0.1:8000`              |
| **LLM_URL**                     | URL del endpoint del motor de inferencia (LLM)                                   | `http://127.0.0.1:8081`              |
| **HTTP_EMBEDDING_SERVER_URL**   | URL del motor de embeddings (texto a vectores)                                   | `http://127.0.0.1:8082`              |
| **SMTP_HOST**                   | Servidor SMTP para envío de emails                                               | `localhost`                          |
| **SMTP_PORT**                   | Puerto del servidor SMTP                                                        | `1025`                               |
| **SMTP_USER**                   | Usuario para autenticación SMTP                                                 | `noreply@silia.test`                 |
| **SMTP_PASS**                   | Contraseña SMTP (obtenida del secret `eca-backend-secrets`)                      | _(from secret)_                      |
| **PUBLIC_URL**                   | URL pública del frontend                                                          | `http://eca.local`                    |
| **REACT_APP_API_HOST**          | URL base de la API backend (Node.js)                                             | `http://eca.local/api`                |
| **REACT_APP_WS_URL**            | URL del servidor WebSocket (Node.js backend)                                     | `ws://eca.local/api`                  |
| **REACT_APP_RABBITMQ_WEBSOCKET_URL** | URL del proxy WebSocket de RabbitMQ                                        | `ws://127.0.0.1:15674/ws`                 |

**Notas importantes**:
- Las variables marcadas como _(from secret)_ deben ser definidas mediante el Secret de Kubernetes y no directamente como texto plano en el manifiesto.
- **Dominio configurable**: El dominio `eca.local` usado en las variables de entorno del frontend (`PUBLIC_URL`, `REACT_APP_API_HOST`, `REACT_APP_WS_URL`) y en los recursos Ingress debe cambiarse según tu necesidad. Reemplázalo con tu dominio real en:
  - Las variables de entorno del deployment del frontend (líneas 173-178 en `k8s-deployment.yaml`)
  - Los recursos Ingress (líneas 232 y 256 en `k8s-deployment.yaml`)
- **Opcional**: Si deseas cambiar el namespace, busca y reemplaza todas las ocurrencias de `namespace: eca-services` con el namespace que prefieras.

### 3. Aplicar los Manifiestos de Kubernetes

```bash
# Aplicar todos los recursos (namespace, services, deployments, ingress)
kubectl apply -f k8s-deployment.yaml
```

**Nota sobre Ingress**: El despliegue incluye dos recursos Ingress separados:
- **eca-ingress-backend**: Maneja las rutas `/api/*` y las redirige al servicio backend en el puerto 3001
- **eca-ingress-frontend**: Maneja la ruta raíz `/` y la redirige al servicio frontend en el puerto 3000

**⚠️ Importante - Configuración del dominio**: Ambos Ingress están configurados para el host `eca.local` por defecto. **Debes cambiar este dominio según tu necesidad** en:
- Los recursos Ingress (busca `host: eca.local` en `k8s-deployment.yaml`)
- Las variables de entorno del frontend (líneas 173-178 en `k8s-deployment.yaml`)

Si usas `eca.local` para desarrollo local, asegúrate de tener este dominio configurado en tu archivo `/etc/hosts` o en tu DNS local:

```bash
# Agregar entrada en /etc/hosts (Linux/Mac)
echo "127.0.0.1 eca.local" | sudo tee -a /etc/hosts
```

**Variables de entorno del frontend configuradas** (usando `eca.local` como ejemplo - cambiar según necesidad):
- `PUBLIC_URL`: `http://eca.local`
- `REACT_APP_API_HOST`: `http://eca.local/api`
- `REACT_APP_WS_URL`: `ws://eca.local/api`

### 4. Verificar el Despliegue

```bash
# Verificar el estado de los pods
kubectl get pods -n eca-services

# Verificar los servicios
kubectl get svc -n eca-services

# Verificar los deployments
kubectl get deployments -n eca-services

# Verificar los ingress
kubectl get ingress -n eca-services

# Ver todos los recursos
kubectl get all -n eca-services
```

## 🧪 Pruebas

### Verificar el Estado del Despliegue

```bash
# Ver todos los recursos en el namespace
kubectl get all -n eca-services

# Ver el estado de los pods
kubectl get pods -n eca-services

# Ver logs del backend
kubectl logs -f deployment/eca-backend-deployment -n eca-services

# Ver logs del frontend
kubectl logs -f deployment/eca-frontend-deployment -n eca-services
```

### Probar el Backend

```bash
# Port forward al servicio backend
kubectl port-forward -n eca-services svc/eca-backend-service 3001:3001

# En otra terminal, probar el endpoint de health
curl http://localhost:3001/health

# Probar el WebSocket del backend
# Si no tienes wscat instalado, primero instálalo con:
npm install -g wscat

# Luego conecta al WebSocket del backend:
wscat -c ws://localhost:3001
```

### Probar el Frontend

```bash
# Port forward al servicio frontend
kubectl port-forward -n eca-services svc/eca-frontend-service 3000:3000

# Abrir en el navegador
# http://localhost:3000
```

## 🐛 Troubleshooting

### Pods no inician

```bash
# Ver logs del pod
kubectl logs <pod-name> -n eca-services

# Ver descripción del pod para más detalles
kubectl describe pod <pod-name> -n eca-services
```

### Errores de pull de imágenes

Verifica que el secret `ghcr-secret` esté configurado correctamente:

```bash
kubectl get secret ghcr-secret -n eca-services
```

### Variables de entorno no configuradas

Verifica que todas las variables estén configuradas en `k8s-deployment.yaml`:

```bash
kubectl get deployment eca-backend-deployment -n eca-services -o yaml | grep -A 20 env:
```

### Health checks fallando

Verifica que los endpoints estén accesibles:

```bash
# Port forward y probar
kubectl port-forward -n eca-services svc/eca-backend-service 3001:3001
curl http://localhost:3001/health
```

## 🔒 Seguridad

- Los secrets nunca deben ser commiteados al repositorio
- Usa `kubectl create secret` en lugar de archivos YAML con valores reales
- Las imágenes se ejecutan como usuario no-root (UID 1000)
- Todas las capacidades están deshabilitadas por defecto

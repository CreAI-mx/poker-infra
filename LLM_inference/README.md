# LLM Inference - Deployment Guide

Esta guía describe cómo desplegar los servicios de inferencia de LLM (Qwen y embeddings) usando Kubernetes o Docker Compose.

## 📋 Requisitos

- **GPU NVIDIA** con drivers instalados
- **Token de Hugging Face** para descargar modelos
- **Kubernetes** (para k8s) o **Docker & Docker Compose** (para docker-compose)

## 🚀 Kubernetes

### Prerrequisitos

- Kubernetes cluster (v1.20+)
- `kubectl` configurado
- Nodos con GPU disponibles
- StorageClass `fast-ssd` (o modificar el PVC en el YAML)

**Nota sobre el Namespace**: El namespace `eca-services` es el valor por defecto configurado en los manifiestos de Kubernetes. Si deseas usar un namespace diferente, puedes cambiarlo en `k8s-qwen-deployment.yaml` (busca `namespace: eca-services` y reemplázalo con el nombre deseado). Asegúrate de usar el mismo namespace en todos los comandos `kubectl`.

### Despliegue

1. **Crear secret de Hugging Face:**
   ```bash
   kubectl create secret generic huggingface-token \
     --from-literal=token="tu_token_huggingface" \
     -n eca-services
   ```
   **Nota**: Si cambiaste el namespace, reemplaza `eca-services` con tu namespace en todos los comandos.

2. **Aplicar manifiestos:**
   ```bash
   kubectl apply -f k8s-qwen-deployment.yaml
   ```

3. **Verificar:**
   ```bash
   kubectl get pods -n eca-services
   kubectl logs -f deployment/qwen-inference -n eca-services -c qwen-inference
   ```

### Pruebas

```bash
# Port forward
kubectl port-forward -n eca-services svc/qwen-inference-service 8081:8081 8082:8082

# Health checks
curl http://localhost:8081/health  # Qwen
curl http://localhost:8082/health  # Embeddings
```

## 🐳 Docker Compose

### Configuración

1. **Crear archivo `.env`:**
   ```bash
   echo "HF_TOKEN=tu_token_huggingface" > .env
   ```

2. **Crear directorio de caché:**
   ```bash
   mkdir -p data/qwen-model-cache
   ```

### Uso

```bash
# Iniciar
docker-compose up -d

# Logs
docker-compose logs -f

# Detener
docker-compose down
```

### Puertos

- **Qwen**: `http://localhost:8081`
- **Embeddings**: `http://localhost:8082`

## 📊 Recursos

- **GPU**: 1 NVIDIA GPU
- **Memoria**: Qwen 12-24Gi, Embeddings 1-2Gi
- **CPU**: Qwen 3-6 cores, Embeddings 0.5-1 core
- **Storage**: 80Gi para caché del modelo Qwen

## 🐛 Troubleshooting

**Pod/Pod en Pending:**
```bash
kubectl describe pod <pod-name> -n eca-services
kubectl get nodes -l accelerator=nvidia-gpu
```

**Verificar secret:**
```bash
kubectl get secret huggingface-token -n eca-services
```

**Verificar GPU en Docker:**
```bash
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

**Nota**: La primera descarga del modelo puede tardar 10-30 minutos.

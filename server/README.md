# Docker Compose Deployment Guide

Esta guía describe cómo desplegar y probar la aplicación usando Docker Compose.

## 📋 Prerrequisitos

- Docker (v20.10+)
- Docker Compose (v1.29+)
- Acceso a internet para descargar imágenes
- Acceso a GitHub Container Registry (GHCR) para obtener las imágenes `ghcr.io/creai-mx/eca-backend:v0.1` y `ghcr.io/creai-mx/eca-frontend:v0.1`

**Nota sobre autenticación GHCR**: Si las imágenes son privadas, necesitarás autenticarte con Docker:

```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
```

## 🚀 Pasos de Despliegue

### 1. Configurar Variables de Entorno

Edita el archivo `docker-compose.yaml` y configura las variables de entorno necesarias para backend y frontend.

A continuación se muestra una tabla con las variables de entorno necesarias. Los valores por defecto están configurados para funcionar con los servicios locales.

| Variable                        | Descripción                                                                     | Valor por Defecto                |
|----------------------------------|---------------------------------------------------------------------------------|-----------------------------------|
| **MONGODB_HOST**                | Host del servidor de MongoDB                                                    | `mongodb`                          |
| **MONGODB_PORT**                | Puerto de conexión del servidor de MongoDB                                      | `27017`                            |
| **MONGODB_DATABASE**            | Nombre de la base de datos a utilizar en MongoDB                                 | `eca_db`                           |
| **MONGODB_USER**                | Usuario para autenticación en MongoDB                                           | `admin`                            |
| **MONGODB_PASSWORD**            | Contraseña de MongoDB                                                           | `secure_password`                 |
| **CHROMA_HOST**                 | Host del servidor de ChromaDB                                                   | `chromadb`                         |
| **CHROMA_PORT**                 | Puerto para ChromaDB                                                            | `8000`                             |
| **CHROMADB_DEFAULT_URI**        | URL base de ChromaDB                                                            | `http://chromadb:8000`             |
| **ORIGINS**                     | Lista de dominios permitidos para peticiones CORS                              | `http://localhost:3000`            |
| **LLM_URL**                     | URL del endpoint del motor de inferencia (LLM)                                   | `http://host.docker.internal:8081` |
| **HTTP_EMBEDDING_SERVER_URL**   | URL del motor de embeddings (texto a vectores)                                  | `http://host.docker.internal:8082` |
| **SMTP_HOST**                   | Servidor SMTP para envío de emails                                               | `localhost`                         |
| **SMTP_PORT**                   | Puerto del servidor SMTP                                                         | `1025`                              |
| **SMTP_USER**                   | Usuario para autenticación SMTP                                                 | `noreply@silia.test`                |
| **SMTP_PASS**                   | Contraseña SMTP                                                                  | `secure_password`                   |
| **EMAIL_FROM**                  | Dirección de correo electrónico remitente para envío de emails                    | `noreply@silia.test`                |
| **RABBITMQ_URL**                | URL de conexión a RabbitMQ (sin credenciales)                                     | `amqp://rabbitmq:5672`               |
| **RABBITMQ_USER**               | Usuario para autenticación en RabbitMQ                                            | `user`                               |
| **RABBITMQ_PASS**               | Contraseña para autenticación en RabbitMQ                                         | `pass`                               |
| **REACT_APP_API_HOST**          | URL base de la API backend (Node.js)                                             | `http://localhost:3001`            |
| **REACT_APP_WS_URL**            | URL del servidor WebSocket (Node.js backend)                                     | `ws://localhost:3001`               |
| **REACT_APP_APP_DOMAIN**        | Dominio de la aplicación para el frontend                                         | `localhost`                         |

**Notas importantes**:
- Para conectar con servicios LLM externos (como los de `LLM_inference`), usa `http://host.docker.internal:8081` para acceder al host desde el contenedor.
- Las variables `REACT_APP_*` se inyectan en tiempo de build del frontend.

### 2. Iniciar los Servicios

```bash
# Iniciar todos los servicios
docker-compose up -d

# Ver logs en tiempo real
docker-compose logs -f

# Ver logs de un servicio específico
docker-compose logs -f backend
docker-compose logs -f frontend
```

### 3. Verificar el Despliegue

```bash
# Ver el estado de los servicios
docker-compose ps

# Verificar que los servicios estén saludables
curl http://localhost:3001/health  # Backend
curl http://localhost:3000          # Frontend
```

## 🧪 Pruebas

### Verificar el Estado de los Servicios

```bash
# Ver todos los servicios
docker-compose ps

# Ver logs del backend
docker-compose logs -f backend

# Ver logs del frontend
docker-compose logs -f frontend
```

### Probar el Backend

```bash
# Probar el endpoint de health
curl http://localhost:3001/health

# Probar el WebSocket del backend
# Si no tienes wscat instalado, primero instálalo con:
npm install -g wscat

# Luego conecta al WebSocket del backend:
wscat -c ws://localhost:3001
```

### Probar el Frontend

```bash
# Abrir en el navegador
# http://localhost:3000
```

## 🐛 Troubleshooting

### Servicios no inician

```bash
# Ver logs del servicio
docker-compose logs backend
docker-compose logs frontend

# Ver detalles del contenedor
docker-compose ps
docker inspect <container-name>
```

### Errores de conexión a MongoDB

Verifica que MongoDB esté corriendo y saludable:

```bash
# Verificar estado de MongoDB
docker-compose ps mongodb

# Ver logs de MongoDB
docker-compose logs mongodb

# Probar conexión manual
docker-compose exec mongodb mongosh -u admin -p secure_password
```

### Errores de conexión a ChromaDB

Verifica que ChromaDB esté corriendo:

```bash
# Verificar estado de ChromaDB
docker-compose ps chromadb

# Ver logs de ChromaDB
docker-compose logs chromadb

# Probar endpoint de health
curl http://localhost:8000/api/v2/heartbeat
```

### Variables de entorno no configuradas

Verifica que todas las variables estén configuradas en `docker-compose.yaml`:

```bash
# Ver variables de entorno del contenedor
docker-compose exec backend env | grep MONGODB
docker-compose exec frontend env | grep REACT_APP
```

### Health checks fallando

Los health checks pueden fallar si los servicios aún se están iniciando. Espera unos minutos y verifica:

```bash
# Verificar health checks
docker-compose ps

# Probar manualmente
curl http://localhost:3001/health  # Backend
curl http://localhost:3000          # Frontend
```

## 🛠️ Comandos Útiles

```bash
# Detener todos los servicios
docker-compose down

# Detener y eliminar volúmenes (elimina datos)
docker-compose down -v

# Reiniciar un servicio específico
docker-compose restart backend
docker-compose restart frontend

# Reconstruir y reiniciar servicios
docker-compose up -d --build

# Ejecutar comandos dentro de un contenedor
docker-compose exec backend sh
docker-compose exec frontend sh
```

## 📊 Puertos

- **3000**: Frontend (React)
- **3001**: Backend (Node.js)
- **27017**: MongoDB
- **8000**: ChromaDB
- **5672**: RabbitMQ (AMQP)
- **15672**: RabbitMQ Management UI

## 🔒 Seguridad

- Cambia las contraseñas por defecto en producción
- No expongas puertos sensibles públicamente
- Usa variables de entorno para información sensible
- Considera usar Docker secrets para producción


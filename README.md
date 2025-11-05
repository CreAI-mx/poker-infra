# 🎰 Poker Infrastructure Project

## 📖 Project Overview

This repository contains the complete infrastructure for a poker application with advanced AI capabilities. The system is designed as a Kubernetes-based microservices architecture, providing scalable and robust backend services for intelligent poker applications.

### 🎯 Main Features

- **🤖 AI-Powered Chatbot**: Intelligent responses using LLM inference servers
- **🔍 Semantic Search**: Vector-based knowledge retrieval with ChromaDB
- **📊 Real-time Communication**: WebSocket-based messaging system
- **🔄 Message Queue**: RabbitMQ for reliable message processing
- **🚀 Scalable Architecture**: Horizontal scaling with Kubernetes

---

## 🏗️ Architecture Overview

The project consists of multiple services that can be deployed using Kubernetes or Docker Compose:

### 1. **Server Infrastructure** (`server/`)
- **Backend**: Node.js application server (port 3001)
- **Frontend**: React application (port 3000)
- **MongoDB**: Database service
- **ChromaDB**: Vector database for semantic search
- **Deployment Options**: Kubernetes manifests (`server/node/`) and Docker Compose (`server/`)

### 2. **LLM Inference** (`LLM_inference/`)
- **Qwen Inference**: LLM inference server (port 8081)
- **Embeddings Inference**: Text embeddings server (port 8082)
- **Deployment Options**: Kubernetes and Docker Compose

---

## 📁 Project Structure

```
poker-infra/
├── 📂 server/                    # Main server infrastructure
│   ├── 📄 README.md              # Docker Compose deployment guide
│   ├── 📦 docker-compose.yaml    # Docker Compose configuration
│   └── 📂 node/                  # Node.js application
│       ├── 📄 README.md          # Kubernetes deployment guide
│       ├── 📦 k8s-deployment.yaml # Kubernetes manifests
│       └── 📦 k8s-secrets-backend.yaml # Secrets template
├── 📂 LLM_inference/             # LLM inference services
│   ├── 📄 README.md              # Deployment guide (K8s + Docker Compose)
│   ├── 📦 k8s-qwen-deployment.yaml # Kubernetes manifests
│   └── 📦 docker-compose.yml      # Docker Compose configuration
└── 📖 README.md                  # This file
```

---

## 🚀 Quick Start

> **📖 Para instrucciones detalladas de despliegue y configuración:**
> - **Kubernetes**: Consulta el [README.md de server/node](server/node/README.md)
> - **Docker Compose**: Consulta el [README.md de server](server/README.md)
> - **LLM Inference**: Consulta el [README.md de LLM_inference](LLM_inference/README.md)





---

## 👥 Team

**Maintainer**: CreAI Team  
**Version**: 1.0.1  
**Last Updated**: November 2025


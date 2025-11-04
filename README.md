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

The project consists of Kubernetes-deployed services:

### 1. **Server Infrastructure** (`server/node/`)
- **Backend**: Node.js application server (port 3001)
- **Frontend**: React application (port 3000)
- **Kubernetes**: Complete deployment manifests with Services, Deployments, and Ingress

---

## 📁 Project Structure

```
poker-infra/
├── 📂 server/                    # Main server infrastructure
│   ├── 📂 node/                  # Node.js application
│   │   ├── 📄 README.md          # Detailed deployment guide
│   │   ├── 📦 k8s-deployment.yaml # Kubernetes manifests
│   │   └── 📦 k8s-secrets-backend.yaml # Secrets template
└── 📖 README.md                  # This file
```

---

## 🚀 Quick Start

> **📖 Para instrucciones detalladas de despliegue y configuración, consulta el [README.md de server/node](server/node/README.md)**





---

## 👥 Team

**Maintainer**: CreAI Team  
**Version**: 1.0.1  
**Last Updated**: November 2025


# Practical 11: Containerization with Docker and Docker Compose

**Course**: Advanced Web Development Frameworks (ITUE301)  
**Department**: Department of Computer Science and Engineering  
**Institute**: Chandubhai S. Patel Institute of Technology (CSPIT), CHARUSAT  
**Academic Year**: 2025–2026  
**Student Name**: Manav Lathiya  
**Student ID**: 24CS048  
**GitHub Repository**: [24CS048Manav/ITUE301-AWDF_Practical11](https://github.com/24CS048Manav/ITUE301-AWDF_Practical11)

---

## 🎯 Practical Objective
To containerize the complete full-stack Task Management application (React frontend, Node.js/Express backend, MongoDB database) into isolated microservice containers and orchestrate them using **Docker Compose** on an isolated bridge network with persistent volume storage.

---

## 🏗️ Architecture & Docker Compose Topology

```
docker-compose.yml
 ├── frontend (React, built + served via Vite preview, port 5173:5173)
 ├── backend (Node/Express with EventEmitter & Caching, port 5000:5000)
 └── mongodb (official mongo image with named volume, port 27017:27017)

Network: app-network (Docker bridge network for internal DNS resolution)
Flow: frontend ──► backend (http://localhost:5000) ──► mongodb (mongodb://mongodb:27017)

All 3 services build and start with a single command:
$ docker-compose up --build
```

---

## 🚀 Quick Start Guide

### Prerequisites
- Docker Engine / Docker Desktop installed:
  ```bash
  docker --version
  docker compose version
  ```

### 1. Launch All Services
Run from the root directory:
```bash
docker-compose up --build
```
*(Or run in detached background mode: `docker-compose up --build -d`)*

### 2. Verify Running Containers
```bash
docker ps
```
Expected running container list:
- `task_manager_frontend` (Port 0.0.0.0:5173->5173/tcp)
- `task_manager_backend` (Port 0.0.0.0:5000->5000/tcp)
- `task_manager_mongodb` (Port 0.0.0.0:27017->27017/tcp)

### 3. Access Services
- **React Frontend**: [http://localhost:5173](http://localhost:5173)
- **Node.js Express Backend API**: [http://localhost:5000](http://localhost:5000)
- **Container Health Endpoint**: [http://localhost:5000/health](http://localhost:5000/health)

### 4. Stop All Services
```bash
docker-compose down
```

---

## 📁 Repository Structure
```text
Prac11/
├── docker-compose.yml           # Orchestration definition for all 3 containers
├── .dockerignore                # Root Docker ignore context
├── .gitignore                   # Git exclusions (node_modules, .env, dist)
├── README.md                    # Practical overview & orchestration guide
├── docs/
│   └── CONTAINERIZATION_REPORT.md # Technical report with answers to Viva questions
├── backend/
│   ├── Dockerfile               # Node.js Express backend container recipe
│   ├── .dockerignore            # Excludes node_modules and .env
│   ├── server.js                # Express REST API, health probes, MongoDB connection
│   ├── events.js                # EventEmitter class and telemetry
│   ├── listeners.js             # Background event workers (task-created, task-deleted, error)
│   ├── cache.js                 # In-memory query caching
│   ├── package.json             # Backend dependencies
│   └── models/                  # Mongoose models (User, Task)
└── frontend/
    ├── Dockerfile               # Multi-stage production container build (Vite)
    ├── .dockerignore            # Excludes node_modules, dist, .env
    ├── package.json             # Frontend dependencies
    ├── vite.config.js           # Host binding for Docker container forwarding
    └── src/
        ├── App.jsx              # Main SPA router with Practical 11 status banner
        ├── api.js               # Central API service with health checks
        └── components/
            ├── DockerMonitor.jsx# Live Docker container status & architecture panel
            ├── CacheMonitor.jsx # Cache hit/miss observability
            ├── EventMonitor.jsx # Live async event stream dashboard
            └── TaskManager.jsx  # Authenticated task management CRUD
```

---

## 💡 Supplementary Problems Implementation

1. **Context Exclusion via `.dockerignore`**:
   - Excludes heavy local `node_modules`, secret `.env` credentials, build output `dist`, and `.git` from the Docker daemon build context.
   - Prevents architecture-dependent native binaries from breaking across host OS and Linux containers.

2. **Multi-Stage Build for React Frontend**:
   - `Stage 1 (build)`: Installs full development toolchain and compiles optimized production assets with Vite.
   - `Stage 2 (production)`: Extracts only the compiled `/dist` directory and production dependencies, drastically shrinking container image size.

3. **Persistent Volume for MongoDB**:
   - Configured named volume `mongodb_data:/data/db`.
   - Data persists safely even when containers are stopped, removed, or rebuilt.

---

## 📝 Answers to Key Questions

1. **Why reference MongoDB by service name (`mongodb`) instead of `localhost`?**  
   Each container possesses its own isolated network interface and loopback address (`127.0.0.1`). Attaching services to the user-defined bridge network (`app-network`) allows Docker's internal DNS to resolve `mongodb` to the specific IP assigned to the MongoDB container.

2. **Difference between `EXPOSE` and `ports` mapping?**  
   `EXPOSE` is informational metadata documenting intended container ports without publishing them to the outside world. `ports` actively maps a port from the host machine to the container, enabling outside network access.

3. **Why is copying `node_modules` into a Docker image bad practice?**  
   It introduces potential binary incompatibility between host OS (e.g. Windows) and container OS (Linux Alpine), bloats build context transfer, and breaks Docker layer caching.

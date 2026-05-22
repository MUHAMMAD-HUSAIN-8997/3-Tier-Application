# Production-Ready Three-Tier DevOps Infrastructure Stack

A mid-level DevOps portfolio project showcasing containerization, security isolation, and data persistence for a full-stack MERN (React, Node.js/Express, MongoDB) application. 

This project bypasses unoptimized development servers to implement an enterprise-grade container strategy, utilizing multi-stage builds, network segmentation, and volume mapping.

---

## 🏗️ Architecture & Component Breakdown
┌───────────────────────────────┐│      Public Web Browser       │└───────────────┬───────────────┘│ (Port 80)▼┌───────────────────────────────────────────────────────────────────────────┐│ FRONTEND NETWORK (frontend-net)                                           ││                                                                           ││   ┌──────────────────────────────┐     ┌──────────────────────────────┐   ││   │      frontend container      │     │      backend container       │   ││   │       (React + Nginx)        ├────►│       (Node.js/Express)      │   ││   └──────────────────────────────┘     └──────────────┬───────────────┘   │└───────────────────────────────────────────────────────┼───────────────────┘│┌───────────────────────────────────────────────────────┼───────────────────┐│ BACKEND NETWORK (backend-net)                         │                   ││                                                       ▼                   ││                                        ┌──────────────────────────────┐   ││                                        │         db container         │   ││                                        │          (MongoDB)           │   ││                                        └──────────────┬───────────────┘   │└───────────────────────────────────────────────────────┼───────────────────┘▼┌──────────────────────────────┐│  Persistent Volume Storage   ││        (mongo-data)          │└──────────────────────────────┘
### 1. Frontend (React + Nginx)
* **Production Build Strategy:** Utilizes a **Multi-Stage Dockerfile**. Stage 1 installs development dependencies and compiles static assets. Stage 2 discards the bulky Node.js runtime environment and uses a lightweight, secure **Nginx Alpine** base image to serve production assets.
* **Result:** Image footprint reduced from **~1.2 GB** down to **~45 MB**, significantly shrinking the attack surface and increasing deployment speeds.

### 2. Backend API (Node.js & Express)
* **Optimization:** Implements optimized Docker layer caching by splitting dependency installation (`package.json`) from source code transfers. Code updates compile in seconds rather than triggering full package downloads.
* **Environment Configuration:** Receives its database connection strings dynamically through standard environment variables (`MONGODB_URI`), making the app fully portable between development, staging, and production clusters.

### 3. Database (MongoDB)
* **Persistence Layer:** Uses an ephemeral-safe state strategy by mapping a managed named volume (`mongo-data`) to the host. Container recreation, crashes, or runtime infrastructure scaling will not trigger data loss.

---

## 🔒 Advanced DevOps Patterns Implemented

* **Network Segmentation & Isolation:** To comply with the security principle of **Least Privilege**, the environment is split into two custom bridges: `frontend-net` and `backend-net`. The database container is entirely shielded within the private backend network; it has no route to the frontend or the public internet, preventing direct injection attacks.
* **Distroless/Alpine Base Images:** Built strictly using official Linux Alpine distributions (`node:18-alpine` and `nginx:alpine`) to strip out unneeded packages, commands, and active shell vulnerabilities.

---

## 🚀 Getting Started & Deployment

### Prerequisites
* Ensure you have [Docker Desktop](https://docker.com) or Docker Engine + Compose installed on your host system.

### Deployment Instructions
1. Clone your configured repository to your local engine:
   ```bash
   git clone <your-repository-url>
   cd react-express-mongodb
   ```

2. Boot the entire three-tier infrastructure stack in detached mode:
   ```bash
   docker compose up -d --build
   ```

3. Verify service stability and runtime functionality:
   * **Frontend Interface:** Navigate to `http://localhost:80`
   * **Backend REST API REST endpoint:** Verify health via `http://localhost:5000`

4. Tearing down the stack safely while preserving data:
   ```bash
   docker compose down
   ```

---

## 🛠️ Verification Commands Used

### 1. Confirm Network Isolation
To prove the database cannot communicate with the frontend network, execute a ping command directly from inside the database container to the frontend:
```bash
docker compose exec db ping frontend
```
*Expected Output:* `ping: bad address 'frontend'` (Confirming the frontend is unreachable and network segmentation is functioning correctly).

### 2. Confirm Data Persistence
1. Open the application UI and add multiple tasks.
2. Force kill and wipe the running stack: `docker compose down`
3. Bring the architecture back up: `docker compose up -d`
4. Refresh your browser—all previously saved tasks persist seamlessly from the host storage volume.

---
# DD Task - MEAN Stack CRUD Application

A full-stack MEAN (MongoDB, Express, Angular, Node.js) application containerized with Docker and deployed using a CI/CD pipeline via GitHub Actions.

---

## 📁 Project Structure

```
crud-dd-task-mean-app/
├── backend/                    # Node.js + Express REST API
│   ├── app/
│   │   ├── config/db.config.js
│   │   ├── controllers/tutorial.controller.js
│   │   ├── models/
│   │   └── routes/turorial.routes.js
│   ├── server.js
│   ├── package.json
│   └── Dockerfile
├── frontend/                   # Angular 15 SPA
│   ├── src/
│   ├── angular.json
│   ├── package.json
│   ├── Dockerfile
│   └── nginx-frontend.conf
├── nginx/
│   └── nginx.conf              # Reverse proxy config
├── .github/
│   └── workflows/
│       └── ci-cd.yml           # GitHub Actions CI/CD pipeline
├── docker-compose.yml
└── README.md
```

---

## 🏗️ Architecture

```
Internet → Port 80
              │
          [Nginx Reverse Proxy]
          /                   \
  /api/* → [Backend:8080]    /* → [Frontend:80]
                │
          [MongoDB:27017]
```

- **Nginx** acts as the entry point on port 80
- **Frontend** (Angular) is served by Nginx inside the frontend container
- **Backend** (Node.js/Express) handles all `/api/` requests
- **MongoDB** stores tutorial data persistently via Docker volumes

---

## ⚙️ Prerequisites

- Docker & Docker Compose installed
- Git
- Docker Hub account
- GitHub account
- Ubuntu VM on AWS/Azure (for deployment)

---

## 🚀 Local Setup & Deployment

### 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/crud-dd-task-mean-app.git
cd crud-dd-task-mean-app
```

### 2. Build Docker Images Locally

```bash
docker build -t <your-dockerhub-username>/dd-backend:latest ./backend
docker build -t <your-dockerhub-username>/dd-frontend:latest ./frontend
```

### 3. Push Images to Docker Hub

```bash
docker login
docker push <your-dockerhub-username>/dd-backend:latest
docker push <your-dockerhub-username>/dd-frontend:latest
```

### 4. Run with Docker Compose

```bash
export DOCKERHUB_USERNAME=<your-dockerhub-username>
docker compose up -d
```

The application will be accessible at: **http://localhost**

---

## ☁️ VM Deployment (Ubuntu)

### Step 1 – Provision Ubuntu VM

Create an Ubuntu 22.04 VM on AWS/Azure. Open inbound ports: **22** (SSH), **80** (HTTP).

### Step 2 – Install Docker on VM

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo usermod -aG docker $USER
newgrp docker
```

### Step 3 – Clone Repo & Deploy on VM

```bash
git clone https://github.com/<your-username>/crud-dd-task-mean-app.git
cd crud-dd-task-mean-app
export DOCKERHUB_USERNAME=<your-dockerhub-username>
docker compose up -d
```

Access the app at: **http://\<VM-PUBLIC-IP\>**

---

## 🔄 CI/CD Pipeline (GitHub Actions)

### How It Works

1. Developer pushes code to the `main` branch
2. GitHub Actions automatically:
   - Builds new Docker images for frontend and backend
   - Pushes them to Docker Hub with `latest` and commit SHA tags
   - SSHs into the VM and runs `docker compose down && docker compose up -d`

### Required GitHub Secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret Name | Description |
|-------------|-------------|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | Docker Hub access token |
| `VM_HOST` | Public IP of your Ubuntu VM |
| `VM_USER` | SSH username (e.g., `ubuntu`) |
| `VM_SSH_KEY` | Full contents of your private SSH key |

---

## 🐳 Docker Images

| Service | Image |
|---------|-------|
| Backend | `<dockerhub-username>/dd-backend:latest` |
| Frontend | `<dockerhub-username>/dd-frontend:latest` |
| MongoDB | `mongo:6.0` (official) |
| Nginx Proxy | `nginx:alpine` (official) |

---

## 🌐 Nginx Configuration

The Nginx reverse proxy listens on port 80 and routes:
- `/api/*` → Backend (Node.js on port 8080)
- `/*` → Frontend (Angular on port 80)

---

## 🩺 Health Check

```bash
docker compose ps
curl http://localhost/api/tutorials
docker compose logs -f
```

---

## 📝 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/tutorials` | Get all tutorials |
| GET | `/api/tutorials/:id` | Get tutorial by ID |
| POST | `/api/tutorials` | Create new tutorial |
| PUT | `/api/tutorials/:id` | Update tutorial |
| DELETE | `/api/tutorials/:id` | Delete tutorial |
| DELETE | `/api/tutorials` | Delete all tutorials |
| GET | `/api/tutorials?title=xyz` | Search by title |

---

## 🔧 Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MONGO_URL` | `mongodb://localhost:27017/dd_db` | MongoDB connection string |
| `PORT` | `8080` | Backend server port |

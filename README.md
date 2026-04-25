# DevOps Lab 2026 – Multi-tier Architecture & CI/CD

This project implements a complete and automated infrastructure using Docker, Ansible, and GitHub Actions. The application is composed of a Vue.js/Nginx frontend, a Java Spring Boot REST API, and a PostgreSQL database.

---

## 🏗 Project Architecture

The architecture follows a **4-tier model**, where the HTTP server acts as a single entry point (**Reverse Proxy**).

- **Frontend / Proxy (Nginx)**: Serves static frontend files and redirects `/api` requests to the backend.  
- **Backend (Spring Boot)**: REST API handling business logic.  
- **Database (PostgreSQL)**: Persistent data storage using a Docker volume.  
- **Network (Docker Network)**: An internal network (`app-network`) ensures isolated communication between containers.

---

## 🚀 Key Features

### 1. Data Persistence

A named volume `pgdata` is used to ensure that database data is not lost when the container restarts.

**Fix applied:**  
Added an Ansible `docker_volume` task to ensure the volume is created before starting the container.

---

### 2. Availability (Healthchecks)

The database container includes a healthcheck (`pg_isready`).  
The backend waits until the database is marked as **healthy** before attempting to connect, preventing startup connection errors.

---

### 3. Reverse Proxy (Frontend)

The Nginx server is configured via `default.conf` to handle routing:

- `location /` → Serves the web interface  
- `location /api/` → Proxies requests to `http://student-api:8080/`

---

## 🤖 CI/CD Pipeline (GitHub Actions)

The workflow `.github/workflows/main.yml` automates the entire lifecycle:

- **Tests**: Run Maven unit tests  
- **Analysis**: Code quality analysis using SonarCloud  
- **Build & Push**: Build Docker images for all 4 services and push them to DockerHub  
- **Deploy**: Automatic deployment via Ansible on a remote instance  

**Manual deployment note:**  
The deployment job can be triggered manually using the **Run workflow** button thanks to the `workflow_dispatch` event.

---

## 🛠 Installation and Deployment

### Prerequisites

- A Linux instance (Takima) accessible via SSH  
- Docker and Ansible installed on the deployment machine  
- GitHub secrets configured:
  - `DOCKER_HUB_TOKEN` : token docker hub
  - `DOCKER_HUB_USERNAME`: username docker hub
  - `SSH_PRIVATE_KEY` : private key to connect to takima's instance
  - `SERVER_HOST` : nom du serveur takima
  - `SONAR_TOKEN`: token de Sonar Cloud
  - `ANSIBLE_INVENTORY`

---

### Deployment via Ansible (Local)

To deploy manually from WSL:

```bash
ansible-playbook -i ansible/inventories/setup.yml ansible/playbook.yml --private-key ./id_rsa
```

### Author 👩‍💻

- Laura-DAMAS
- Group : ING2-APP-BDML1
- DevOps Lab Project – Mars-Avril 2026
- Course: DevOps
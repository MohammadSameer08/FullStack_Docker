# Fullstack Docker Application

A full-stack application with backend, frontend, and database services containerized using Docker.

## Project Structure

```
.
├── docker-compose.yml       # Docker orchestration configuration
├── backend/                 # Node.js Express backend service
│   ├── Dockerfile
│   ├── index.js
│   └── package.json
├── frontend/                # React + Vite frontend service
│   ├── Dockerfile
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   ├── src/
│   └── public/
└── README.md
```

## Services Overview

### Docker Compose Configuration

The `docker-compose.yml` orchestrates three main services:

#### 1. **Backend Service**
- **Image**: Built from `./backend/Dockerfile`
- **Container**: `backend`
- **Port**: `5001:5000` (Host:Container)
- **Framework**: Node.js (port 5000 internally)
- **Environment Variables**:
  - `DB_HOST=database` (PostgreSQL service hostname)
  - `DB_USER=myuser`
  - `DB_PASSWORD=mypassword`
  - `DB_NAME=mydatabase`
- **Dependencies**: Depends on `database` service
- **Network**: `my-custom-network`

#### 2. **Frontend Service**
- **Image**: Built from `./frontend/Dockerfile`
- **Container**: `frontend`
- **Port**: `3000:80` (Host:Container)
- **Framework**: React + Vite served via Nginx
- **Dependencies**: Depends on `backend` service
- **Network**: `my-custom-network`

#### 3. **Database Service**
- **Image**: `postgres:latest`
- **Container**: `database`
- **Database**: PostgreSQL
- **Credentials**:
  - User: `myuser`
  - Password: `mypassword`
  - Database: `mydatabase`
- **Volume**: `pgData:/var/lib/postgresql/data` (persistent storage)
- **Network**: `my-custom-network`

## Dockerfiles

### Backend Dockerfile
```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["npm", "start"]
```
- Uses lightweight `node:22-alpine` image
- Installs dependencies from package.json
- Runs the Node.js application on port 5000

### Frontend Dockerfile
```dockerfile
# build stage
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# serve stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
- **Multi-stage build** for optimized image size
- **Build stage**: Compiles React app using Vite
- **Serve stage**: Serves built files via Nginx on port 80

## Networking

All services are connected via a custom bridge network `my-custom-network`:
- Services can communicate using container names as hostnames
- Backend connects to database as `DB_HOST=database`
- Frontend connects to backend via the custom network

## Volumes

- **pgData**: Persistent PostgreSQL database storage
  - Path in container: `/var/lib/postgresql/data`
  - Survives container restarts

## Getting Started

### Start all services
```bash
docker compose up
```

### Stop all services
```bash
docker compose down
```

### Rebuild images
```bash
docker compose up --build
```

### View logs
```bash
docker compose logs -f
```

## Port Mapping

| Service | Container Port | Host Port | Access URL |
|---------|---|---|---|
| Frontend | 80 | 3000 | http://localhost:3000 |
| Backend | 5000 | 5001 | http://localhost:5001 |
| Database | 5432 | N/A | postgres://myuser@database:5432/mydatabase |

## Environment Configuration

The backend connects to PostgreSQL using environment variables defined in `docker-compose.yml`. The database credentials match the PostgreSQL service configuration to enable seamless connection.

### Connection String (Backend → Database)
```
postgres://myuser:mypassword@database:5432/mydatabase
```

## Notes

- The frontend uses a **multi-stage Docker build** to reduce final image size
- Service startup order is managed via `depends_on`
- The custom network allows services to communicate by container name
- PostgreSQL data persists in the `pgData` volume even after container shutdown

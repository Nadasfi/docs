# Docker Setup

Nadas.fi is designed with Docker-first architecture, providing consistent environments across development, staging, and production.

## Quick Start

```bash
# Clone repository
git clone <repository-url>
cd nadas/backend

# Configure environment
cp .env.example .env.development

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f api
```

## Docker Architecture

### Service Overview

```yaml
services:
  api:              # FastAPI application
  celery-worker:    # Background task processing
  celery-beat:      # Task scheduling
  db:              # PostgreSQL database
  redis:           # Cache and message broker
  nginx:           # Reverse proxy (production)
  prometheus:      # Metrics collection (monitoring profile)
  grafana:         # Dashboard (monitoring profile)
```

### Network Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Nginx     │    │   API       │    │   Worker    │
│   :80/443   │◄───┤   :8000     │    │   (Celery)  │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
              ┌─────────────▼─────────────┐
              │      nadas-network        │
              │    (172.20.0.0/16)       │
              └─────────────┬─────────────┘
                           │
                 ┌─────────┴─────────┐
                 │                   │
        ┌────────▼────────┐ ┌────────▼────────┐
        │   PostgreSQL    │ │     Redis       │
        │     :5432       │ │     :6379       │
        └─────────────────┘ └─────────────────┘
```

## Docker Compose Configuration

### Development Setup

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
    container_name: nadas-api
    ports:
      - "8000:8000"
    environment:
      - ENVIRONMENT=development
    env_file:
      - .env.development
    volumes:
      - .:/app
      - ./logs:/app/logs
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
```

### Production Configuration

```yaml
# docker-compose.production.yml
version: '3.8'

services:
  api:
    build:
      target: production
    restart: unless-stopped
    environment:
      - ENVIRONMENT=production
    env_file:
      - .env.production
    volumes:
      - ./logs:/app/logs
      - ./static:/app/static
    healthcheck:
      test: ["CMD", "python", "/app/healthcheck.py"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Dockerfile

### Multi-stage Build

```dockerfile
# Base stage
FROM python:3.11-slim as base

WORKDIR /app
RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Dependencies stage
FROM base as dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Development stage
FROM dependencies as development
COPY . .
ENV ENVIRONMENT=development
CMD ["uvicorn", "app.main:app", "--reload", "--host", "0.0.0.0", "--port", "8000"]

# Production stage  
FROM dependencies as production
COPY . .
ENV ENVIRONMENT=production
RUN useradd --create-home --shell /bin/bash nadas
USER nadas
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Environment Configuration

### Development Environment

```bash
# .env.development
ENVIRONMENT=development
DEBUG=true
DATABASE_URL=postgresql+asyncpg://nadas:nadas123@db:5432/nadas_development
REDIS_URL=redis://redis:6379/0
HYPERLIQUID_NETWORK=testnet
LOG_LEVEL=DEBUG
```

### Production Environment

```bash
# .env.production
ENVIRONMENT=production
DEBUG=false
DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/nadas_production
REDIS_URL=redis://redis:6379/0
HYPERLIQUID_NETWORK=mainnet
LOG_LEVEL=INFO
```

## Service Profiles

### Development Profile (Default)

```bash
# Start development services
docker-compose up -d

# Services included:
# - api (with hot reload)
# - db (PostgreSQL)
# - redis
# - celery-worker
# - celery-beat
```

### Production Profile

```bash
# Start production services
docker-compose --profile production up -d

# Additional services:
# - nginx (reverse proxy)
# - SSL termination
# - Load balancing
```

### Monitoring Profile

```bash
# Start with monitoring
docker-compose --profile monitoring up -d

# Additional services:
# - prometheus (metrics collection)
# - grafana (dashboards)
```

## Volume Management

### Persistent Volumes

```yaml
volumes:
  postgres-data:        # Database persistence
    driver: local
  redis-data:          # Redis persistence  
    driver: local
  celery-beat-data:    # Scheduler data
    driver: local
  prometheus-data:     # Metrics data
    driver: local
  grafana-data:       # Dashboard configuration
    driver: local
```

### Bind Mounts

```yaml
volumes:
  - ./logs:/app/logs              # Application logs
  - ./static:/app/static          # Static files
  - ./media:/app/media            # User uploads
  - ./nginx/ssl:/etc/nginx/ssl    # SSL certificates
```

## Health Checks

### API Health Check

```python
# healthcheck.py
import httpx
import sys

try:
    response = httpx.get("http://localhost:8000/health", timeout=10.0)
    if response.status_code == 200:
        sys.exit(0)
    else:
        sys.exit(1)
except:
    sys.exit(1)
```

### Database Health Check

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U nadas -d nadas_${ENVIRONMENT}"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### Redis Health Check

```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 10s
  timeout: 5s
  retries: 3
```

## Container Management

### Basic Commands

```bash
# Build and start services
docker-compose build
docker-compose up -d

# View running containers
docker-compose ps

# View logs
docker-compose logs api
docker-compose logs -f --tail=100 api

# Execute commands in containers
docker-compose exec api bash
docker-compose exec db psql -U nadas nadas_development

# Scale services
docker-compose up -d --scale celery-worker=3
```

### Service Management

```bash
# Start specific service
docker-compose start api

# Stop specific service
docker-compose stop api

# Restart service
docker-compose restart api

# Remove containers
docker-compose down

# Remove containers and volumes
docker-compose down -v
```

## Development Workflow

### Hot Reload Setup

```yaml
# Development configuration for hot reload
api:
  volumes:
    - .:/app          # Mount source code
  command: uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Database Development

```bash
# Apply migrations
docker-compose exec api alembic upgrade head

# Create new migration
docker-compose exec api alembic revision --autogenerate -m "Add feature"

# Reset database (development only)
docker-compose exec api alembic downgrade base
docker-compose exec api alembic upgrade head
```

### Background Tasks

```bash
# Monitor Celery workers
docker-compose exec celery-worker celery -A celery_app inspect active

# Purge task queue
docker-compose exec celery-worker celery -A celery_app purge

# Monitor task execution
docker-compose logs -f celery-worker
```

## Production Deployment

### SSL Configuration

```nginx
# nginx/nginx.prod.conf
server {
    listen 443 ssl http2;
    server_name api.nadas.fi;
    
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;
    
    location / {
        proxy_pass http://api:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Resource Limits

```yaml
# Production resource limits
services:
  api:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '1.0'
          memory: 1G
```

### Security Configuration

```yaml
# Security hardening
services:
  api:
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETUID
      - SETGID
```

## Monitoring Setup

### Prometheus Configuration

```yaml
# monitoring/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'nadas-api'
    static_configs:
      - targets: ['api:8000']
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres_exporter:9187']
  - job_name: 'redis'
    static_configs:
      - targets: ['redis_exporter:9121']
```

### Grafana Dashboard

```yaml
# monitoring/grafana/provisioning/dashboards/dashboard.yml
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    options:
      path: /etc/grafana/provisioning/dashboards
```

## Troubleshooting

### Common Issues

#### Port Conflicts
```bash
# Find processes using ports
lsof -i :8000
lsof -i :5432

# Change port mapping
ports:
  - "8001:8000"  # Use different host port
```

#### Volume Permissions
```bash
# Fix volume permissions
sudo chown -R $USER:$USER ./logs
sudo chown -R 999:999 ./postgres-data  # PostgreSQL UID
```

#### Memory Issues
```bash
# Check container memory usage
docker stats

# Increase memory limits
deploy:
  resources:
    limits:
      memory: 4G
```

### Debug Mode

```bash
# Run with debug logging
COMPOSE_LOG_LEVEL=DEBUG docker-compose up

# Access container shell
docker-compose exec api bash

# View container logs
docker-compose logs --details api
```

### Performance Optimization

```yaml
# Optimize for production
services:
  api:
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
  
  db:
    command: postgres -c shared_preload_libraries=pg_stat_statements
    environment:
      - POSTGRES_SHARED_PRELOAD_LIBRARIES=pg_stat_statements
```

This Docker setup provides a robust, scalable foundation for deploying Nadas.fi across different environments with proper monitoring, security, and performance optimizations.
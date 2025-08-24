# Installation

This guide will help you set up the Nadas.fi backend development environment on your local machine.

## Prerequisites

Before installing the Nadas.fi backend, ensure you have the following installed:

### Required Software

- **Python 3.10+** - The backend is built with modern Python features
- **PostgreSQL 13+** - Primary database
- **Redis 6+** - Caching and message broker
- **Git** - Version control

### Optional (for full development experience)

- **Docker & Docker Compose** - For containerized development
- **Node.js 18+** - For frontend development (if working with full stack)

## Installation Methods

Choose one of the following installation methods:

### Method 1: Local Development Setup

#### 1. Clone the Repository

```bash
git clone <repository-url>
cd nadas/backend
```

#### 2. Create Virtual Environment

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate
# On Windows:
venv\Scripts\activate
```

#### 3. Install Python Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

#### 4. Set Up Database

**PostgreSQL Setup:**
```bash
# Install PostgreSQL (macOS with Homebrew)
brew install postgresql
brew services start postgresql

# Create database and user
psql postgres
CREATE DATABASE nadas_development;
CREATE USER nadas WITH PASSWORD 'nadas123';
GRANT ALL PRIVILEGES ON DATABASE nadas_development TO nadas;
\q
```

**Redis Setup:**
```bash
# Install Redis (macOS with Homebrew)
brew install redis
brew services start redis

# Test Redis connection
redis-cli ping
# Should return: PONG
```

#### 5. Configure Environment

```bash
# Copy environment template
cp .env.example .env.development

# Edit the environment file
nano .env.development
```

#### 6. Run Database Migrations

```bash
# Initialize Alembic (if not already done)
alembic init alembic

# Run migrations
alembic upgrade head
```

#### 7. Start the Development Server

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Method 2: Docker Development Setup

#### 1. Clone the Repository

```bash
git clone <repository-url>
cd nadas/backend
```

#### 2. Configure Environment

```bash
cp .env.example .env.development
```

#### 3. Start Services with Docker Compose

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f api

# Stop services
docker-compose down
```

## Verification

After installation, verify everything is working:

### 1. Check API Health

```bash
curl http://localhost:8000/health
```

Expected response:
```json
{
  "status": "healthy",
  "environment": "development",
  "version": "1.0.0",
  "services": {
    "supabase": {"status": "connected"},
    "redis": {"status": "connected"}
  }
}
```

### 2. Check API Documentation

Visit: `http://localhost:8000/docs`

You should see the interactive FastAPI documentation.

### 3. Test Database Connection

```bash
# If using local setup
python -c "from app.core.database import get_db_session; print('Database connected successfully')"
```

## Common Installation Issues

### Python Dependencies

**Issue**: Package installation fails
```bash
# Solution: Update pip and try again
pip install --upgrade pip setuptools wheel
pip install -r requirements.txt
```

### PostgreSQL Connection

**Issue**: Database connection fails
```bash
# Check if PostgreSQL is running
pg_isready -h localhost -p 5432

# If not running, start it:
# macOS with Homebrew:
brew services start postgresql
# Linux with systemd:
sudo systemctl start postgresql
```

### Redis Connection

**Issue**: Redis connection fails
```bash
# Check if Redis is running
redis-cli ping

# If not running, start it:
# macOS with Homebrew:
brew services start redis
# Linux with systemd:
sudo systemctl start redis-server
```

### Port Conflicts

**Issue**: Port 8000 is already in use
```bash
# Find process using port 8000
lsof -i :8000

# Kill the process
kill -9 <PID>

# Or use a different port
uvicorn app.main:app --reload --host 0.0.0.0 --port 8001
```

## Next Steps

Once installation is complete:

1. [Configure your environment](configuration.md)
2. [Follow the quick start guide](quick-start.md)
3. Explore the [API documentation](../api-reference/)

## Development Tools

### Recommended IDE Extensions

**Visual Studio Code:**
- Python
- Pylance
- GitLens
- Docker
- Thunder Client (API testing)

**PyCharm:**
- Built-in Python support
- Database tools
- Docker integration

### Useful Commands

```bash
# Run tests
pytest

# Format code
black app/
isort app/

# Type checking
mypy app/

# Database operations
alembic revision --autogenerate -m "Description"
alembic upgrade head
alembic downgrade -1

# Celery operations (if using background tasks)
celery -A celery_app worker --loglevel=info
celery -A celery_app beat --loglevel=info
```

## Troubleshooting

If you encounter issues during installation, check:

1. **Python version**: Ensure you're using Python 3.10+
2. **Virtual environment**: Make sure it's activated
3. **Database permissions**: Ensure your database user has proper permissions
4. **Port availability**: Check that required ports (8000, 5432, 6379) are available
5. **Environment variables**: Verify all required environment variables are set

For additional help, see our [Development Guide](../development/) or open an issue in the repository.
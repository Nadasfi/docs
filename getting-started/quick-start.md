# Quick Start

Get up and running with Nadas.fi in minutes! This guide assumes you've completed the [installation](installation.md) and [configuration](configuration.md) steps.

## 🚀 Start the Development Server

### Option 1: Local Development

```bash
# Navigate to backend directory
cd nadas/backend

# Activate virtual environment
source venv/bin/activate  # On macOS/Linux
# or
venv\Scripts\activate     # On Windows

# Start the server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Option 2: Docker Development

```bash
# Navigate to backend directory
cd nadas/backend

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f api
```

## ✅ Verify Installation

Once the server is running, verify everything works:

### 1. Health Check

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

### 2. API Documentation

Visit: `http://localhost:8000/docs`

You should see the interactive Swagger UI with all API endpoints.

## 🔐 Authentication Setup

### 1. Create a Test User

```bash
# Using curl
curl -X POST "http://localhost:8000/api/v1/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "wallet_address": "0x1234567890123456789012345678901234567890",
    "username": "testuser",
    "email": "test@example.com"
  }'
```

### 2. Get Access Token

```bash
# Login to get access token
curl -X POST "http://localhost:8000/api/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "wallet_address": "0x1234567890123456789012345678901234567890",
    "signature": "0x...",
    "message": "Login to Nadas.fi"
  }'
```

## 📊 Test Core Features

### 1. Portfolio Data

```bash
# Get portfolio information
curl -X GET "http://localhost:8000/api/v1/portfolio/overview" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### 2. Live Trading (Testnet)

```bash
# Execute a test trade
curl -X POST "http://localhost:8000/api/v1/live-trading/execute-trade" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{
    "symbol": "ETH",
    "side": "buy",
    "size": 100.0,
    "order_type": "market"
  }'
```

### 3. AI Strategy Generation

```bash
# Generate an AI strategy
curl -X POST "http://localhost:8000/api/v1/ai/generate-strategy" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{
    "user_input": "Create a DCA strategy for ETH with $100 weekly purchases"
  }'
```

## 🔄 Background Services

### Start Celery Worker (if using background tasks)

```bash
# Terminal 1: Start Celery worker
celery -A celery_app worker --loglevel=info

# Terminal 2: Start Celery beat (scheduler)
celery -A celery_app beat --loglevel=info
```

### Test Background Task

```bash
# Create an automation rule that will run in the background
curl -X POST "http://localhost:8000/api/v1/automation-simple/create-rule" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{
    "name": "Test DCA Rule",
    "automation_type": "dca",
    "config": {
      "symbol": "ETH",
      "amount": 100,
      "frequency": "daily"
    }
  }'
```

## 🌐 WebSocket Testing

### Connect to WebSocket

```javascript
// JavaScript example for WebSocket connection
const ws = new WebSocket('ws://localhost:8000/api/v1/websocket/market-data');

ws.onopen = function(event) {
    console.log('WebSocket connected');
    
    // Subscribe to ETH price updates
    ws.send(JSON.stringify({
        action: 'subscribe',
        channel: 'price',
        symbol: 'ETH'
    }));
};

ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log('Received:', data);
};
```

### Python WebSocket Client

```python
import asyncio
import websockets
import json

async def test_websocket():
    uri = "ws://localhost:8000/api/v1/websocket/market-data"
    
    async with websockets.connect(uri) as websocket:
        # Subscribe to price updates
        await websocket.send(json.dumps({
            "action": "subscribe",
            "channel": "price",
            "symbol": "ETH"
        }))
        
        # Listen for messages
        async for message in websocket:
            data = json.loads(message)
            print(f"Received: {data}")

# Run the client
asyncio.run(test_websocket())
```

## 🧪 Run Tests

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/unit/test_ai_strategy_generator.py

# Run with coverage
pytest --cov=app tests/

# Run integration tests
pytest tests/integration/

# Run end-to-end tests
pytest tests/e2e/
```

## 🔍 Explore API Endpoints

### Core Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Root health check |
| `/health` | GET | Detailed health check |
| `/docs` | GET | API documentation |
| `/api/v1/auth/login` | POST | User authentication |
| `/api/v1/portfolio/overview` | GET | Portfolio summary |
| `/api/v1/live-trading/execute-trade` | POST | Execute live trades |

### Automation Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/automation/create-strategy` | POST | Create automation strategy |
| `/api/v1/automation-simple/create-rule` | POST | Create simple automation rule |
| `/api/v1/sub-accounts/create` | POST | Create sub-account |

### AI Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/ai/generate-strategy` | POST | Generate AI strategy |
| `/api/v1/ai/analyze-portfolio` | POST | AI portfolio analysis |

## 🛠️ Development Workflow

### 1. Code Changes

```bash
# The development server automatically reloads on code changes
# Edit any file in the app/ directory and see changes instantly
```

### 2. Database Changes

```bash
# Create a new migration
alembic revision --autogenerate -m "Add new feature"

# Apply migrations
alembic upgrade head

# Rollback if needed
alembic downgrade -1
```

### 3. Add New Dependencies

```bash
# Add to requirements.txt
echo "new-package==1.0.0" >> requirements.txt

# Install
pip install -r requirements.txt
```

## 🔧 Useful Commands

### Database Management

```bash
# Reset database (development only)
alembic downgrade base
alembic upgrade head

# View current migration
alembic current

# View migration history
alembic history
```

### Redis Management

```bash
# Connect to Redis CLI
redis-cli

# View all keys
redis-cli keys "*"

# Clear all data (development only)
redis-cli flushall
```

### Docker Management

```bash
# Rebuild containers
docker-compose build

# View container logs
docker-compose logs api
docker-compose logs db
docker-compose logs redis

# Execute commands in containers
docker-compose exec api bash
docker-compose exec db psql -U nadas nadas_development
```

## 🚨 Troubleshooting

### Common Issues

1. **Port 8000 already in use**
   ```bash
   # Find and kill process
   lsof -i :8000
   kill -9 <PID>
   ```

2. **Database connection error**
   ```bash
   # Check if PostgreSQL is running
   pg_isready -h localhost -p 5432
   
   # Start PostgreSQL
   brew services start postgresql  # macOS
   sudo systemctl start postgresql  # Linux
   ```

3. **Redis connection error**
   ```bash
   # Check if Redis is running
   redis-cli ping
   
   # Start Redis
   brew services start redis  # macOS
   sudo systemctl start redis-server  # Linux
   ```

4. **Import errors**
   ```bash
   # Make sure virtual environment is activated
   source venv/bin/activate
   
   # Reinstall dependencies
   pip install -r requirements.txt
   ```

### Debug Mode

```bash
# Run with debug logging
LOG_LEVEL=DEBUG uvicorn app.main:app --reload

# Or set in environment
export LOG_LEVEL=DEBUG
```

## 📚 Next Steps

Now that you have the basic setup working:

1. **Explore Features**: Try the different [features](../features/) like cross-chain trading and AI strategies
2. **Read API Docs**: Understand the [API reference](../api-reference/) for detailed endpoint documentation
3. **Understand Architecture**: Learn about the [system architecture](../architecture/)
4. **Deploy**: When ready, check the [deployment guide](../deployment/)

## 🎯 Quick Test Scenarios

### Scenario 1: Simple Trading

1. Create user account
2. Execute a small testnet trade
3. Check portfolio update

### Scenario 2: Automation Setup

1. Create a DCA automation rule
2. Verify it appears in the database
3. Trigger manual execution

### Scenario 3: AI Integration

1. Generate a trading strategy using AI
2. Review the generated plan
3. Test strategy validation

### Scenario 4: Cross-Chain

1. Check available cross-chain routes
2. Simulate a cross-chain transfer
3. Monitor execution status

## 💡 Tips for Development

- **Use the interactive docs** at `/docs` to test endpoints
- **Monitor logs** in real-time during development
- **Use Redis CLI** to inspect cached data
- **Check database** with psql for data verification
- **Test WebSocket connections** with browser dev tools

Happy coding! 🚀
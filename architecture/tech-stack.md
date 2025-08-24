# Technology Stack

This document details the technology choices made for the Nadas.fi platform, explaining the rationale behind each decision and how they work together to create a robust DeFi automation platform.

## Backend Framework

### FastAPI
**Version**: 0.95.0+

**Why FastAPI?**
- **High Performance**: Built on Starlette and Pydantic, offering exceptional performance
- **Async Support**: Native async/await support for non-blocking operations
- **Auto Documentation**: Automatic OpenAPI/Swagger documentation generation
- **Type Safety**: Full Python type hints integration with runtime validation
- **Modern Python**: Leverages Python 3.10+ features for clean, maintainable code

**Use Cases in Nadas.fi**:
- RESTful API endpoints for trading and automation
- WebSocket connections for real-time data streaming
- Auto-generated API documentation for developers
- Request/response validation and serialization

## Database Technologies

### PostgreSQL 15+
**Why PostgreSQL?**
- **ACID Compliance**: Strong consistency for financial data
- **JSON Support**: Native JSON columns for flexible configuration storage
- **Performance**: Excellent performance with proper indexing
- **Extensions**: PostGIS for geo data, pg_stat_statements for monitoring
- **Reliability**: Battle-tested in production environments

**Schema Design**:
```sql
-- Core tables
users                 -- User accounts and profiles
portfolios            -- Portfolio tracking and analytics
automation_rules      -- Trading strategy definitions  
automation_executions -- Execution tracking and history
notifications         -- Alert and notification management
cross_chain_transactions -- Cross-chain operation tracking
```

### Supabase
**Why Supabase?**
- **Real-time subscriptions**: Live data updates for UI
- **Row Level Security**: Fine-grained access control
- **Built-in Auth**: Additional authentication layer
- **API Generation**: Auto-generated REST and GraphQL APIs
- **Backup & Sync**: Additional data redundancy

**Integration**:
- Primary database: PostgreSQL (local/production)
- Backup/sync: Supabase for additional redundancy
- Real-time features: Supabase subscriptions for live updates

## Caching & Message Broker

### Redis 7+
**Why Redis?**
- **In-Memory Performance**: Sub-millisecond response times
- **Data Structures**: Rich data types for different use cases
- **Pub/Sub**: Message broker capabilities
- **Persistence**: Configurable persistence options
- **Lua Scripting**: Complex atomic operations

**Use Cases**:
```python
# Caching strategy
market_data_cache = redis.get(f"price:{symbol}")  # Price caching
user_session = redis.hget(f"session:{user_id}")   # Session storage
rate_limit = redis.incr(f"rate:{ip_address}")     # Rate limiting
task_queue = redis.lpush("celery", task_data)     # Task queuing
```

## Background Processing

### Celery 5.3+
**Why Celery?**
- **Distributed Tasks**: Scale across multiple workers
- **Scheduling**: Cron-like scheduling with Celery Beat
- **Result Backend**: Task result storage and retrieval
- **Error Handling**: Retry mechanisms and dead letter queues
- **Monitoring**: Rich monitoring and management tools

**Task Types**:
```python
# Automation tasks
@celery.task
def execute_dca_strategy(rule_id: str)

@celery.task  
def check_notification_rules(user_id: str)

@celery.task
def sync_portfolio_data(user_id: str)

@celery.task
def process_cross_chain_transaction(tx_id: str)
```

## Database ORM

### SQLAlchemy 2.0+
**Why SQLAlchemy 2.0?**
- **Async Support**: Full async/await support
- **Type Safety**: Integration with Python type hints
- **Performance**: Optimized query generation and execution
- **Migration Support**: Alembic integration for schema changes
- **Relationship Management**: Complex relationship handling

**Modern Patterns**:
```python
# Async session management
async with get_db_session() as session:
    result = await session.execute(
        select(User).where(User.wallet_address == address)
    )
    user = result.scalar_one_or_none()

# Declarative models with type hints
class User(Base):
    id: Mapped[UUID] = mapped_column(primary_key=True)
    wallet_address: Mapped[str] = mapped_column(String(42), unique=True)
    created_at: Mapped[datetime] = mapped_column(DateTime(timezone=True))
```

## Authentication & Security

### JWT (JSON Web Tokens)
**Libraries**: PyJWT 2.0+

**Why JWT?**
- **Stateless**: No server-side session storage required
- **Scalable**: Works across multiple server instances
- **Flexible**: Custom claims for user permissions
- **Standards-based**: RFC 7519 compliant

**Security Implementation**:
```python
# Token structure
{
  "user_id": "uuid",
  "wallet_address": "0x...",
  "exp": timestamp,
  "iat": timestamp,
  "jti": "token_id"  # For revocation tracking
}
```

### Privy Web3 Authentication
**Why Privy?**
- **Wallet Integration**: Native Web3 wallet support
- **Social Login**: Traditional OAuth alongside Web3
- **Multi-chain**: Support for multiple blockchain networks
- **Security**: Built-in security best practices

## Blockchain Integration

### Hyperliquid Python SDK 0.18.0+
**Why Official SDK?**
- **Native Integration**: Direct access to all Hyperliquid features
- **Real-time Data**: WebSocket connections for live market data
- **Type Safety**: Proper Python typing for all API responses
- **Maintained**: Regular updates from Hyperliquid team

**Integration Architecture**:
```python
# Trading operations
from hyperliquid.info import Info
from hyperliquid.exchange import Exchange

info = Info(base_url, skip_ws=False)
exchange = Exchange(account, base_url)

# WebSocket subscriptions
ws_manager.subscribe('orderUpdates')
ws_manager.subscribe('fills')
ws_manager.subscribe('positions')
```

### Web3.py 6.15.0+
**For HyperEVM Integration**:
- **EVM Operations**: Smart contract interactions
- **Precompile Access**: HyperEVM-specific precompiles
- **Transaction Simulation**: Gas estimation and dry runs
- **Event Filtering**: Real-time event monitoring

```python
# HyperEVM precompile interactions
from web3 import Web3

# Connect to HyperEVM
w3 = Web3(Web3.HTTPProvider(settings.HYPEREVM_MAINNET_RPC))

# Interact with precompiles
precompile_contract = w3.eth.contract(
    address=HYPERLIQUID_PRECOMPILE_ADDRESS,
    abi=PRECOMPILE_ABI
)
```

## Cross-Chain Integration

### LI.FI Integration
**HTTP Client**: httpx 0.23.0+

**Why LI.FI?**
- **Multi-chain**: Support for 20+ blockchain networks
- **Best Routes**: Automatic route optimization
- **DeFi Protocols**: Integration with major DeFi protocols
- **Developer Tools**: Comprehensive API and SDKs

### GlueX Integration
**HTTP Client**: aiohttp 3.9.0+

**Why GlueX?**
- **High Performance**: Optimized for speed
- **Low Fees**: Competitive cross-chain fees
- **Security**: Audited smart contracts
- **Hyperliquid Focus**: Specialized for Hyperliquid ecosystem

## AI Integration

### Anthropic Claude 3.5 Sonnet
**Client**: anthropic 0.21.0+

**Why Claude?**
- **Code Understanding**: Excellent at analyzing trading strategies
- **Complex Reasoning**: Advanced multi-step problem solving
- **Safety**: Built-in safety features for financial applications
- **Context Length**: Large context windows for complex analysis

### OpenAI GPT-4 (Fallback)
**Client**: openai 0.27.0+

**Backup AI Provider**:
- **Availability**: High uptime and reliability
- **Performance**: Fast response times
- **Integration**: Well-documented API

### AWS Bedrock (Enterprise)
**Client**: boto3 1.26.0+

**Enterprise AI Option**:
- **Security**: VPC and private endpoints
- **Compliance**: SOC 2, HIPAA compliance
- **Cost Control**: Predictable pricing

## HTTP Clients

### HTTPX (Primary)
**Version**: 0.23.0+

**Why HTTPX?**
- **Async Support**: Native async/await support
- **HTTP/2**: Better performance with HTTP/2
- **Type Hints**: Full typing support
- **Middleware**: Request/response middleware support

### aiohttp (WebSocket)
**Version**: 3.8.0+

**For Real-time Connections**:
- **WebSocket Client**: Robust WebSocket implementation
- **Session Management**: Connection pooling and reuse
- **Error Handling**: Automatic reconnection logic

## Monitoring & Observability

### Structured Logging
**Library**: structlog 23.0.0+

**Why structlog?**
- **Machine Readable**: JSON-formatted logs
- **Context Preservation**: Correlation IDs across requests
- **Performance**: Lazy evaluation of log messages
- **Integration**: Works with standard Python logging

```python
import structlog

logger = structlog.get_logger(__name__)
logger.info(
    "trade_executed",
    user_id=user_id,
    symbol=symbol,
    size=size,
    price=price,
    correlation_id=request_id
)
```

### Prometheus & Grafana
**Metrics Collection**: Custom metrics for business logic

```python
from prometheus_client import Counter, Histogram, Gauge

# Business metrics
trades_total = Counter('trades_total', 'Total trades executed')
trade_duration = Histogram('trade_duration_seconds', 'Trade execution time')
active_users = Gauge('active_users_total', 'Currently active users')
```

## Development Tools

### Testing Framework

**pytest 7.4.0+**:
- **Async Testing**: pytest-asyncio for async test support
- **Fixtures**: Reusable test fixtures for database and mocks
- **Parametrization**: Data-driven testing
- **Coverage**: pytest-cov for code coverage

```python
# Test structure
tests/
├── unit/           # Fast, isolated tests
├── integration/    # Service integration tests  
└── e2e/           # End-to-end workflow tests
```

### Code Quality

**Formatting & Linting**:
- **black**: Code formatting
- **isort**: Import sorting
- **mypy**: Static type checking
- **flake8**: Linting and style checking

### Environment Management

**python-dotenv 1.0.0+**:
- Environment-specific configuration
- Secure secret management
- Development/staging/production separation

## Deployment Technologies

### Docker & Docker Compose
**Why Docker?**
- **Consistency**: Same environment across dev/staging/prod
- **Isolation**: Service isolation and resource management
- **Scalability**: Easy horizontal scaling
- **Portability**: Deploy anywhere Docker runs

**Container Architecture**:
```yaml
services:
  api:          # FastAPI application
  celery-worker: # Background task processing
  celery-beat:   # Task scheduling
  db:           # PostgreSQL database
  redis:        # Cache and message broker
  nginx:        # Reverse proxy and load balancer
```

### Nginx (Production)
**Why Nginx?**
- **Performance**: High-performance reverse proxy
- **Load Balancing**: Built-in load balancing
- **SSL Termination**: TLS/SSL handling
- **Static Files**: Efficient static file serving

## Performance Optimizations

### Connection Pooling
```python
# Database connection pooling
engine = create_async_engine(
    DATABASE_URL,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True
)

# Redis connection pooling
redis_pool = redis.ConnectionPool(
    host='localhost',
    port=6379,
    max_connections=20
)
```

### Caching Strategy
```python
# Multi-level caching
@lru_cache(maxsize=1000)  # In-memory cache
async def get_market_data(symbol: str):
    # Try Redis first
    cached = await redis.get(f"market:{symbol}")
    if cached:
        return json.loads(cached)
    
    # Fetch from API and cache
    data = await hyperliquid_api.get_market_data(symbol)
    await redis.setex(f"market:{symbol}", 60, json.dumps(data))
    return data
```

### Async Operations
```python
# Concurrent API calls
async def get_multi_asset_data(symbols: List[str]):
    tasks = [get_market_data(symbol) for symbol in symbols]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    return results
```

## Security Stack

### Input Validation
**Pydantic Models**: Runtime validation and serialization

```python
class TradeRequest(BaseModel):
    symbol: str = Field(regex=r'^[A-Z]{2,10}$')
    size: Decimal = Field(gt=0, le=1000000)
    price: Optional[Decimal] = Field(gt=0)
    
    @validator('symbol')
    def validate_symbol(cls, v):
        if v not in ALLOWED_SYMBOLS:
            raise ValueError('Invalid trading symbol')
        return v
```

### Rate Limiting
```python
from slowapi import Limiter

limiter = Limiter(key_func=get_client_ip)

@app.post("/api/v1/trades")
@limiter.limit("10/minute")
async def execute_trade():
    pass
```

### Circuit Breakers
```python
from circuitbreaker import circuit

@circuit(failure_threshold=5, recovery_timeout=30)
async def call_external_api():
    # Protected external API call
    pass
```

## Technology Decision Matrix

| Category | Primary Choice | Alternative | Rationale |
|----------|---------------|-------------|-----------|
| **Web Framework** | FastAPI | Django/Flask | Performance + async + documentation |
| **Database** | PostgreSQL | MongoDB | ACID properties for financial data |
| **Cache** | Redis | Memcached | Data structures + pub/sub |
| **Queue** | Celery | RQ | Feature richness + monitoring |
| **ORM** | SQLAlchemy | Django ORM | Flexibility + async support |
| **HTTP Client** | HTTPX | requests | Async support + HTTP/2 |
| **AI Provider** | Claude | OpenAI | Code understanding + safety |
| **Monitoring** | Prometheus | DataDog | Open source + flexibility |
| **Container** | Docker | Kubernetes | Simplicity for current scale |

## Future Technology Considerations

### Potential Upgrades
- **Database**: Consider TimescaleDB for time-series data
- **Search**: Elasticsearch for advanced log searching
- **Message Queue**: Apache Kafka for high-throughput scenarios
- **Container Orchestration**: Kubernetes for large-scale deployment
- **Service Mesh**: Istio for microservices communication

### Monitoring Evolution
- **Distributed Tracing**: Jaeger or Zipkin for request tracing
- **APM**: Application Performance Monitoring tools
- **Log Aggregation**: ELK stack for centralized logging

## Performance Benchmarks

### Target Performance Metrics
- **API Response Time**: < 100ms (95th percentile)
- **Database Query Time**: < 50ms (average)
- **Cache Hit Ratio**: > 80%
- **Trade Execution Time**: < 2 seconds
- **WebSocket Latency**: < 10ms

### Scalability Targets
- **Concurrent Users**: 10,000+
- **Requests per Second**: 1,000+
- **Database Connections**: 100+
- **Background Tasks**: 500+ per minute

This technology stack provides a solid foundation for building a high-performance, scalable, and maintainable DeFi automation platform while maintaining security and reliability standards required for financial applications.
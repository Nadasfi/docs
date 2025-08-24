# Project Structure

This document provides a detailed overview of the Nadas.fi backend codebase organization, explaining the purpose of each directory and key files.

## Root Directory Structure

```
nadas/backend/
├── app/                    # Main application code
├── alembic/               # Database migrations
├── tests/                 # Test suite
├── nginx/                 # Nginx configuration
├── scripts/               # Utility scripts
├── logs/                  # Application logs (created at runtime)
├── requirements.txt       # Python dependencies
├── docker-compose.yml     # Container orchestration
├── Dockerfile            # Container build instructions
├── pytest.ini           # Test configuration
├── alembic.ini          # Database migration configuration
└── celery_app.py        # Celery configuration
```

## Application Structure (`app/`)

```
app/
├── __init__.py           # Package initialization
├── main.py              # FastAPI application entry point
├── api/                 # API layer (controllers)
├── core/                # Core infrastructure
├── models/              # Database models
├── schemas/             # Pydantic schemas
├── services/            # Business logic layer
├── adapters/            # External service integrations
├── utils/               # Utility functions
├── workers/             # Background task workers
└── middleware/          # Custom middleware
```

## API Layer (`app/api/`)

The API layer contains all HTTP endpoints organized by version and functionality.

```
api/
├── __init__.py
└── v1/                  # API version 1
    ├── __init__.py
    ├── auth.py          # Authentication endpoints
    ├── live_trading.py  # Live trading execution
    ├── automation_engine.py        # Strategy automation
    ├── automation_simple.py        # Simple automation rules
    ├── automation_wallets.py       # Wallet management
    ├── sub_account_automation.py   # Sub-account management
    ├── portfolio.py               # Portfolio management
    ├── portfolio_simple.py        # Simplified portfolio
    ├── ai_assistant.py           # AI-powered features
    ├── trading.py               # Client-side trading
    ├── simulator.py             # HyperEVM simulation
    ├── twap.py                 # TWAP order execution
    ├── lifi.py                # LI.FI cross-chain
    ├── gluex.py              # GlueX cross-chain
    ├── orchestrator.py       # Cross-chain orchestrator
    ├── websocket_api.py     # WebSocket endpoints
    ├── notifications.py     # Notification system
    └── demo.py             # Demo endpoints
```

### Key API Files

#### `auth.py`
```python
# Authentication and user management
@router.post("/register")        # User registration
@router.post("/login")           # User login
@router.post("/refresh")         # Token refresh
@router.post("/logout")          # User logout
@router.get("/profile")          # Get user profile
```

#### `live_trading.py`
```python
# Real trading execution with risk management
@router.post("/execute-trade")    # Execute live trade
@router.post("/risk-check")       # Pre-trade risk assessment
@router.post("/cancel-order")     # Cancel pending order
@router.get("/positions")         # Get current positions
@router.get("/order-history")     # Trading history
```

#### `automation_engine.py`
```python
# Advanced automation strategies
@router.post("/create-strategy")   # Create automation strategy
@router.get("/strategies")         # List user strategies
@router.put("/strategy/{id}")      # Update strategy
@router.delete("/strategy/{id}")   # Delete strategy
@router.post("/execute/{id}")      # Manual execution
```

## Core Infrastructure (`app/core/`)

Contains foundational services and utilities used throughout the application.

```
core/
├── __init__.py
├── config.py             # Application configuration
├── database.py           # Database connection and session management
├── redis_client.py       # Redis client configuration
├── security.py           # Security utilities (JWT, hashing)
├── dependencies.py       # FastAPI dependency injection
├── logging.py           # Structured logging configuration
├── error_handling.py    # Error handling and circuit breakers
├── monitoring_startup.py # Monitoring initialization
├── response.py          # Standard response utilities
└── supabase.py         # Supabase integration
```

### Key Core Files

#### `config.py`
```python
class Settings(BaseSettings):
    """Application settings with environment variable support"""
    
    # Application settings
    ENVIRONMENT: str = "development"
    DEBUG: bool = True
    
    # Database configuration
    DATABASE_URL: str
    DATABASE_POOL_SIZE: int = 10
    
    # External service configurations
    HYPERLIQUID_NETWORK: str = "testnet"
    ANTHROPIC_API_KEY: str = ""
    # ... additional settings
```

#### `database.py`
```python
# Async database engine and session management
engine = create_async_engine(settings.DATABASE_URL)
async_session = async_sessionmaker(engine)

async def get_db_session():
    """Dependency for database sessions"""
    async with async_session() as session:
        yield session
```

#### `security.py`
```python
# JWT token management and security utilities
def create_access_token(data: dict) -> str
def verify_token(token: str) -> dict
def hash_password(password: str) -> str
def verify_password(password: str, hashed: str) -> bool
```

## Models Layer (`app/models/`)

SQLAlchemy ORM models representing the database schema.

```
models/
├── __init__.py
├── user.py              # User account models
├── portfolio.py         # Portfolio and asset models
├── automation.py        # Automation rule models
├── simulation.py        # Simulation and backtesting models
├── notification.py      # Notification system models
├── cross_chain.py       # Cross-chain transaction models
└── ai.py               # AI-related models
```

### Key Model Files

#### `user.py`
```python
class User(Base):
    """User account model"""
    __tablename__ = "users"
    
    id: Mapped[UUID] = mapped_column(primary_key=True)
    wallet_address: Mapped[str] = mapped_column(String(42), unique=True)
    username: Mapped[Optional[str]] = mapped_column(String(50))
    email: Mapped[Optional[str]] = mapped_column(String(255))
    created_at: Mapped[datetime] = mapped_column(DateTime(timezone=True))
    
    # Relationships
    automation_rules = relationship("AutomationRule", back_populates="user")
    portfolios = relationship("Portfolio", back_populates="user")
```

#### `automation.py`
```python
class AutomationRule(Base):
    """Automation rule model"""
    __tablename__ = "automation_rules"
    
    id: Mapped[UUID] = mapped_column(primary_key=True)
    user_id: Mapped[UUID] = mapped_column(ForeignKey("users.id"))
    name: Mapped[str] = mapped_column(String(255))
    automation_type: Mapped[str] = mapped_column(String(50))
    config: Mapped[dict] = mapped_column(JSON)
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)
```

## Services Layer (`app/services/`)

Business logic implementation with clear separation of concerns.

```
services/
├── __init__.py
├── hyperliquid_client.py        # Hyperliquid trading client
├── ai_service.py                # AI integration service
├── ai_strategy_generator.py     # AI strategy generation
├── portfolio_service.py         # Portfolio management
├── portfolio_service_simple.py  # Simplified portfolio service
├── automation_engine.py         # Automation execution engine
├── sub_account_automation.py    # Sub-account automation
├── cross_chain_orchestrator.py  # Cross-chain coordination
├── transaction_simulator.py     # HyperEVM simulation
├── twap_executor.py            # TWAP order execution
├── live_trading.py             # Live trading service
├── user_wallet_integration.py  # Wallet integration
├── web3_provider.py            # Web3 provider management
└── websocket_manager.py        # WebSocket connection management
```

### Key Service Files

#### `hyperliquid_client.py`
```python
class HyperliquidClient:
    """Production-ready Hyperliquid trading client"""
    
    def __init__(self, private_key: Optional[str] = None)
    async def place_order(self, order: OrderRequest) -> OrderResult
    async def cancel_order(self, order_id: str) -> bool
    async def get_positions(self) -> List[Position]
    async def get_market_data(self, symbol: str) -> MarketData
```

#### `ai_strategy_generator.py`
```python
class AIStrategyGenerator:
    """AI-powered strategy generation"""
    
    async def generate_strategy_from_text(
        self, user_input: str, user_address: str
    ) -> Dict[str, Any]
    
    async def analyze_portfolio(
        self, portfolio_data: Dict
    ) -> Dict[str, Any]
```

## Adapters Layer (`app/adapters/`)

External service integration with consistent interfaces.

```
adapters/
├── __init__.py
├── hyperliquid.py       # Hyperliquid SDK adapter
├── alchemy.py          # Alchemy Web3 provider adapter
├── lifi.py             # LI.FI cross-chain adapter
├── gluex.py            # GlueX cross-chain adapter
├── liquid_labs.py      # Liquid Labs integration
└── notifications.py    # Notification service adapter
```

### Key Adapter Files

#### `hyperliquid.py`
```python
class HyperliquidAdapter:
    """Hyperliquid API adapter with caching and error handling"""
    
    async def get_market_data(self, symbol: str) -> Dict
    async def get_orderbook(self, symbol: str) -> Dict
    async def get_user_positions(self, user: str) -> List[Dict]
    async def subscribe_to_updates(self, callback: Callable)
```

#### `lifi.py`
```python
class LiFiAdapter:
    """LI.FI cross-chain bridge adapter"""
    
    async def get_quote(self, from_chain: str, to_chain: str, 
                       from_token: str, to_token: str, 
                       amount: Decimal) -> Dict
    async def execute_transfer(self, quote: Dict) -> Dict
    async def get_transfer_status(self, tx_hash: str) -> Dict
```

## Background Workers (`app/workers/`)

Celery task definitions for background processing.

```
workers/
├── __init__.py
├── automation_tasks.py    # Automation execution tasks
├── monitoring_tasks.py    # System monitoring tasks
└── notification_tasks.py  # Notification processing tasks
```

### Key Worker Files

#### `automation_tasks.py`
```python
@celery.task
def execute_automation_rule(rule_id: str):
    """Execute a single automation rule"""

@celery.task
def check_all_automation_rules():
    """Check all active automation rules"""

@celery.task
def cleanup_completed_executions():
    """Clean up old execution records"""
```

## Database Migrations (`alembic/`)

Database schema versioning and migrations.

```
alembic/
├── env.py               # Alembic environment configuration
├── script.py.mako       # Migration script template
└── versions/            # Migration files
    ├── fe0fe3607d2a_initial_migration.py
    ├── add_cross_chain_support.py
    ├── add_hyperliquid_sdk_models.py
    └── add_security_and_monitoring.py
```

## Test Suite (`tests/`)

Comprehensive test coverage with different test types.

```
tests/
├── unit/                # Unit tests
│   ├── test_ai_strategy_generator.py
│   └── test_cross_chain_orchestrator.py
├── integration/         # Integration tests
│   └── test_orchestrator_api.py
├── e2e/                # End-to-end tests
│   └── test_cross_chain_workflow.py
├── fixtures/           # Test fixtures
├── mocks/             # Mock objects
└── conftest.py        # Test configuration
```

### Test Organization

#### Unit Tests (`tests/unit/`)
```python
# Fast, isolated tests for individual components
def test_strategy_generation():
    generator = AIStrategyGenerator()
    result = await generator.generate_strategy_from_text("Buy ETH")
    assert result["success"] is True
```

#### Integration Tests (`tests/integration/`)
```python
# Tests for service interactions
async def test_automation_engine_integration():
    # Test automation engine with real services
    pass
```

#### E2E Tests (`tests/e2e/`)
```python
# Full workflow tests
async def test_complete_trading_workflow():
    # Test from user request to trade execution
    pass
```

## Configuration Files

### `requirements.txt`
```bash
# Core Framework
fastapi>=0.95.0
uvicorn[standard]>=0.20.0

# Database & ORM
sqlalchemy>=2.0.0
alembic>=1.10.0
asyncpg>=0.27.0

# Hyperliquid Integration
hyperliquid-python-sdk>=0.18.0
eth-account>=0.10.0

# AI Providers
openai>=0.27.0
anthropic>=0.21.0

# ... additional dependencies
```

### `docker-compose.yml`
```yaml
version: '3.8'
services:
  api:              # FastAPI application
    build: .
    ports: ["8000:8000"]
    
  celery-worker:    # Background task processor
    build: .
    command: celery -A celery_app worker
    
  db:              # PostgreSQL database
    image: postgres:15-alpine
    
  redis:           # Cache and message broker
    image: redis:7-alpine
```

### `pytest.ini`
```ini
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    -v
    --tb=short
    --asyncio-mode=auto
```

## Development Guidelines

### File Naming Conventions
- **Snake case**: All Python files use snake_case
- **Descriptive names**: Files clearly indicate their purpose
- **Consistent prefixes**: Test files start with `test_`

### Directory Organization Principles
1. **Separation of Concerns**: Each layer has a specific responsibility
2. **Dependency Direction**: Higher layers depend on lower layers
3. **Feature Grouping**: Related functionality is grouped together
4. **Clear Interfaces**: Well-defined boundaries between layers

### Import Structure
```python
# Standard library imports
import asyncio
from datetime import datetime
from typing import Dict, List, Optional

# Third-party imports
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Mapped

# Local application imports
from app.core.config import settings
from app.models.user import User
from app.services.hyperliquid_client import HyperliquidClient
```

### Code Organization Best Practices

1. **Single Responsibility**: Each module has one clear purpose
2. **Dependency Injection**: Use FastAPI's dependency system
3. **Error Handling**: Consistent error handling across layers
4. **Type Hints**: Full type annotation for better IDE support
5. **Documentation**: Docstrings for all public functions and classes

## Navigation Quick Reference

| Need | Location |
|------|----------|
| **Add new endpoint** | `app/api/v1/` |
| **Add business logic** | `app/services/` |
| **Add external integration** | `app/adapters/` |
| **Add database model** | `app/models/` |
| **Add background task** | `app/workers/` |
| **Modify configuration** | `app/core/config.py` |
| **Add database migration** | `alembic revision` |
| **Add tests** | `tests/unit/`, `tests/integration/`, or `tests/e2e/` |

This structure provides a clear, maintainable, and scalable foundation for the Nadas.fi backend, making it easy for developers to understand where to find and add functionality.
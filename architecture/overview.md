# System Overview

Nadas.fi is a sophisticated DeFi automation platform built specifically for the Hyperliquid ecosystem. This document provides a high-level overview of the system architecture, design principles, and key components.

## Architecture Principles

### 1. Microservices Architecture
- **Modular Design**: Each feature is implemented as a separate service or adapter
- **Loose Coupling**: Services communicate through well-defined interfaces
- **Independent Scaling**: Components can be scaled independently based on demand

### 2. Security-First Design
- **Wallet Security**: User private keys never touch the server
- **Client-side Signing**: All transactions are signed on the client side
- **JWT Authentication**: Secure user session management
- **Circuit Breakers**: Fault-tolerant external service integration

### 3. Performance Optimization
- **Async/Await**: Non-blocking I/O operations throughout
- **Connection Pooling**: Efficient database and Redis connections
- **Caching Strategy**: Redis-based caching for frequently accessed data
- **Background Processing**: Celery for heavy computational tasks

### 4. Observability
- **Structured Logging**: Consistent, searchable log format
- **Health Checks**: Comprehensive system health monitoring
- **Metrics Collection**: Prometheus-based metrics
- **Error Tracking**: Circuit breaker patterns for resilience

## System Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Mobile App    │    │  External APIs  │
│   (React)       │    │   (React Native)│    │  (Hyperliquid)  │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌─────────────▼───────────────┐
                    │       Load Balancer        │
                    │       (Nginx/Docker)       │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │      FastAPI Backend       │
                    │    (Python 3.10+)         │
                    └─────────────┬───────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
┌───────▼────────┐    ┌──────────▼───────────┐    ┌────────▼─────────┐
│   PostgreSQL   │    │       Redis         │    │     Celery       │
│   Database     │    │   Cache & Queue     │    │  Background      │
│   (Primary)    │    │                     │    │   Workers        │
└────────────────┘    └──────────────────────┘    └──────────────────┘
        │                         │                         │
        │              ┌──────────▼───────────┐             │
        │              │     Supabase        │             │
        │              │   (Backup/Sync)     │             │
        │              └──────────────────────┘             │
        │                                                   │
        └─────────────────────┬─────────────────────────────┘
                              │
              ┌───────────────▼────────────────┐
              │      External Services         │
              │                                │
              │  ┌──────────┐ ┌──────────┐    │
              │  │Hyperliquid│ │  LI.FI   │    │
              │  │    SDK    │ │Cross-Chain│   │
              │  └──────────┘ └──────────┘    │
              │                                │
              │  ┌──────────┐ ┌──────────┐    │
              │  │  GlueX   │ │ Claude AI │    │
              │  │Router API│ │  Service  │    │
              │  └──────────┘ └──────────┘    │
              └────────────────────────────────┘
```

## Core Components

### 1. API Layer (`app/api/`)
- **FastAPI Framework**: High-performance async web framework
- **RESTful Endpoints**: Standard HTTP API design
- **WebSocket Support**: Real-time data streaming
- **Auto Documentation**: OpenAPI/Swagger integration

#### Key Routers:
- `auth.py` - Authentication and user management
- `live_trading.py` - Real trading execution
- `automation_engine.py` - Strategy automation
- `portfolio.py` - Portfolio management
- `ai_assistant.py` - AI-powered features
- `websocket_api.py` - Real-time data streams

### 2. Service Layer (`app/services/`)
Business logic implementation with clear separation of concerns.

#### Core Services:
- **HyperliquidClient** - Trading execution service
- **AIStrategyGenerator** - AI-powered strategy creation
- **CrossChainOrchestrator** - Multi-chain coordination
- **PortfolioService** - Portfolio analytics
- **AutomationEngine** - Strategy execution
- **WebSocketManager** - Real-time data management

### 3. Adapter Layer (`app/adapters/`)
External service integration with consistent interfaces.

#### Available Adapters:
- **HyperliquidAdapter** - Native Hyperliquid integration
- **AlchemyAdapter** - Web3 provider integration
- **LiFiAdapter** - Cross-chain bridge integration
- **GlueXAdapter** - Cross-chain router integration
- **NotificationAdapter** - Multi-channel notifications

### 4. Data Layer (`app/models/`)
SQLAlchemy ORM models for data persistence.

#### Core Models:
- **User** - User account management
- **Portfolio** - Portfolio tracking
- **AutomationRule** - Strategy definitions
- **AutomationExecution** - Execution tracking
- **Notification** - Alert management

### 5. Core Infrastructure (`app/core/`)
Foundation services and utilities.

#### Infrastructure Components:
- **Database** - SQLAlchemy async engine
- **Redis** - Caching and session storage
- **Config** - Environment-based configuration
- **Security** - JWT and authentication
- **Logging** - Structured logging setup
- **Error Handling** - Circuit breakers and retries

## Data Flow

### 1. User Request Flow
```
User Request → FastAPI → Authentication → Service Layer → Adapter Layer → External API
                  ↓
Response ← JSON Serialization ← Business Logic ← Data Processing ← API Response
```

### 2. Trading Flow
```
Trade Request → Risk Validation → Strategy Analysis → Execution → Confirmation
      ↓              ↓                  ↓               ↓           ↓
   Database ← Logging ← AI Analysis ← Hyperliquid ← Transaction Hash
```

### 3. Automation Flow
```
Trigger Event → Rule Evaluation → Strategy Execution → Result Tracking
      ↓               ↓                   ↓                ↓
   Celery Task → Background Worker → Trade Execution → Database Update
```

## Integration Architecture

### 1. Hyperliquid Integration
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ HyperLiquid SDK │    │   WebSocket     │    │   REST API      │
│  (Trading)      │◄───┤  (Real-time)    │◄───┤ (Market Data)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         ▲                       ▲                       ▲
         │                       │                       │
┌────────▼────────┐    ┌─────────▼───────┐    ┌─────────▼───────┐
│ Position Mgmt   │    │ Price Streaming │    │ Order Book Data │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 2. Cross-Chain Integration
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│      LI.FI      │    │     GlueX       │    │   Hyperliquid   │
│   (Bridging)    │    │   (Routing)     │    │  (Execution)    │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────┬───────────┼──────────┬───────────┘
                     │           │          │
           ┌─────────▼───────────▼──────────▼───────┐
           │    Cross-Chain Orchestrator           │
           │   (Strategy Coordination)             │
           └───────────────────────────────────────┘
```

### 3. AI Integration
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Claude 3.5     │    │     OpenAI      │    │  AWS Bedrock    │
│   Sonnet        │    │    GPT-4        │    │   (Backup)      │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────┬───────────┼──────────┬───────────┘
                     │           │          │
           ┌─────────▼───────────▼──────────▼───────┐
           │       AI Service Layer                │
           │  (Strategy Generation & Analysis)     │
           └───────────────────────────────────────┘
```

## Scalability Design

### Horizontal Scaling
- **Stateless API**: No server-side session state
- **Load Balancing**: Nginx reverse proxy with multiple instances
- **Database Pooling**: Connection pooling for PostgreSQL
- **Cache Distribution**: Redis cluster support

### Vertical Scaling
- **Async Operations**: Non-blocking I/O throughout
- **Connection Pooling**: Efficient resource utilization
- **Background Processing**: CPU-intensive tasks in Celery workers
- **Memory Management**: Efficient data structures and caching

### Performance Optimization
- **Database Indexing**: Optimized queries and indexes
- **Response Caching**: Redis-based response caching
- **Connection Reuse**: Keep-alive connections to external APIs
- **Batch Processing**: Bulk operations where possible

## Security Architecture

### Authentication & Authorization
```
Client Request → JWT Validation → Role-Based Access → Resource Access
      ↓                ↓                ↓                ↓
  Rate Limiting → IP Allowlist → Permission Check → Audit Log
```

### Data Security
- **Encryption at Rest**: Database encryption
- **Encryption in Transit**: TLS/HTTPS everywhere
- **Secret Management**: Environment-based secrets
- **Input Validation**: Pydantic schema validation

### Trading Security
- **Client-side Signing**: Private keys never on server
- **Transaction Verification**: Multi-level validation
- **Position Limits**: Configurable risk limits
- **Circuit Breakers**: Automatic failure protection

## Monitoring & Observability

### Metrics Collection
```
Application Metrics → Prometheus → Grafana Dashboard
        ↓                ↓             ↓
   Custom Metrics ← Time Series ← Visualization
```

### Logging Strategy
```
Application Logs → Structured Format → Centralized Storage
        ↓               ↓                    ↓
  Correlation ID ← JSON Format ← Search & Analysis
```

### Health Monitoring
- **Application Health**: `/health` endpoint with service status
- **Database Health**: Connection and query performance
- **Redis Health**: Cache availability and performance
- **External Service Health**: API availability monitoring

## Development & Deployment

### Development Flow
```
Code Changes → Local Testing → CI/CD Pipeline → Staging → Production
     ↓              ↓              ↓             ↓          ↓
  Hot Reload ← Unit Tests ← Integration Tests ← E2E Tests ← Monitoring
```

### Container Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   API Service   │    │  Celery Worker  │    │  Celery Beat    │
│   (FastAPI)     │    │ (Background)    │    │  (Scheduler)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         ▲                       ▲                       ▲
         │                       │                       │
┌────────▼────────┐    ┌─────────▼───────┐    ┌─────────▼───────┐
│   PostgreSQL    │    │     Redis       │    │     Nginx       │
│   (Database)    │    │ (Cache/Queue)   │    │ (Load Balancer) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Next Steps

To dive deeper into the architecture:

1. [Technology Stack](tech-stack.md) - Detailed technology choices
2. [Project Structure](project-structure.md) - Code organization
3. [Database Schema](database-schema.md) - Data model design

For implementation details, see:
- [API Reference](../api-reference/) - Endpoint documentation
- [Features](../features/) - Feature-specific architecture
- [Deployment](../deployment/) - Production deployment architecture
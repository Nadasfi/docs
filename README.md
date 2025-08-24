# Nadas.fi Backend Documentation

Welcome to the comprehensive documentation for Nadas.fi, a cutting-edge Hyperliquid DeFi Automation Platform built for the Hyperliquid ecosystem.

## 🚀 About Nadas.fi

Nadas.fi is an advanced DeFi automation platform that provides:

- **Live Trading** - Real execution with comprehensive risk management
- **Automation Engine** - Intelligent trading strategies with user wallet delegation
- **Cross-Chain Operations** - Seamless integration with LI.FI and GlueX protocols
- **AI-Powered Strategies** - Claude 3.5 Sonnet powered automation analysis
- **HyperEVM Simulation** - Transaction simulation and testing capabilities
- **TWAP Orders** - Time-weighted average price execution
- **Real-time Data** - WebSocket streaming for live market data

## 🏗️ Architecture Overview

The backend is built using:

- **FastAPI** - High-performance async web framework
- **SQLAlchemy** - Advanced ORM with async support
- **PostgreSQL** - Primary database with Supabase integration
- **Redis** - Caching and message broker
- **Celery** - Background task processing
- **Hyperliquid Python SDK** - Native trading integration
- **Claude AI** - Advanced strategy generation

## 📋 Key Features

### Trading & Automation
- Live trading with risk management
- Sub-account based automation
- Simple notification-based automation
- Hybrid wallet management

### Cross-Chain Integration
- **LI.FI** - Cross-chain bridge integration ($5k bounty)
- **GlueX** - Cross-chain router integration ($7k bounty)
- Cross-chain orchestrator with AI-powered strategies

### Advanced Features
- **HyperEVM Simulator** - Transaction simulation ($30k bounty)
- **TWAP Executor** - Advanced order execution ($5k bounty)
- **AI Assistant** - Natural language strategy generation
- **WebSocket API** - Real-time data streaming

## 🔧 Technology Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Web Framework** | FastAPI | High-performance async API |
| **Database** | PostgreSQL + SQLAlchemy | Data persistence |
| **Cache/Queue** | Redis | Caching and task queue |
| **Background Tasks** | Celery | Async task processing |
| **Authentication** | JWT + Privy | Secure user authentication |
| **Trading** | Hyperliquid SDK | Native trading integration |
| **AI** | Anthropic Claude | Strategy generation |
| **Monitoring** | Prometheus + Grafana | System monitoring |

## 📚 Documentation Sections

- [Getting Started](getting-started/) - Installation and setup guides
- [Architecture](architecture/) - System design and structure
- [API Reference](api-reference/) - Complete API documentation
- [Features](features/) - Detailed feature documentation
- [Deployment](deployment/) - Production deployment guides
- [Development](development/) - Developer resources

## 🏆 Hackathon Bounties

This project includes implementations for several Hyperliquid hackathon bounties:

- **$30k HyperEVM Transaction Simulator** - Advanced precompile simulation
- **$7k GlueX Cross-Chain Integration** - Seamless cross-chain trading
- **$5k LI.FI Cross-Chain Integration** - Bridge protocol integration
- **$5k TWAP Order Executor** - Time-weighted execution
- **$3k Notification System** - Real-time alerts and notifications

## 🔒 Security Features

- JWT-based authentication with refresh tokens
- Wallet signature verification
- Rate limiting and CORS protection
- Circuit breaker pattern for external services
- Comprehensive input validation
- Structured logging and monitoring

## 🌐 Environment Support

- **Development** - Full debugging and hot reload
- **Production** - Optimized with monitoring and logging
- **Testing** - Comprehensive test suite with mocks
- **Docker** - Containerized deployment with orchestration

## 📊 Monitoring & Observability

- Structured logging with correlation IDs
- Prometheus metrics collection
- Grafana dashboards
- Health check endpoints
- Error tracking and circuit breakers

## 🚀 Quick Start

```bash
# Clone the repository
git clone <repository-url>
cd nadas/backend

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env.development

# Run the development server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

For detailed setup instructions, see [Getting Started](getting-started/).

## 🤝 Contributing

This project welcomes contributions! Please see our [Development Guide](development/) for details on:

- Setting up the development environment
- Running tests
- Code standards and conventions
- Submitting pull requests

## 📄 License

This project is part of the Hyperliquid hackathon submission and follows the competition guidelines.

---

**Built with ❤️ for the Hyperliquid ecosystem**
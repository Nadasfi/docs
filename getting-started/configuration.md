# Configuration

This guide covers the configuration of the Nadas.fi backend environment variables and settings.

## Environment Files

The application uses environment-specific configuration files:

- `.env.development` - Development environment
- `.env.production` - Production environment
- `.env.test` - Testing environment

## Core Configuration

### Application Settings

```bash
# Application Environment
ENVIRONMENT=development
API_VERSION=v1
PROJECT_NAME=Nadas.fi
DEBUG=true
HOST=0.0.0.0
PORT=8000

# Security
ALLOWED_HOSTS=*
ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:3000,https://nadas.fi
```

### Database Configuration

```bash
# PostgreSQL Database
DATABASE_URL=postgresql+asyncpg://nadas:nadas123@localhost:5432/nadas
DATABASE_POOL_SIZE=10
DATABASE_MAX_OVERFLOW=20
```

### Redis Configuration

```bash
# Redis Cache & Message Broker
REDIS_URL=redis://localhost:6379/0
REDIS_CACHE_TTL=3600
```

## Authentication & Security

### JWT Configuration

```bash
# JWT Security
SECRET_KEY=your-super-secret-jwt-key-here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
REFRESH_TOKEN_EXPIRE_DAYS=7
```

### Privy Authentication

```bash
# Privy Web3 Authentication
PRIVY_APP_ID=your-privy-app-id-here
PRIVY_APP_SECRET=your-privy-app-secret
PRIVY_VERIFICATION_KEY=your-privy-verification-key
```

## Hyperliquid Configuration

### Trading Settings

```bash
# Hyperliquid Network Configuration
HYPERLIQUID_TESTNET_URL=https://api.hyperliquid-testnet.xyz
HYPERLIQUID_MAINNET_URL=https://api.hyperliquid.xyz
HYPERLIQUID_NETWORK=testnet  # testnet or mainnet

# Trading Parameters
HYPERLIQUID_USE_WEBSOCKET=true
HYPERLIQUID_MAX_POSITION_SIZE=1000.0
HYPERLIQUID_DEFAULT_SLIPPAGE=0.01
```

### HyperEVM Configuration

```bash
# HyperEVM RPC Configuration
HYPEREVM_TESTNET_RPC=https://api.hyperliquid-testnet.xyz/evm
HYPEREVM_MAINNET_RPC=https://api.hyperliquid.xyz/evm
HYPEREVM_CHAIN_ID_TESTNET=998
HYPEREVM_CHAIN_ID_MAINNET=999
```

## External Integrations

### Supabase Configuration

```bash
# Supabase Database & Auth
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-supabase-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-supabase-service-role-key
```

### Alchemy Configuration

```bash
# Alchemy Web3 Provider
ALCHEMY_API_KEY=your-alchemy-api-key
ALCHEMY_HYPERLIQUID_URL=https://hyperliquid-mainnet.g.alchemy.com/v2/your-api-key
```

## Cross-Chain Integration

### LI.FI Configuration

```bash
# LI.FI Cross-Chain Bridge ($5k Bounty)
LIFI_API_BASE=https://li.quest/v1
LIFI_API_KEY=your-lifi-api-key  # Optional for higher rate limits
LIFI_DEFAULT_SLIPPAGE=0.03
LIFI_ENABLE_CROSS_CHAIN=true
```

### GlueX Configuration

```bash
# GlueX Cross-Chain Router ($7k Bounty)
GLUEX_API_KEY=your-gluex-api-key
GLUEX_ROUTER_URL=https://router.gluex.xyz/v1
GLUEX_EXCHANGE_RATES_URL=https://exchange-rates.gluex.xyz
GLUEX_DEFAULT_SLIPPAGE=0.005
GLUEX_MAX_PRICE_IMPACT=0.05
GLUEX_ENABLE_CROSS_CHAIN=true
```

## AI Services

### Claude AI Configuration

```bash
# Anthropic Claude AI
ANTHROPIC_API_KEY=your-anthropic-api-key
ANTHROPIC_MODEL=claude-3-5-sonnet-20241022
ANTHROPIC_MAX_TOKENS=8192
ANTHROPIC_TEMPERATURE=0.7

# OpenAI (Alternative)
OPENAI_API_KEY=your-openai-api-key
OPENAI_MODEL=gpt-4-1106-preview
OPENAI_MAX_TOKENS=4096

# AWS Bedrock (Alternative)
AWS_ACCESS_KEY_ID=your-aws-access-key
AWS_SECRET_ACCESS_KEY=your-aws-secret-key
AWS_REGION=us-east-1
BEDROCK_MODEL_ID=anthropic.claude-3-sonnet-20240229-v1:0
```

## Notification System

### Notification Configuration

```bash
# Notification System ($3k Bounty)
NOTIFICATION_CHECK_INTERVAL_SECONDS=30
NOTIFICATION_WEBSOCKET_PORT=8001
NOTIFICATION_MAX_RULES_PER_USER=50
NOTIFICATION_COOLDOWN_MINUTES=15
NOTIFICATION_RETENTION_DAYS=30

# Email Configuration
EMAIL_SMTP_SERVER=smtp.gmail.com
EMAIL_SMTP_PORT=587
EMAIL_USERNAME=your-email@gmail.com
EMAIL_PASSWORD=your-email-password

# Push Notifications
PUSH_NOTIFICATION_API_KEY=your-push-api-key
```

## Monitoring & Logging

### Logging Configuration

```bash
# Logging
LOG_LEVEL=INFO
LOG_FORMAT=json
LOG_FILE=logs/nadas.log
STRUCTLOG_ENABLE=true
```

### Monitoring Configuration

```bash
# Prometheus Metrics
PROMETHEUS_ENABLED=true
PROMETHEUS_PORT=9090

# Grafana Dashboard
GRAFANA_ADMIN_PASSWORD=your-grafana-password
```

## Development vs Production

### Development Configuration

```bash
# .env.development
ENVIRONMENT=development
DEBUG=true
LOG_LEVEL=DEBUG
HYPERLIQUID_NETWORK=testnet
ALLOWED_HOSTS=*
```

### Production Configuration

```bash
# .env.production
ENVIRONMENT=production
DEBUG=false
LOG_LEVEL=INFO
HYPERLIQUID_NETWORK=mainnet
ALLOWED_HOSTS=nadas.fi,www.nadas.fi,api.nadas.fi
```

## Configuration Validation

The application automatically validates configuration on startup. Required variables:

### Required Variables

- `SECRET_KEY` - JWT signing key
- `DATABASE_URL` - Database connection
- `REDIS_URL` - Redis connection

### Optional Variables

Most integration-specific variables are optional and will disable the corresponding features if not provided.

## Security Best Practices

### 1. Secret Management

```bash
# Use strong, unique secrets
SECRET_KEY=$(python -c 'import secrets; print(secrets.token_urlsafe(32))')

# Never commit secrets to version control
echo ".env*" >> .gitignore
```

### 2. Database Security

```bash
# Use strong database passwords
DATABASE_PASSWORD=$(python -c 'import secrets; print(secrets.token_urlsafe(16))')

# Restrict database access
DATABASE_HOST=127.0.0.1  # Only local access
```

### 3. API Security

```bash
# Restrict allowed origins in production
ALLOWED_ORIGINS=https://nadas.fi,https://www.nadas.fi

# Restrict allowed hosts
ALLOWED_HOSTS=nadas.fi,www.nadas.fi,api.nadas.fi
```

## Environment-Specific Examples

### Local Development

```bash
# .env.development
ENVIRONMENT=development
DEBUG=true
DATABASE_URL=postgresql+asyncpg://nadas:nadas123@localhost:5432/nadas_dev
REDIS_URL=redis://localhost:6379/0
HYPERLIQUID_NETWORK=testnet
ALLOWED_HOSTS=*
ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:3000
```

### Docker Development

```bash
# .env.development (for Docker)
ENVIRONMENT=development
DEBUG=true
DATABASE_URL=postgresql+asyncpg://nadas:nadas123@db:5432/nadas_dev
REDIS_URL=redis://redis:6379/0
HYPERLIQUID_NETWORK=testnet
```

### Production

```bash
# .env.production
ENVIRONMENT=production
DEBUG=false
DATABASE_URL=postgresql+asyncpg://user:pass@prod-db:5432/nadas_prod
REDIS_URL=redis://prod-redis:6379/0
HYPERLIQUID_NETWORK=mainnet
ALLOWED_HOSTS=api.nadas.fi
ALLOWED_ORIGINS=https://nadas.fi,https://www.nadas.fi
```

## Configuration Loading

The application loads configuration in this order:

1. Default values in `app/core/config.py`
2. Environment-specific file (`.env.{ENVIRONMENT}`)
3. System environment variables
4. Command-line arguments (if applicable)

Higher priority sources override lower priority ones.

## Troubleshooting Configuration

### Common Issues

1. **Database Connection Fails**
   ```bash
   # Check connection string format
   DATABASE_URL=postgresql+asyncpg://user:password@host:port/database
   ```

2. **Redis Connection Fails**
   ```bash
   # Check Redis URL format
   REDIS_URL=redis://host:port/database_number
   ```

3. **API Keys Not Working**
   ```bash
   # Verify API keys are correct and have proper permissions
   # Check for extra whitespace in .env files
   ```

### Testing Configuration

```python
# Test configuration loading
from app.core.config import settings
print(f"Environment: {settings.ENVIRONMENT}")
print(f"Database URL: {settings.DATABASE_URL}")
```

## Next Steps

After configuration:

1. [Follow the quick start guide](quick-start.md)
2. Test the [API endpoints](../api-reference/)
3. Set up [monitoring](../deployment/monitoring.md)
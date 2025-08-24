# Hyperliquid Integration

Nadas.fi provides comprehensive integration with the Hyperliquid protocol, offering real-time trading, portfolio management, and advanced automation features.

## Overview

The Hyperliquid integration is built using the official Hyperliquid Python SDK, providing:

- **Live Trading**: Real trading execution with risk management
- **Real-time Data**: WebSocket connections for live market data
- **Portfolio Tracking**: Comprehensive position and PnL monitoring
- **HyperEVM Support**: Smart contract interactions and transaction simulation

## Core Features

### 1. Trading Execution

**Supported Order Types:**
- Market Orders
- Limit Orders
- Stop Market Orders
- Stop Limit Orders
- Take Profit Orders

**Trading Features:**
- Real-time order placement and cancellation
- Position management with automatic risk controls
- Slippage protection and position limits
- Multi-asset trading support

### 2. Real-time Market Data

**WebSocket Subscriptions:**
- Price updates for all trading pairs
- Order book depth and changes
- Trade history and volume data
- User-specific position updates

**Data Processing:**
- Real-time price feeds with < 10ms latency
- Automatic reconnection and error handling
- Rate limiting and connection pooling
- Data validation and normalization

### 3. Portfolio Management

**Portfolio Analytics:**
- Real-time position tracking
- PnL calculations (realized and unrealized)
- Risk metrics and exposure analysis
- Performance attribution and analytics

**Asset Support:**
- All Hyperliquid trading pairs
- Cross-margined and isolated margin positions
- Multi-timeframe performance analysis
- Historical data and backtesting

## Technical Implementation

### SDK Integration

```python
from hyperliquid.info import Info
from hyperliquid.exchange import Exchange
from hyperliquid.utils import constants

# Initialize connections
info = Info(constants.MAINNET_API_URL, skip_ws=False)
exchange = Exchange(account, constants.MAINNET_API_URL)

# Real-time data subscriptions
info.subscribe({'type': 'orderUpdates', 'user': user_address})
info.subscribe({'type': 'fills', 'user': user_address})
```

### WebSocket Manager

```python
class WebSocketManager:
    async def subscribe_to_market_data(self, symbols: List[str]):
        """Subscribe to real-time market data"""
        
    async def handle_price_update(self, data: Dict):
        """Process incoming price updates"""
        
    async def handle_order_update(self, data: Dict):
        """Process order status changes"""
```

### Trading Client

```python
class HyperliquidClient:
    async def place_order(self, order: OrderRequest) -> OrderResult:
        """Execute trade with risk management"""
        
    async def cancel_order(self, order_id: str) -> bool:
        """Cancel pending order"""
        
    async def get_positions(self) -> List[Position]:
        """Get current positions"""
```

## API Endpoints

### Live Trading

**POST /api/v1/live-trading/execute-trade**
```json
{
  "symbol": "ETH",
  "side": "buy",
  "size": 1000.0,
  "order_type": "market",
  "reduce_only": false
}
```

**GET /api/v1/live-trading/positions**
```json
{
  "success": true,
  "positions": [
    {
      "symbol": "ETH",
      "size": 10.5,
      "side": "long",
      "entry_price": 2850.0,
      "mark_price": 2875.0,
      "unrealized_pnl": 262.5,
      "leverage": 5.0
    }
  ]
}
```

### Portfolio Data

**GET /api/v1/portfolio/overview**
```json
{
  "success": true,
  "portfolio": {
    "total_value": 125000.0,
    "available_balance": 25000.0,
    "margin_used": 100000.0,
    "unrealized_pnl": 5000.0,
    "daily_pnl": 1250.0,
    "positions_count": 5
  }
}
```

## Risk Management

### Position Limits
- Maximum position size per asset
- Maximum total exposure limits
- Leverage restrictions
- Stop-loss automation

### Risk Controls
```python
class RiskManager:
    def validate_trade(self, order: OrderRequest) -> RiskAssessment:
        """Validate trade against risk parameters"""
        
    def calculate_position_risk(self, position: Position) -> float:
        """Calculate position risk metrics"""
        
    def check_exposure_limits(self, user_id: str) -> bool:
        """Verify exposure within limits"""
```

## Error Handling

### Circuit Breakers
- Automatic disconnection on repeated failures
- Exponential backoff for reconnections
- Fallback to alternative data sources

### Error Recovery
```python
@circuit_breaker(failure_threshold=5, timeout=30)
async def execute_trade_with_retry(order: OrderRequest):
    """Trade execution with automatic retry"""
```

## Performance Optimizations

### Connection Management
- Persistent WebSocket connections
- Connection pooling for REST API calls
- Automatic reconnection with backoff

### Data Caching
```python
# Price data caching
@lru_cache(maxsize=1000)
async def get_cached_price(symbol: str) -> Decimal:
    """Get cached price data"""

# Redis caching for market data
await redis.setex(f"price:{symbol}", 60, price_data)
```

### Batch Operations
```python
async def get_multiple_positions(user_addresses: List[str]):
    """Batch fetch positions for multiple users"""
    tasks = [get_user_positions(addr) for addr in user_addresses]
    return await asyncio.gather(*tasks)
```

## Configuration

### Environment Variables
```bash
# Hyperliquid Configuration
HYPERLIQUID_NETWORK=mainnet  # or testnet
HYPERLIQUID_MAX_POSITION_SIZE=1000000
HYPERLIQUID_DEFAULT_SLIPPAGE=0.01
HYPERLIQUID_USE_WEBSOCKET=true

# Risk Management
RISK_MAX_LEVERAGE=10.0
RISK_MAX_DAILY_LOSS=0.05  # 5%
RISK_POSITION_LIMIT_USD=50000
```

### Trading Parameters
```python
# Default trading configuration
DEFAULT_CONFIG = {
    "slippage_tolerance": 0.01,  # 1%
    "max_position_size": 1000000,  # $1M
    "leverage_limit": 5.0,
    "stop_loss_threshold": 0.05,  # 5%
}
```

## Monitoring & Analytics

### Performance Metrics
- Trade execution latency
- WebSocket connection uptime
- API response times
- Error rates and types

### Business Metrics
```python
# Trading metrics
trades_executed = Counter('trades_total')
trade_volume = Histogram('trade_volume_usd')
position_pnl = Gauge('position_pnl_usd')

# Connection metrics
websocket_connections = Gauge('websocket_connections_active')
api_requests = Counter('api_requests_total')
```

## Security Considerations

### Wallet Security
- Private keys never stored on server
- Client-side transaction signing
- Signature verification for all operations

### API Security
```python
# Rate limiting
@limiter.limit("100/minute")
async def execute_trade():
    pass

# Input validation
class TradeRequest(BaseModel):
    symbol: str = Field(regex=r'^[A-Z]{2,10}$')
    size: Decimal = Field(gt=0, le=1000000)
```

## Testing & Validation

### Unit Tests
```python
async def test_trade_execution():
    client = HyperliquidClient(testnet=True)
    result = await client.place_order(test_order)
    assert result.success is True
```

### Integration Tests
```python
async def test_websocket_connection():
    ws_manager = WebSocketManager()
    await ws_manager.connect()
    assert ws_manager.is_connected()
```

## Troubleshooting

### Common Issues
1. **Connection Failures**: Check network connectivity and API keys
2. **Order Rejection**: Verify account balance and position limits
3. **WebSocket Disconnections**: Monitor connection health and implement reconnection logic

### Debug Mode
```python
# Enable debug logging
LOG_LEVEL=DEBUG uvicorn app.main:app --reload
```

This integration provides a robust foundation for building advanced trading applications on the Hyperliquid protocol with comprehensive risk management and real-time capabilities.
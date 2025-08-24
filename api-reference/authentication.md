# Authentication API

The Nadas.fi authentication system supports Web3 wallet-based authentication with JWT tokens for session management.

## Overview

- **Base URL**: `/api/v1/auth`
- **Authentication**: JWT Bearer tokens
- **Wallet Support**: Ethereum-compatible wallets (MetaMask, WalletConnect, etc.)

## Endpoints

### POST /register
Register a new user account with wallet address.

**Request Body:**
```json
{
  "wallet_address": "0x742d35cc6eab4c06c346c9bf1e6d8ae7b7b9e99b",
  "username": "john_doe",
  "email": "john@example.com"
}
```

**Response:**
```json
{
  "success": true,
  "user": {
    "id": "uuid",
    "wallet_address": "0x742d35cc6eab4c06c346c9bf1e6d8ae7b7b9e99b",
    "username": "john_doe",
    "created_at": "2024-01-15T10:30:00Z"
  }
}
```

### POST /login
Authenticate user with wallet signature.

**Request Body:**
```json
{
  "wallet_address": "0x742d35cc6eab4c06c346c9bf1e6d8ae7b7b9e99b",
  "signature": "0x...",
  "message": "Login to Nadas.fi - Nonce: 12345"
}
```

**Response:**
```json
{
  "success": true,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "refresh_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

### POST /refresh
Refresh access token using refresh token.

**Request Body:**
```json
{
  "refresh_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

### GET /profile
Get current user profile (requires authentication).

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response:**
```json
{
  "success": true,
  "user": {
    "id": "uuid",
    "wallet_address": "0x742d35cc6eab4c06c346c9bf1e6d8ae7b7b9e99b",
    "username": "john_doe",
    "email": "john@example.com",
    "created_at": "2024-01-15T10:30:00Z",
    "last_login": "2024-01-20T14:22:00Z"
  }
}
```

## Authentication Flow

1. **Get Nonce**: Request a login nonce from the server
2. **Sign Message**: User signs the message with their wallet
3. **Verify Signature**: Server verifies the signature and issues tokens
4. **Use Token**: Include JWT token in subsequent API requests

## Error Codes

| Code | Description |
|------|-------------|
| 401 | Invalid credentials or signature |
| 403 | Account not found or inactive |
| 429 | Rate limit exceeded |
| 500 | Internal server error |
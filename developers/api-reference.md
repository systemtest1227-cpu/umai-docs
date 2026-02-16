# API Reference

The UmAI automation bot exposes a REST API on **port 3001** for health checks, status monitoring, and referral management. This page documents every endpoint and the key smart contract functions.

---

## Authentication

API authentication is **optional** and controlled by the `API_KEY` environment variable. When set, requests must include the key in the `X-API-KEY` header.

```bash
# Authenticated request
curl -H "X-API-KEY: your-secret-key" http://localhost:3001/api/status
```

If `API_KEY` is not configured, all endpoints are accessible without authentication.

---

## Health & Status Endpoints

### GET /health

Returns a simple health check response. Use this for uptime monitoring and load balancer probes.

**Request:**

```bash
curl http://localhost:3001/health
```

**Response:**

```json
{
  "status": "ok",
  "timestamp": "2025-12-01T00:00:00.000Z"
}
```

| Status Code | Meaning |
|-------------|---------|
| `200` | Bot is running |

---

### GET /api/status

Returns detailed operational status including module states, circuit breaker status, and recent activity.

**Request:**

```bash
curl http://localhost:3001/api/status
```

**Response:**

```json
{
  "status": "ok",
  "uptime": 86400,
  "modules": {
    "harvester": {
      "enabled": true,
      "lastRun": "2025-12-01T00:00:00.000Z",
      "lastResult": "success",
      "consecutiveFailures": 0,
      "circuitBreakerTripped": false
    },
    "rebalancer": {
      "enabled": true,
      "autoRebalance": false,
      "lastCheck": "2025-12-01T12:05:00.000Z",
      "positionInRange": true,
      "consecutiveFailures": 0,
      "circuitBreakerTripped": false
    }
  },
  "wallet": {
    "address": "0x...",
    "ethBalance": "0.15"
  }
}
```

---

## Referral Code Endpoints

### GET /api/referrals/codes

Retrieves all referral codes.

**Request:**

```bash
curl http://localhost:3001/api/referrals/codes
```

**Response:**

```json
{
  "codes": [
    {
      "code": "ALPHA100",
      "ownerAddress": "0x1234...abcd",
      "rewardBps": 500,
      "active": true,
      "createdAt": "2025-11-01T00:00:00.000Z",
      "usageCount": 42
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `code` | string | The referral code string |
| `ownerAddress` | string | Ethereum address of the code owner |
| `rewardBps` | number | Reward rate in basis points (500 = 5%) |
| `active` | boolean | Whether the code is currently active |
| `createdAt` | string | ISO 8601 creation timestamp |
| `usageCount` | number | Number of users who used this code |

---

### POST /api/referrals/codes

Creates a new referral code.

**Request:**

```bash
curl -X POST http://localhost:3001/api/referrals/codes \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: your-secret-key" \
  -d '{
    "code": "NEWCODE",
    "ownerAddress": "0x1234567890abcdef1234567890abcdef12345678",
    "rewardBps": 500
  }'
```

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | string | Yes | Unique referral code (alphanumeric) |
| `ownerAddress` | string | Yes | Ethereum address of the code owner |
| `rewardBps` | number | Yes | Reward rate in basis points |

**Response (201 Created):**

```json
{
  "code": "NEWCODE",
  "ownerAddress": "0x1234567890abcdef1234567890abcdef12345678",
  "rewardBps": 500,
  "active": true,
  "createdAt": "2025-12-01T00:00:00.000Z"
}
```

**Error Response (409 Conflict):**

```json
{
  "error": "Referral code already exists"
}
```

---

### PATCH /api/referrals/codes/:code

Updates an existing referral code's properties.

**Request:**

```bash
curl -X PATCH http://localhost:3001/api/referrals/codes/NEWCODE \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: your-secret-key" \
  -d '{
    "rewardBps": 750
  }'
```

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `rewardBps` | number | No | Updated reward rate in basis points |
| `ownerAddress` | string | No | Updated owner address |

**Response (200 OK):**

```json
{
  "code": "NEWCODE",
  "ownerAddress": "0x1234567890abcdef1234567890abcdef12345678",
  "rewardBps": 750,
  "active": true
}
```

---

### DELETE /api/referrals/codes/:code

Permanently deletes a referral code.

**Request:**

```bash
curl -X DELETE http://localhost:3001/api/referrals/codes/NEWCODE \
  -H "X-API-KEY: your-secret-key"
```

**Response (200 OK):**

```json
{
  "message": "Referral code deleted",
  "code": "NEWCODE"
}
```

**Error Response (404 Not Found):**

```json
{
  "error": "Referral code not found"
}
```

---

### POST /api/referrals/codes/:code/toggle

Toggles a referral code between active and inactive states.

**Request:**

```bash
curl -X POST http://localhost:3001/api/referrals/codes/ALPHA100/toggle \
  -H "X-API-KEY: your-secret-key"
```

**Response (200 OK):**

```json
{
  "code": "ALPHA100",
  "active": false,
  "message": "Referral code deactivated"
}
```

---

## Referral User Endpoints

### GET /api/referrals/users

Lists all users associated with referral codes.

**Request:**

```bash
curl http://localhost:3001/api/referrals/users
```

**Response:**

```json
{
  "users": [
    {
      "userAddress": "0xabcd...1234",
      "referralCode": "ALPHA100",
      "registeredAt": "2025-11-15T10:30:00.000Z",
      "totalRewardsEarned": "150.50"
    }
  ]
}
```

---

### POST /api/referrals/users

Registers a user under a referral code.

**Request:**

```bash
curl -X POST http://localhost:3001/api/referrals/users \
  -H "Content-Type: application/json" \
  -d '{
    "userAddress": "0xabcdef1234567890abcdef1234567890abcdef12",
    "referralCode": "ALPHA100"
  }'
```

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `userAddress` | string | Yes | Ethereum address of the user |
| `referralCode` | string | Yes | Referral code to associate with |

**Response (201 Created):**

```json
{
  "userAddress": "0xabcdef1234567890abcdef1234567890abcdef12",
  "referralCode": "ALPHA100",
  "registeredAt": "2025-12-01T00:00:00.000Z"
}
```

---

## Referral Distribution Endpoints

### GET /api/referrals/distributions

Retrieves the history of referral reward distributions.

**Request:**

```bash
curl http://localhost:3001/api/referrals/distributions
```

**Response:**

```json
{
  "distributions": [
    {
      "id": "dist_001",
      "executedAt": "2025-12-01T00:05:00.000Z",
      "totalDistributed": "500.00",
      "recipients": 15,
      "txHash": "0x..."
    }
  ]
}
```

---

### POST /api/referrals/distributions/execute

Triggers an immediate referral reward distribution.

**Request:**

```bash
curl -X POST http://localhost:3001/api/referrals/distributions/execute \
  -H "X-API-KEY: your-secret-key"
```

**Response (200 OK):**

```json
{
  "status": "success",
  "distributed": "500.00",
  "recipients": 15,
  "txHash": "0x...",
  "executedAt": "2025-12-01T12:00:00.000Z"
}
```

**Error Response (400 Bad Request):**

```json
{
  "error": "No pending rewards to distribute"
}
```

---

## Endpoint Summary Table

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET` | `/health` | No | Health check |
| `GET` | `/api/status` | Optional | Detailed bot status |
| `GET` | `/api/referrals/codes` | Optional | List all referral codes |
| `POST` | `/api/referrals/codes` | Yes | Create a referral code |
| `PATCH` | `/api/referrals/codes/:code` | Yes | Update a referral code |
| `DELETE` | `/api/referrals/codes/:code` | Yes | Delete a referral code |
| `POST` | `/api/referrals/codes/:code/toggle` | Yes | Toggle code active/inactive |
| `GET` | `/api/referrals/users` | Optional | List referred users |
| `POST` | `/api/referrals/users` | Optional | Register a referred user |
| `GET` | `/api/referrals/distributions` | Optional | Distribution history |
| `POST` | `/api/referrals/distributions/execute` | Yes | Execute distribution |

---

## Smart Contract ABI -- Key Functions

The UmAI Vault exposes the following key functions that can be called directly or through the frontend:

### User Functions

#### `deposit(uint256 usdcAmount, uint256 maxSlippage, uint8 lockPeriod, bytes32 referralCode)`

Deposits USDC into the vault, selects a lock period and optional referral code, and mints non-transferable vault shares to the caller.

```solidity
function deposit(
    uint256 usdcAmount,
    uint256 maxSlippage,
    uint8 lockPeriod,
    bytes32 referralCode
) external nonReentrant;
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `usdcAmount` | uint256 | Amount of USDC to deposit (6 decimals). Minimum: 500 USDC. |
| `maxSlippage` | uint256 | Maximum slippage tolerance in basis points (e.g., 50 = 0.5%). Capped at 100 (1%). |
| `lockPeriod` | uint8 | Lock duration selection: 3, 6, or 12 months. Determines performance fee rate. |
| `referralCode` | bytes32 | Optional referral code. Pass `bytes32(0)` for no referral. |

#### `withdraw(uint256 shares, uint256 maxSlippage)`

Burns vault shares and returns the proportional USDC to the caller. Reverts if the caller's lock period has not expired.

```solidity
function withdraw(
    uint256 shares,
    uint256 maxSlippage
) external nonReentrant;
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `shares` | uint256 | Number of vault shares to burn |
| `maxSlippage` | uint256 | Maximum slippage tolerance in basis points (e.g., 50 = 0.5%). Capped at 100 (1%). |

#### `claimPendingRewards()`

Claims accumulated pending rewards for the calling address. Rewards accrue when the vault's push-based fee distribution fails for a depositor.

```solidity
function claimPendingRewards() external nonReentrant;
```

### Manager Functions

#### `rebalance(int24 newTickLower, int24 newTickUpper, uint256 minAmount0, uint256 minAmount1)`

Withdraws liquidity from the current position and redeploys it to a new tick range. Restricted to the Manager role.

```solidity
function rebalance(
    int24 newTickLower,
    int24 newTickUpper,
    uint256 minAmount0,
    uint256 minAmount1
) external onlyManager;
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `newTickLower` | int24 | Lower bound of the new tick range |
| `newTickUpper` | int24 | Upper bound of the new tick range |
| `minAmount0` | uint256 | Minimum WETH to deploy (slippage protection) |
| `minAmount1` | uint256 | Minimum USDC to deploy (slippage protection) |

#### `harvestAndAllocate()`

Collects accumulated trading fees from the Uniswap V3 position, takes the performance fee, and reinvests the remainder. Restricted to the Manager role.

```solidity
function harvestAndAllocate() external onlyManager;
```

---

## Error Codes

| HTTP Status | Meaning |
|-------------|---------|
| `200` | Success |
| `201` | Created |
| `400` | Bad request (invalid parameters) |
| `401` | Unauthorized (missing or invalid API key) |
| `404` | Resource not found |
| `409` | Conflict (duplicate resource) |
| `500` | Internal server error |

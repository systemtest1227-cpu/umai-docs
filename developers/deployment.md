# Deployment

This guide covers deploying the UmAI system, which consists of three components:

1. **Smart Contracts** -- The on-chain vault and related contracts (Foundry/Solidity)
2. **Automation Bot** -- The Node.js service that manages harvesting, rebalancing, and referrals
3. **Frontend** -- The Next.js web application for user interaction

---

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| Node.js | 18+ | Bot and frontend runtime |
| npm or yarn | Latest | Package management |
| Docker & Docker Compose | Latest | Containerized deployment |
| Foundry (forge, cast) | Latest | Smart contract toolchain |
| Git | Latest | Source control |
| PM2 | Latest | Process manager (VPS deployment) |

---

## 1. Docker Compose Deployment (Recommended)

Docker Compose is the recommended deployment method. It runs both the frontend and bot as containers with a single command.

### Directory Structure

```
umai/
├── docker-compose.yml
├── bot/
│   ├── Dockerfile
│   ├── package.json
│   └── .env
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── .env.local
└── contracts/
    └── ...
```

### docker-compose.yml

```yaml
version: "3.8"

services:
  bot:
    build: ./bot
    container_name: umai-bot
    restart: unless-stopped
    ports:
      - "3001:3001"
    env_file:
      - ./bot/.env
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    build: ./frontend
    container_name: umai-frontend
    restart: unless-stopped
    ports:
      - "3000:3000"
    env_file:
      - ./frontend/.env.local
    depends_on:
      bot:
        condition: service_healthy
```

### Starting the Stack

```bash
# Build and start all services
docker compose up -d --build

# View logs
docker compose logs -f

# Stop all services
docker compose down

# Rebuild a single service
docker compose up -d --build bot
```

---

## 2. Environment Variables

### Bot Environment Variables (`.env`)

Create a `.env` file in the bot directory:

```bash
# === Required ===

# Base Mainnet RPC endpoint
RPC_URL=https://mainnet.base.org

# Operator wallet private key (with ETH for gas on Base)
PRIVATE_KEY=0x_your_private_key_here

# UmAI Vault proxy contract address
VAULT_ADDRESS=0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5

# === Optional: Server ===

# Bot HTTP server port
BOT_PORT=3001

# API authentication key (leave empty to disable auth)
API_KEY=your-secret-api-key

# === Optional: Scheduling ===

# Harvest cron schedule (default: midnight UTC)
HARVEST_CRON=0 0 * * *

# Rebalance check interval (default: every 5 minutes)
REBALANCE_CRON=*/5 * * * *

# === Optional: Rebalancer ===

# Enable automatic rebalancing (default: false)
AUTO_REBALANCE=false

# Cooldown between rebalances in minutes (default: 60)
REBALANCE_COOLDOWN=60

# Tick range width (default: 4000 ticks ≈ ±5%)
RANGE_WIDTH=4000

# TWAP observation window in seconds (default: 1800 = 30 min)
TWAP_WINDOW=1800

# Maximum TWAP deviation in basis points (default: 200 = 2%)
TWAP_MAX_DEVIATION=200

# === Optional: Safety ===

# Minimum ETH balance to execute transactions (default: 0.01)
GAS_THRESHOLD=0.01

# Consecutive failures before circuit breaker trips (default: 5)
CIRCUIT_BREAKER_THRESHOLD=5

# Max retry attempts per operation (default: 3)
MAX_RETRIES=3

# Base delay between retries in ms (default: 2000)
RETRY_BASE_DELAY=2000

# === Optional: Notifications ===

# Discord webhook URL for alerts
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/YOUR_WEBHOOK_ID/YOUR_WEBHOOK_TOKEN
```

> **Security:** Never commit the `.env` file to version control. Add it to `.gitignore`.

### Frontend Environment Variables (`.env.local`)

Create a `.env.local` file in the frontend directory:

```bash
# === Required ===

# UmAI Vault contract address
NEXT_PUBLIC_VAULT_ADDRESS=0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5

# WETH contract address on Base
NEXT_PUBLIC_WETH_ADDRESS=0x4200000000000000000000000000000000000006

# USDC contract address on Base
NEXT_PUBLIC_USDC_ADDRESS=0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913

# Base Mainnet Chain ID
NEXT_PUBLIC_CHAIN_ID=8453

# Base Mainnet RPC URL
NEXT_PUBLIC_RPC_URL=https://mainnet.base.org

# === Optional ===

# WalletConnect Project ID (for wallet connections)
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id

# Bot API URL (for frontend-to-bot communication)
NEXT_PUBLIC_BOT_API_URL=http://localhost:3001

# Enable/disable testnet mode
NEXT_PUBLIC_TESTNET=false
```

---

## 3. Manual VPS Deployment with PM2

For deployment on a bare VPS (Ubuntu/Debian) without Docker.

### System Setup

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Node.js 18+ via NodeSource
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install PM2 globally
npm install -g pm2

# Install nginx (reverse proxy)
sudo apt install -y nginx
```

### Deploy the Bot

```bash
# Clone the repository
git clone https://github.com/your-org/umai.git
cd umai/bot

# Install dependencies
npm install

# Create environment file
cp .env.example .env
nano .env  # Edit with your values

# Start with PM2
pm2 start index.js --name umai-bot
pm2 save
pm2 startup  # Enable auto-start on reboot
```

### Deploy the Frontend

```bash
cd umai/frontend

# Install dependencies
npm install

# Create environment file
cp .env.example .env.local
nano .env.local  # Edit with your values

# Build for production
npm run build

# Start with PM2
pm2 start npm --name umai-frontend -- start
pm2 save
```

### Nginx Reverse Proxy

Create an Nginx configuration to serve both services:

```nginx
# /etc/nginx/sites-available/umai
server {
    listen 80;
    server_name your-domain.com;

    # Frontend
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Bot API
    location /api/ {
        proxy_pass http://127.0.0.1:3001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Bot health check
    location /health {
        proxy_pass http://127.0.0.1:3001;
    }
}
```

Enable the site and restart Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/umai /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### SSL with Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

### PM2 Monitoring Commands

```bash
# View all processes
pm2 list

# View logs
pm2 logs umai-bot
pm2 logs umai-frontend

# Restart a service
pm2 restart umai-bot

# Monitor resources
pm2 monit
```

---

## 4. Smart Contract Deployment with Foundry

### Install Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

### Build Contracts

```bash
cd umai/contracts

# Install dependencies
forge install

# Build
forge build

# Run tests
forge test
```

### Deploy the Vault (Upgradeable)

The UmAI Vault uses the UUPS upgradeable proxy pattern. Deployment is handled by the `DeployUpgradeable.s.sol` script.

```bash
# Set environment variables
export RPC_URL=https://mainnet.base.org
export PRIVATE_KEY=0x_your_deployer_private_key
export ETHERSCAN_API_KEY=your_basescan_api_key

# Deploy to Base Mainnet
forge script script/DeployUpgradeable.s.sol:DeployUpgradeable \
  --rpc-url $RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --verify \
  --etherscan-api-key $ETHERSCAN_API_KEY
```

### Deployment Script Parameters

The `DeployUpgradeable.s.sol` script configures the vault with the following parameters at deployment:

| Parameter | Value | Description |
|-----------|-------|-------------|
| `pool` | `0x6c561B446416E1A00E8E93E221854d6eA4171372` | WETH/USDC 0.3% pool |
| `positionManager` | `0x03a520b32C04BF3bEEf7BEb72E919cf822Ed34f1` | Uniswap V3 NFT Position Manager |
| `swapRouter` | `0x2626664c2603336E57B271c5C0b26F421741e481` | Uniswap V3 Swap Router |
| `manager` | Deployer address | Initial manager (can be transferred) |
| `feeRecipient` | Deployer address | Receives performance fees |

### Verify on BaseScan

If verification was not included in the deploy command:

```bash
forge verify-contract \
  --chain-id 8453 \
  --num-of-optimizations 200 \
  --watch \
  --compiler-version v0.8.19 \
  --etherscan-api-key $ETHERSCAN_API_KEY \
  0x_IMPLEMENTATION_ADDRESS \
  src/UmAIVault.sol:UmAIVault
```

### Upgrading the Vault

To deploy a new implementation and upgrade the proxy:

```bash
forge script script/UpgradeVault.s.sol:UpgradeVault \
  --rpc-url $RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --verify \
  --etherscan-api-key $ETHERSCAN_API_KEY
```

> **Warning:** Only the contract owner can upgrade the proxy. Always test upgrades on a testnet (Base Sepolia) before deploying to mainnet.

---

## 5. Post-Deployment Checklist

After deploying all components, verify the following:

- [ ] Smart contract is deployed and verified on BaseScan
- [ ] Vault is initialized with correct pool, position manager, and swap router addresses
- [ ] Manager and fee recipient addresses are set correctly
- [ ] Bot `.env` has the correct vault address, RPC URL, and private key
- [ ] Bot health endpoint responds: `curl http://localhost:3001/health`
- [ ] Frontend `.env.local` has the correct contract addresses and chain ID
- [ ] Frontend loads and connects to the correct network
- [ ] Discord webhook is configured and test alert sent
- [ ] Operator wallet has sufficient ETH on Base for gas
- [ ] Nginx is configured with SSL (production)
- [ ] PM2 or Docker restart policies are enabled
- [ ] `.env` files are excluded from version control

---

## Troubleshooting

### Bot fails to start

```bash
# Check logs
pm2 logs umai-bot --lines 50
# or
docker compose logs bot --tail 50
```

Common causes:
- Missing or invalid `PRIVATE_KEY`
- Invalid `RPC_URL` (check your provider)
- `VAULT_ADDRESS` is incorrect

### Frontend shows wrong data

- Verify `NEXT_PUBLIC_VAULT_ADDRESS` matches the deployed proxy address
- Ensure `NEXT_PUBLIC_CHAIN_ID` is `8453` for Base Mainnet
- Check that `NEXT_PUBLIC_RPC_URL` is reachable

### Transactions failing

- Check operator wallet ETH balance: needs at least 0.01 ETH
- Verify the operator address has the Manager role on the vault contract
- Check Base network status at [base.statuspage.io](https://base.statuspage.io)

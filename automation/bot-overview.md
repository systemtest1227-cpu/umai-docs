# Bot Overview

The UmAI automation bot is a Node.js service that performs ongoing vault maintenance without human intervention. It harvests trading fees, rebalances liquidity positions, distributes referral rewards, and sends real-time alerts to Discord.

---

## Architecture

The bot runs as an **Express.js HTTP server** on **port 3001**. It exposes health-check and management API endpoints while running background cron jobs for each automated task.

```
┌──────────────────────────────────────────────┐
│              UmAI Automation Bot              │
│             Express.js :3001                  │
├──────────────────────────────────────────────┤
│                                              │
│  ┌────────────┐  ┌─────────────┐             │
│  │ Harvester  │  │ Rebalancer  │             │
│  └────────────┘  └─────────────┘             │
│  ┌──────────────┐ ┌──────────────────┐       │
│  │ Distributor  │ │ ReferralManager  │       │
│  └──────────────┘ └──────────────────┘       │
│  ┌────────────┐                              │
│  │  Alerter   │                              │
│  └────────────┘                              │
│                                              │
│  Cron Scheduler  ─────────────────────────── │
│  Circuit Breaker ─────────────────────────── │
│  Gas Monitor     ─────────────────────────── │
│                                              │
└──────────────────────────────────────────────┘
        │               │              │
        ▼               ▼              ▼
   Base Mainnet    Discord Webhook   API Clients
```

### Tech Stack

| Component | Technology |
|-----------|------------|
| Runtime | Node.js |
| HTTP Framework | Express.js |
| Blockchain | ethers.js (v6) |
| Scheduling | node-cron |
| Alerts | Discord Webhooks |

---

## Modules

### Harvester

Collects accumulated trading fees from the Uniswap V3 position, splits them according to the fee structure, and reinvests the remainder into the vault. Runs on a configurable cron schedule (default: daily at midnight UTC).

See [Harvester](harvester.md) for full details.

### Rebalancer

Monitors the current tick of the WETH/USDC pool and repositions the vault's liquidity when price drifts outside the active range. Checks every 5 minutes and enforces a 60-minute cooldown between rebalances.

See [Rebalancer](rebalancer.md) for full details.

### Distributor

Handles the distribution of referral rewards to eligible users. Works in concert with the ReferralManager to calculate and execute payouts.

### ReferralManager

Manages referral codes, tracks user-to-code mappings, and calculates reward distributions. Exposes CRUD API endpoints for referral code administration.

### Alerter

Sends structured notifications to a Discord channel via webhook. All other modules call the Alerter to report successes, failures, and warnings.

---

## Cron Scheduling

Each automated task is registered as a cron job using `node-cron`. Schedules are configured through environment variables so operators can tune timing without code changes.

| Job | Default Schedule | Env Variable |
|-----|-----------------|--------------|
| Harvest | `0 0 * * *` (midnight UTC) | `HARVEST_CRON` |
| Rebalance Check | `*/5 * * * *` (every 5 min) | `REBALANCE_CRON` |
| Referral Distribution | Manual / on-demand | `DISTRIBUTE_CRON` |

```bash
# Example: run harvest at 2:00 AM UTC instead of midnight
HARVEST_CRON="0 2 * * *"
```

---

## Circuit Breaker Pattern

To prevent runaway failures from draining gas or spamming the network, the bot implements a **circuit breaker** that pauses operations after repeated failures.

### How It Works

1. Each module maintains a **consecutive failure counter**.
2. When a task fails, the counter increments by 1.
3. After **5 consecutive failures**, the circuit breaker **trips** and the module enters a paused state.
4. While paused, the cron job continues to fire but the module skips execution and logs a warning.
5. The circuit resets automatically after a configurable cooldown period, or an operator can manually reset it via the API.
6. Any successful execution immediately resets the failure counter to 0.

```
Normal ──[failure]──▶ Count=1 ──[failure]──▶ Count=2 ... ──[5th failure]──▶ PAUSED
  ▲                                                                          │
  └──────────────────────[cooldown elapsed / manual reset]───────────────────┘
```

### Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `CIRCUIT_BREAKER_THRESHOLD` | `5` | Failures before tripping |
| `CIRCUIT_BREAKER_COOLDOWN` | `3600` | Seconds before auto-reset |

---

## Retry Logic with Exponential Backoff

Before a failure is counted against the circuit breaker, each operation is retried with exponential backoff:

| Attempt | Delay |
|---------|-------|
| 1st retry | 2 seconds |
| 2nd retry | 4 seconds |
| 3rd retry | 8 seconds |

After 3 retries (4 total attempts), the operation is marked as failed and the circuit breaker counter increments.

```
Attempt 1 ──[fail]──▶ wait 2s ──▶ Attempt 2 ──[fail]──▶ wait 4s ──▶ Attempt 3 ──[fail]──▶ wait 8s ──▶ Attempt 4 ──[fail]──▶ FAILURE
```

The maximum retry count and base delay are configurable:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `MAX_RETRIES` | `3` | Number of retries after initial attempt |
| `RETRY_BASE_DELAY` | `2000` | Base delay in milliseconds |

---

## Gas Balance Monitoring

Before executing any on-chain transaction, the bot checks the operator wallet's ETH balance on Base. If the balance falls below the configured threshold, the transaction is skipped and a warning is sent to Discord.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `GAS_THRESHOLD` | `0.01` | Minimum ETH balance required (in ETH) |

```
Check Balance ──▶ Balance ≥ 0.01 ETH? ──[yes]──▶ Execute Transaction
                                        ──[no]──▶ Skip + Alert Discord
```

The gas monitor runs before every harvest, rebalance, and distribution operation. This prevents transactions from failing mid-execution due to insufficient gas, which would waste the gas spent on the failed transaction.

---

## Discord Webhook Notifications

The bot sends real-time notifications to a Discord channel for all significant events. Messages are color-coded by severity and include structured embeds with transaction details.

### Event Types

| Event | Color | Example |
|-------|-------|---------|
| Harvest Success | Green | "Harvested 0.05 ETH + 120 USDC in fees" |
| Rebalance Success | Blue | "Rebalanced to tick range [-204360, -200360]" |
| Operation Failed | Red | "Harvest failed: insufficient gas" |
| Low Gas Warning | Yellow | "Operator balance below 0.01 ETH" |
| Circuit Breaker Tripped | Red | "Harvester paused after 5 failures" |

### Configuration

```bash
# Discord webhook URL (required for alerts)
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/YOUR_WEBHOOK_ID/YOUR_WEBHOOK_TOKEN"
```

To set up a Discord webhook:

1. Open your Discord server settings.
2. Navigate to **Integrations** > **Webhooks**.
3. Click **New Webhook** and configure the channel.
4. Copy the webhook URL and set it as the `DISCORD_WEBHOOK_URL` environment variable.

---

## Environment Variables Summary

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `RPC_URL` | Yes | -- | Base Mainnet RPC endpoint |
| `PRIVATE_KEY` | Yes | -- | Operator wallet private key |
| `VAULT_ADDRESS` | Yes | -- | UmAI Vault proxy address |
| `BOT_PORT` | No | `3001` | HTTP server port |
| `DISCORD_WEBHOOK_URL` | No | -- | Discord alerts webhook |
| `HARVEST_CRON` | No | `0 0 * * *` | Harvest schedule |
| `REBALANCE_CRON` | No | `*/5 * * * *` | Rebalance check schedule |
| `GAS_THRESHOLD` | No | `0.01` | Min ETH for transactions |
| `CIRCUIT_BREAKER_THRESHOLD` | No | `5` | Failures before pause |
| `MAX_RETRIES` | No | `3` | Retry attempts per operation |
| `API_KEY` | No | -- | Optional API authentication key |

> **Security Note:** Never commit `PRIVATE_KEY` or `DISCORD_WEBHOOK_URL` to version control. Use environment files (`.env`) that are excluded via `.gitignore`.

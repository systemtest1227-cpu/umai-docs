# UnAI

## AI-Powered Concentrated Liquidity Optimization

UnAI is an AI-driven protocol that automatically manages concentrated liquidity positions on Uniswap V3. It eliminates the complexity of manual LP management by using intelligent algorithms to optimize price ranges, rebalance positions, and compound earned fees -- all on your behalf.

---

## The Problem

Providing liquidity on Uniswap V3 is fundamentally different from V2. Concentrated liquidity demands that LPs choose specific price ranges, and the consequences of getting it wrong are severe:

- **Range management is complex.** Choosing optimal tick ranges requires deep understanding of volatility, price action, and pool dynamics. Most retail users lack the tools and expertise to do this effectively.
- **Impermanent loss is amplified.** Concentrated positions magnify IL compared to full-range V2 positions. A 5% price move can wipe out weeks of fee earnings.
- **Ranges go stale.** When price moves outside your selected range, your position earns zero fees. Manual repositioning is slow, gas-intensive, and often too late.
- **Fee compounding is manual.** Earned fees sit idle unless the LP actively collects and reinvests them, leaving significant yield on the table.
- **24/7 monitoring is unrealistic.** Markets move around the clock. Human LPs cannot watch positions continuously.

The result: most Uniswap V3 LPs underperform a simple buy-and-hold strategy.

---

## The Solution

UnAI solves these problems with a fully automated vault system:

| What UnAI Does | How |
|---|---|
| **Optimizes price ranges** | AI calculates optimal tick ranges based on current volatility and pool conditions |
| **Auto-rebalances** | Monitors positions 24/7 and repositions when the price drifts outside the range, with anti-fake-breakout confirmation to avoid unnecessary rebalances |
| **Compounds fees** | Earned trading fees are automatically reinvested into the position, maximizing APR |
| **Handles swaps** | Accepts USDC-only deposits and automatically swaps to the optimal token ratio for the LP position |
| **Distributes rewards** | Earned WETH rewards are pushed directly to your wallet daily |

### How It Works (Simplified)

```
1. Deposit USDC into the UnAI Vault
2. The vault swaps to the optimal WETH/USDC ratio
3. AI calculates the best concentrated liquidity range
4. A Uniswap V3 LP position is minted
5. The AI monitors and rebalances as needed
6. Earned fees are compounded automatically
7. WETH rewards are pushed to your wallet daily
8. Withdraw anytime (after lock period) back to USDC
```

---

## Key Features

### USDC-Only Deposits
No need to manage multiple tokens. Deposit USDC, and the vault handles all token swaps internally using optimal swap calculations for concentrated liquidity.

### Lock Periods
Choose your commitment level:
- **3 months** -- 30% performance fee
- **6 months** -- 30% performance fee
- **12 months** -- 30% performance fee

All lock periods share the same 30% performance fee rate. Lock periods determine when you can withdraw your principal.

### WETH Reward Distribution
Your share of trading fees (70%) is automatically pushed to your wallet as WETH every day. No need to manually claim -- rewards arrive directly. If a push transfer fails, rewards are stored as pending and can be claimed via `claimRewards()`.

### Referral Affiliate Program
UnAI offers an approval-based referral program. Users apply for a referral code through the app, and once approved by the platform, they receive a unique code to share. Referral code holders earn a commission from the protocol's fee share when users they referred generate trading fees. This is an affiliate model -- referred users pay the same 30% performance fee as everyone else.

### Auto-Rebalance with Breakout Confirmation
The vault continuously checks whether positions need rebalancing. When the price moves out of range, the bot uses a **Breakout Confirmation** system that waits for multiple consecutive confirmations before executing, preventing unnecessary rebalances during temporary price spikes.

### Non-Transferable Shares
Vault shares (ERC20) are non-transferable by design, preventing lock period bypass through secondary markets.

---

## Network

UnAI is live on **Base Mainnet**, targeting the WETH/USDC 0.30% fee tier pool.

**Production Vault Address:**
```
0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5
```

---

## Documentation

| Section | Description |
|---|---|
| [Architecture Overview](architecture/overview.md) | System components, data flow, and contract evolution |
| [Smart Contracts Overview](smart-contracts/overview.md) | Contract versions, interfaces, and access control |
| [Vault Mechanics](smart-contracts/vault-mechanics.md) | Deposit, withdraw, rebalance, and share token flows |
| [Fee Structure](smart-contracts/fee-structure.md) | Lock periods, fee calculation, referrals, and distribution |
| [Security](smart-contracts/security.md) | Reentrancy guards, slippage protection, and more |

---

## Links

- **Launch App**: [un-ai.io/app](https://un-ai.io/app)
- **Website**: [un-ai.io](https://un-ai.io)
- **Base Explorer (Vault)**: [basescan.org/address/0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5](https://basescan.org/address/0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5)

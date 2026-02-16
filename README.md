# UmAI

## AI-Powered Concentrated Liquidity Optimization

UmAI is an AI-driven protocol that automatically manages concentrated liquidity positions on Uniswap V3 (and soon V4). It eliminates the complexity of manual LP management by using intelligent algorithms to optimize price ranges, rebalance positions, and compound earned fees -- all on your behalf.

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

UmAI solves these problems with a fully automated vault system:

| What UmAI Does | How |
|---|---|
| **Optimizes price ranges** | AI calculates optimal tick ranges based on current volatility and pool conditions |
| **Auto-rebalances** | Monitors positions 24/7 and repositions when the price drifts outside the range, with TWAP oracle protection against manipulation |
| **Compounds fees** | Earned trading fees are automatically reinvested into the position, maximizing APR |
| **Handles swaps** | Accepts USDC-only deposits and automatically swaps to the optimal token ratio for the LP position |
| **Protects against MEV** | Slippage protection, deadline checks, and TWAP validation on every operation |

### How It Works (Simplified)

```
1. Deposit USDC into the UmAI Vault
2. The vault swaps to the optimal WETH/USDC ratio
3. AI calculates the best concentrated liquidity range
4. A Uniswap V3 LP position is minted
5. The AI monitors and rebalances as needed
6. Earned fees are compounded automatically
7. Withdraw anytime (after lock period) back to USDC
```

---

## Key Features

### USDC-Only Deposits
No need to manage multiple tokens. Deposit USDC, and the vault handles all token swaps internally using optimal swap calculations for concentrated liquidity.

### Lock Periods with Variable Fees
Choose your commitment level:
- **3 months** -- 50% performance fee
- **6 months** -- 40% performance fee
- **12 months** -- 30% performance fee

Longer locks reward you with lower fees on earned yield.

### Referral System
Referral codes can unlock custom fee rates below the default 35%. Each code is applied once at deposit time and persists for the life of the position.

### Auto-Rebalance with TWAP Protection
The vault continuously checks whether positions need rebalancing. When triggered, it uses a 30-minute TWAP oracle to validate the current price against manipulation, ensuring rebalances execute at fair market prices.

### Non-Transferable Shares
Vault shares (ERC20) are non-transferable by design, preventing lock period bypass through secondary markets.

---

## Network

UmAI is live on **Base Mainnet**, targeting the WETH/USDC 0.30% fee tier pool. Multi-chain expansion (Arbitrum, Polygon, Ethereum Mainnet) is on the roadmap.

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
| [Security](smart-contracts/security.md) | Reentrancy guards, TWAP checks, slippage protection, and more |

---

## Links

- **Launch App**: [app.umai.finance](https://app.umai.finance)
- **Website**: [umai.finance](https://umai.finance)
- **GitHub**: [github.com/UmAI-protocol](https://github.com/UmAI-protocol)
- **Base Explorer (Vault)**: [basescan.org/address/0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5](https://basescan.org/address/0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5)

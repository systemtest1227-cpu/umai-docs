# Frequently Asked Questions

---

## General

### What is UmAI?

UmAI is an **AI-powered concentrated liquidity optimizer**. It uses artificial intelligence to automatically manage Uniswap V3/V4 liquidity positions on your behalf. Instead of manually setting price ranges and monitoring markets 24/7, you simply deposit USDC and let UmAI's AI engine handle everything -- optimizing ranges, rebalancing positions, and maximizing your trading fee yield.

### What chain is UmAI on?

UmAI is currently live on **Base**, an Ethereum Layer 2 network. Multi-chain expansion to Ethereum mainnet, Arbitrum, Polygon, and other networks is planned for the future.

### Who is behind UmAI?

UmAI is a protocol with a dedicated team focused on AI-driven DeFi strategies. The smart contracts are deployed on-chain and operate transparently. A manager role handles operational tasks like rebalancing and harvesting, but your funds are always held in the non-custodial smart contract -- never by a person.

---

## Deposits

### What tokens can I deposit?

**USDC only.** When you deposit, the vault automatically converts your USDC into a 50/50 WETH/USDC liquidity position on Uniswap. You do not need to manually acquire WETH or set up the LP position yourself.

### What is the minimum deposit?

The minimum deposit is **500 USDC**. There is no maximum limit.

### Can I deposit multiple times?

Yes. You can make as many deposits as you want. Each deposit is tracked independently with its own lock period, share count, and earnings. See [Lock Periods](./user-guide/lock-periods.md) for more details.

### How do I get USDC on Base?

You can:
- **Bridge from Ethereum** using the [Base Bridge](https://bridge.base.org) or a third-party bridge
- **Swap on a Base DEX** like Uniswap or Aerodrome if you have ETH on Base
- **Withdraw from a centralized exchange** that supports Base (e.g., Coinbase)

See [Getting Started](./user-guide/getting-started.md) for detailed instructions.

---

## How It Works

### How does the AI work?

UmAI's AI engine runs continuously and performs three core functions:

1. **Price prediction** -- Analyzes market data to anticipate price movements.
2. **Range optimization** -- Sets the optimal tick range for concentrated liquidity to capture maximum trading fees.
3. **Auto-rebalancing** -- When the price moves outside the current range, the AI triggers a rebalance to reposition the liquidity into a new optimal range.

The result is a hands-free yield experience where your capital is always working efficiently.

### What is impermanent loss?

Impermanent loss (IL) is a risk that occurs when you provide liquidity and the price of the assets in the pool changes. If one token's price moves significantly relative to the other, the value of your position can be lower than if you had simply held the tokens.

**How UmAI minimizes it:**
- The AI actively monitors price movements and adjusts ranges to keep positions well-centered around the current price.
- Dynamic rebalancing reduces exposure to large price swings.
- Concentrated liquidity in tight, AI-optimized ranges earns more fees, which can offset IL in most market conditions.

> **Note:** Impermanent loss cannot be fully eliminated. It is an inherent risk of providing liquidity. UmAI aims to minimize it and ensure that fee earnings outpace any IL experienced.

### Who manages the vault?

The vault has a **manager role** that is responsible for:
- Triggering rebalances when the AI determines a new optimal range
- Harvesting earned fees from the Uniswap position
- Executing operational functions on the smart contract

As a user, you do not need to do anything. You just deposit and the manager handles all operational tasks. Your funds are always held in the smart contract itself -- the manager cannot take or move your principal.

### What happens if the position goes out of range?

If the market price moves outside the current liquidity range:
- The position temporarily stops earning trading fees.
- The AI engine detects the out-of-range condition.
- An **auto-rebalance** is triggered to move the liquidity into a new range centered around the current price.
- Alternatively, the vault manager can manually trigger a rebalance.
- Once rebalanced, the position resumes earning fees.

This process usually happens within minutes, thanks to UmAI's continuous monitoring.

---

## Earnings and Fees

### What is the APR?

APR (Annual Percentage Rate) represents the projected annual yield based on current trading fee earnings. It is **variable and not fixed** -- it changes based on:

- Trading volume on the underlying Uniswap WETH/USDC pool
- Market volatility (more volatility generally means more trading, which means more fees)
- How efficiently the AI manages tick ranges
- The performance fee rate applied to your position

The dashboard shows the current live APR so you can track performance in real time.

### How are fees calculated?

UmAI charges a **performance fee** -- a percentage of the trading fees your position earns. The fee rate depends on your [lock period](./user-guide/lock-periods.md):

| Lock Period | Performance Fee | You Keep |
|---|---|---|
| 3 months | 50% | 50% of earned fees |
| 6 months | 40% | 60% of earned fees |
| 12 months | 30% | 70% of earned fees |

There are **no deposit fees, no withdrawal fees, and no management fees**. The only fee is the performance fee on earnings.

If you do not use a referral code, the default performance fee rate is 35%. See [Referral Program](./user-guide/referral-program.md) for details.

### Can I see my earnings?

Yes. The UmAI dashboard shows:

- **Pending rewards** -- The fees you have earned that are ready to claim.
- **Earnings history chart** -- A visual chart of your yield over time, with 1-hour, 1-day, and 1-week views.
- **Total earnings** -- The cumulative amount you have earned across all your deposits.
- **Current APR** -- The live annualized rate your position is earning.

---

## Withdrawals

### Can I withdraw anytime?

After your **lock period expires**, yes -- you can withdraw at any time. There is no deadline to withdraw after unlock; your funds keep earning yield until you choose to pull them out.

During the lock period, your principal cannot be withdrawn. However, you can **claim earned rewards at any time**, even while locked.

See [Deposit and Withdraw](./user-guide/deposit-withdraw.md) for the full withdrawal process.

### What do I receive when I withdraw?

When you withdraw, the vault:
1. Removes your share of liquidity from the Uniswap position.
2. Converts the WETH/USDC back to USDC.
3. Deducts the performance fee from your earned fees.
4. Sends the remaining USDC to your wallet.

You receive **USDC** -- your original deposit plus your net share of earned fees.

### Can I claim rewards during the lock period?

**Yes.** Pending rewards are claimable at any time, regardless of your lock status. Claiming rewards does not affect your lock timer or your deposited principal. See [Lock Periods](./user-guide/lock-periods.md) for details.

---

## Security

### Are my funds safe?

UmAI is designed with security as a top priority:

- **Non-custodial smart contract** -- Your funds are held in the vault smart contract, not by any individual or team.
- **Audited code** -- The smart contracts have been audited for security vulnerabilities.
- **ReentrancyGuard** -- Protection against reentrancy attacks, one of the most common smart contract exploits.
- **TWAP protection** -- Time-Weighted Average Price checks are used to prevent price manipulation during swaps and rebalances.
- **On-chain transparency** -- All transactions, rebalances, and fee collections are verifiable on the Base block explorer.

> **Important:** While these measures significantly reduce risk, no DeFi protocol is completely risk-free. Smart contract risk, market risk, and impermanent loss are inherent to DeFi. Only deposit funds you can afford to have at risk.

### Has the smart contract been audited?

Yes. The UmAI vault smart contract includes industry-standard protections such as ReentrancyGuard and TWAP oracle validation. Details of specific audits are available through UmAI's official channels.

---

## Referrals

### How do referral codes work?

Referral codes are applied by visiting UmAI through a special link: `https://app.umai.finance/?ref=CODE`. Once applied, the code is permanently linked to your wallet and cannot be changed. Referred users may receive reduced performance fees, and referral code holders earn a share of fees from their referrals.

See [Referral Program](./user-guide/referral-program.md) for the full breakdown.

---

## Troubleshooting

### My transaction failed. What should I do?

Common causes and solutions:

1. **Insufficient gas (ETH on Base)** -- Make sure you have a small amount of ETH on Base to pay for transaction fees.
2. **Slippage too low** -- During high volatility, the default 0.5% slippage may not be enough. Try increasing it slightly (up to the 1% maximum enforced by the contract).
3. **Deposit below minimum** -- Make sure you are depositing at least 500 USDC.
4. **Wrong network** -- Confirm your wallet is connected to Base (Chain ID: 8453).

### I do not see my deposit on the dashboard.

- Wait a few seconds for the transaction to be confirmed on Base.
- Refresh the page.
- Check the transaction on [BaseScan](https://basescan.org) to confirm it was successful.
- Make sure you are connected with the same wallet you used to deposit.

### I cannot switch to the Base network.

- Try adding Base manually in your wallet settings (see [Getting Started](./user-guide/getting-started.md) for network details).
- Some older wallet versions may not support Base. Update your wallet to the latest version.
- If using WalletConnect, try disconnecting and reconnecting.

---

## Still Have Questions?

Join the UmAI community for help:

- **Discord** -- Chat with the team and other users
- **Twitter / X** -- Follow for updates and announcements
- **Telegram** -- Join the discussion

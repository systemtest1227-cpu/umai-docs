# Depositing and Withdrawing

This guide explains the full process for depositing USDC into UmAI, how your funds are managed behind the scenes, and how to withdraw when you are ready.

---

## How Deposits Work

Depositing into UmAI is straightforward. You deposit USDC, choose a lock period, and the vault handles the rest.

### Step-by-Step Deposit Process

1. **Connect your wallet** and make sure you are on the Base network. (See [Getting Started](./getting-started.md) if you need help.)

2. **Enter your USDC amount.**
   - The minimum deposit is **500 USDC**.
   - Type or paste the amount you want to deposit.

3. **Select a lock period.**
   - Choose from **3 months**, **6 months**, or **12 months**.
   - Longer lock periods give you a lower performance fee, meaning you keep more of your earnings.
   - See [Lock Periods](./lock-periods.md) for a detailed comparison.

4. **Approve USDC spending.**
   - If this is your first time depositing, you will need to approve the UmAI vault smart contract to spend your USDC.
   - Click "Approve" and confirm the approval transaction in your wallet.
   - This is a one-time step (unless you revoke the approval later).

5. **Confirm the deposit transaction.**
   - After approval, click "Deposit" and confirm the transaction in your wallet.
   - Wait for the transaction to be confirmed on Base (usually a few seconds).

6. **Receive vault shares.**
   - Once confirmed, you will receive vault shares representing your ownership stake in the vault.
   - Your deposit and lock timer will appear on your dashboard.

> **Tip:** You can make multiple deposits with different lock periods. Each deposit is tracked independently with its own lock timer and share count.

---

## What Happens Behind the Scenes

When you deposit USDC, the vault does not simply hold it. Here is what happens:

1. **Optimal swap to WETH/USDC ratio** -- A portion of your USDC is automatically swapped to WETH (Wrapped ETH) at the optimal ratio for the current concentrated liquidity range. The exact split depends on the position's tick bounds relative to the current price -- it is not necessarily a 50/50 split.

2. **LP position created** -- The vault deploys your WETH/USDC into a concentrated liquidity position on Uniswap V3 within an AI-optimized price range.

3. **AI monitors and rebalances** -- UmAI's AI engine continuously monitors the market price and adjusts the tick range to keep your position earning maximum fees.

4. **Fees accumulate** -- As traders swap through the Uniswap pool, your position earns a share of the trading fees proportional to your liquidity.

> **Note:** The auto-swap means you are exposed to both USDC and ETH price movements. If ETH drops significantly, the value of your position may decrease even though you are earning fees. This is known as impermanent loss, which UmAI's AI actively works to minimize.

---

## How Withdrawals Work

You can withdraw your funds after your lock period has expired.

### Step-by-Step Withdrawal Process

1. **Check your lock status.**
   - Go to your dashboard and find the deposit you want to withdraw.
   - If the status shows **"Locked"**, you must wait until the lock period expires.
   - If the status shows **"Unlocked"**, you can proceed with the withdrawal.

2. **Select the deposit to withdraw.**
   - If you have multiple deposits, each one has its own lock timer.
   - Click on the unlocked deposit you want to withdraw.

3. **Confirm the withdrawal.**
   - Click "Withdraw" and confirm the transaction in your wallet.
   - The vault will remove your liquidity from the Uniswap position and convert everything back to USDC.

4. **Receive USDC back.**
   - Your USDC (original deposit plus your share of earned fees, minus the performance fee) will be sent to your wallet.
   - The transaction will appear on the Base block explorer.

> **Important:** You cannot withdraw a deposit that is still locked. Each deposit must individually reach its unlock date before it can be withdrawn. There is no early withdrawal option.

---

## Claiming Rewards

You do not have to wait until your lock period ends to claim your earned rewards.

- **Pending rewards can be claimed at any time**, even while your principal is still locked.
- Go to your dashboard and look for the "Claim Rewards" button.
- Click it, confirm the transaction, and your earned fees will be sent to your wallet in USDC.

> **Tip:** Claiming rewards does not affect your lock timer or your deposited principal. You can claim as often as you like without impacting your position.

---

## Slippage Settings

When depositing or withdrawing, a small amount of slippage may occur during the token swaps (USDC to WETH/USDC or back).

- **Default slippage tolerance: 0.5%**
- This means the transaction will fail if the price moves more than 0.5% between when you submit and when it executes.
- If you are experiencing failed transactions during high volatility, you may need to increase the slippage tolerance slightly.

> **Note:** The smart contract enforces a maximum slippage cap of 1% to protect against unfavorable swap rates. In most cases, the default 0.5% works well.

---

## Summary

| Action | When Available | What You Receive |
|---|---|---|
| Deposit | Anytime (min 500 USDC) | Vault shares |
| Claim Rewards | Anytime (even during lock) | Earned fees in USDC |
| Withdraw | After lock period expires | Principal + remaining fees in USDC |

---

## Related Guides

- [Getting Started](./getting-started.md)
- [Understanding Lock Periods](./lock-periods.md)
- [Referral Program](./referral-program.md)
- [FAQ](../faq.md)

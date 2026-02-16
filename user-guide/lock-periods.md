# Lock Periods

Lock periods are a core part of how UmAI works. By committing your funds for a set period, you unlock lower performance fees and keep a bigger share of the yield. This page explains exactly how lock periods work, what to expect, and how to make the best choice for your situation.

---

## Why Lock Periods Exist

UmAI uses lock periods for a practical reason: **commitment enables lower fees.**

When the vault knows that your funds will remain deposited for a guaranteed duration, it can plan liquidity deployment more efficiently. This leads to better position management and lower operational costs. Those savings are passed on to you in the form of reduced performance fees.

In short: the longer you commit, the more you earn.

---

## The Three Options

UmAI offers three lock period durations. The only difference between them is the **performance fee rate** -- the percentage of your earned trading fees that goes to the protocol.

### 3-Month Lock

| Detail | Value |
|---|---|
| Lock Duration | 3 months |
| Performance Fee | 50% |
| You Keep | **50%** of earned fees |

Best for: Users who want flexibility and are testing UmAI for the first time.

### 6-Month Lock

| Detail | Value |
|---|---|
| Lock Duration | 6 months |
| Performance Fee | 40% |
| You Keep | **60%** of earned fees |

Best for: Users with medium-term conviction who want a meaningful fee reduction.

### 12-Month Lock

| Detail | Value |
|---|---|
| Lock Duration | 12 months |
| Performance Fee | 30% |
| You Keep | **70%** of earned fees |

Best for: Long-term believers who want maximum yield retention.

> **Example:** If the vault earns $1,000 in trading fees on your deposit over your lock period:
> - **3-month lock:** You keep $500, protocol takes $500
> - **6-month lock:** You keep $600, protocol takes $400
> - **12-month lock:** You keep $700, protocol takes $300

---

## When Does the Lock Start?

Your lock period begins **at the exact moment your deposit transaction is confirmed** on the Base network. The unlock date is calculated from that timestamp.

For example, if you deposit with a 3-month lock on March 1, your funds will unlock on approximately June 1.

---

## What Happens During the Lock?

While your deposit is locked:

- **Your funds are actively working.** They are deployed as concentrated liquidity on Uniswap, earning trading fees 24/7.
- **The AI continuously optimizes your position.** Ranges are adjusted, rebalances happen, and fees are collected -- all automatically.
- **You cannot withdraw your principal.** Your deposited USDC (now in the form of vault shares) cannot be removed until the lock expires.
- **You CAN claim rewards.** See the next section.

---

## Can You Claim Rewards During the Lock?

**Yes.** Pending rewards can be claimed at any time, even while your principal is locked.

This means you do not have to wait months to see returns. As the vault earns fees from trading activity, your share of those fees accumulates and can be claimed whenever you want.

To claim:

1. Go to your dashboard.
2. Check the "Pending Rewards" section.
3. Click "Claim Rewards."
4. Confirm the transaction in your wallet.
5. Receive your earned fees in USDC.

Claiming rewards has no effect on your lock timer or your deposited principal. It is a separate action.

> **Tip:** You can claim rewards as often as you want. There is no penalty or cooldown for frequent claims.

---

## What Happens After Unlock?

Once your lock period expires:

- Your deposit status changes from **"Locked"** to **"Unlocked"** on your dashboard.
- You can **withdraw at any time** -- there is no rush. Your funds continue earning fees even after unlock.
- When you withdraw, the vault converts your position back to USDC and sends it to your wallet.

There is no auto-withdrawal. Your funds stay in the vault earning yield until you actively choose to withdraw.

---

## Multiple Deposits

You can make multiple deposits, and each one has its own independent lock timer.

For example:

| Deposit | Amount | Lock Period | Deposit Date | Unlock Date |
|---|---|---|---|---|
| Deposit 1 | 1,000 USDC | 3 months | Jan 1 | Apr 1 |
| Deposit 2 | 2,000 USDC | 12 months | Feb 15 | Feb 15 (next year) |
| Deposit 3 | 500 USDC | 6 months | Mar 10 | Sep 10 |

Each deposit tracks its own:
- Lock duration and unlock date
- Performance fee rate
- Share count
- Earned rewards

You can withdraw each deposit individually once it unlocks, without affecting any others.

> **Note:** There is no way to change the lock period after depositing. Choose carefully before confirming your transaction.

---

## Quick Comparison

| | 3 Months | 6 Months | 12 Months |
|---|---|---|---|
| Performance Fee | 50% | 40% | 30% |
| You Keep | 50% | 60% | 70% |
| Flexibility | Highest | Medium | Lowest |
| Yield Retention | Lower | Medium | Highest |
| Claim Rewards Early? | Yes | Yes | Yes |
| Early Withdrawal? | No | No | No |

---

## Related Guides

- [Getting Started](./getting-started.md)
- [How to Deposit and Withdraw](./deposit-withdraw.md)
- [Referral Program](./referral-program.md)
- [FAQ](../faq.md)

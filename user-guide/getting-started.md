# Getting Started

Welcome to UmAI -- the AI-powered concentrated liquidity optimizer built on Base. This guide walks you through everything you need to start earning yield in just a few minutes.

***

## Prerequisites

Before you begin, make sure you have the following ready:

### 1. A Compatible Wallet

UmAI supports the most popular Ethereum wallets through RainbowKit:

* **MetaMask** -- The most widely used browser extension wallet
* **Coinbase Wallet** -- Available as a browser extension or mobile app
* **WalletConnect** -- Connects any WalletConnect-compatible mobile wallet (Trust Wallet, Rainbow, etc.)

> **New to crypto wallets?** We recommend starting with [MetaMask](https://metamask.io). Install the browser extension, create an account, and securely back up your seed phrase before proceeding.

### 2. USDC on the Base Network

UmAI accepts **USDC only** as the deposit token. You need USDC on the **Base** network (not Ethereum mainnet, not Arbitrum -- specifically Base).

The minimum deposit is **500 USDC**.

***

## Connecting Your Wallet

1. Go to the UmAI app.
2. Click the **"Connect Wallet"** button in the top-right corner.
3. A RainbowKit modal will appear showing your wallet options.
4. Select your preferred wallet (MetaMask, Coinbase Wallet, or WalletConnect).
5. Approve the connection request in your wallet.
6. Once connected, your wallet address will appear in the header.

> **Tip:** If you have used the app before, your wallet may reconnect automatically.

***

## Switching to Base Network

UmAI runs on the **Base** network. If your wallet is connected to a different network (such as Ethereum mainnet), you will need to switch.

1. After connecting, UmAI will detect if you are on the wrong network.
2. You will see a prompt or a **"Switch Network"** button.
3. Click the button and approve the network switch in your wallet.
4. Your wallet will add Base automatically if it is not already configured.

**Base Network Details** (for manual setup):

| Field           | Value                      |
| --------------- | -------------------------- |
| Network Name    | Base                       |
| RPC URL         | `https://mainnet.base.org` |
| Chain ID        | 8453                       |
| Currency Symbol | ETH                        |
| Block Explorer  | `https://basescan.org`     |

***

## Getting USDC on Base

If you do not have USDC on the Base network yet, here are the most common ways to get it:

### Option A: Bridge from Ethereum

1. Go to the [Base Bridge](https://bridge.base.org) or use a third-party bridge like Superbridge.
2. Connect your wallet on Ethereum mainnet.
3. Select USDC as the token to bridge.
4. Enter the amount and confirm the bridge transaction.
5. Wait for the bridging process to complete (usually a few minutes).
6. Your USDC will appear in your wallet on the Base network.

### Option B: Buy on a Base DEX

1. If you already have ETH on Base, visit a decentralized exchange like Uniswap or Aerodrome on Base.
2. Swap ETH (or another token) for USDC.
3. Confirm the swap transaction in your wallet.

### Option C: Withdraw from a Centralized Exchange

Some centralized exchanges (like Coinbase) support direct withdrawals to the Base network. Check if your exchange supports Base withdrawals for USDC.

> **Important:** Always double-check that you are sending USDC to the Base network, not Ethereum mainnet or another chain. Sending tokens to the wrong network may result in lost funds.

***

## Dashboard Overview

Once your wallet is connected and you are on the Base network, you will see the UmAI dashboard. Here is what each section shows:

### Main Stats

* **Total Value Locked (TVL)** -- The total amount of assets currently deposited across the entire UmAI vault.
* **Fee APR** -- The current annualized percentage rate being earned from trading fees, after the AI optimizes your liquidity ranges.
* **Your Earnings** -- The total fees you have earned from your deposited positions.

### Earnings History Chart

A visual chart showing your earnings over time. You can toggle between different time frames (1 hour, 1 day, 1 week) to see how your yield has grown.

### Position Range

A visual indicator showing the current price range of the vault's concentrated liquidity position relative to the current market price:

* **"In Range"** (green) means the position is actively earning fees.
* **"Out of Range"** means the current price has moved outside the set range, and a rebalance is needed (the AI or vault manager handles this automatically).

***

## Understanding Your Position

After you deposit, here are the key metrics to track:

### Shares

When you deposit USDC, you receive **vault shares** that represent your proportional ownership of the vault. The more shares you hold, the larger your share of the earned fees.

### TVL (Total Value Locked)

This is the total value of all assets in the vault. As more users deposit, TVL grows. Your share of the TVL is determined by how many vault shares you own.

### APR (Annual Percentage Rate)

APR represents the projected annual return based on current fee earnings. This rate is **variable** -- it depends on:

* Trading volume on the underlying Uniswap pool
* How efficiently the AI sets the liquidity ranges
* Market conditions

> **Note:** APR is not fixed or guaranteed. It fluctuates based on market activity. The AI works to maximize it, but past performance does not guarantee future results.

***

## Next Steps

Now that you understand the basics, you are ready to make your first deposit.

* [How to Deposit and Withdraw](deposit-withdraw.md)
* [Understanding Lock Periods](lock-periods.md)
* [Referral Program](referral-program.md)
* [Frequently Asked Questions](../resources/faq.md)

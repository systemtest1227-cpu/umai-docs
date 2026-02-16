# Contract Addresses

All UmAI smart contracts are deployed on **Base Mainnet** (Chain ID: 8453). This page lists every contract address used by the protocol.

---

## Network Information

| Property | Value |
|----------|-------|
| Network | Base Mainnet |
| Chain ID | `8453` |
| RPC URL | `https://mainnet.base.org` |
| Block Explorer | [BaseScan](https://basescan.org) |
| Native Token | ETH |

---

## Core Protocol Contracts

| Contract | Address | BaseScan |
|----------|---------|----------|
| **UmAI Vault (UUPS Proxy)** | `0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5` | [View](https://basescan.org/address/0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5) |

The Vault is deployed behind a **UUPS (Universal Upgradeable Proxy Standard)** proxy. The proxy address above is the address users interact with. The implementation contract behind the proxy can be upgraded by the contract owner.

> **Important:** Always interact with the **proxy address**, not the implementation address. The proxy delegates all calls to the current implementation.

---

## Token Contracts

| Token | Address | Decimals | BaseScan |
|-------|---------|----------|----------|
| **WETH** | `0x4200000000000000000000000000000000000006` | 18 | [View](https://basescan.org/address/0x4200000000000000000000000000000000000006) |
| **USDC** | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` | 6 | [View](https://basescan.org/address/0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913) |

- **WETH** is the canonical Wrapped Ether on Base, used as `token0` in the pool.
- **USDC** is the official bridged USDC on Base (issued by Circle), used as `token1` in the pool.

---

## Uniswap V3 Contracts

| Contract | Address | BaseScan |
|----------|---------|----------|
| **Pool (WETH/USDC 0.3%)** | `0x6c561B446416E1A00E8E93E221854d6eA4171372` | [View](https://basescan.org/address/0x6c561B446416E1A00E8E93E221854d6eA4171372) |
| **NonfungiblePositionManager** | `0x03a520b32C04BF3bEEf7BEb72E919cf822Ed34f1` | [View](https://basescan.org/address/0x03a520b32C04BF3bEEf7BEb72E919cf822Ed34f1) |
| **SwapRouter** | `0x2626664c2603336E57B271c5C0b26F421741e481` | [View](https://basescan.org/address/0x2626664c2603336E57B271c5C0b26F421741e481) |

---

## Pool Parameters

The UmAI Vault operates on a single Uniswap V3 pool with the following parameters:

| Parameter | Value |
|-----------|-------|
| Pair | WETH / USDC |
| Fee Tier | 0.3% (3000) |
| Tick Spacing | 60 |
| Token0 | WETH |
| Token1 | USDC |

### Fee Tier Explained

The **0.3% fee tier** means that every swap through this pool charges traders a 0.3% fee, which is distributed to liquidity providers (including the UmAI Vault) proportional to their share of active liquidity.

### Tick Spacing

A tick spacing of **60** means that liquidity can only be placed at tick values that are multiples of 60. When the Rebalancer calculates new tick ranges, it rounds to the nearest valid tick:

```
validTick = floor(rawTick / 60) * 60
```

---

## Adding to Your Wallet

### MetaMask / Wallet Configuration

To add Base Mainnet to your wallet:

| Field | Value |
|-------|-------|
| Network Name | Base |
| RPC URL | `https://mainnet.base.org` |
| Chain ID | `8453` |
| Currency Symbol | ETH |
| Block Explorer | `https://basescan.org` |

### Importing the Vault Token

To track your UmAI Vault shares in MetaMask:

1. Open MetaMask and switch to Base network.
2. Click **Import tokens**.
3. Enter the Vault proxy address: `0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5`
4. The token symbol and decimals will auto-populate.
5. Click **Add Custom Token**.

---

## Verifying Contracts

All contracts can be verified on BaseScan. To verify the vault implementation:

1. Navigate to the [proxy address on BaseScan](https://basescan.org/address/0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5).
2. Click the **Contract** tab.
3. If the proxy is verified, click **Read as Proxy** to interact with the implementation ABI.
4. Click **Write as Proxy** to execute transactions through the BaseScan interface.

---

## Quick Reference

```
# UmAI Vault (interact with this address)
0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5

# Tokens
WETH:  0x4200000000000000000000000000000000000006
USDC:  0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913

# Uniswap V3
Pool:       0x6c561B446416E1A00E8E93E221854d6eA4171372
PosManager: 0x03a520b32C04BF3bEEf7BEb72E919cf822Ed34f1
SwapRouter: 0x2626664c2603336E57B271c5C0b26F421741e481

# Network
Chain ID: 8453 (Base Mainnet)
```

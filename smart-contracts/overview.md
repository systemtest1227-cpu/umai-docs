# Smart Contracts Overview

This page provides a high-level view of the UmAI smart contract system, including the version history, production deployment details, core interfaces, and access control model.

---

## Contract Versions

UmAI's vault contract has evolved through four versions. Each version added significant features while maintaining backward compatibility with existing depositor state.

| Feature | V1 | V2 | V3 | V3 Upgradeable |
|---|:---:|:---:|:---:|:---:|
| USDC deposits | Yes | Yes | Yes | Yes |
| ERC20 share tokens | Yes | Yes | Yes | Yes |
| Optimal swap calculation | Yes | Yes | Yes | Yes |
| Manager-triggered harvest | Yes | Yes | Yes | Yes |
| Manager-triggered rebalance | Yes | Yes | Yes | Yes |
| Fee distribution to feeRecipient | Yes | Yes | Yes | Yes |
| Lock periods (3/6/12 months) | -- | Yes | Yes | Yes |
| Non-transferable shares | -- | Yes | Yes | Yes |
| Weighted fee calculation | -- | Yes | Yes | Yes |
| Deadline-based slippage | -- | Yes | Yes | Yes |
| Referral system | -- | -- | Yes | Yes |
| Auto-rebalance (`needsRebalance`) | -- | -- | Yes | Yes |
| TWAP oracle validation | -- | -- | Yes | Yes |
| Rebalance cooldown | -- | -- | Yes | Yes |
| Push-based fee distribution | -- | -- | Yes | Yes |
| Pending rewards fallback | -- | -- | Yes | Yes |
| UUPS proxy upgradeability | -- | -- | -- | Yes |
| `initialize()` function | -- | -- | -- | Yes |

---

## Production Contract

The current production deployment uses **MellowLiteVaultV3Upgradeable** behind a UUPS (ERC1967) proxy.

| Property | Value |
|---|---|
| **Contract Name** | MellowLiteVaultV3Upgradeable |
| **Network** | Base Mainnet |
| **Proxy Address** | `0x331a49587A42b92fd0bC2B31cdB07BD0cfC7C8f5` |
| **Proxy Standard** | ERC1967 (UUPS) |
| **Solidity Version** | ^0.8.20 |
| **OpenZeppelin Base** | ERC20Upgradeable, OwnableUpgradeable, ReentrancyGuardUpgradeable, UUPSUpgradeable |

> **Note:** The internal contract names (`MellowLiteVault`, `UniLink`) are legacy development names. The public-facing brand is **UmAI**. All user documentation and interfaces should use "UmAI Vault" or "UmAI" exclusively.

### Inheritance Chain

```solidity
contract MellowLiteVaultV3Upgradeable is
    Initializable,
    ERC20Upgradeable,
    OwnableUpgradeable,
    ReentrancyGuardUpgradeable,
    UUPSUpgradeable
{
    // ...
}
```

The contract inherits from five OpenZeppelin upgradeable base contracts:

- **Initializable** -- Provides the `initializer` modifier for proxy-safe construction
- **ERC20Upgradeable** -- Share token functionality (mint, burn, balance tracking)
- **OwnableUpgradeable** -- Single-owner access control
- **ReentrancyGuardUpgradeable** -- Reentrancy protection via `nonReentrant` modifier
- **UUPSUpgradeable** -- Upgrade authorization logic

---

## Core Interfaces

The vault interacts with three primary Uniswap V3 interfaces:

### INonfungiblePositionManager

Used for creating, modifying, and closing LP positions.

```solidity
interface INonfungiblePositionManager {
    struct MintParams {
        address token0;
        address token1;
        uint24 fee;
        int24 tickLower;
        int24 tickUpper;
        uint256 amount0Desired;
        uint256 amount1Desired;
        uint256 amount0Min;
        uint256 amount1Min;
        address recipient;
        uint256 deadline;
    }

    function mint(MintParams calldata params)
        external
        payable
        returns (
            uint256 tokenId,
            uint128 liquidity,
            uint256 amount0,
            uint256 amount1
        );

    function increaseLiquidity(IncreaseLiquidityParams calldata params)
        external
        payable
        returns (uint128 liquidity, uint256 amount0, uint256 amount1);

    function decreaseLiquidity(DecreaseLiquidityParams calldata params)
        external
        payable
        returns (uint256 amount0, uint256 amount1);

    function collect(CollectParams calldata params)
        external
        payable
        returns (uint256 amount0, uint256 amount1);
}
```

**Vault usage:**
- `mint()` -- Called during initial deposit or rebalance to create a new LP position
- `increaseLiquidity()` -- Called on subsequent deposits to add to the existing position
- `decreaseLiquidity()` -- Called during withdrawals and rebalances to remove liquidity
- `collect()` -- Called during harvests and rebalances to collect earned trading fees

### ISwapRouter

Used for all token swaps (USDC to WETH on deposit, WETH to USDC on withdrawal).

```solidity
interface ISwapRouter {
    struct ExactInputSingleParams {
        address tokenIn;
        address tokenOut;
        uint24 fee;
        address recipient;
        uint256 deadline;
        uint256 amountIn;
        uint256 amountOutMinimum;
        uint160 sqrtPriceLimitX96;
    }

    function exactInputSingle(ExactInputSingleParams calldata params)
        external
        payable
        returns (uint256 amountOut);
}
```

**Vault usage:**
- Deposit: swap USDC to WETH at the calculated optimal ratio
- Withdrawal: swap WETH back to USDC so the user receives a single token
- Rebalance: swap tokens to match the new range's optimal ratio

### IUniswapV3Pool

Used for reading pool state (current price, tick, TWAP observation).

```solidity
interface IUniswapV3Pool {
    function slot0()
        external
        view
        returns (
            uint160 sqrtPriceX96,
            int24 tick,
            uint16 observationIndex,
            uint16 observationCardinality,
            uint16 observationCardinalityNext,
            uint8 feeProtocol,
            bool unlocked
        );

    function observe(uint32[] calldata secondsAgos)
        external
        view
        returns (
            int56[] memory tickCumulatives,
            uint160[] memory secondsPerLiquidityCumulativeX128s
        );
}
```

**Vault usage:**
- `slot0()` -- Read current price (`sqrtPriceX96`) and tick for swap calculations
- `observe()` -- Compute TWAP over 30-minute window for manipulation protection

---

## Access Control

The vault uses a three-tier access control model:

```
┌──────────────────────────────────────────────────┐
│                     Owner                         │
│  - Set manager address                           │
│  - Set fee recipient                             │
│  - Set fee rates and referral codes              │
│  - Emergency token recovery                      │
│  - Authorize contract upgrades (UUPS)            │
│  - All Manager permissions                       │
├──────────────────────────────────────────────────┤
│                    Manager                        │
│  - Trigger harvest (collect and distribute fees) │
│  - Trigger rebalance (reposition liquidity)      │
│  - Set new tick ranges                           │
├──────────────────────────────────────────────────┤
│                      User                         │
│  - Deposit USDC (with lock period selection)     │
│  - Withdraw USDC (after lock expiry)             │
│  - View position value and pending rewards       │
│  - Claim pending rewards                         │
└──────────────────────────────────────────────────┘
```

### Role Definitions

**Owner** (`onlyOwner` modifier)
- The deployer or designated admin address
- Has full control over contract configuration
- Is the only address authorized to call `_authorizeUpgrade` for UUPS upgrades
- Can set the manager address, fee recipient, default fee rate, and referral codes
- Can call `recoverToken()` for emergency recovery of accidentally sent tokens (but cannot drain the vault's own WETH or USDC)

**Manager** (`onlyManager` modifier)
- An operational address (typically the keeper bot)
- Set by the owner via `setManager(address)`
- Can trigger `harvest()` to collect and distribute trading fees
- Can trigger `rebalance()` to reposition liquidity into new tick ranges
- Cannot modify contract configuration or access user funds directly

**User** (no special modifier beyond standard checks)
- Any address that calls `deposit()` with USDC
- Receives non-transferable ERC20 share tokens representing their proportion of the vault
- Can call `withdraw()` after their lock period expires
- Can claim any pending reward distributions

---

## Related Pages

- [Architecture Overview](../architecture/overview.md) -- System-level architecture and data flows
- [Vault Mechanics](vault-mechanics.md) -- Detailed deposit, withdraw, and rebalance logic
- [Fee Structure](fee-structure.md) -- Lock periods, fee calculation, and referrals
- [Security](security.md) -- Reentrancy, TWAP, slippage, and emergency recovery

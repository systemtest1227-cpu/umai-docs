# Security

This page details the security mechanisms built into the UmAI vault smart contracts, covering reentrancy protection, price oracle validation, slippage controls, access restrictions, and emergency recovery.

---

## Security Model Overview

The UmAI vault handles user funds in a non-custodial manner through Uniswap V3 LP positions. Security is enforced at multiple layers:

```
┌─────────────────────────────────────────────────────────────┐
│                    Access Control Layer                       │
│  Owner / Manager / User role separation                      │
├─────────────────────────────────────────────────────────────┤
│                  Reentrancy Protection Layer                  │
│  ReentrancyGuard on all state-changing functions             │
├─────────────────────────────────────────────────────────────┤
│                   Price Validation Layer                      │
│  TWAP oracle check (30-min window, 2% max deviation)         │
├─────────────────────────────────────────────────────────────┤
│                   Slippage Protection Layer                   │
│  User-specified slippage, max 1% cap, deadline checks        │
├─────────────────────────────────────────────────────────────┤
│                   Token Safety Layer                          │
│  SafeERC20, non-transferable shares, emergency recovery      │
├─────────────────────────────────────────────────────────────┤
│                   Upgrade Safety Layer                        │
│  UUPS onlyOwner authorization, initializer guard             │
└─────────────────────────────────────────────────────────────┘
```

---

## Reentrancy Protection

All state-changing external functions in the vault use OpenZeppelin's `ReentrancyGuardUpgradeable` via the `nonReentrant` modifier. This prevents reentrancy attacks where a malicious contract could re-enter the vault during an external call (e.g., during a token transfer or swap callback).

### Protected Functions

| Function | Modifier |
|---|---|
| `deposit()` | `nonReentrant` |
| `withdraw()` | `nonReentrant` |
| `harvest()` | `nonReentrant` |
| `rebalance()` | `nonReentrant` |
| `claimPendingRewards()` | `nonReentrant` |

```solidity
function deposit(
    uint256 amount,
    uint256 maxSlippage,
    uint8 lockPeriod,
    bytes32 referralCode
) external nonReentrant {
    // State changes and external calls are safe from reentrancy
    // ...
}
```

### Why This Matters

The vault makes multiple external calls during a single transaction (token transfers, Uniswap swaps, LP position operations). Without reentrancy protection, an attacker could exploit the intermediate state between these calls to drain funds or manipulate share calculations.

---

## TWAP Price Oracle Check

The vault validates the pool's current spot price against a Time-Weighted Average Price (TWAP) before executing rebalances. This is the primary defense against price manipulation attacks.

### Parameters

| Parameter | Value | Purpose |
|---|---|---|
| TWAP Window | 30 minutes (1800 seconds) | Time period for averaging |
| Max Deviation | 2% | Maximum allowed difference between spot and TWAP |

### Implementation

```solidity
function _validateTWAP() internal view {
    // Query the pool for tick cumulatives at t=0 and t=-1800s
    uint32[] memory secondsAgos = new uint32[](2);
    secondsAgos[0] = 1800; // 30 minutes ago
    secondsAgos[1] = 0;    // current

    (int56[] memory tickCumulatives, ) = pool.observe(secondsAgos);

    // Calculate TWAP tick
    int24 twapTick = int24(
        (tickCumulatives[1] - tickCumulatives[0]) / int56(int32(1800))
    );

    // Get current spot tick
    (, int24 currentTick, , , , , ) = pool.slot0();

    // Calculate absolute deviation
    int24 deviation = currentTick > twapTick
        ? currentTick - twapTick
        : twapTick - currentTick;

    require(deviation <= maxTwapDeviation, "TWAP: price deviation too high");
}
```

### Attack Scenarios Prevented

| Attack | How TWAP Prevents It |
|---|---|
| **Flash loan price manipulation** | Flash loans execute within a single block. The 30-minute TWAP is unaffected by single-block price spikes. |
| **Sandwich attacks on rebalances** | An attacker who moves the price before a rebalance would cause the TWAP check to fail, blocking the transaction. |
| **Oracle manipulation** | The TWAP is derived directly from the Uniswap V3 pool's built-in accumulator, which requires sustained capital to manipulate over 30 minutes. |

### When TWAP Is Checked

TWAP validation is performed at the beginning of every `rebalance()` call, before any liquidity is removed or re-added. If the check fails, the entire transaction reverts with no state changes.

---

## Slippage Protection

All swap operations enforce slippage limits to protect against unfavorable execution prices.

### User-Specified Slippage

Users pass a `maxSlippage` parameter (in basis points) with their `deposit()` and `withdraw()` calls. The vault uses this to calculate the minimum acceptable output for swaps:

```solidity
uint256 amountOutMinimum = expectedOutput * (10000 - maxSlippage) / 10000;
```

### Maximum Slippage Cap

The vault enforces a **maximum slippage of 1%** (100 basis points) regardless of what the user specifies. This prevents users from accidentally setting excessively high slippage that could be exploited by MEV bots:

```solidity
require(maxSlippage <= MAX_SLIPPAGE, "Slippage too high");
uint256 constant MAX_SLIPPAGE = 100; // 1% in basis points
```

### Deadline Checks

All Uniswap V3 interactions include a `deadline` parameter set to the current block timestamp. This ensures transactions that are stuck in the mempool for an extended period will fail rather than execute at a stale price:

```solidity
ISwapRouter.ExactInputSingleParams({
    // ...
    deadline: block.timestamp,
    amountOutMinimum: calculateMinOut(amountIn, maxSlippage),
    // ...
})
```

---

## Non-Transferable Shares

Vault share tokens are **non-transferable** between users. Only minting (deposit) and burning (withdrawal) are allowed.

### Implementation

```solidity
function _update(
    address from,
    address to,
    uint256 value
) internal override {
    // Minting: from == address(0) -- ALLOWED
    // Burning: to == address(0)   -- ALLOWED
    // Transfer: both non-zero     -- BLOCKED
    if (from != address(0) && to != address(0)) {
        revert("Transfers disabled");
    }
    super._update(from, to, value);
}
```

### Why Non-Transferable?

| Risk | Mitigation |
|---|---|
| **Lock period bypass** | Without transfer restrictions, a user could sell their locked shares on a secondary market, effectively exiting early. |
| **Fee rate arbitrage** | Shares tied to different fee rates (from referral codes or lock periods) could be mixed, breaking the per-depositor fee model. |
| **Accounting complexity** | Non-transferability ensures the depositor list remains accurate and fee calculations remain consistent. |

---

## Emergency Token Recovery

The vault includes an emergency recovery function that allows the owner to rescue tokens accidentally sent to the contract. This is critical because users sometimes mistakenly transfer tokens directly to a contract address.

### Implementation

```solidity
function recoverToken(
    address token,
    uint256 amount,
    address recipient
) external onlyOwner {
    require(token != address(weth), "Cannot recover WETH");
    require(token != address(usdc), "Cannot recover USDC");

    IERC20(token).safeTransfer(recipient, amount);
    emit TokenRecovered(token, amount, recipient);
}
```

### Safety Constraints

- **Cannot drain vault tokens:** The function explicitly blocks recovery of WETH and USDC, which are the vault's operational tokens. This prevents the owner from draining user funds.
- **Owner-only:** Only the contract owner can call this function.
- **Event emission:** All recovery operations emit an event for transparency and auditability.

---

## Access Control

The vault enforces strict role-based access control across all sensitive operations.

### Role Matrix

| Operation | Owner | Manager | User | Anyone |
|---|:---:|:---:|:---:|:---:|
| `deposit()` | | | Yes | |
| `withdraw()` | | | Yes | |
| `claimPendingRewards()` | | | Yes | |
| `harvest()` | | Yes | | |
| `rebalance()` | | Yes | | |
| `setManager()` | Yes | | | |
| `setFeeRecipient()` | Yes | | | |
| `setDefaultFeeRate()` | Yes | | | |
| `setReferralCode()` | Yes | | | |
| `recoverToken()` | Yes | | | |
| `upgradeTo()` | Yes | | | |
| `needsRebalance()` | | | | Yes (view) |
| `balanceOf()` | | | | Yes (view) |

### Modifier Implementations

```solidity
// Owner: inherited from OwnableUpgradeable
modifier onlyOwner() {
    require(owner() == msg.sender, "Ownable: caller is not the owner");
    _;
}

// Manager: custom modifier
modifier onlyManager() {
    require(msg.sender == manager, "Not manager");
    _;
}
```

### Separation of Concerns

The Owner and Manager roles are intentionally separate:

- **Owner** controls configuration and cannot trigger operational functions like rebalance (unless also set as manager). This is typically a multisig or governance contract.
- **Manager** can only trigger operational functions and cannot modify contract parameters. This is typically the keeper bot's hot wallet, which has a smaller attack surface if compromised.

---

## SafeERC20 Usage

All ERC20 token interactions use OpenZeppelin's `SafeERC20` library, which wraps `transfer`, `transferFrom`, and `approve` calls with return value checks:

```solidity
using SafeERC20 for IERC20;

// Instead of:
IERC20(usdc).transfer(recipient, amount);      // Unsafe: ignores return value

// The vault uses:
IERC20(usdc).safeTransfer(recipient, amount);   // Safe: reverts on failure
```

This protects against tokens that:
- Return `false` instead of reverting on failure
- Do not return a value at all (non-standard ERC20)
- Have non-standard `approve` behavior

---

## UUPS Upgrade Authorization

The vault uses the UUPS proxy pattern, which places the upgrade logic in the implementation contract rather than the proxy. The `_authorizeUpgrade` function controls who can trigger upgrades:

```solidity
function _authorizeUpgrade(
    address newImplementation
) internal override onlyOwner {
    // Only the owner can authorize upgrades
    // No additional validation -- the owner is fully trusted
}
```

### Upgrade Safety Measures

| Measure | Description |
|---|---|
| `onlyOwner` restriction | Only the contract owner can initiate an upgrade |
| `initializer` modifier | The `initialize()` function can only be called once, preventing re-initialization attacks on the implementation contract |
| Storage layout preservation | Upgradeable contracts use OpenZeppelin's storage gap pattern to prevent storage collisions between versions |
| Implementation validation | The UUPS pattern requires the new implementation to also be UUPS-compatible, preventing accidental bricking |

### Re-Initialization Protection

The `initialize` function uses the `initializer` modifier, ensuring it can only be called once:

```solidity
function initialize(
    address _usdc,
    address _weth,
    address _positionManager,
    address _swapRouter,
    address _pool,
    address _feeRecipient,
    string memory _name,
    string memory _symbol
) public initializer {
    __ERC20_init(_name, _symbol);
    __Ownable_init(msg.sender);
    __ReentrancyGuard_init();
    __UUPSUpgradeable_init();

    // Set immutable-like configuration
    usdc = IERC20(_usdc);
    weth = IERC20(_weth);
    // ...
}
```

If an attacker attempted to call `initialize()` again (e.g., on the implementation contract directly), the `initializer` modifier would revert the transaction.

---

## Rebalance Cooldown

To prevent excessive rebalancing (which incurs swap fees and potential slippage), the vault enforces a cooldown period between rebalances:

```solidity
uint256 public lastRebalanceTime;
uint256 public rebalanceCooldown;

function rebalance(...) external onlyManager nonReentrant {
    require(
        block.timestamp >= lastRebalanceTime + rebalanceCooldown,
        "Cooldown active"
    );
    // ...
    lastRebalanceTime = block.timestamp;
}
```

This prevents scenarios where a compromised manager key could rapidly rebalance the vault, burning value through repeated swap fees.

---

## Security Checklist Summary

| Category | Mechanism | Status |
|---|---|---|
| Reentrancy | `nonReentrant` on all state-changing functions | Implemented |
| Price manipulation | 30-min TWAP oracle, 2% max deviation | Implemented |
| Slippage | User-specified + 1% max cap | Implemented |
| Transaction expiry | `deadline: block.timestamp` on all Uniswap calls | Implemented |
| Lock bypass | Non-transferable ERC20 shares | Implemented |
| Fund drainage | `recoverToken` blocks WETH/USDC withdrawal | Implemented |
| Access control | Owner/Manager/User role separation | Implemented |
| Token safety | SafeERC20 for all ERC20 operations | Implemented |
| Upgrade safety | UUPS `onlyOwner` + `initializer` guard | Implemented |
| Rebalance spam | Cooldown period between rebalances | Implemented |
| Fee distribution failure | Pending rewards fallback mapping | Implemented |

---

## Related Pages

- [Smart Contracts Overview](overview.md) -- Contract versions and access control summary
- [Vault Mechanics](vault-mechanics.md) -- Deposit, withdraw, and rebalance flows
- [Fee Structure](fee-structure.md) -- Fee distribution and referral system
- [Architecture Overview](../architecture/overview.md) -- System-level architecture

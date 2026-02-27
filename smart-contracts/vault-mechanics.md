# Vault Mechanics

This page explains the core operational flows of the UnAI vault: how deposits, withdrawals, rebalances, and share tokens work at the smart contract level.

---

## Deposit Flow

Users deposit USDC into the vault, which is converted into a Uniswap V3 concentrated liquidity position. In return, the user receives non-transferable ERC20 share tokens.

### Step-by-Step

1. **User calls `deposit()`** with the USDC amount, maximum slippage tolerance, minimum output amounts, lock period selection, and a deadline.

2. **Lock period is recorded.** The vault stores the user's chosen lock period (3, 6, or 12 months) and calculates the unlock timestamp. The fee rate associated with the lock period is also recorded.

3. **Referral tracking** is handled off-chain by the bot's referral system, not by the smart contract. Users register with referral codes via the bot API before depositing.

4. **Optimal swap amount is calculated.** The vault determines how much of the deposited USDC should be swapped to WETH, based on the current tick range and price. This is the most mathematically complex step (see [Optimal Swap Calculation](#optimal-swap-calculation) below).

5. **USDC is swapped to WETH** via Uniswap V3 SwapRouter's `exactInputSingle`. Slippage protection and a deadline are enforced.

6. **Liquidity is added to Uniswap V3:**
   - If this is the vault's first position: `mint()` is called on the NonfungiblePositionManager to create a new LP position at the configured tick range.
   - If a position already exists: `increaseLiquidity()` is called to add to the existing position.

7. **Share tokens are minted** to the depositor, proportional to the value they contributed relative to the total vault value.

```solidity
// Simplified deposit logic
function deposit(
    uint256 usdcAmount,
    uint256 maxSlippage,
    uint256 minAmount0,
    uint256 minAmount1,
    uint8 lockPeriod,
    uint256 deadline
) external nonReentrant {
    // Transfer USDC from user
    IERC20(usdc).safeTransferFrom(msg.sender, address(this), usdcAmount);

    // Record lock period and fee rate
    depositors[msg.sender].lockExpiry = block.timestamp + lockDuration(lockPeriod);
    depositors[msg.sender].feeRate = getFeeRate(lockPeriod);

    // Calculate optimal swap
    uint256 swapAmount = calculateOptimalSwap(usdcAmount);

    // Swap USDC → WETH
    uint256 wethReceived = swapRouter.exactInputSingle(
        ISwapRouter.ExactInputSingleParams({
            tokenIn: usdc,
            tokenOut: weth,
            fee: poolFee,
            recipient: address(this),
            deadline: block.timestamp,
            amountIn: swapAmount,
            amountOutMinimum: calculateMinOut(swapAmount, maxSlippage),
            sqrtPriceLimitX96: 0
        })
    );

    // Add liquidity (mint or increase)
    if (tokenId == 0) {
        _mintPosition(usdcAmount - swapAmount, wethReceived);
    } else {
        _increaseLiquidity(usdcAmount - swapAmount, wethReceived);
    }

    // Mint shares to depositor
    uint256 shares = calculateShares(usdcAmount);
    _mint(msg.sender, shares);
}
```

---

## Optimal Swap Calculation

When a user deposits USDC, the vault needs to split it into two tokens (USDC and WETH) in the exact ratio required by the concentrated liquidity position's current tick range. This is significantly more complex than V2 liquidity math because the required ratio depends on the position's tick bounds relative to the current price.

### The Math

For a Uniswap V3 position with tick range `[tickLower, tickUpper]` at current tick `tickCurrent`:

- If `tickCurrent < tickLower`: the position is 100% token0 (WETH)
- If `tickCurrent > tickUpper`: the position is 100% token1 (USDC)
- If `tickLower <= tickCurrent <= tickUpper`: the position requires a specific ratio of both tokens, determined by:

```
sqrtPrice = sqrt(1.0001^tickCurrent)
sqrtPriceLower = sqrt(1.0001^tickLower)
sqrtPriceUpper = sqrt(1.0001^tickUpper)

amount0 = liquidity * (sqrtPriceUpper - sqrtPrice) / (sqrtPrice * sqrtPriceUpper)
amount1 = liquidity * (sqrtPrice - sqrtPriceLower)
```

The vault's `calculateOptimalSwap` function solves for the exact USDC amount to swap into WETH so that the remaining USDC and received WETH match this ratio, accounting for:

- The swap's price impact on the pool
- The pool's fee tier (0.3%)
- Slippage tolerance

---

## Withdrawal Flow

Users burn their share tokens to receive USDC back, proportional to their share of the vault's total value.

### Step-by-Step

1. **User calls `withdraw()`** with the number of shares to burn and a maximum slippage tolerance.

2. **Lock period is validated.** The vault checks that `block.timestamp >= depositors[msg.sender].lockExpiry`. If the lock has not expired, the transaction reverts.

3. **Proportional liquidity is calculated.** The vault determines how much liquidity to remove:
   ```
   liquidityToRemove = totalLiquidity * sharesToBurn / totalShares
   ```

4. **Liquidity is removed** by calling `decreaseLiquidity()` on the NonfungiblePositionManager, which converts the LP position back into WETH and USDC.

5. **Tokens are collected** by calling `collect()` on the NonfungiblePositionManager.

6. **WETH is swapped to USDC** via the SwapRouter so the user receives a single token.

7. **Share tokens are burned** from the withdrawer's balance.

8. **USDC is transferred** to the user.

```solidity
// Simplified withdrawal logic
function withdraw(
    uint256 shares,
    uint256 maxSlippage
) external nonReentrant {
    require(block.timestamp >= depositors[msg.sender].lockExpiry, "Lock active");
    require(shares <= balanceOf(msg.sender), "Insufficient shares");

    // Calculate proportional liquidity
    uint128 liquidityToRemove = uint128(
        uint256(totalLiquidity) * shares / totalSupply()
    );

    // Remove liquidity from Uniswap V3
    positionManager.decreaseLiquidity(
        INonfungiblePositionManager.DecreaseLiquidityParams({
            tokenId: tokenId,
            liquidity: liquidityToRemove,
            amount0Min: 0,
            amount1Min: 0,
            deadline: block.timestamp
        })
    );

    // Collect tokens
    (uint256 amount0, uint256 amount1) = positionManager.collect(...);

    // Swap WETH → USDC
    uint256 usdcFromWeth = swapRouter.exactInputSingle(...);

    // Burn shares and transfer USDC
    _burn(msg.sender, shares);
    IERC20(usdc).safeTransfer(msg.sender, amount1 + usdcFromWeth);
}
```

---

## Harvest Flow

Harvesting collects earned trading fees from the Uniswap V3 position and distributes them according to the fee structure.

### Step-by-Step

1. **Manager calls `harvest()`** (or it is triggered as part of a rebalance).

2. **Fees are collected** from the Uniswap V3 position using `collect()`. This returns the accumulated WETH and USDC trading fees.

3. **WETH fees are swapped to USDC** for uniform accounting.

4. **Protocol fee is calculated:**
   ```
   protocolFee = totalFees * weightedAverageFeeRate
   userFees = totalFees - protocolFee
   ```

5. **Protocol fee is sent** to the `feeRecipient` address.

6. **User fees are distributed** proportionally to all depositors based on their share balance:
   - **Push distribution:** The vault attempts to transfer USDC directly to each depositor.
   - **Fallback:** If a direct transfer fails (e.g., the recipient is a contract that reverts), the amount is added to `pendingRewards[depositor]` for later manual claim.

7. **Remaining funds are compounded** back into the Uniswap V3 position by calling `increaseLiquidity()`.

---

## Rebalance Flow

Rebalancing repositions the vault's entire liquidity into a new tick range. This is the most complex operation.

### Step-by-Step

1. **Manager or keeper bot calls `rebalance()`** with new tick bounds (`newTickLower`, `newTickUpper`) and minimum output amounts (`minAmount0`, `minAmount1`) for slippage protection.

2. **Breakout Confirmation** ensures the current pool price is not being manipulated. The vault reads the 30-minute time-weighted average from the pool's `observe()` function and compares it to the current spot price. If the deviation exceeds 2%, the rebalance reverts.

3. **Cooldown check** ensures a minimum time has passed since the last rebalance.

4. **Harvest is triggered** to collect and distribute any pending fees (steps from [Harvest Flow](#harvest-flow) above).

5. **All liquidity is removed** from the current position via `decreaseLiquidity(100%)` and `collect()`.

6. **Optimal swap is recalculated** for the new tick range. Since the ratio of WETH:USDC required depends on the new range's position relative to the current price, the vault recalculates and executes a swap to achieve the correct ratio.

7. **New position is minted** at the new tick range via `mint()`.

8. **Internal state is updated:** the old `tokenId` is replaced with the new one, and tick bounds are stored.

```solidity
// Simplified rebalance logic
function rebalance(
    int24 newTickLower,
    int24 newTickUpper,
    uint256 minAmount0,
    uint256 minAmount1
) external onlyManager nonReentrant {
    // 1. Validate via Breakout Confirmation
    _validateBreakoutConfirmation();

    // 2. Check cooldown
    require(block.timestamp >= lastRebalanceTime + rebalanceCooldown, "Cooldown");

    // 3. Harvest existing fees
    _harvest();

    // 4. Remove all liquidity
    _removeAllLiquidity();

    // 5. Recalculate and swap to optimal ratio for new range
    uint256 swapAmount = calculateOptimalSwap(newTickLower, newTickUpper);
    _executeSwap(swapAmount, minAmount0, minAmount1);

    // 6. Mint new position
    _mintPosition(newTickLower, newTickUpper);

    // 7. Update state
    lastRebalanceTime = block.timestamp;
    currentTickLower = newTickLower;
    currentTickUpper = newTickUpper;
}
```

---

## Auto-Rebalance

The vault includes an on-chain `needsRebalance()` view function that the keeper bot (or anyone) can call to check whether a rebalance is needed.

### `needsRebalance()` Logic

```solidity
function needsRebalance() public view returns (bool) {
    (, int24 currentTick, , , , , ) = pool.slot0();

    // Check if price has moved outside the current range
    if (currentTick < currentTickLower || currentTick > currentTickUpper) {
        // Check cooldown has elapsed
        if (block.timestamp >= lastRebalanceTime + rebalanceCooldown) {
            return true;
        }
    }
    return false;
}
```

The keeper bot polls this function periodically. When it returns `true`, the bot calculates new optimal tick bounds off-chain and submits a `rebalance()` transaction.

### Breakout Confirmation Safety

Before executing a rebalance, the vault validates the pool price against a time-weighted average oracle:

```solidity
function _validateTWAP() internal view {
    uint32[] memory secondsAgos = new uint32[](2);
    secondsAgos[0] = 1800; // 30 minutes ago
    secondsAgos[1] = 0;    // now

    (int56[] memory tickCumulatives, ) = pool.observe(secondsAgos);

    int24 twapTick = int24((tickCumulatives[1] - tickCumulatives[0]) / 1800);
    (, int24 currentTick, , , , , ) = pool.slot0();

    int24 deviation = currentTick > twapTick
        ? currentTick - twapTick
        : twapTick - currentTick;

    require(deviation <= maxTwapDeviation, "Price deviation too high");
}
```

This Breakout Confirmation prevents rebalances during flash loan attacks or price manipulation events. The 30-minute window and 2% maximum deviation provide a strong safety margin.

---

## Share Token Mechanics

The vault issues ERC20 share tokens to represent each depositor's proportional ownership of the vault's total liquidity.

### Minting

On deposit, shares are minted based on the deposit's value relative to the vault's total value:

```
If totalSupply == 0:
    shares = depositAmount  (1:1 initial minting)
Else:
    shares = depositAmount * totalSupply / totalVaultValue
```

`totalVaultValue` is calculated by querying the Uniswap V3 position's current value (both tokens converted to USDC terms) plus any undeployed USDC held by the vault.

### Burning

On withdrawal, shares are burned and the corresponding proportion of liquidity is returned:

```
usdcReturned = (shares / totalSupply) * totalVaultValueInUSDC
```

### Non-Transferable

Starting from V2, share tokens are **non-transferable**. This is enforced by overriding the ERC20 `_update` function:

```solidity
function _update(address from, address to, uint256 value) internal override {
    // Allow minting (from == address(0)) and burning (to == address(0))
    // Block all transfers between non-zero addresses
    if (from != address(0) && to != address(0)) {
        revert("Transfers disabled");
    }
    super._update(from, to, value);
}
```

**Why non-transferable?**

- Prevents lock period bypass via secondary market sales
- Ensures fee rates remain tied to the original depositor's lock period
- Simplifies fee accounting since shares cannot change hands

---

## Related Pages

- [Architecture Overview](../architecture/overview.md) -- System-level architecture and data flows
- [Smart Contracts Overview](overview.md) -- Contract versions and interfaces
- [Fee Structure](fee-structure.md) -- Lock periods, fee calculation, and distribution
- [Security](security.md) -- Breakout Confirmation validation, slippage protection, and reentrancy guards

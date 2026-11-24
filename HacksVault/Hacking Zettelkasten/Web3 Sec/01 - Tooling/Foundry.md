---
Created: 23-11-2026
Tags:
  - web3
  - foundry
  - tooling
  - solidity
  - poc
---
# Foundry Cheatsheet & PoC Writing

**Foundry** is the industry-standard Rust toolkit for compiling Solidity, running local EVM forks, and writing executable Proof-of-Concept (PoC) exploits.


```bash
# 1. Download & Install Foundry
curl -L https://foundry.paradigm.xyz | bash

# 2. 
source ~/.zshrc

# 3. Download & Install forge, cast, anvil e chisel
foundryup
```

Check:

```bash
forge --version
```

---

## 1. Core CLI Commands

| Command | Purpose |
| :--- | :--- |
| `forge build` | Compiles all smart contracts in `src/` and `test/`. |
| `forge test` | Runs all test files (`*.t.sol`) in `test/`. |
| `forge test --match-test test_unstoppable -vvvv` | Runs a single test with **level 4 verbosity** (shows full execution trace, internal calls, gas, and revert reasons). |
| `forge test --fork-url <RPC_URL>` | Runs tests against a live mainnet/testnet fork state. |
| `anvil` | Starts a local EVM node on `http://127.0.0.1:8545` with 10 funded test accounts. |
| `cast call <ADDR> "balanceOf(address)(uint256)" <USER>` | Queries a read-only view function from the terminal. |

---

## 2. Essential VM Cheatcodes for Exploit PoCs

Inside any Foundry test inheriting from `Test` (`forge-std/Test.sol`), the `vm` object lets you manipulate the EVM state:

### Impersonation (Acting as the Attacker or Victim)

```js
// Executes ONLY the very next call as `attacker`
vm.prank(attacker);
vault.withdraw();

// Executes ALL subsequent calls as `attacker` until `vm.stopPrank()`
vm.startPrank(attacker);
token.approve(address(vault), 100e18);
vault.deposit(100e18);
vm.stopPrank();
```

### Funding Accounts (ETH & ERC-20)

```js
// Give 10 ETH to `attacker`
vm.deal(attacker, 10 ether);

// Give 1,000 ERC-20 tokens to `attacker` (modifies storage directly)
deal(address(token), attacker, 1000e18);
```

### Time & Block Manipulation

```js
// Fast-forward block.timestamp by 7 days
vm.warp(block.timestamp + 7 days);

// Advance block.number by 100 blocks
vm.roll(block.number + 100);
```

### Snapshots & Assertions

```js
assertEq(token.balanceOf(attacker), 1000e18, "Attacker did not get tokens");
assertGt(attacker.balance, initialBalance, "No ETH profit");
vm.expectRevert(); // Next call MUST revert for the test to pass
```

---

## 3. Standard PoC Test Template (`Exploit.t.sol`)

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {Test, console} from "forge-std/Test.sol";
import {TargetContract} from "../src/TargetContract.sol";

contract ExploitTest is Test {
    TargetContract target;
    address attacker = makeAddr("attacker");

    function setUp() public {
        // Deploy contracts and set initial balances
        target = new TargetContract();
        vm.deal(attacker, 1 ether);
    }

    function test_exploit() public {
        vm.startPrank(attacker);

        // --- WRITE EXPLOIT STEPS HERE ---

        vm.stopPrank();

        // Verify exploit success
        assertEq(address(target).balance, 0);
    }
}
```

---

## Quick Debugging Tip

* Always use **`-vvvv`** (4 `v`s) when a `PoC` fails: Foundry prints the exact call stack in red showing which `require` or arithmetic error caused the `revert`.

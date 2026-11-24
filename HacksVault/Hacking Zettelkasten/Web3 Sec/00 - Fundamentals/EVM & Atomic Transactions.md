---
Created: 24-11-2026
Tags:
  - web3
  - solidity
  - fundamentals
---

Before reading Solidity code or hunting for vulnerabilities, you need a solid grasp of the physical rules governing how the **Ethereum Virtual Machine (EVM)** executes bytecode and manages state.

## 1. The Two Account Types on EVM

There are only two categories of 20-byte addresses (`address`) on Ethereum and EVM-compatible chains:

| Feature                        | EOA (Externally Owned Account)            | Contract Account (Smart Contract)                                   |
| :----------------------------- | :---------------------------------------- | :------------------------------------------------------------------ |
| **Controlled by**              | A user via a private key (e.g., MetaMask) | The immutable bytecode stored at its address                        |
| **Can initiate transactions?** | **Yes** (pays gas fees)                   | **No** (can only execute when called by an EOA or another contract) |
| **Has internal Storage?**      | No (holds native ETH balance only)        | **Yes** (persistent key-value database)                             |

> [!important] Golden Rule for Auditing
> Smart contracts are completely passive—they never run background jobs on their own. Every `public` or `external` function is an entry point that anyone can invoke with arbitrary inputs.

---

## 2. Transaction Atomicity (`revert`)

Every EVM transaction is strictly **atomic** (all-or-nothing):

1. A single transaction can chain dozens of external calls across multiple smart contracts.
2. If an error occurs at any point during execution (failed `require`, explicit `revert`, division by zero, out-of-gas), **the entire transaction is rolled back**.
3. Every `storage` modification made up to that millisecond is wiped clean as if the transaction never happened (the caller only loses the gas spent up to the failure point).

> [!tip] Security Implication (Flash Loans)
> Atomicity allows attackers to borrow millions of dollars with zero collateral (**Flash Loans**), execute a multi-step price manipulation, and repay the loan within the same transaction. If the exploit is unprofitable at the final step, they trigger a `revert` and lose zero principal.

---

## 3. Data Locations: Storage, Memory, and Calldata

In Solidity, where data lives directly impacts both gas costs and critical state-corruption bugs:

* **`storage`:** Permanent database persisted on the blockchain (expensive in gas, survives across transactions). Organized into **32-byte slots** (`slot 0`, `slot 1`, etc.).
* **`memory`:** Temporary RAM allocated during a function call; wiped clean once execution finishes.
* **`calldata`:** Read-only, non-modifiable space containing the raw input arguments passed to an `external` function.

---

## 4. Security-Critical Global Variables

Every execution context exposes global variables accessible inside Solidity:

* **`msg.sender`:** The address that made the **direct** call to the current contract (can be an EOA or an intermediary smart contract).
* **`tx.origin`:** The original EOA that signed the transaction.
  * *Security Note:* Using `require(tx.origin == owner)` for authorization is a **known vulnerability** (enables phishing attacks via malicious intermediary contracts). Always validate against `msg.sender`.
* **`msg.value`:** Amount of native ETH (denominated in **Wei**, where $1 \text{ ETH} = 10^{18} \text{ Wei}$) sent alongside the call.
* **`block.timestamp`:** Current block's Unix timestamp (can be slightly manipulated by validators within a few seconds; never use it as a source of randomness).

---

## Quick Takeaways

* Smart contracts are reactive state machines—they only run when poked from the outside.
* A failure on the last line reverts state changes made on the first line.
* Never rely on `tx.origin` for access control.

---
Created: 24-11-2026
Tags:
  - web3
  - solidity
  - erc20
  - erc4626
  - vaults
  - fundamentals
---


Over 70% of real-world DeFi protocols and Sherlock audit contests revolve around two standards: **ERC-20** (fungible tokens) and **ERC-4626** (tokenized yield-bearing vaults). Understanding their mechanics and edge cases is essential for finding high-severity accounting bugs.

---

## 1. The ERC-20 Token Standard

An ERC-20 token is simply a smart contract maintaining a `mapping(address => uint256)` of balances.

### Core Functions & Approval Flow

There are two distinct ways tokens move:

1. **Direct Push (`transfer`):**
   ```solidity
   token.transfer(recipient, amount);
   ```
   
2. Moves `amount` directly from `msg.sender` to `recipient`.

3. **Pull Pattern (`approve` + `transferFrom`):**
   Smart contracts cannot detect when someone sends them ERC-20 tokens via `transfer()`. Therefore, DeFi protocols use a two-step pull pattern:
   * **Step 1:** User calls `token.approve(protocolAddress, amount)` to grant spending permission.
   * **Step 2:** User calls `protocol.deposit(amount)`, and inside that function the protocol executes:
   * 
     ```solidity
     token.transferFrom(msg.sender, address(this), amount);
     ```

### Weird ERC-20 Edge Cases (Frequent Audit Bugs)
Never assume all ERC-20 tokens behave like standard OpenZeppelin tokens:

* **Fee-on-Transfer Tokens:** Some tokens take a tax (e.g., 2%) on every transfer. If a user calls `transferFrom(user, vault, 100)`, the vault might only receive `98` tokens. If the vault credits `100` to the user's balance, the vault becomes insolvent.
  * *Fix:* Always measure balance before and after: `received = balanceAfter - balanceBefore`.
* **Missing Return Value (e.g., USDT):** Standard ERC-20 returns `bool`, but Tether (USDT) returns nothing (`void`), causing standard `IERC20(usdt).transfer(...)` calls to revert.
  * *Fix:* Protocols must use OpenZeppelin's `SafeERC20` (`token.safeTransfer(...)`).
* **Decimals Mismatch:** USDC and USDT use **6 decimals** ($1 \text{ USDC} = 10^6$), while ETH, DAI, and most tokens use **18 decimals** ($1 \text{ DAI} = 10^{18}$). Failing to normalize decimals causes massive math/oracle errors.

---

## ERC-4626 Tokenized Vaults

An **ERC-4626 Vault** is a standard smart contract where users deposit an underlying ERC-20 **asset** (e.g., USDC) and receive **shares** (an ERC-20 receipt token representing their proportional ownership of the pool).

### The Core Accounting Formula
As the vault earns yield (more `totalAssets` enter the vault while `totalSupply` of shares stays constant), each share becomes worth more underlying assets:

$$\text{Shares Minted} = \text{Assets Deposited} \times \frac{\text{totalSupply}}{\text{totalAssets}}$$

$$\text{Assets Redeemed} = \text{Shares Burned} \times \frac{\text{totalAssets}}{\text{totalSupply}}$$

### Vulnerable Vault Implementation Example

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract SimpleVault {
    IERC20 public immutable asset;
    uint256 public totalSupply;
    mapping(address => uint256) public balanceOf;

    constructor(address _asset) {
        asset = IERC20(_asset);
    }

    // 1. Deposit underlying assets and mint proportional shares
    function deposit(uint256 assets) external returns (uint256 shares) {
        uint256 totalAssets = asset.balanceOf(address(this));

        if (totalSupply == 0) {
            shares = assets;
        } else {
            // Precision loss / rounding down happens here!
            shares = (assets * totalSupply) / totalAssets;
        }

        require(shares > 0, "Zero shares minted");
        asset.transferFrom(msg.sender, address(this), assets);

        totalSupply += shares;
        balanceOf[msg.sender] += shares;
    }

    // 2. Burn shares and withdraw proportional underlying assets
    function withdraw(uint256 shares) external returns (uint256 assets) {
        uint256 totalAssets = asset.balanceOf(address(this));
        assets = (shares * totalAssets) / totalSupply;

        balanceOf[msg.sender] -= shares;
        totalSupply -= shares;

        asset.transfer(msg.sender, assets);
    }
}
```

---

## 3. Classic Vault Vulnerability: First Depositor / Inflation Attack

Look closely at `SimpleVault` above when `totalAssets` is read directly via `asset.balanceOf(address(this))`:

1. **Attacker deposits `1 wei`** when the vault is empty (`totalSupply == 0`) and receives `1 share`.
2. **Attacker directly transfers (`donate`) `10,000 USDC`** to the vault address (bypassing `deposit()`). Now `totalSupply = 1` and `totalAssets = 10,000e6 + 1`.
3. **Victim deposits `19,999 USDC`:**
   $$\text{shares} = \frac{19,999 \times 10^6 \times 1}{10,000 \times 10^6 + 1} = 1 \text{ share (truncated due to integer division!)}$$
4. **Attacker withdraws their `1 share`** and gets half of the entire pool (`~15,000 USDC`), stealing `~5,000 USDC` from the victim!

> [!warning] Audit Red Flag: Spot Balance Accounting
> Whenever a contract calculates share prices or internal accounting using `token.balanceOf(address(this))` instead of tracking deposits with an internal storage variable (`internalTotalAssets`), anyone can manipulate the denominator via a direct token transfer (or Flash Loan).
> *(Note: This exact concept is the core of Challenge #1 in Damn Vulnerable DeFi: [Unstoppable](../03%20-%20Damn%20Vulnerable%20DeFi/Unstoppable.md)).*

---

## Quick Takeaways

* Solidity has **no floating-point numbers**: integer division always rounds **down** (`5 / 2 == 2`). Always multiply before dividing.
* Always check if a protocol tracks balances via internal accounting vs `address(this).balance` / `token.balanceOf(address(this))`.
* Check whether `ERC-20` transfers handle fee-on-transfer tokens and non-standard return values (`SafeERC20`).

---
Created: 24-11-2026
tags:
  - web3
---
First Install [Foundry](../01%20-%20Tooling/Foundry.md) then clone and build the "gym" from `damn-vulnerable-defi`:

```bash
git clone https://github.com/theredguild/damn-vulnerable-defi.git
cd damn-vulnerable-defi
forge build
```

Test:

```bash
forge test --match-path test/unstoppable/Unstoppable.t.sol
```
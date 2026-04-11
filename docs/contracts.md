# Smart Contract Development

← [Back to README](../README.md)

---

## Overview

OPNET smart contracts are written in **AssemblyScript** (a TypeScript-like language), compiled to **WebAssembly (WASM)**, and deployed on Bitcoin L1 via Tapscript witness data.

The contract runtime is provided by [@btc-vision/btc-runtime](https://github.com/btc-vision/btc-runtime).

---

## Setup

```bash
# Install the runtime and compiler
npm install @btc-vision/btc-runtime
npm install --save-dev assemblyscript @btc-vision/unit-test-framework
```

The AssemblyScript compiler transform `@btc-vision/opnet-transform` is bundled with `btc-runtime` and handles decorator injection automatically.

---

## Key Differences from Solidity / EVM

| Concept | Ethereum / Solidity | OPNET / AssemblyScript |
|---------|--------------------|-----------------------|
| Language | Solidity | AssemblyScript |
| Compile target | EVM bytecode | WebAssembly (WASM) |
| Hash function | keccak-256 | SHA-256 |
| Number type | uint256 | `u256` — needs `SafeMath` |
| Signatures | ECDSA | ECDSA + ML-DSA (quantum-resistant) |
| Deployment | Single transaction | Two-phase Taproot (funding + reveal) |
| Storage | Slot-based (32-byte slots) | Pointer-based (`Blockchain.nextPointer`) |
| Token approval | `approve()` | `increaseAllowance()` |
| Constructor | Runs once at deployment | Runs on **every** interaction |
| `msg.sender` | ECDSA public key | Identity key (≠ visible address) |
| Block time | ~12 seconds | ~10 minutes |
| Event history | `eth_getLogs` | No native equivalent — must use indexer |
| Loops | Unbounded while loops OK | No unbounded loops — use bounded `for` |

---

## Project Structure

```
my-contract/
├── package.json
├── asconfig.json          # AssemblyScript compiler config
├── tsconfig.json
├── src/my-contract/
│   ├── index.ts           # Factory function + selector routing
│   └── MyContract.ts      # @final contract class
├── __test__/unit/
│   ├── contracts/
│   │   └── MyContract.ts  # Test wrapper (ContractRuntime / OP20 subclass)
│   ├── tests/
│   │   └── my-contract.ts # Test suite
│   └── helpers/
│       └── cheatcodes.ts  # Foundry-style test helpers
└── build/                 # WASM output
```

---

## Base Classes

| Class | Use when |
|-------|---------|
| `OP20` | Building a fungible token |
| `OP_NET` | Building any other contract (NFT, AMM, custom) |
| `ReentrancyGuard` | Contract handles token transfers or calls other contracts |

Import from `@btc-vision/btc-runtime/runtime`.

---

## Contract Lifecycle

```typescript
constructor() {
    super();
    // Runs on EVERY interaction — only wire up storage field references here.
    // Do NOT put initialization logic here.
}

public override onDeployment(_calldata: Calldata): void {
    // Runs ONCE at deployment time.
    // Set initial values, mint tokens, configure ownership.
    this.owner.value = Blockchain.tx.sender;
}

public override onUpdate(_calldata: Calldata): void {
    // Called on contract upgrade.
    // Required — implement even if empty.
}
```

---

## Storage Model

Every persistent value occupies a **storage pointer** allocated sequentially via `Blockchain.nextPointer`.

```typescript
import { StoredU256, StoredAddress, EMPTY_POINTER, Blockchain } from '@btc-vision/btc-runtime/runtime';

// Each Blockchain.nextPointer call returns a unique sequential pointer
private readonly owner:  StoredAddress = new StoredAddress(Blockchain.nextPointer);
private readonly supply: StoredU256    = new StoredU256(Blockchain.nextPointer, EMPTY_POINTER);
// NOTE: StoredU256 second arg must be EMPTY_POINTER — not u256.Zero
```

**Stored types:**

| Type | Constructor | Notes |
|------|------------|-------|
| `StoredU256` | `(pointer, EMPTY_POINTER)` | Second arg MUST be `EMPTY_POINTER` |
| `StoredAddress` | `(pointer)` | One arg only |
| `StoredString` | `(pointer)` | One arg only |
| `StoredBoolean` | `(pointer)` | One arg only |

**Maps:**

```typescript
import { Map, AddressMap } from '@btc-vision/btc-runtime/runtime';

// Simple map — field-level init is fine
private readonly balances: Map<Address, u256> = new Map(Blockchain.nextPointer);

// Nested map — MUST be initialized in constructor body (not at field level)
private allowances!: AddressMap;
constructor() {
    super();
    this.allowances = new AddressMap(Blockchain.nextPointer);
}
```

---

## Method Decorators

```typescript
// Read-only view call — no state changes, no gas cost for caller
@method({ readable: true })
public getBalance(addr: Address): BytesWriter {
    const writer = new BytesWriter(32);
    writer.writeU256(this.balances.get(addr));
    return writer;
}

// Write — modifies state
@method()
@emit('Transfer')    // emits event named 'Transfer' with the return bytes
public transfer(to: Address, amount: u256): BytesWriter {
    // ... state changes ...
    const writer = new BytesWriter(64);
    writer.writeAddress(to);
    writer.writeU256(amount);
    return writer;
}
```

---

## OP-20 Token Example

```typescript
import {
    Address, Blockchain, SafeMath, u256,
    OP20, OP20InitParameters, Calldata, BytesWriter,
    StoredAddress, EMPTY_POINTER,
} from '@btc-vision/btc-runtime/runtime';

@final
export class MyToken extends OP20 {
    private readonly owner: StoredAddress = new StoredAddress(Blockchain.nextPointer);

    constructor() { super(); }

    public override onDeployment(_calldata: Calldata): void {
        const params = new OP20InitParameters(
            u256.fromString('1000000000000000000000000'), // maxSupply (1M with 18 decimals)
            18,           // decimals
            'My Token',   // name
            'MTK',        // symbol
        );
        this.instantiate(params);
        this.owner.value = Blockchain.tx.sender;
        this._mint(Blockchain.tx.sender, u256.fromString('1000000000000000000000000'));
    }

    public override onUpdate(_calldata: Calldata): void {}
}
```

---

## Common Pitfalls

### 1. `SafeMath` is mandatory — raw operators silently overflow

```typescript
// WRONG
const result = a + b;         // silently wraps on overflow

// CORRECT
const result = SafeMath.add(a, b);    // reverts on overflow
const result = SafeMath.sub(a, b);    // reverts on underflow
const result = SafeMath.mul(a, b);    // reverts on overflow
const result = SafeMath.div(a, b);    // reverts on divide-by-zero
```

### 2. `EMPTY_POINTER`, not `u256.Zero`

```typescript
// WRONG
private readonly v: StoredU256 = new StoredU256(Blockchain.nextPointer, u256.Zero);

// CORRECT
private readonly v: StoredU256 = new StoredU256(Blockchain.nextPointer, EMPTY_POINTER);
```

### 3. Constructor runs on every call — use `onDeployment` for init

```typescript
// WRONG — owner changes on every tx
constructor() { super(); this.owner.value = Blockchain.tx.sender; }

// CORRECT
public override onDeployment(_calldata: Calldata): void {
    this.owner.value = Blockchain.tx.sender;
}
```

### 4. Pointer collision causes silent data corruption

```typescript
// WRONG — two fields share the same pointer
const ptr = Blockchain.nextPointer;
private readonly a: StoredU256 = new StoredU256(ptr, EMPTY_POINTER);
private readonly b: StoredU256 = new StoredU256(ptr, EMPTY_POINTER); // overwrites a!

// CORRECT — each field independently calls nextPointer
private readonly a: StoredU256 = new StoredU256(Blockchain.nextPointer, EMPTY_POINTER);
private readonly b: StoredU256 = new StoredU256(Blockchain.nextPointer, EMPTY_POINTER);
```

### 5. `u256.fromString()` for large numbers

```typescript
// WRONG — fromU64 silently overflows for values > 2^64
const supply = u256.fromU64(1_000_000_000_000_000_000);

// CORRECT
const supply = u256.fromString('1000000000000000000');
```

### 6. No unbounded loops

```typescript
// WRONG — while loops not supported
while (condition) { ... }

// CORRECT — bounded for loop
for (let i = 0; i < MAX_ITERATIONS; i++) {
    if (!condition) break;
}
```

### 7. Never import `ABIDataTypes` — it is globally injected

```typescript
// WRONG — causes compile error
import { ABIDataTypes } from '@btc-vision/btc-runtime/runtime';

// CORRECT — just use it directly (compiler injects it)
```

### 8. SHA-256 selectors — not keccak-256

Function selectors use the first 4 bytes of `SHA-256(signature)`, not keccak-256. The same method signature produces a different selector than on Ethereum.

### 9. `increaseAllowance`, not `approve`

OP-20 does not have an `approve()` function. Always use `increaseAllowance()` to grant spending permissions.

### 10. Always simulate before sending

```typescript
const sim = await contract.myMethod(args);
if (sim.revert) {
    console.error('Would revert:', sim.revert);
    return;
}
await sim.sendTransaction({ ... });
```

### 11. `MapOfMap` must be initialized in constructor body

```typescript
// WRONG
private allowances: AddressMap = new AddressMap(Blockchain.nextPointer);

// CORRECT
private allowances!: AddressMap;
constructor() {
    super();
    this.allowances = new AddressMap(Blockchain.nextPointer);
}
```

### 12. `maximumAllowedSatToSpend` must not exceed wallet balance

The SDK throws "Insufficient UTXOs" before signing if this value exceeds the actual UTXO balance. Always derive it from the actual available balance.

---

## Testing

Tests use `@btc-vision/unit-test-framework` with the `opnet()` runner. Test wrappers extend `ContractRuntime` (or `OP20` for token tests) and expose helper methods that call into the contract.

See [btc-vision/unit-test-framework](https://github.com/btc-vision/unit-test-framework) for the full API reference.

---

## Deployment Checklist

1. Compile with release optimization
2. Run all tests (including fuzz if available)
3. Dry-run to estimate fees
4. Deploy to testnet first, verify bytecode
5. Deploy to mainnet with `--verify` flag to wait for confirmation

---

## Community Tips (sourced from OPNET Telegram)

The following gotchas come from Bob (OPNET core developer) and community members sharing solutions in the official Telegram.

### T1. Use `networks.opnetTestnet`, not `networks.testnet`

```typescript
// WRONG — wrong chain ID, transactions will fail silently
import { networks } from '@btc-vision/bitcoin';
const provider = new JSONRpcProvider('https://testnet.opnet.org', networks.testnet);

// CORRECT
const provider = new JSONRpcProvider('https://testnet.opnet.org', networks.opnetTestnet);
```

`networks.testnet` is the standard Bitcoin testnet chain ID. `networks.opnetTestnet` is the OPNET-specific constant with the correct chain ID.

### T2. Always install with `@latest` — avoid beta/rc/alpha tags

```bash
# CORRECT — install stable versions
npm install opnet@latest @btc-vision/transaction@latest @btc-vision/bitcoin@latest @btc-vision/btc-runtime@latest

# WRONG — beta/rc packages may be broken or incompatible
npm install opnet@beta
```

Current stable as of April 2026: `opnet@1.8.4`. Check npm for the latest.

### T3. ML-DSA LEVEL2 is the default — do not override unless intentional

The SDK defaults to **MLDSA-44** (1312-byte public key). This is the correct level for mainnet. Do not set a custom level unless you have a specific reason.

### T4. Deployment transaction options

When deploying a contract, always include these flags:

```typescript
const deployTx = await deployer.deploy(bytecode, {
    linkMLDSAPublicKeyToAddress: true,   // required for ML-DSA key linking
    revealMLDSAPublicKey: true,          // required for key visibility
});
```

Omitting these will result in keys not being linked on-chain, causing subsequent calls to fail authentication.

### T5. `setTransactionDetails()` before simulate for extra outputs

When your transaction needs to send BTC to additional addresses (beyond the contract call itself), call `setTransactionDetails()` before `simulate()`:

```typescript
contract.setTransactionDetails({
    extraOutputs: [
        { address: recipientAddress, value: BigInt(satoshiAmount) }
    ]
});

const sim = await contract.myMethod(args);
if (sim.revert) throw new Error(sim.revert);
await sim.sendTransaction({ ... });
```

**Index 0 is reserved by the protocol.** Extra outputs start at index 1.

### T6. "Legacy public key not set" error

This error means you passed `wallet.p2tr` (a string) to `getContract()` instead of `wallet.address` (an `Address` object).

```typescript
// WRONG — p2tr is a plain string
const contract = getContract(contractAddress, ABI, provider, wallet.p2tr);

// CORRECT — pass the Address object
const contract = getContract(contractAddress, ABI, provider, wallet.address);
```

### T7. Clean install when hitting dependency issues

```bash
rm -rf node_modules package-lock.json && npm install
```

Run this whenever you see version mismatch errors or unexpected import failures after updating packages.

---

## Further Reading

- [btc-vision/btc-runtime](https://github.com/btc-vision/btc-runtime) — runtime library
- [btc-vision/unit-test-framework](https://github.com/btc-vision/unit-test-framework) — test runner
- [btc-vision/example-tokens](https://github.com/btc-vision/example-tokens) — reference implementations
- [docs.opnet.org](https://docs.opnet.org) — official documentation

# OPNET Protocol

> How OPNET works — architecture, transaction structure, epochs, and key properties.

← [Back to README](../README.md)

---

## Overview

OPNET is a **metaprotocol** layered on top of Bitcoin's base layer. It does not require a sidechain, rollup, or bridge. Bitcoin L1 is both the settlement layer and the execution environment's anchor.

Smart contracts are written in **AssemblyScript**, compiled to **WebAssembly (WASM)**, and deployed as Bitcoin transactions using Tapscript witness data. OPNET nodes run the WASM in the **op-vm** (Rust-based) and maintain consensus over the resulting state.

---

## Key Properties

| Property | Value |
|----------|-------|
| Language | AssemblyScript (TypeScript-like), compiled to WASM |
| VM | op-vm — Rust WebAssembly runtime |
| Hash function | SHA-256 (not keccak-256) |
| Signature scheme | ECDSA + ML-DSA (CRYSTALS-Dilithium, post-quantum) |
| Block time | ~10 minutes (Bitcoin L1) |
| Deployment | Two-phase Taproot transaction (funding + reveal) |
| Storage | Pointer-based (sequential `Blockchain.nextPointer`) |
| Token approvals | `increaseAllowance()` — not `approve()` |
| Protocol token | None — all fees paid in native BTC |
| Mempool | Fully public — no private mempool |

---

## How a Contract is Deployed

Deployment uses a **two-phase Taproot transaction**:

1. **Funding transaction (commit):** Sends BTC to a Taproot address that commits to the WASM bytecode hash.
2. **Reveal transaction:** Spends the funding output and includes the full WASM bytecode in the Tapscript witness.

This is the standard Taproot inscription pattern. The WASM is permanently stored in the Bitcoin transaction witness data.

---

## How Contract Calls Work

To call a deployed contract, a user broadcasts a Bitcoin transaction that embeds OPNET calldata in the witness field. OPNET nodes:

1. Detect the transaction as an OPNET interaction
2. Load the target contract's WASM
3. Execute the WASM in op-vm with the provided calldata
4. Reach consensus on the resulting state changes
5. Record the state update and emitted events

---

## Transaction Types

| Type | Description |
|------|-------------|
| `deployment` | Reveals WASM bytecode — creates a new contract |
| `interaction` | Calls a method on an existing contract |
| `generic` | Regular Bitcoin transaction — not an OPNET interaction |

---

## Address Formats

OPNET addresses appear in two forms that refer to the same identity:

| Format | Example | Used for |
|--------|---------|---------|
| **Bech32m** | `op1sq...` or `bc1p...` | Human-readable, explorers, config files |
| **32-byte hex** | `0x000...` | Contract calldata, event payloads, on-chain references |

Convert between them with the `btc_getPublicKeyInfo` RPC method or any OPNET SDK.

### The Two-Key System

Every OPNET wallet has two distinct public keys:

| Key | Derivation | Role |
|-----|------------|------|
| **Tweaked pubkey** | Bitcoin Taproot tweak applied | Derives the `bc1p...` / `op1sq...` address |
| **Identity key** | Original internal key | `Blockchain.tx.sender` — the `msg.sender` equivalent |

**Important:** The address you see in an explorer is derived from the tweaked key. The `msg.sender` inside a contract is the identity key. These are different values. OPNET SDKs handle the conversion, but you need to be aware of this when building scripts or custom tooling.

---

## Arithmetic Safety

OPNET's `u256` type does NOT throw on overflow — raw operators (`+`, `-`, `*`) silently wrap. Always use `SafeMath` for arithmetic in contracts:

```typescript
// WRONG — silently overflows
const result = a + b;

// CORRECT
const result = SafeMath.add(a, b);  // reverts on overflow
const result = SafeMath.sub(a, b);  // reverts on underflow
```

---

## Epochs & Mining

OPNET uses an epoch system on top of Bitcoin's block production. Validators submit epoch submissions that advance the OPNET state. Each epoch has a mining template that determines which state updates are finalized.

See [docs.opnet.org/epochs-and-mining](https://docs.opnet.org/epochs-and-mining) for the full specification.

---

## No Native `eth_getLogs`

Unlike Ethereum, OPNET has no equivalent of `eth_getLogs`. There is no API to query historical contract events by filter. To access event history you must either:

1. Scan every block from genesis individually (slow — one RPC call per block)
2. Run or connect to a dedicated indexer that has pre-scanned and stored the data

The genesis block for indexing is **941,400** (mainnet launch, March 19, 2026).

---

## Public Mempool

OPNET has no private mempool. All pending Bitcoin transactions are visible to everyone simultaneously as soon as they are broadcast. There is no front-running protection built into the protocol — all mempool activity is transparent.

---

## SlowFi

OPNET's native DeFi philosophy. Bitcoin's ~10-minute block time fundamentally changes DeFi dynamics compared to fast EVM chains:

- Price discrepancies between pools can persist for hours rather than milliseconds
- Liquidity stays in protocols longer, enabling yield strategies not feasible on fast chains
- The competitive edge in the ecosystem comes from data quality and model sophistication, not raw execution speed

---

## Further Reading

- [docs.opnet.org/learn/what-is-opnet](https://docs.opnet.org/learn/what-is-opnet)
- [docs.opnet.org/learn/native-bitcoin](https://docs.opnet.org/learn/native-bitcoin)
- [docs.opnet.org/epochs-and-mining](https://docs.opnet.org/epochs-and-mining)
- [docs.opnet.org/bitcoin-address-system](https://docs.opnet.org/bitcoin-address-system)

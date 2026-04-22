# SDKs & Packages

← [Back to README](../README.md)

---

## TypeScript / JavaScript SDK

### opnet — Primary Provider SDK

The main TypeScript SDK for interacting with OPNET from dApps, backends, and scripts.

- **npm:** [npmjs.com/package/opnet](https://www.npmjs.com/package/opnet)
- **GitHub:** [github.com/btc-vision/opnet](https://github.com/btc-vision/opnet)
- **Audited by:** Verichains

```bash
npm install opnet@latest
```

> **Version guidance:** Always use `@latest`. Do not install `@beta`, `@rc`, or `@alpha` tagged packages — they may be unstable or incompatible with other `@btc-vision/*` packages. Current stable: `opnet@1.8.4` (April 2026).

**Key exports:**

| Export | Description |
|--------|-------------|
| `JSONRpcProvider` | HTTP JSON-RPC client |
| `WebSocketProvider` | WebSocket provider with push subscriptions |
| Contract classes | Type-safe contract interaction handles |
| ABI decoders | Decode OP-20, NativeSwap, Motoswap events |
| `getContract()` | Instantiate a typed contract handle from an ABI |

**Quick start:**

```typescript
import { JSONRpcProvider } from 'opnet';

const provider = new JSONRpcProvider('https://mainnet.opnet.org');
const block = await provider.getBlockNumber();
console.log('Block:', block);
```

**Full documentation:** See `node_modules/opnet/docs/` after installing, or [docs.opnet.org](https://docs.opnet.org).

---

### @btc-vision/transaction — Transaction Builder

Low-level utilities for transaction signing and broadcasting.

- **npm:** [npmjs.com/package/@btc-vision/transaction](https://www.npmjs.com/package/@btc-vision/transaction)

```bash
npm install @btc-vision/transaction
```

Key utilities: `Wallet` (keypair + ML-DSA key container), `ABICoder` (encode/decode ABI data), `TransactionFactory` (deployments and plain BTC transfers).

**Suggested use cases:** deployments (`TransactionFactory.signDeployment`), plain BTC transfers (`TransactionFactory.createBTCTransfer`), and wallet construction (`new Wallet(WIF, MLDSA_KEY, network)`).

For contract calls (simulate + sendTransaction), use the [`opnet`](https://www.npmjs.com/package/opnet) package and `getContract()` instead.

---

### @btc-vision/bitcoin — Bitcoin Primitives

Bitcoin-layer utilities: key derivation, address formats, UTXO management.

- **npm:** [npmjs.com/package/@btc-vision/bitcoin](https://www.npmjs.com/package/@btc-vision/bitcoin)

```bash
npm install @btc-vision/bitcoin
```

---

### @btc-vision/bsi-bitcoin-rpc — Bitcoin Core RPC Client

Low-level Bitcoin Core JSON-RPC client. Used when you need direct Bitcoin node access (e.g. for fee estimation via `estimatesmartfee`).

- **npm:** [npmjs.com/package/@btc-vision/bsi-bitcoin-rpc](https://www.npmjs.com/package/@btc-vision/bsi-bitcoin-rpc)

---

### @btc-vision/walletconnect — Browser Wallet Integration

WalletConnect integration for the OP_WALLET browser extension. Used in dApp frontends to connect user wallets.

- **GitHub:** [github.com/btc-vision/walletconnect](https://github.com/btc-vision/walletconnect)

```bash
npm install @btc-vision/walletconnect
```

---

### @btc-vision/btc-runtime — Contract Runtime

AssemblyScript runtime library for writing OPNET smart contracts.

- **GitHub:** [github.com/btc-vision/btc-runtime](https://github.com/btc-vision/btc-runtime)

```bash
npm install @btc-vision/btc-runtime
```

Key exports: `Blockchain`, `Address`, `StoredU256`, `StoredAddress`, `Map`, `SafeMath`, `OP20`, `OP_NET`, `ReentrancyGuard`, `BytesWriter`, `Calldata`, `TransferHelper`.

---

### @btc-vision/unit-test-framework — Contract Test Runner

Official test framework for OPNET smart contracts.

- **GitHub:** [github.com/btc-vision/unit-test-framework](https://github.com/btc-vision/unit-test-framework)

```bash
npm install --save-dev @btc-vision/unit-test-framework
```

Key exports: `opnet()` test runner, `ContractRuntime`, `OP20` (test base class), `Assert`, `OPNetUnit`.

---

## Typical Install Patterns

**For a dApp frontend:**
```bash
npm install opnet @btc-vision/transaction @btc-vision/bitcoin @btc-vision/walletconnect
```

**For a backend / indexer:**
```bash
npm install opnet @btc-vision/transaction @btc-vision/bitcoin
```

**For smart contract development:**
```bash
npm install @btc-vision/btc-runtime
npm install --save-dev assemblyscript @btc-vision/unit-test-framework
```

---

## Further Reading

- [docs.opnet.org/developers/getting-started/setting-up-environment/for-nodejs](https://docs.opnet.org/developers/getting-started/setting-up-environment/for-nodejs)
- [docs.opnet.org/developers/walletconnect/setup](https://docs.opnet.org/developers/walletconnect/setup)
- [github.com/btc-vision](https://github.com/btc-vision) — full GitHub organization

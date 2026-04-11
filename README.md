# Awesome OPNET

> A curated encyclopedia of OPNET — the Bitcoin-native smart contract metaprotocol.
> Resources, tools, dApps, SDKs, and developer references, organized for both humans and LLMs.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

---

## Contents

- [What is OPNET?](#what-is-opnet)
- [Official Links](#official-links)
- [Networks & RPC](#networks--rpc)
- [Block Explorers](#block-explorers)
- [Wallets & Faucet](#wallets--faucet)
- [Token Standards](#token-standards)
- [DEXes & DeFi](#dexes--defi)
- [Mainnet Contract Addresses](#mainnet-contract-addresses)
- [Official SDKs & Packages](#official-sdks--packages)
- [Smart Contract Development](#smart-contract-development)
- [Infrastructure](#infrastructure)
- [Ecosystem & dApps](#ecosystem--dapps)
- [Community & Social](#community--social)
- [Media & Coverage](#media--coverage)
- [Deep Dives](#deep-dives)

---

## What is OPNET?

OPNET is a **consensus layer on Bitcoin L1** that enables fully expressive smart contracts, DeFi, and post-quantum security — directly on Bitcoin, with no sidechains, bridges, or rollups.

It is a **metaprotocol** built on Bitcoin's Taproot/SegWit stack. Smart contracts are written in **AssemblyScript**, compiled to **WebAssembly (WASM)**, and deployed via Tapscript-encoded Bitcoin transactions. All fees are paid in native BTC. There is no separate protocol token.

**Mainnet launched: March 19, 2026 — Block 941,400**

Key properties at a glance:

| Property | Value |
|----------|-------|
| Language | AssemblyScript → WASM |
| VM | WebAssembly (Rust-based op-vm) |
| Hash function | SHA-256 (not keccak-256) |
| Signatures | ECDSA + ML-DSA (post-quantum) |
| Block time | ~10 minutes (Bitcoin L1) |
| Protocol token | None — all fees in native BTC |
| Deployment | Two-phase Taproot tx (funding + reveal) |

→ Full protocol overview: [docs/protocol.md](docs/protocol.md)

---

## Official Links

| Resource | URL |
|----------|-----|
| Website | https://opnet.org |
| Documentation | https://docs.opnet.org |
| GitHub Organization | https://github.com/btc-vision |
| Block Explorer | https://opscan.org |
| MotoSwap DEX | https://motoswap.org |
| OPTools | https://optools.org |
| AI Dev Assistant (Bob) | https://ai.opnet.org |

---

## Networks & RPC

| Network | RPC Endpoint |
|---------|-------------|
| Mainnet | `https://mainnet.opnet.org` |
| Testnet | `https://testnet.opnet.org` |

The environment variable **`OPNET_RPC_URL`** is the standard override across all tooling.

→ Full networks reference: [docs/networks.md](docs/networks.md)

---

## Block Explorers

| Name | URL | Network |
|------|-----|---------|
| OpScan | https://opscan.org | Mainnet |
| OpScan | https://testnet.opscan.org | Testnet |
| Transaction Explorer | https://transaction.opnet.org | Mainnet |
| Mempool Explorer | https://github.com/btc-vision/mempool-opnet | Self-hosted |

---

## Wallets & Faucet

| Resource | URL |
|----------|-----|
| OP_WALLET (browser extension) | Install via [docs.opnet.org](https://docs.opnet.org) |
| Testnet Faucet | https://faucet.opnet.org/ |

→ Wallet & key concepts: [docs/wallets.md](docs/wallets.md)

---

## Token Standards

| Standard | Description | Status |
|----------|-------------|--------|
| **OP-20** | Fungible token standard (ERC-20 equivalent). Uses `increaseAllowance()` instead of `approve()`. | Live |
| **OP-721** | Non-fungible token standard (ERC-721 equivalent). | Live |
| **OP-20S** | Stablecoin extension of OP-20. | Expected Q2 2026 |

→ Full token standards reference: [docs/token-standards.md](docs/token-standards.md)

---

## DEXes & DeFi

| Protocol | Type | URL |
|----------|------|-----|
| **NativeSwap** | BTC ↔ OP-20 AMM (2-phase swaps, no wrapped BTC) | [github.com/btc-vision/native-swap](https://github.com/btc-vision/native-swap) |
| **Motoswap** | OP-20/OP-20 AMM (Uniswap V2 style, MOTO as hub) | https://motoswap.org |
| **MotoChef** | LP staking & MOTO farming rewards | (deployed on mainnet) |

→ Full DeFi mechanics: [docs/defi.md](docs/defi.md)

---

## Mainnet Contract Addresses

| Contract | Address |
|----------|---------|
| NativeSwap | `op1sqzjrvunc39nahkr6a0d4vx7rjdfjfc7qtqhp9wks` |
| Motoswap Factory | `op1sqpud62qd6gewyz4vd8x6lyrtwv6dkrd8n56j5ujw` |
| MotoChef (staking) | `op1sqq9axjh9mm03sz962n5dvgfae0pnvwr0zvzhm28r` |
| MOTO Token | `0x0a6732489a31e6de07917a28ff7df311fc5f98f6e1664943ac1c3fe7893bdab5` |

> OPNET addresses appear in two equivalent forms: Bech32m (`op1sq...`) and 32-byte hex (`0x...`).
> Convert between them with the `getPublicKeysInfoRaw` RPC method.

→ Full address reference: [docs/addresses.md](docs/addresses.md)

---

## Official SDKs & Packages

### TypeScript / JavaScript

| Package | npm | Description |
|---------|-----|-------------|
| `opnet` | [npmjs.com/package/opnet](https://www.npmjs.com/package/opnet) | Primary SDK — `JSONRpcProvider`, contract calls, ABI. Audited by Verichains. |
| `@btc-vision/btc-runtime` | [github.com/btc-vision/btc-runtime](https://github.com/btc-vision/btc-runtime) | AssemblyScript runtime for writing contracts |
| `@btc-vision/transaction` | [npmjs.com/package/@btc-vision/transaction](https://www.npmjs.com/package/@btc-vision/transaction) | Transaction building & broadcasting |
| `@btc-vision/bitcoin` | [npmjs.com/package/@btc-vision/bitcoin](https://www.npmjs.com/package/@btc-vision/bitcoin) | Bitcoin primitives — keys, addresses, UTXOs |
| `@btc-vision/walletconnect` | [github.com/btc-vision/walletconnect](https://github.com/btc-vision/walletconnect) | WalletConnect integration for OP_WALLET |
| `@btc-vision/unit-test-framework` | [github.com/btc-vision/unit-test-framework](https://github.com/btc-vision/unit-test-framework) | Smart contract test runner |

```bash
# dApp / backend development
npm install opnet @btc-vision/transaction @btc-vision/bitcoin

# Smart contract development
npm install @btc-vision/btc-runtime @btc-vision/unit-test-framework
```

→ Full SDK reference: [docs/sdks.md](docs/sdks.md)

---

## Smart Contract Development

Contracts are written in **AssemblyScript** and compiled to WASM using `@btc-vision/btc-runtime`.

**Key differences from Solidity / EVM:**

| Concept | Ethereum | OPNET |
|---------|----------|-------|
| Language | Solidity | AssemblyScript |
| Compile target | EVM bytecode | WASM |
| Hashing | keccak-256 | SHA-256 |
| Signatures | ECDSA | ECDSA + ML-DSA (quantum-resistant) |
| Deployment | Single tx | Two-phase Taproot (funding + reveal) |
| Storage | Slot-based | Pointer-based (`Blockchain.nextPointer`) |
| Token approval | `approve()` | `increaseAllowance()` |
| Constructor | Runs once | Runs on **every** interaction |
| Block time | ~12 s | ~10 min |
| Event logs | `eth_getLogs` | No native equivalent |

**Official resources:**
- [btc-vision/btc-runtime](https://github.com/btc-vision/btc-runtime) — runtime library with base classes
- [btc-vision/unit-test-framework](https://github.com/btc-vision/unit-test-framework) — test runner
- [btc-vision/example-tokens](https://github.com/btc-vision/example-tokens) — reference OP-20 implementations
- [docs.opnet.org](https://docs.opnet.org) — official developer documentation

→ Full contract development guide: [docs/contracts.md](docs/contracts.md)

---

## Infrastructure

| Repository | Description |
|------------|-------------|
| [btc-vision/opnet-node](https://github.com/btc-vision/opnet-node) | Full OPNET node implementation |
| [btc-vision/op-vm](https://github.com/btc-vision/op-vm) | Rust WebAssembly VM for contract execution |
| [btc-vision/mempool-opnet](https://github.com/btc-vision/mempool-opnet) | Self-hostable mempool explorer |
| [btc-vision/native-swap](https://github.com/btc-vision/native-swap) | NativeSwap contract source |
| [btc-vision/motoswap](https://github.com/btc-vision/motoswap) | Motoswap DEX contract source |

---

## Ecosystem & dApps

Community-built projects on OPNET:

| Project | URL | Description |
|---------|-----|-------------|
| **OPNet Hub** | https://vibecode.finance | All-in-one dashboard — token launcher, DEX UI, epoch miner, 26+ dApp directory |
| **BlockFeed** | https://blockfeed.online/docs | Public OPNET data API — REST/GraphQL/WebSocket, 35+ endpoints, no API key |
| **OP-Sign** | — | Trustless document notarization on Bitcoin L1 |
| **Eternal Sentinel** | — | Decentralized inheritance / dead man's switch for OP-20 tokens |
| **BlockTip** | — | Send any OP-20 token to anyone in one click on Bitcoin L1 |
| **Satoshi's Market** | — | NFT marketplace for OP-721 tokens |
| **op-bet** | https://mint.op-bet.xyz/ | On-chain betting dApp |

→ Full ecosystem listing with 26+ dApps: [vibecode.finance/ecosystem](https://vibecode.finance/ecosystem) · [docs/resources.md](docs/resources.md)

---

## Community & Social

| Platform | Link |
|----------|------|
| Twitter / X | https://x.com/opnetbtc |
| Discord | https://discord.com/invite/opnet |
| Official Site | https://opnet.org |
| Documentation | https://docs.opnet.org |
| Faucet | https://faucet.opnet.org/ |

---

## Media & Coverage

| Source | Article |
|--------|---------|
| CoinDesk | [Bitcoin's Biggest DeFi Drawback Under Attack as OpNet Unlocks Smart Contracts on Mainnet](https://www.coindesk.com/markets/2026/03/19/bitcoin-s-biggest-defi-drawback-under-attack-as-opnet-unlocks-smart-contracts-on-mainnet) |
| The Defiant | [Bitcoin Gets Native DeFi Stack as OP_NET Goes Live](https://thedefiant.io/news/defi/bitcoin-gets-native-defi-stack-as-opnet-goes-live-on-mainnet) |
| CoinTelegraph | [OP_NET Launches "SlowFi" DeFi Stack on Bitcoin](https://cointelegraph.com/news/opnet-brings-native-defi-to-bitcoin) |
| MoneyCheck | [Bitcoin Native DeFi Arrives: OpNet Unlocks OP-20 Tokens](https://moneycheck.com/bitcoin-native-defi-arrives-opnet-unlocks-op-20-tokens-and-on-chain-trading/) |
| Sovryn | [What Is OP_NET? A Guide to the New Bitcoin Metaprotocol](https://sovryn.com/all-things-sovryn/op-net-bitcoin-metaprotocol) |

---

## Deep Dives

Detailed reference documents:

| Doc | Contents |
|-----|----------|
| [docs/protocol.md](docs/protocol.md) | How OPNET works, epochs, transaction structure, key properties |
| [docs/networks.md](docs/networks.md) | Networks, RPC endpoints, JSON-RPC method reference |
| [docs/wallets.md](docs/wallets.md) | OP_WALLET, key formats, the two-key system, testnet faucet |
| [docs/token-standards.md](docs/token-standards.md) | OP-20, OP-721, OP-20S — mechanics and differences from ERC equivalents |
| [docs/defi.md](docs/defi.md) | NativeSwap 2-phase mechanics, Motoswap architecture, MotoChef staking |
| [docs/contracts.md](docs/contracts.md) | AssemblyScript contracts, btc-runtime, storage model, common pitfalls |
| [docs/sdks.md](docs/sdks.md) | opnet SDK, @btc-vision packages, WalletConnect integration |
| [docs/addresses.md](docs/addresses.md) | Mainnet contract addresses, genesis block, address formats |
| [docs/resources.md](docs/resources.md) | Full GitHub org listing, documentation links, community |

---

*Last updated: 2026-04-11 — Mainnet launched March 19, 2026 (block 941,400). Ecosystem is early.*

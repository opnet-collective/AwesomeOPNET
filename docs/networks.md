# Networks & RPC

← [Back to README](../README.md)

---

## Networks

| Network | Description | Genesis Block |
|---------|-------------|---------------|
| **Mainnet** | Live Bitcoin — real BTC | 941,400 (March 19, 2026) |
| **Testnet** | Bitcoin testnet — free test BTC | — |

---

## RPC Endpoints

| Network | URL |
|---------|-----|
| Mainnet | `https://mainnet.opnet.org` |
| Testnet | `https://testnet.opnet.org` |

The environment variable **`OPNET_RPC_URL`** is the standard override across all OPNET tooling and SDKs.

---

## JSON-RPC Method Reference

All methods use standard JSON-RPC 2.0 format over HTTP POST.

### Read Methods

| Method | Description |
|--------|-------------|
| `btc_blockNumber` | Current block height |
| `btc_getBalance` | BTC balance in satoshis for an address |
| `btc_getBlockByNumber` | Full block details by height |
| `btc_call` | Read-only (view) contract call — no transaction |
| `btc_getCode` | Deployed bytecode at an address |
| `btc_getStorageAt` | Raw storage read by pointer |
| `btc_getTransactionByHash` | Transaction details |
| `btc_getTransactionReceipt` | Transaction receipt |
| `btc_gasParameters` | Current gas price parameters |
| `btc_getPublicKeyInfo` | Resolve address to tweaked + identity pubkeys |
| `btc_getChallenge` | ML-DSA signing challenge |
| `btc_getUTXOs` | UTXOs for an address |

### Write Methods

| Method | Description |
|--------|-------------|
| `btc_sendRawTransaction` | Broadcast a signed transaction |

---

## Further Reading

- [docs.opnet.org/learn/networks](https://docs.opnet.org/learn/networks)
